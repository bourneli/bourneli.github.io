---
layout: post
title:  "Spark优化那些事(2)-graphx实现介数估算踩坑总结"
categories: [scala,spark,sns,graphx,graph]
---

## 背景
最近一段时间在使用spark graphx实现介数估算。介数（betweenness）可分为点介数和边介数，在图中衡量一个点或边的重要程度，后面的讨论均是针对点介数，边介数实现方式类似。这个指标虽然好，但是计算开销非常之大，如果没有优化，纯粹按照定义计算，复杂度为$O(n^3)$(n是节点数)，定义如下:

$$
	Bet(v) = \sum_\limits{s \ne t \ne v \in V}  \frac{\sigma_{st}(v)}{\sigma_{st}},
$$

其中$\sigma_{st}$是s，t之间最短路径的数量，$\sigma_{st}(v)$是s，t最短路径中包含节点v的数量。  


根据[Brandes的快速介数计算][1]，可以将复杂度降到$O((m+n)n)$(m，n分别是边数和节点数)。但是，即使是spark，面对此计算量计算上百万节点的图仍然不太现实。所以，只能采取折中的方法，也就是估算每个点的介数。最后，仍然是采用[Brandes的介数估算算法][2]，才使复杂度降到可接受的$O(m+n)$。虽然牺牲了部分精度，但是在样本达到一定数据量时，误差可以降到可接受的范围。

本文主要介绍实现该算法时，遇到的性能问题以及解决方法，不会涉及过多算法细节，需要详细了解的读者可以参考文章最后的**参考资料**中的相关论文。

## 算法框架
首先需要从精确计算介数谈起，首先定义一个变量，称之为dependency，简单来讲，在原始定义中，将源点s固定，v和t变化，形式的定义如下：

$$
	\delta_{s\bullet}(v) = \sum_\limits{t \ne v \in V}  \frac{\sigma_{st}(v)}{\sigma_{st}}
$$

Brandes的快速介数计算算法中，每计算一轮会将所有v针对固定s的dependency计算出来，计算n轮，然后针对每个v，合并所有dependency，就可以计算每个点介数值。每一轮的计算量包括：一个单源最短路径，一个前置节点搜集，一个依赖合并过程，需要三个pregel过程和一个vertexLeftJoin过程，每一轮的复杂度为$O(m+n)$。介数估算的方法是随机抽取k节点，计算对应的dependency，然后计算每个v的平均dependency并乘以n，得到最终估算结果，理论依据是[霍夫丁不等式][3](有届随机变量样本均值与均值期望的误差可以控制在一定范围内)。由于$k\ll m$，所以最终复杂度不变。以上就是计算介数估算的大体框架，具体细节请参考[Brades介数估算论文][2]，里面有估算的误差估计证明以及实验效果。

## 性能瓶颈
算法过程虽然不复杂，但是在实现时却遇到了性能瓶颈。经过排查，每一轮dependency计算开销相对稳定，主要不稳定的开销在最后按点聚合dependency的过程中。在百万点，千万边的无标度图中，该过程有时候只需要**几分钟**，但是大多数时候却需要**几小时甚至更多**！所以，主要的优化点就是如何提高聚合这块的性能。此过程经过了几轮改造，最后将此步骤的耗时稳定在几分钟，下面分享此过程的演进过程。
 

## 版本1：逐步聚合 vs 合并后聚合
聚合的第一个版本，可以用下面简化代码演示，

{% highlight scala %}
// 计算依赖
def dependency(id:Int, graph:Graph[Double, Double]):RDD[(Long, Double)] = { ... }  
val data:Graph[Double, Double] = ... // loading from hdfs

// 随机抽样
val idList = data.vertices.takeSample(10)
// 方案1：全部聚合，然后统一reduceByKey，只有一次shuffle
val rst1 = idList.map(id => dependency(id, data))
                 .reduce(_ union _).reduceByKey(_+_)			 
// 方案2：每一步都聚合，减少内存使用，但有多次shuffle
val rst2 = idList.map(id => dependency(id, data))
                 .reduce((l,r) => (l union r).reduceByKey(_+_))
{% endhighlight %}
在第一个版本中，尝试过上面两个方案:1)先全部union，然后一次shuffle，这样在聚合的时会对内存要求过高；2)逐步聚合，虽然有多次shuffle，但是减少内存使用。两个方案可以理解为空间与实践的置换，前置空间换时间，后者用时间换空间。但很不幸，两方案均有性能瓶颈！后来经过分析与网上求助，最终找到问题所在：两个方案都使血缘(lineage)变长了。比如第一个方案，第一个dependency的结果在最后的结果中，会经过9次血缘，第二个dependency经过了8此，以此类推。而第二个方案，更加可怕，第一个经过了18次，第二个是16次。这么长的血缘，出错的几率是非常大的，所以需要大量的重跑，这也是导致最后shuffle不稳定的原因，有大量出错导致重计算。网上有个类似问题，可以参考[Stackoverflow due to long RDD Lineage][4]。

## 版本2：批量union，减少血缘
spark context对象提供一个union方法，用于批量聚合若干个rdd，并且没有血缘叠加效果，最终，将代码改为如下：

{% highlight scala %}
// 计算依赖
def dependency(id:Int, graph:Graph[Double, Double]):RDD[(Long, Double)] = { ... }  
val data:Graph[Double, Double] = ... // loading from hdfs

// 随机抽样
val idList = data.vertices.takeSample(10)
val rst1 = sc.union(idList.map(id => dependency(id, data))).reduceByKey(_+_) 	 
{% endhighlight %}
上面的代码虽然减少了每个dependency RDD的血缘，但是shuffle还是很慢且不稳定。shuffle阶段有时需要几分钟，有时需要数个小时。不知道是不是由于需要同时容纳数个rdd，占据了大量内存，然后不断出现错误，进而导致重新计算，最终shuffle变得不稳定，这一点还有待证实。

## 版本3：去掉shuffle，空间换时间
前两个版本的问题都出在shuffle阶段，最后痛定思痛，决定去掉shuffle。大致思路是将每一轮每个点的dependency对象存储在节点中，该值在计算单轮dependency时不参与任何计算，只在计算完后，与当前的合并，这样就华丽的避开了shuffle过程。简化代码如下：

{% highlight scala %}
// 计算依赖
def dependency(id:Int, g:Graph[Double, Double]):Graphx[(Double, Double)] = { ... }  
val data:Graph[Double, Double] = ... // loading from hdfs

val sampleVertices = data.vertices.takeSample(10)
var dep = data
for(source <- sampleVertices) {
    val oldDep = dep // 迭代使用图对象，同时最多只要两个图对象
    dep = dependency(source, dep).persist(cacheLevel)  
    oldDep.unpersist(false) // 释放之前的资源，已经无用
}
val rst = dep.mapVertices((_, attr) => attr * verticesNum / sampleSize) // 合并
{% endhighlight %}

## 计算效果
经过上面几轮优化后，效果有了大幅度提升。还有一些优化的小技巧，这里捎带提一下，比如使用EdgePartition2D的边分区策略可以进一步提高执行效率，合理的释放和缓存图对象也可以减少血缘，减少重算时间。现在使用100个executor，2核，每个executor 14G内存，对450,000,000点，1,253,792,736 边的图，随机迭代5轮，只需要**279分钟**，且大部分时间用在第一轮迭代（耗时209分钟），随后的几轮的迭代会显著递减，当达到第5轮时，只需要5分钟。可能在前面1,2轮中缓存了大部分图结构，导致了后面计算的加速。

## spark优化总结
spark底层api优化其实就两点 ：

1. **减少血缘** 合理仔细的利用persisit，checkpoint和unpersisit，缓存中间变量并去掉无用的对象，避免过长的血缘重计算与合理的利用内存。但是，如果不适当的释放内存，可能导致没有缓存对象，仍然导致过长的血缘，这一点可以参考[Spark优化那些事(1)-请在action之后unpersisit!](http://bourneli.github.io/scala/spark/2016/06/17/spark-unpersist-after-action.html)。
2. **减少shuffling** shuffling需要网络开销，能少就少，能不用就不用。

上面的迭代过程其实就遵循上面两个原则进行的，最后得到了不错的效果。

## 参考资料
* [Brandes的快速介数计算][1]
* [Brandes的介数估算算法][2]
* [Stackoverflow due to long RDD Lineage][4]


[1]:http://algo.uni-konstanz.de/publications/b-fabc-01.pdf
[2]:http://algo.uni-konstanz.de/publications/bp-celn-06.pdf
[3]:https://en.wikipedia.org/wiki/Hoeffding%27s_inequality
[4]:http://stackoverflow.com/questions/34461804/stackoverflow-due-to-long-rdd-lineage
