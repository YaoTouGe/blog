---
title: GPU节能技术
date: 2021-03-29 23:58:21
tags:
- gpu
- graphics
---

最近在学习GPU的一些知识，看到一篇[论文](https://personals.ac.upc.edu/jarnau/Thesis_JoseMaria.pdf)讲的挺清楚。但自己对芯片底层设计不太了解，只能看个大概（其实只是把论文的综述认真看了一下，哈哈，不过对目前的移动GPU有个大概的认识吧），做笔记总结一下：

移动设备以及GPU的能耗占比（图片来自于论文）：
![mobile energy](https://pic3.zhimg.com/v2-2d1a10b9d62bedfd956fe4bbaae35196_b.png)

# 移动GPU的渲染方式：
* Tegra的GPU采用立即渲染模式（Immediate-Mode Rendering)
* Mali采用Tile Based Rendering。
* PowerVR采用也是Tile Based Rendering，但着重解决了 overdraw的问题
* Adreno高通的采用混合架构，会在立即模式和tile based之间运行时切换，取名FlexRender技术。
* 还有其他的小众GPU，如Vivante，把GUI相关的任务交给了单独的处理器，从GPU中剥离，很好地降低GPU的功耗。
* Digital Medial Professional的PICA-200 GPU，用在任天堂3DS上。

# 移动GPU的两大功耗杀手：
## Big register file
A 2MB register file consumes more than 9 watts. GPU采取高并行的策略来抵消高内存延迟的影响，而高并行则导致大的register file（每个 GPU thread都需要对应的register file），导致功耗增加。移动GPU将thread的数量从几万降低到几百的数量级，性能降低了很多，为了降低功耗实属无奈。
## Expensive off-chip memory access
移动GPU访问 off-chip 内存（移动GPU没有显存，或者内存当做显存用？）比访问on-chip内存的功耗要高一个数量级的功耗，甚至占用了一半以上的GPU总功耗。

# 目前的GPU节能技术概况：
## memory latency
### Register file optimization
没看太懂，提到了 register file cache，通过cache减少对 main register file的访问，然后通过优化GPU thread的调度，使得active thread数量减少，进一步减少register file的大小。
### Prefetch
减少cache miss造成的 memory latency，与CPU的prefetch算法不同，mobile gpu也是以节能为主的。CPU的prefetch算法通常性能很好，但是取到很多不必要的数据，所以能耗表现差。
### Decouple Access/Execute
把memory access和执行计算两个步骤解耦，prefetch可以提前很多，这样cache miss和计算可以overlap，不造成stall，虽然也是对prefetch的优化，但其它prefetch的地址是预测的，这种方式的prefetch地址却是计算得到的，因此更精确，造成的浪费更少。后面没看太懂。。。反正因为限制很多，实际效果没有那么好，好像并没有商业CPU采用，但好像也没说GPU上应用没有。
## Bandwidth saving：
之前一直有个误解，以为显卡的带宽消耗是指从数据从内存传输到显存的消耗，这个带宽应该是消耗的数据总线的带宽（移动端可能没有这个概念？）。
GPU本身访问显存要消耗带宽，类比CPU访问内存是一个道理，这个才是显存带宽，同样GPU也会有cache miss等情况。GPU的渲染过程中每帧都需要从main memory（显存） 获取大量的数据（如Vertex，Texture等）。
带宽消耗是GPU能耗的主要来源之一。
### Tile-Based Rendering
Immediate Mode Rendering(IMR)中geometry经过vertex processor后立即进入管线的其他步骤，进行pixel processing，并更新framebuffer，当发生overdraw时，framebuffer中的像素会被写入很多次，造成带宽的浪费，桌面GPU对功耗不敏感，采用这种方式。
TBR将屏幕划分为小tile，framebuffer也是逐tile去绘制，每个tile只渲染其所覆盖到的三角形，tile的渲染过程只访问 on-chip memory，也就是overdraw的时候，不会出现大量的main memory 访问。当tile渲染结束之后，才会写一次framebuffer，这样节省了pixel bandwidth。
不过由于每个tile只渲染其所覆盖的三角形，不像IMR那样用完即扔，因此全部的三角形是存储在main memory中的，渲染单个tile时从main memory读回on-chip，因此TBR是增加了 geometry bandwidth来减少 pixel bandwidth，但通常 geometry数据量是小于像素写入framebuffer的数据量的，所以总体带宽是减少的。
### Compression
压缩既能降低内存占用，也能减少带宽消耗。通过硬件支持纹理压缩，GPU访问main memory中的texture时，传输的是压缩过的数据，传输到GPU中后再实时解压缩（在on-chip meomory中？），这样减少带宽的消耗，因此，纹理压缩算法有几个特性（来自维基百科）：
* Decoding speed，要支持渲染时实时解压缩，因此解压速度一定要快。
* Random access，纹理访问的位置难以预测，因此压缩后的纹理要支持随机访问。
* Compression rate and visual quality，渲染系统是能容忍一定的纹理精度损失的，因此在压缩率和视觉效果之间可以做出一些平衡。
* Encoding speed，通常压缩过程是离线完成的，因此压缩算法可以不对称，即压缩速度和解压速度不对等。

OpenGL ES 3.0 引入了ETC2的支持，各个GPU厂商也有自己的纹理压缩格式，ARM的MaliGPU支持ASTC格式，PowerVR支持PVRTC格式，NVIDIA的Tegra支持DXT格式。
除了纹理压缩外，Framebuffer也可以通过压缩节省带宽，原理和纹理压缩类似。Framebuffer的数据存在main memory 中，GPU读入时，传入压缩后的数据，然后GPU端实时解压。写回Framebuffer时则相反，GPU端实时压缩后，将压缩了的数据写入main memory中。
### Transaction Elimination
由ARM引入，在Mali GPU上实现，每个tile有一个CRC校验码，通过CRC来判断tile内容是否发生了变化，对于没有变化的tile不写入Framebuffer，达到节省带宽的目的，尤其对静态背景的游戏以及UI的绘制有效，因为通常它们变化的tile很少。
## Multi-View and Multi-Frame Rendering
VR显示中需要每帧从不同的view渲染，为了最大化cache的命中率，渲染框架对每个三角形并行渲染多个view，而不是逐个依次进行。
NVIDIA的SLI技术中利用多GPU的一种方式叫做Alternate Frame Rendering，驱动将每帧轮流交给不同的GPU渲染，比如按照奇数和偶数分给两个GPU，通过在多个GPU间共享同一个的地址空间，进而共享部分渲染数据可以达到节省显存带宽的目的。
## 其他相关技术
Tegra中采用 Early-Z的方式，在fragment shader执行前进行depth test，减少对不可见fragment的计算，从前往后渲染时效果最好。
Tile-Based Rendering中虽然一个tile只写一次Framebuffer，但在on-chip memory中依然存在overdraw，浪费fragment的计算。ARM的mali采用了Forward Pixel Kill（FPK）技术，已经发起的thread不再是不可取消的了，一旦发现一个像素在后面将要被写入不透明的颜色，正在执行的thread可以随时终止。
PowerVR采用Hidden Surface Removal(HSR)技术，在执行fragment shader之前，会对每个tile的可见性进行完全解析，找出最靠前的fragment，然后进行着色，因此每个像素只会执行一次fragment shader（真的有这么厉害吗？）。
