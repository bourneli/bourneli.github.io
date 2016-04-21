---
layout: post
title:  "线代随笔10-极大似然计算马尔科夫矩阵"
categories: [linear-algebra, statistic, optimization]
---

马尔科夫链可用于观察物体状态转换，日常生活中有大量的数据与状态有关。本文将介绍如何使用样本数据计算马尔科夫矩阵，并且给出scala spark实现的示例代码。首先，定义一些符号，

* $X^n$表示随机序列，长度为n。$x^n \equiv x_1,x_2,\cdots, x_n$表示该随机序列样本。
* $X_n$表示随机状态，$x_n$表示样本。（注意：n在X右上与右下的意义是不同的。）
* $p_{ij}=Pr(X_n=j\\|X_{n-1}=i)$表示状态从i到j的概率。
* $n_{ij}$表示状态从i到j的样本个数。
* k为状态的数量

那么出现样本$x^n\equiv x_1,x_2,\cdots, x_n$的概率如下

$$
\begin{align}
	Pr(X^n=x^n) &= Pr(X_1 = x_1)Pr(X_2=x_2|X^1 = x^1)\cdots Pr(X_n=x_n|X^{n-1}=x^{n-1}) \\
				&= Pr(X_1 = x_1) \prod_{t=2}^n{Pr(X_t=x_t|X^{t-1}=x^{t-1})} \\
				&= Pr(X_1 = x_1) \prod_{t=2}^n{Pr(X_t=x_t|X_{t-1}=x_{t-1})} \\
\end{align}
$$

上面的等式中，第一，二行直接使用条件概率，计算样本概率，第三行利用了马尔科夫链的性质，即第t个状态只与t-1的状有关，与之前的状态无关。由于各个状态转换是固定的，变的是不同转换的次数，所以上面的等式可以换一种方式表示，如下：

$$
	Pr(X^n=x^n) = Pr(X_1 = x_1)\prod_{i=1}^k\prod_{j=1}^k{p_{ij}^{n_{ij}}}
$$

现在概率公式已经化简的比较简单了，但是还有一个问题：现在只观察了一个对象的转换序列，如果多个对象呢？接下来，将上面的公式对m个对象进行扩展，先定义一些补充变量，

* 总共有$m$个对象。
* $X(l)^n$是第$l$个对象的随机序列。
* $X(l)_n$是第$l$个对象的第n个状态的随机变量。
* $n(l)$表示第$l$个对象的状态转换次数。
* $n(l)_{ij}$表示第$l$个对象状态从i到j的样本个数。
* $N_{ij} = \sum_{l=1}^m{n(l)_{ij}}$，表示所有样本中状态i到j的样本数。

并且，**假设m个对象相互独立**，扩展后的样本概率为

$$
\begin{align}
	\prod_{l=1}^m{Pr(X(l)^{n(l)} = x(l)^{n(l)})} 
		&= \prod_{l=1}^m{\left( Pr(X(l)_1 = x(l)_1) \prod_{i=1}^k\prod_{j=1}^k{p_{ij}^{n(l)_{ij}}} \right)} \\
		&= \prod_{l=1}^m{\left( Pr(X(l)_1 = x(l)_1) \right)} \prod_{l=1}^m{\left( \prod_{i=1}^k\prod_{j=1}^k{p_{ij}^{n(l)_{ij}}} \right)} \\					 
		&= \prod_{l=1}^m{\left( Pr(X(l)_1 = x(l)_1) \right)}  \prod_{i=1}^k\prod_{j=1}^k{p_{ij}^{\sum_{l=1}^m{n(l)_{ij}}}} \\
		&= \prod_{l=1}^m{\left( Pr(X(l)_1 = x(l)_1) \right)}  \prod_{i=1}^k\prod_{j=1}^k{p_{ij}^{N_{ij}}} \\
\end{align}
$$

上面就是目标函数，设为$L(p)$，p是转换矩阵。现在需要计算L(p)的极值。


# 参考
* [马尔科夫链](https://zh.wikipedia.org/wiki/%E9%A9%AC%E5%B0%94%E5%8F%AF%E5%A4%AB%E9%93%BE)
* [拉格朗日乘子式](https://zh.wikipedia.org/wiki/%E6%8B%89%E6%A0%BC%E6%9C%97%E6%97%A5%E4%B9%98%E6%95%B0)
* [Note: Maximum Likelihood Estimation for Markov Chains](http://www.stat.cmu.edu/~cshalizi/462/lectures/06/markov-mle.pdf)




