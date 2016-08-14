---
layout: post
title:  "线代随笔07-关于基的那些事"
categories: [linear-algebra]
---

线性空间的基非常重要，正如其名字，它是线性空间的基石，线性空间围绕其构建。每个特定的线性空间，可以有无数不同的基，最好用，也是最常用的基是正交基，因为它乘起来得到单位矩阵。虽然基千变万化，但是基的数目稳定，称之为维度；且基一旦确定，可以唯一表示线性空间中的任意向量。下面记录基的一些性质并且给出相关证明。

## 基唯一表示任意向量
设A的列向量$\vec{a_1},\cdots, \vec{a_k}$线性独立，$\vec{x}$为$C(A)$中任意向量，那么假设

$$
	\vec{x}=\sum^k_{i=1}c_i\vec{a_i}=\sum^k_{i=1}d_i\vec{a_i} \Rightarrow \vec{0} = \sum^k_{i=1}(c_i - d_i)\vec{a_i}
$$

由于线性独立，$c_i=d_i$,证毕。

## 维度不变
设空间$S$有两组基

$$
	A = \begin{bmatrix} a_1 & \cdots & a_m \end{bmatrix},
	B = \begin{bmatrix} b_1 & \cdots & b_n \end{bmatrix}
$$

即$S=C(A)=C(B)$

用B表示A

$\vec{a_1} = c_{11}\vec{b_1} + \cdots + c_{1n}\vec{b_n}$

$\vdots$

$\vec{a_m} = c_{m1}\vec{b_1} + \cdots + c_{mn}\vec{b_n}$

所以 

$$
A = \begin{bmatrix} b_1 & \cdots & b_n \end{bmatrix}
    \underbrace{
		\begin{bmatrix}
			c_{11} & \cdots & c_{m1} \\
			\vdots & \vdots & \vdots \\
			c_{1n} & \cdots & c_{mn} \\
		\end{bmatrix}
	}_C = BC
$$	 

观察C的形状，假设$m \gt n$，那么C是一个宽行的矩阵，也就有如下结论:

$$
	m = rank(A) = rank(BC) \le rank(C) \le n
$$

上面现象与假设矛盾，所以假设不成立，假设的逆命题$n \ge m$成立。


同样，用A表示B

$\vec{b_1} = d_{11}\vec{a_1} + \cdots + d_{1m}\vec{a_m}$

$\vdots$

$\vec{b_n} = d_{n1}\vec{a_1} + \cdots + d_{nm}\vec{a_m}$

使用矩阵表示，总结如下

$$
B = \begin{bmatrix} a_1 & \cdots & a_m \end{bmatrix}
    \underbrace{
		\begin{bmatrix}
			d_{11} & \cdots & d_{n1} \\
			\vdots & \vdots & \vdots \\
			d_{1m} & \cdots & d_{nm} \\
		\end{bmatrix}
	}_D = AD
$$	 

现在$n \ge m$，所以$D$是一个宽矩阵，有如下结论：

$$
	n = rank(B) = rank(AD) \le rank(D) \le m
$$

所以，综合上面结论，只有$m = n$这一种情况，在上面两种情况下均成立，证毕。

上面的证明再一次的演示了矩阵表示的简洁与优美，可以将问题简化，方便观察特征，找到解决办法。

P.S.: $rank(AD) \le rank(D)$的证明参考[线代随笔09-矩阵乘法与秩](/linear-algebra/2016/04/17/linear-algebra-09-BTA-inverse.html)。

## 维度过多必定冗余
在空间$R^n$中，任意两子空间$V$和$W$，如果$dim(W)+dim(V) \gt n$，那么V，W交集必有非0向量。
证明：假设V，W的结构如下

$V=span({v_1, \cdots, v_k}), 其中v_i线性独立$
$W=span({w_1, \cdots, w_{n-k}, \dots, w_j}), 其中w_i线性独立$

令$A = \begin{bmatrix}v_1 & \cdots & v_k & w_1 & \cdots & w_{n-k} \end{bmatrix}$,且列向量线性独立，否则无需证明。

那么$C(A)=R^n$，有$w_j = \sum^k_{i=1}{c_iv_i} + \sum^{n-k}_{i=1}{d_iw_i}$

令
$$
	w_{j1} = \sum^{n-k}_{i=1}{d_iw_i}, w_{j2}= \sum^{k}_{i=1}{c_iv_i} \Rightarrow  w_j = w_{j1}+w_{j2}
$$

所以，$w_{j2} \in V 且 w_{j2} \in W$，证毕。


