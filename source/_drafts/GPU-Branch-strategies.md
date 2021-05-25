---
title: GPU Flow-Control Idioms 笔记
tags:
- graphics
- GPU
---

一直只知道shader中用分支会有额外开销，但总想搞清楚，GPU执行分支的时候发生了什么。这点不同架构，不同厂商的GPU底层细节可能都不同，他们也不会公开太多消息。我找到[GPU Gems2](https://developer.nvidia.com/gpugems/gpugems2/part-iv-general-purpose-computation-gpus-primer/chapter-34-gpu-flow-control-idioms)的一篇老文章学习（貌似是好多年前的，里面还在讲GeForce 6系列显卡，不过基本原理应该还是有参考价值的），做笔记记录一下。

首先看看CPU分支的性能开销，CPU的指令流水线通常比较长（能达到10到20个时钟周期），为了让效率最大化，理想情况是不希望流水线被打断的。所以CPU的[分支预测](https://en.wikipedia.org/wiki/Branch_predictor)很重要。如果没有分支预测，在条件跳转（condition jump）执行完毕之前，CPU必须等待，流水线只能空着。为了改善这个问题，CPU会对分支进行预测，根据结果提前让指令进入流水线执行，如果预测失败，也不过是把提前执行的指令中断，重新执行正确的指令即可。

