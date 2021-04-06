---
title: Ambient Occlusion
tags:
- graphics
- unity
- post process
---

环境光遮蔽(AO, ambient occlusion)是提升画面质感相当重要的一个后处理特效，人们感受物体间的位置关系非常依赖光影，宏观的阴影有ShadowMap来解决，微观上的则主要靠AO。最近在学习在Unity中实现AO效果，找到一篇13年的论文[A comparative study of ssao methods](https://www.gamedevs.org/uploads/comparative-study-of-ssao-methods.pdf)，把当时已有的AO算法做了对比分析（似乎从那个时候到现在，AO算法还是那么几种，可能已经够用了）。

首先上来是老路子，渲染方程列出来：

$L_o=\int fL_i\cdot(n\cdot\omega)d\omega$

眼睛接收到P点处的辐射率（radiance）$L_o$，通过在法线n决定的半球面上，对所有方向$\omega$的入射辐射率$L_i$做积分得到，f是物体表面的BRDF，根据入射方向l和视线方向v确定反射光的比例。之所以会有AO产生，是因为物体表面凹凸不平，凹陷部位的半球面上被遮挡了部分的光照，因而表现黯淡。凹陷的越厉害，这种现象就越明显。如果我们完整的对渲染方程去求解，在半球积分的过程中，很自然就会产生AO。然而在实时渲染中，这样的计算消耗根本不可能实现（甚至在很多离线CG渲染中，也是用近似的方式来渲染AO的），为了能满足实时交互的需求，必须对AO的算法进行简化，只要视觉上能“欺骗”眼睛就行，并不要求物理真实。

于是做出如下的假设：

* 假设BRDF是独立于入射方向和视线的常量，物体表面是Lambertian。
* 假设各个方向的入射光的radiance是常量。

简化之后，渲染方程就变成了：

$L_{oa}=fL_{ia}\int (n\cdot\omega)d\omega$

有两项被移出积分外，现在$L_{oa}$是p点处ambient的出射辐射率，$L_{ia}$是常量的ambient入射辐射率，f是常量的BRDF。（这种技巧似乎挺常见，为了加速渲染方程的求解过程，要么假设某个变量为常量，将之移出积分。亦或是把积分中的表达式拆开，将能预计算的部分剥离出来离线烘焙）。



![半球面](https://bl3301files.storage.live.com/y4mL3up2eBBjrXw_e-MkVyXraz030aLqD___CBg0iM0wFaMAh9HimDeCaqW4oi23k6BW5xUmLcWJyDlh0wpfdysGht19E8-qcyHgrY-DhXLIRiqNGeY_iAvX_jKG-axusO3o44JuqZ7VQd2JRjSvz3WtS_BWY-S0Y5kPDSkPLJeqtRiW4G1Y_WUgymiECDad2Uv?width=574&height=376&cropmode=none)

### AlchemyAO:
 https://casual-effects.com/research/McGuire2011AlchemyAO/VV11AlchemyAO.pdf