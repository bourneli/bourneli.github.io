---
layout: post
title:  "线代随笔07-关于基的那些事"
categories: [linear-algebra]
---

线性空间的基非常重要，正如其名字，它是线性空间的基石，线性空间围绕其构建。每个特定的线性空间，可以有无数不同的基，最好用，也是最常用的基是正交基，因为它乘起来得到单位矩阵。虽然基千变万化，但是基的数目稳定，称之为维度；且基一旦确定，可以唯一表示线性空间中的任意向量。

## 基唯一表示任意向量
设A的列向量$\vec{a_1},\cdots, \vec{a_k}$线性独立，$\vec{x}$为$C(A)$中任意向量，那么假设

$$
	\vec{x}=\sum^k_{i=1}c_i\vec{a_i}=\sum^k_{i=1}d_i\vec{a_i} \Rightarrow \vec{0} = \sum^k_{i=1}(c_i - d_i)\vec{a_i}
$$

由于线性独立，$c_i=d_i$,证毕。

## 维度不变
设有两组基

$$
	A = \begin{bmatrix} a_1 & \cdots & a_m \end{bmatrix},
	B = \begin{bmatrix} b_1 & \cdots & b_n \end{bmatrix},
	m \ne n，且 C(A)=C(B)=S
$$

那么，用A表示B

$\vec{a_1} = c_{11}\vec{b_1} + c_{12}\vec{b_2} + \cdots + c_{1n}\vec{b_n}$

$\vdots$

$\vec{a_m} = c_{m1}\vec{b_1} + c_{m2}\vec{b_2} + \cdots + c_{mn}\vec{b_n}$

所以 

$$
A = \begin{bmatrix} b_1 & \cdots & b_n \end{bmatrix}
    \underbrace{
		\begin{bmatrix}
			c_{11} & \cdots & c_{m1} \\
			\vdots & \vdots & \vdots \\
			c_{1n} & \cdots & c_{mn} \\
		\end{bmatrix}
	}_D = BD
$$	 

计算0空间，由于A线性独立，所以x=0恒成立

$$
	Ax = BDx=0 \Rightarrow B^TBDx=0 \Rightarrow (B^TB)^{-1}B^TBDx=Dx=0
$$

$B^T$列线性独立,[$B^TB必可逆$](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)。因为D是一个$n \times m$向量，所以 $n \le m$才能满足$x=0$恒成立。

同理，使用B表示A，可以推出$n \ge m$。最后得到$m=n$，证毕。

## 维度过多必定余
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


