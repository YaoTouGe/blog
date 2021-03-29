---
title: reversed-z
date: 2021-03-19 18:13:45
mathjax: true
tags:
- graphics
---

之前说到z-fighting，我只能想到两个解决方法，一个是把近平面推远，调整z值在视锥里的分布。另一种是用cascade的方式，多级视锥来把z的精度匀开，其实也是变相把近平面推远了，只不过有好几个近平面而已。

最近学习到的一个技术，reversed-z，感觉是一种更行之有效的办法。通过对投影矩阵的调整，很巧妙的把深度缓冲中的z在世界空间中的分布，和浮点数的精度分布结合起来，让深度的精度更合理。

传统的透视投影形如：

$M_p P_w=\begin{bmatrix} s & 0 & 0 & 0 \\\ 0 & s & 0 & 0 \\\ 0 & 0 & \frac{f}{f-n} & -\frac{fn}{f-n} \\\ 0 & 0 & 1 & 0 \end{bmatrix} \begin{bmatrix} x \\\ y \\\ z \\\ 1 \end{bmatrix}$

$d=\frac{f}{f-n}-\frac{fn}{f-n} * \frac{1}{z}$

d为乘以投影矩阵后，再经过透视除法，写入深度缓冲的值，0对应near，1对应far。

d是一个浮点数，浮点数有一个让人比较头疼的特性，就是其所表述的值越大，精度就越低。因为底数的bits只表示小数，大数值完全是靠着指数撑起来的，而且指数可以为负，因此0附近的精度尤其高。

![float bits](https://bl3301files.storage.live.com/y4m12E6E-flIVXMTTnJE9UJD3fkbQSS-T71vOR4zljvCxrtOaiI51cOIuuaNnPBvf68pPku_UhGOm5w-h4izkQM74rhlmq132FeFFrtPPfWlr1rnpbkaIn4xfGD58GvXGLUjO2khu_QZfZFBtUfP5fJqhuYOrMjp3jLIF3c0pH96NYVugQt3zjzpA4luSNMCvXg?width=948&height=204&cropmode=none)

我们如果将float32在[0, 1]区间上的精度上可视化，如下图，蓝色的刻度表示每次给底数加上1 << 20后，实际所表示的数值。（模拟每次给底数加1，但那样刻度数量太多，间距也不够明显）：

![float precision](https://bl3301files.storage.live.com/y4m7wDnXg-sMAuLU7m1Etsk_4KVd2SmG-FuChEdXngpzChXXlRl4edNKMzvog58jev1brOp5LoHbmQF9U52fs7y4hWLtJPgIZj21oJAVlOEtHrUHo8Dmk30MKqMcxJkmHb98-ah41G0fLgpub13OnLc7ZB3nkSaQf9V61MRaXdOlm5ROYPbiEOs0HKfkbShmW_L?width=1080&height=288&cropmode=none)

越接近0，数精度越高，蓝色刻度越密集，越接近1，精度越低，刻度间隔越大。

对应的，z-buffer值与word z的函数曲线上，将精度可视化，橙色十字表示z-buffer中的精度分布，绿色十字表示每个橙色十字对应到word z的值：

![depth buffer precision](https://bl3301files.storage.live.com/y4mb3eevO7zVZdoeJ_ryqJmk_nzeADWPzog1KuKaqYdsX49VLenUyFG83EmEnfEi8rY_Cbu-wulbE8f5w7GRWRQoblIQAfJ3-7_MOtp3WRUyfQWiM-4Ke_lMwb3lLemxMqhGzMNhraWVvqPO1IE36VveTa8cFu1y_PrV5oB1YiJv4m9mbBt33EdfSqp4ycw-fuq?width=1440&height=720&cropmode=none)

这下更明显了，函数曲线本身的特点就是，word z接近near的部分占用了[0, 1]]区间的一大半值域，再加上浮点数的精度分布在0附近更高，二者互相增益，导致z-buffer的精度在世界空间的分布更集中于near附近，极度不均匀。

实验做到这里，不得不佩服前人的智慧了，既然这两个效果互相增益，导致精度更不均匀，那如果能让二者互相抵消，是不是能够很大程度的缓解这个问题呢？于是有人想办法将z-buffer的值反过来，0对应far，1对应near，这样浮点的精度分布和透视投影的z值函数曲线的分布就能互相抵消了。这里对projection矩阵做了修改，左乘一个转换矩阵：

$T M_p P_w=\begin{bmatrix} 1&0&0&0 \\\ 0&1&0&0 \\\ 0&0&-1&1 \\\ 0&0&0&1 \end{bmatrix} \begin{bmatrix} s & 0 & 0 & 0 \\\ 0 & s & 0 & 0 \\\ 0 & 0 & \frac{f}{f-n} & -\frac{fn}{f-n} \\\ 0 & 0 & 1 & 0 \end{bmatrix} \begin{bmatrix} x \\\ y \\\ z \\\ 1 \end{bmatrix}$

$d=-\frac{n}{f-n}+\frac{fn}{f-n}*\frac{1}{z}$

这样得到的z-buffer正好与之前相反，0对应far，1对应near，再对新的函数曲线以及精度进行可视化：

![reversed depth buffer precision](https://bl3301files.storage.live.com/y4mFXVlYzbuJbY3r01RZpiT7BSwxgYJ8BsK-9MoJWETydeGG2rl-2-21UVYWVv4G7e9lWyltBpsA8wfpGI1dywBNRqQJRJLgtnggJC06C33Frfge6VDHwhihq7WBgNIT3Gv7FpDr2LjQTti4K1HFh91Qu3PDQZuf5dATUgl5PelIx0c-JLb88ht5HOHvS0Zv31L?width=1440&height=720&cropmode=none)

分布比之前合理太多了！world z的near和far区间内，对应到z-buffer上都有较为均匀的精度分布了。实际使用时，由于z-buffer的大小关系反过来了，因此要把depth test改成 greater。如果在OpenGL上应用，还有一个问题，就是OpenGL的ndc的z范围为[-1, 1]，为了能用上reversed-z，还需要支持 glClipControl 这个api，OpenGL4.5以上的版本才有，更低版本只能看是否有对应的扩展了，详细可以看[这篇文章](https://nlguillemot.wordpress.com/2016/12/07/reversed-z-in-opengl)。

最后附上绘图的代码[reverse_z.ipynb](/notebooks/reverse_z.ipynb)，为了画这几张图，临时入门了一下matplotlib。

