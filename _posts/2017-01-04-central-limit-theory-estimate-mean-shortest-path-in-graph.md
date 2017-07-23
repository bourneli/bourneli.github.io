---
layout: post
title:  中心极限定理评估平均最短路径
categories: [graph,statistics]
---


前一阵子在寻找计算图的平均最短路径的方法。如果要精确计算，一般需要$O(n^2)$的时间复杂度，对于节点较多的图而言，该时间复杂度是不能接受的。一般而言，都是用统计的方法估算平均最短路径。最终，使用中心极限定理评估比较合理，并且在公开的数据集上验证，效果明显。同时，使用[ANF](http://www.cs.cmu.edu/~christos/PUBLICATIONS/kdd02-anf.pdf)方法评估平均最短路径，发现后者的误差较大大，而且总是估计过高。下面简单介绍中心极限定理估计最短路径的过程以及相关实验数据。


## 为什么使用中心极限定理？
[中心极限定理](https://zh.wikipedia.org/zh-cn/%E4%B8%AD%E5%BF%83%E6%9E%81%E9%99%90%E5%AE%9A%E7%90%86)可以使用样本均值估计整体均值，并且给出置信区间。平均最短路径就是整体均值，而我们可以通过随机选取节点，计算两点之间距离，这样选取k对后，通过样本均值估计整体均值，完全符合中心极限定理的应用场景。

## 实现方法

首先需要得到全联通图，如果是有向图，必须是有向全联通。否则可能抽到两点不可达，导致无法准确的评估。设置样本数量k，经验数据，对于上千万节点的图，k=100的精度可以接受。计算这k对节点的[最短路径](https://zh.wikipedia.org/wiki/%E6%88%B4%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95)。得到k个样本后，计算样本均值$\bar{X}=\frac{1}{k}\sum_{i=1}^kx_i$和样本方差$S^2=\frac{1}{k-1}\sum_{i=1}^k(x_i-\bar{X})^2$。

根据中心极限定理，无论原始分布是什么，样本均值的分布满足

$$
  N(\mu, \sigma^2/k)
$$

其中$\bar{X}$是$\mu$的无偏估计，$S^2$是$\sigma^2$的无偏估计。所以，得到分布，就可以轻松的计算置信区间了。更为简单的方法，使用R中的[t.test](http://www.statmethods.net/stats/ttest.html)函数，直接将上面k个样本输入，即可算出置信区间。

## 实验数据

使用[豆瓣的数据](http://konect.uni-koblenz.de/networks/douban)做试验。该图的平均最短路径的真实值是**5.10**。使用中心极限定理，令k=120，可得到95%的置信区间为[5.02,5.20]，挺接近真实值的。而使用ANF，得到的直径为**6.57**，误差挺大，且没有置信区间,可能是由于需要多次估算集数，导致误差累加。