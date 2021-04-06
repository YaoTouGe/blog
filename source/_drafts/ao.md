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

眼睛接收到P点处的辐射率（radiance）$L_o$，通过在法线n决定的半球面上，对所有无遮挡方向$\omega$的入射辐射率$L_i$做积分得到。之所以会有AO产生，是因为物体表面凹凸不平，凹陷的部位无法接受来自整个半球面上的光照，因而表现黯淡。凹陷的越厉害，这种现象就越明显。

![半球面](https://bl3301files.storage.live.com/y4mL3up2eBBjrXw_e-MkVyXraz030aLqD___CBg0iM0wFaMAh9HimDeCaqW4oi23k6BW5xUmLcWJyDlh0wpfdysGht19E8-qcyHgrY-DhXLIRiqNGeY_iAvX_jKG-axusO3o44JuqZ7VQd2JRjSvz3WtS_BWY-S0Y5kPDSkPLJeqtRiW4G1Y_WUgymiECDad2Uv?width=574&height=376&cropmode=none)

### AlchemyAO:
 https://casual-effects.com/research/McGuire2011AlchemyAO/VV11AlchemyAO.pdf