---
title: GPU Flow-Control Idioms 笔记
tags:
  - graphics
  - GPU
date: 2021-08-09 00:21:28
---


一直只知道shader中用分支会有额外开销，但总想搞清楚，GPU执行分支的时候发生了什么。这点不同架构，不同厂商的GPU底层细节可能都不同，他们也不会公开太多消息。我找到[GPU Gems2](https://developer.nvidia.com/gpugems/gpugems2/part-iv-general-purpose-computation-gpus-primer/chapter-34-gpu-flow-control-idioms)的一篇老文章学习（貌似是好多年前的，里面还在讲GeForce 6系列显卡，有些内容可能比较过时，不过基本原理应该还是有参考价值的），做笔记记录一下。

## CPU和GPU的分支模型

首先看看CPU分支的性能开销，CPU的指令流水线通常比较长（能达到10到20个时钟周期），为了让效率最大化，理想情况是不希望流水线被打断的。所以CPU的[分支预测](https://en.wikipedia.org/wiki/Branch_predictor)很重要。如果没有分支预测，在条件跳转（condition jump）执行完毕之前，CPU必须等待，流水线只能空着。为了改善这个问题，CPU会对分支进行预测，根据预测结果提前让指令进入流水线执行，如果预测失败，也不过是把提前执行的指令中断，重新让正确的指令进入流水线。

在GPU上，最新的GPU（那个年代还是GeForce 6xxx系列）也有类似的分支指令，但性能与CPU有些许不同。更老的GPU则没有原生的分支指令，而是用其他方式模拟分支的行为。并行架构中最常见的机制是SIMD（单指令多数据）和MIMD（多指令多数据）。SIMD架构中的所有处理单元在同一时刻执行相同的指令。MIMD中不同的处理器则可以同时执行不同的指令。GPU目前用三种方式来实现分支：MIMD分支，SIMD分支和条件码。

*MIMD*

MIMD是最理想的情况，每个处理单元可以像GPU那样，在没有性能损失的情况下执行不同的分支代码，GeForce 6系列的顶点处理器中支持MIMD分支。

*SIMD*

SIMD支持程序中出现分支以及循环，但一组中所有处理器执行完全一致的指令，所以不同处理器执行出现分歧(divergent)时性能会损失。举个例子，如果一个片元着色器在条件判断中，根据输入的随机数来决定执行哪个分支，然后输出不同的值。那么同一组处理器中，条件为False的处理器，在条件为True的处理器执行完毕之前就必须等待（SIMD中的一组处理器执行完全相同的代码，先当作全部为True执行一遍，对于False的那些处理器，会抛弃本轮的结果，然后全部当作False执行一遍）。结果就是总时间等于两个分支的总和加上条件判断指令的时间。

![SIMD的执行模型](https://tva1.sinaimg.cn/large/006uGrbwly1gt9hr8y1kwj30jd0iyn1v.jpg)

因此SIMD在分支条件的分布比较“连续”的时候相当有用（比如大多数warp里的处理器都走相同的分支，只有少数warp发生分歧），如果条件分布很不“连贯”（比如每个warp里面有一个处理器走不同的分值），性能会严重下降，变成两个分支的消耗之和。

*条件码*

在更老的GPU中采用条件码的方式来模拟分支的行为，条件判断执行后会设置条件码，if的true和false分支全都会被执行（不像SIMD中只有发生分歧才会这样），随后根据条件码的状态决定采用哪个分支的结果。所以这种架构中，分支的代价一定等于所有分支的执行时间之和加上条件判断的时间，如果每个分支的指令很少，这种方式的效率还能接受。当分支指令变得很复杂时，就需要采取其他方式了。所以这种架构中应该尽可能的避免分支。

## 控制流的基本策略

### 将分支前移

由于GPU的分支的机制相当复杂，我们需要有不同策略来应对各种情况，一种比较有效的就是将条件判断尽可能前移，在越早的阶段做判断效率就越高。

*静态分支的粒度*

在CPU上进行数据运算时，很多人都知道要尽可能避免在内层循环钟使用分支，否则当分支预测错误时会导致流水线的停滞，不断地重新填充指令流水线。例如在离散空间网格中计算偏微分方程（PDE）时，最直接的CPU实现就是遍历所有网格，然后判断每个网格是否为区域的边界，并使用不同的计算方法。更好的实现是用两个循环，一个遍历区域内部的网格，一个遍历边界网格，相当于将分支判断提到循环外面。

这种策略对于GPU而言更为重要，因为GPU的分支开销更大。在GPU实现中会把计算分为两个fragment program（那时还没有compute shader，GPGPU都是通过fragment shader计算的，还需要调用绘制命令），一个计算区域内部的网格，绘制内部的网格四边形，一个计算区域边界的网格，绘制边界线。

*预计算*

在上一个例子中，分支的结果在大部分的输入/输出值中都是不变的。有时候分支结果在一个时间段内是不变的，这种情况下，我们可以只在分支发生变化的时候进行计算，然后将结果预存起来供后面使用。

在NVIDIA SDK的 gpgpu_fluid例子中，在计算障碍物的边界处时就采用了这种技巧。当流体计算的网格与障碍物相邻时需要做额外的计算，我们必须检查相邻的网格来确定障碍物的朝向，然后根据朝向进行下一步计算。这些障碍物只有在用户进行操作时才会发生改变，因此我们可以将朝向的计算结果保存下来并复用，直到障碍物发生变化才重新计算。

### Z-Cull

如果在预计算的基础上更进一步，我们可以采用另一个GPU的特性来完全跳过不必要的计算。z-cull是一种用于避免对不可见的fragment进行着色的技术（就是early-z），如果某个fragment没有通过深度测试，则在进行着色计算前取消掉这个fragment，避免不必要的计算。

在GPGPU中也可以利用这个特性，还是在流体计算的那个例子中，有的计算网格是完全“包裹”在障碍物内部的，我们并不需要对它们进行任何计算。为了跳过这些网格，当障碍物发生变化时，我们会通过一个fragment shader来检查所有的网格以及其邻居，discard那些没有被完全包裹的网格，对于被包裹的网格，则会输出0到z-buffer中。而其他的cell则用深度1画出来，这样当我们用深度0.5绘制一个全屏的quad时，障碍物内的cell就被mask掉了。

要注意的是，z-cull的分辨率通常比屏幕分辨率要粗糙（类似Hi-Z中的高等级的mipmap），所以只有一小片区域中的fragment都没有通过深度测试才会被discard掉，不同GPU的z-culling的分辨率也不同，但总的来说，只有当分支的条件（是否通过深度测试）具有一定的局部性时，z-cull才会有性能的提升。

为了展示这种局部性，我们进行一个实验，对比在通过深度测试的fragment比例相同的条件下，z值分布具有高度的空间局部性，和随机分布这两种情况下z-cull的性能，如下图。可以观察到，具有空间局部性的情况下效率是最高的，时间消耗与执行的fragment数量几乎是线性关系。而随机的情况下，即时通过深度测试的fragment比例不是太高，依然会有较多的时间消耗，好消息是当这个比例很高或者很低时（曲线的两端），天然就具备了这种空间局部性。但如果是其他情况，该方法的性能并不必条件码好多少。

![对比结果](https://tvax3.sinaimg.cn/large/006uGrbwly1gt9skvbqyxj30dw044mxi.jpg)

最后一点要注意的是z-cull和fragment shader中的分支区别还是很大的，z-cull是完全避免fragment shader的执行，比fragment中的分支更早，节省更多资源。如果分支的条件是静态的预计算好的，z-cull是一个不错的选择。

### 分支指令

第一个在fragment shader中支持分支的是GeForce 6系列，随着越来越多的GPU支持分支，条件码的方式会逐渐被淘汰。但是类似前面z-cull中所展示的局部性问题那样，GPU分组并行的执行fragment shader，组内只能执行相同的代码，如果分支发生分歧，就会退化成条件码的方式，下图很好的表示这个问题：

![分支指令](https://tvax4.sinaimg.cn/large/006uGrbwly1gt9thory60j309q05m3yn.jpg)

## 遮挡查询

硬件遮挡查询也是设计用于防止绘制不可见fragment的一个特性。通过它可以查询渲染过程中有多少像素更新了，并且不会阻塞GPU的流水线。而通常在GPGPU计算中，计算所覆盖到的像素数量都是已知的，所以通过硬件遮挡查询，我们可以知道更新的fragment数量以及discard的数量，然后根据查询结果在CPU端做一些决策，比如作为循环的终止条件等等。

## 总结

虽然GPU上能支持分支指令了，但是不“紧凑”的分支计算依然还有性能损失，采用诸如预计算，将分支提到流水线前端（不管是vertex shader或者是到CPU上）的技巧，仍然是很有必要的。