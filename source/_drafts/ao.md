---
title: Ambient Occlusion
tags:
- graphics
- unity
- post process
---

环境光遮蔽(AO, ambient occlusion)是提升画面质感相当重要的一个后处理特效，人们感受物体间的位置关系非常依赖光影，宏观的阴影有ShadowMap来解决，微观上的则主要靠AO。最近在学习在Unity中实现AO后处理，找到一篇13年的论文[A comparative study of ssao methods](https://www.gamedevs.org/uploads/comparative-study-of-ssao-methods.pdf)，把当时已有的AO算法做了对比分析（似乎从那个时候到现在，AO算法还是那么几种，可能已经够用了），并且讲清了AO算法发展的来龙去脉，很值得一读。

首先上来是老路子，渲染方程列出来：

$L_o=\int fL_i\cdot(n\cdot\omega)d\omega$

眼睛接收到P点处的辐射率（radiance）为$L_o$，通过在法线n决定的半球面上，对所有方向$\omega$的入射辐射率$L_i$做积分得到，f是物体表面的BRDF，根据入射方向l和视线方向v确定反射光的比例。之所以会有AO产生，是因为物体表面凹凸不平，凹陷部位的半球面上被遮挡了部分的光照，因而表现黯淡。凹陷的越厉害，这种现象就越明显。

![半球面遮挡](https://bl3301files.storage.live.com/y4mL3up2eBBjrXw_e-MkVyXraz030aLqD___CBg0iM0wFaMAh9HimDeCaqW4oi23k6BW5xUmLcWJyDlh0wpfdysGht19E8-qcyHgrY-DhXLIRiqNGeY_iAvX_jKG-axusO3o44JuqZ7VQd2JRjSvz3WtS_BWY-S0Y5kPDSkPLJeqtRiW4G1Y_WUgymiECDad2Uv?width=574&height=376&cropmode=none)

如果我们完整的对渲染方程去求解，在半球积分的过程中，很自然就会产生AO。然而在实时渲染中，这样的计算消耗根本不可能实现（甚至在很多离线CG渲染中，也是用近似的方式来渲染AO的），为了能满足实时交互的需求，必须对AO的算法进行简化，只要视觉上能“欺骗”眼睛就行，并不要求物理真实。

于是做出如下的假设：

* 假设BRDF是独立于入射方向和视线的常量，物体表面是Lambertian。
* 假设各个方向的入射光的radiance是常量。

简化之后，渲染方程就变成了：

{% raw %}
$L_{oa}=fL_{ia}\int(n\cdot\omega)d\omega$
{% endraw %}

有两项被移出积分外，现在$L_{oa}$是p点处ambient的出射辐射率，$L_{ia}$是常量的ambient入射辐射率，f是常量的BRDF。（这种技巧似乎挺常见，为了加速渲染方程的求解过程，要么假设某个变量为常量，将之移出积分。亦或是把积分中的表达式拆开，将能预计算的部分剥离出来离线烘焙）。现在，方程只剩下一个积分项了，我们将简化后的方程记作

$A(p)=\frac{1}{\pi}\int(n\cdot\omega)d\omega$


A(p)表示p点处的遮挡，$\frac{1}{\pi}$是lambertian brdf的归一项，因为cos经过半球积分会得到一个$\pi$系数，我们要将其归一化（brdf积分应该等于1，能量守恒）。详细可以看这篇文章[如何求计算半球积分](http://www.pbr-book.org/3ed-2018/Color_and_Radiometry/Working_with_Radiometric_Integrals.html)。

传统的光照模型中，会把A(p)简化为常数1，也就是没有环境光遮蔽。现在为了计算这个积分，我们需要定义一个可见函数$V(p,\omega)$，表示p处沿$\omega$方向是否有遮挡。

$V(p,\omega)
\begin{cases}
0& \text{Ray from p in direction} \quad \omega \quad \text{hits anything}\\
1& \text{Otherwise}
\end{cases}$

这样就得到AO的计算公式：

$AO(p)=\frac{1}{\pi}\int_\Omega V(p,\omega)(n\cdot \omega)d\omega$

用计算机求解时，可以采用蒙特卡洛积分进行求和计算：

$AO \approx\frac{1}{N}\sum_{n=1}^NV\cdot(n\cdot \omega_n)$

详细可以参照[关于蒙特卡洛积分的资料](https://blog.csdn.net/hellocsz/article/details/94400402)，大概思想是，利用强大数定理，把对随机变量的函数的积分看作是求期望，然后转化为对随机变量序列的和求平均，在N趋近无穷大时，平均值是无穷接近于期望值的。

### AlchemyAO:
 https://casual-effects.com/research/McGuire2011AlchemyAO/VV11AlchemyAO.pdf