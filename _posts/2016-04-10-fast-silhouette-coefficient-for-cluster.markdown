---
layout: post
title:  轮廓系数的快速计算方法
categories: [inequality,cluster,kmeans]
---

[轮廓系数（Silhouette Coefficient）](https://en.wikipedia.org/wiki/Silhouette_(clustering))用来评估聚类的效果。但是，其缺陷是计算复杂度为$O(n^2)$，需要计算距离矩阵，那么当数据量上到百万，甚至千万级别时，计算开销会非常巨大（想想也是醉了，从来不敢在这个量级上计算轮廓系数）。现在讨论一种快速计算轮廓系数的方法，将复杂度降为$O(n)$，同时简单的分析快速计算方法与原始方法的效果差异。

## 快速方法
大致思路是计算每个点与k个簇的平均中心点的距离的平方，使用这个距离表示每个点与簇的距离关系，而不是原始定义中使用每个点与每个簇的平均距离平方和。使用$d_1$表示原始方法的欧式距离，${d_2}$表示快速计算方法，假设某个簇有n个点，任意点为$b$，如下

$$
	d_1 = {1 \over n} \sum_{i=1}^n(b-c_i)^2, d_2 = (b-{1 \over n}\sum_{i=1}^nc_i)^2
$$

现在计算两个距离的差值，

$$
	\Delta = d_1 - d_2 = {1 \over n} \sum_{i=1}^n(b-c_i)^2 - (b-{1 \over n}\sum_{i=1}^nc_i)^2 
	       = {1 \over n^2}\bigg(n\sum_{i=1}^nc_i - \bigg(\sum_{i=1}nc_i\bigg)^2 \bigg)
$$

根据[柯西不等式变形式](/inequality/2016/04/10/cauchy-schwarz-inequality.html)，$\Delta \ge 0$恒成立
所以，**快速计算法计算的欧式距离永远小于原始计算方法**。
 
## 轮廓系数应用于Kmeans聚类评估
轮廓系数的计算公式如下

$$
	s(i) = \begin{cases}
		  1-a(i)/b(i), & \mbox{if } a(i) < b(i) \\
		  0,  & \mbox{if } a(i) = b(i) \\
		  b(i)/a(i)-1, & \mbox{if } a(i) > b(i) \\
	\end{cases}
$$

其中$a(i)$是到本类的聚类，$b(i)$是到最近的其他簇的距离，由于kmeans算法，$a(i) \le b(i)$恒成立，所以$s(i) \ge 0$。但是由于快速算法的结果均比原始方法小，导致并不能明显的看出轮廓系数在两种计算方法上的大小关系，所以从目前来看，并不能否认快速计算方法与原始计算方法有明显的差异。
 
## 参考
* [Silhouette (clustering)](https://en.wikipedia.org/wiki/Silhouette_(clustering))
* [柯西-施瓦茨不等式推广到向量空间$\Bbb R^m$](/inequality/2016/04/10/cauchy-schwarz-inequality.html)