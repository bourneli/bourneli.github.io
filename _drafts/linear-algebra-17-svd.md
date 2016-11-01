---
layout: post
title:  "线代随笔17-奇异值分解(SVD)推导"
categories: [linear-algebra]
---

奇异值分解可以说是基础线性代数的高潮部分，它将很多基础线性代数的内容串联起来，比如正交矩阵，基，正定，矩阵的秩，特征值，特征向量，矩阵的逆等等。奇异值分解的理论非常漂亮，它可以对任意形状的矩阵进行分解，而且得到分解形式也很特殊。本博文以一种倒叙的方式介绍奇异值分解，从中可以领会到数学美感。

假设矩阵A为任意维度，A的秩为$r$，如果不为方正，那么自然是得不到特征值和特征向量的。但是$A^TA$与$AA^T$却是可逆的，而且根据正定性，它们是半正定矩阵。所以，可以计算相关的特征值和特征向量。那么，这两个矩阵的特征向量是否有什么关系呢？

假设向量$v_1,\cdots.v_r \in R(A)$,$u_1,\cdots,u_r \in C(A)$，存在

$$
	Av_1 = \rho_1u_1, Av_2 = \rho_2u_2,\cdots , Av_r = \rho_ru_r
$$

那么可以得到

$$
	A\begin{bmatrix}v_1 & \cdots & v_r\end{bmatrix} = \begin{bmatrix}u_1 & \cdots & u_r\end{bmatrix}
	\begin{bmatrix}
		\rho_1 & & \\ 
               & \ddots & \\
         & & \rho_r
    \end{bmatrix}
$$


(可以扩展基到满秩)