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

我们如果将float32的精度在数轴上可视化，如下图，蓝色的刻度表示每次给底数加上1 << 20后，实际所表示的数值。（模拟每次给底数加1，但那样刻度数量太多，间距也不够明显）：

![float-precision](https://bl3301files.storage.live.com/y4mZs9BaRFkk6zfsKuDVtSFfzseIipDkX0_REULMj33JpkrtclBKsszVN510SzDm4ZcUrIxbRaqEdxxyfEGGNXPXLuUiugXMARdxgECRG9vEc4EVxID-s-mnWwj638fRoU32fbI9MqYxtvEd9CJJlz3Bt17fyx8lM8nslBMBe_dU1PukDoQGA4EpfLcSHvr9t_s?width=1080&height=109&cropmode=none)

仅仅在0~10000区间就可能看出，越接近10000，相邻两个刻度的数值差别越大，精度也就越差。

对应的，在depth-buffer中数值与世界空间z的函数曲线上，将精度可视化：

![depth-buffer-precision](https://bl3301files.storage.live.com/y4m3oRe2dinbk00w5sa-30BowrhBQCs4jcyrkhPckcdMp6f3JDUXLx2qZwrE8OfcjATpVLGEMQlPzDmFApk_iBceLoO9s1sMpiihiomsdbBH5TIKWYv5QS7NUXU0MCLxiEez8vC8hzIRmmjKCEloDKJRdqLTzDwMJaOX279O6R-pllDnaheN1sSg89BHO_7LAHu?width=1440&height=720&cropmode=none)

原本函数曲线的特点就是，dpeth value的 0

reversed-z 对投影矩阵的改动很简单，乘以一个特意构造的矩阵