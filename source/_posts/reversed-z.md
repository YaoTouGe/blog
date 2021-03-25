---
title: reversed-z
date: 2021-03-19 18:13:45
mathjax: true
tags:
- graphics
---

之前说到z-fighting，我能想到的解决办法就两个，一个是把近平面推远，调整z值在视锥里的分布。另一种是用cascade的方式，多级视锥来把z的精度匀开，其实也是变相把近平面推远了，只不过有好几个近平面而已。

最近学习到的一个新技术，reversed-z，感觉是一种更行之有效的办法。通过对投影矩阵的调整，很巧妙的把深度缓冲中的z在世界空间中的分布，和浮点数的精度分布结合起来，让深度的精度更合理。

传统的透视投影矩阵形如：

$P=\begin{bmatrix} s & 0 & 0 & 0 \\\ 0 & s & 0 & 0 \\\ 0 & 0 & b & a \\\ 0 & 0 & 1 & 0
\end{bmatrix}$

其中a,b为根据远近平面距离计算的常量。顶点乘以透视矩阵，再经过透视除法后，写入深度缓冲的值为

 $d=a\frac{1}{z}+b$

a通常是一个负值，也就是说深度缓冲的d和世界空间z的倒数之间，是线性函数，且负相关。z等于near时，d等于0，z等于far时，d等于1，而精度问题就是在此处发生的。

z是一个浮点数，浮点数有一个让人比较头疼的特性，就是其所表述的值越大，精度就越低。因为底数的bits只表示小数，大数值完全是靠着指数撑起来的。

![float-bits](https://bl3301files.storage.live.com/y4m12E6E-flIVXMTTnJE9UJD3fkbQSS-T71vOR4zljvCxrtOaiI51cOIuuaNnPBvf68pPku_UhGOm5w-h4izkQM74rhlmq132FeFFrtPPfWlr1rnpbkaIn4xfGD58GvXGLUjO2khu_QZfZFBtUfP5fJqhuYOrMjp3jLIF3c0pH96NYVugQt3zjzpA4luSNMCvXg?width=948&height=204&cropmode=none)

我们如果将浮点数的精度在数轴上可视化：

![float-precision](https://bl3301files.storage.live.com/y4mabPnvRJE1sb7uZGfd4s783xXHxZBko-KlJ4oULQBSC6OBj-nwwWHPTTbDrGc67jhBGl8Pf1ojynrLuydb6zW774hRft9s4ZawL33sLXTPnA4wXCRMWEe0LWSLEHbngPxHEMNwKy_us3D8F8BM-Bps5tb6m2SYGlVPwCtUueI3RH4cHUCogbIROWEDEOv-3A1?width=1528&height=142&cropmode=none)

对应的，在z-buffer中的精度可视化：

reversed-z 对投影矩阵的改动很简单，乘以一个特意构造的矩阵