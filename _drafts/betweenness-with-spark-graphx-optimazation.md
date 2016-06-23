---
layout: post
title:  "Spark优化那些事(2)-graphx实现介数估算踩坑总结"
categories: [scala,spark,sns,graphx,graph]
---

## 背景
最近一段时间在使用spark graphx实现介数估算。介数（betweenness），点介数和边介数之分，在图中衡量一个点或边的重要程度，后面的讨论均是针对点介数，边介数实现方式类似。这个指标虽然好，但是计算开销非常之大，如果没有优化，纯粹按照定义计算，复杂度为$O(n^3)$(n是节点数)，定义如下:

$$
	Bet(v) = \sum_\limits{s \ne t \ne v \in V}  \frac{\sigma_{st}(v)}{\sigma_{st}},
$$

其中$\sigma_{st}$是s，t之间最短路径的数量，$\sigma_{st}(v)$是s，t最短路径中包含节点v的数量。  


根据[Brandes的快速介数计算][1]，可以将复杂度降到$O((m+n)n)$(m，n分别是边数和节点数)。但是，即使是spark，面对此计算量计算上百万节点的图仍然不太现实。所以，只能采取折中的方法，也就是估算每个点的介数。最后，仍然是采用[Brandes的介数估算算法][2]，才最复杂度降到可接受的$O(m+n)$。虽然牺牲了部分精度，但是在样本达到一定的时候，误差可以接受。

本文主要介实现该算法时，遇到的优化问题以及解决方法，不会涉及过多算法细节，需要详细了解的读者可以参考文章最后的"参考资料部分"。

## 算法框架
首先需要从精确计算介数谈起，首先定义一个变量，称之为dependency，简单来讲，在原始定义中，将源点s固定，v和t变化，形式的定义如下：

$$
	\delta_{s\bullet}(v) = \sum_\limits{t \ne v \in V}  \frac{\sigma_{st}(v)}{\sigma_{st}}
$$

Brandes的快速介数计算每计算一轮，会将所有v针对固定s的dependency计算出来。计算n轮，然后针对每个v，合并所有dependency，就可以计算每个点的dependency。每一轮的计算量包括：一个单元最短路径，一个前置节点搜集，一个依赖合并过程，需要三个pregel过程和一个vertexLeftJoin过程，每一轮的复杂度为$O(m+n)$。估算的方法式随机抽取k节点，计算对应的dependency，然后计算没个v的平均dependency并乘以n，得到最终估算结果。由于$k\ll m$，所以最终复杂度不变。以上就是计算介数估算的大体框架，具体细节请参考[Brades介数估算论文][2]，里面有估算的误差估计证明以及实现效果。


## 性能瓶颈


每一轮计算时间不长，但是聚合及其耗时。


## 逐步聚合
reduce by key one by one 
union one by one and reduce by key 
血缘月shuffle的问题

## union再聚合 
sc.union and reduce by key
shuffling的问题


## 去掉聚合，空间换时间
不用reduce by key,图节点保持每一轮的状态。执行的cache与unpersisit，避免血缘问题。

## 计算效果
不同边分区策略，导致计算效果的不同。


## spark优化总结
1 减少血缘
2 减少shuffling


## 参考资料
* [Brandes的快速介数计算][1]
* [Brandes的介数估算算法][2]
* spark优化策略
* 我的blog，请在action后unpersist


[1]:http://algo.uni-konstanz.de/publications/b-fabc-01.pdf
[2]:http://algo.uni-konstanz.de/publications/bp-celn-06.pdf