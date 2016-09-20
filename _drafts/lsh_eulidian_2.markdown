---
layout: post
title:  LSH在欧式空间的应用(2)--工作原理
categories: [probability,LSH]
---

本文主要介绍LSH的整个工作原理，并结合欧式距离给出实例。

## $(r_1,r_2,p_1,p_2)-sensitive$

LSH的核心工作原理就是找到一组hash函数，它们必须符合$(r_1,r_2,p_1,p_2)-sensitive$条件。这类函数定义：$H=\lbrace h: S \rightarrow U \rbrace$，对于任意一种距离度量方法D，任意向量$q,v \in S$

$$
	如果 v \in B(q, r1), P(h(v)=h(q)) \ge p1 \\
	如果 v \notin B(q, r2), P(h(v)=h(q)) \le p2
$$

其中$B(v,r) = \lbrace q \in X \| d(v,q) \le r \rbrace$，表示以点$v$为中心，距离为r范围内的所有点的集合。

上面定义的直观解释就是Locality Sensitive Hashing。
即将相距较近($r_1$以内)的向量hashing到一起的概率要大($p_1$较接近1)；距离较远($r_2$以外，且$r_2 > r_1$)的对象hashing到一起的概率小($p_2$较接近0)。

## 欧式空间中的$(r_1,r_2,p_1,p_2)-sensitive$应用

在上一篇文章[LSH在欧式空间的应用(1)--碰撞概率分析](/probability/lsh/2016/09/15/lsh_eulidian_1.html)中讨论的投射方法和碰撞概率函数，就是分别对应上面定义的$h$和$P$。尤其是碰撞函数，可以根据给出的$r_1$和$r_2$情况估算$p_1$和$p_2$，并且计算最优的$w$的范围。首先，回顾一下碰撞函数，

$$
	P(h(v)=h(q)) = g(c) = 1 - 2F(-\frac{1}{c}) + \sqrt{\frac{2}{\pi}}c(e^{-\frac{1}{2c^2}}-1)
$$

其中$c=\frac{u}{w}$,假设$r_1=5,r_2=50,p_1 = 0.95, p_2 = 0.1$。根据碰撞概率的解析公式，可以得到$p_1,p_2$对应的数值解。即$c_1 = g^{-1}(p_1),c_2=g^{-1}(p_2)$。由于概率公式是减函数，所以

$$
	\begin{align}
	\frac{r1}{w} \le c_1, \frac{r2}{w} \ge c_2 &\Rightarrow \frac{r1}{g^{-1}(p_1)} \le w \le \frac{r2}{g^{-1}(p_2)} \\
	&\Rightarrow \frac{5}{g^{-1}(0.95)} \le w \le \frac{50}{g^{-1}(0.1)} \\
	\end{align}
$$

$p_1,p_2与r_1,r_2$可以根据应用精度和数据范围，人工估算。通过碰撞公式，完美的解决了$w$的取值问题，非常具有指导意义。但是，上述不等式不是永远成立的，部分$p_1,p_2与r_1,r_2$的组合可能导致不等式下界大于上界。


## 强化LSH函数

假设有一个$(r_1,r_2,p_1,p_2)-sensitive$函数族$F$，可以通过**逻辑和**的方法构造一个新的函数族$F'$。

假设$f \in F'$并且$f_i \in F, i = 1,2,\cdots r$。令$f(x)=f(y)$为$f_i(x)=f_y(i)$，此时$F'$是$(r_1, r_2, p_1^r, p_2^r) - sensitive$。如果$p_2$较小,$p_1$较大，可以通过此操作将其进一步的缩小，同时有不会将$p_1$变得太小。

同理，可以使用**逻辑与**的方法，将其变成一个$(r_1, r_2, 1-(1-p_1)^r, 1-(1-p_2)^r) - sensitive$。其效果与逻辑和相反，它将概率均变大。

常用的方法是将逻辑和嵌套到逻辑与中使用，得到$(r_1, r_2, 1-(1-p_1^k)^L, 1-(1-p_2^k)^L) - sensitive$函数族（先使用逻辑与，再使用逻辑和也可）。这种组合的意义是去掉那些碰巧hash到一起的情况，如果真的很近，在L组计算中，总有一组k个hash均相等。k和L需要设置合理，L如果设置太大，计算开销会增加。


## LSH的工作流程


参考Mining of Massive Dataset section 3.6.3

<div align='center'>
	<img src='/img/lsh_create_table.png' />
</div>

<div align='center'>
	<img src='/img/lsh_query.png' />
</div>

