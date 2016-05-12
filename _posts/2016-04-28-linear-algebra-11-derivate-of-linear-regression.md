---
layout: post
title:  "线代随笔11-线性回归相关的向量求导"
categories: [linear-algebra,calculus]
---

线性回归的计算推导方法有许多，其中有一种使用矩阵运算，涉及到标量对向量的求导，本文主要介绍相关的两个向量求导计算过程：$\frac{\partial x^TAx}{\partial x}$与$\frac{\partial b^TAx}{\partial x}$。

## 标量对向量求导
令$f(x_1,\cdots,x_n)$为多元可导函数，记作$f(x)$，其中$x=(x_1,\cdots,x_n)^T$。$f$对$x$的倒导数定义为下面的n维向量：

$$
\nabla f = \frac{\partial f(x)}{\partial x} 
		 = \begin{bmatrix}
				\frac{\partial f(x)}{\partial x_1} \\	
				\vdots \\
				\frac{\partial f(x)}{\partial x_n} \\
		   \end{bmatrix} \qquad (1)
$$

即$f(x)$偏导组成的列向量。

## $x^TAx$向量求导

令$A=\begin{bmatrix}c_1 & \cdots & c_n \end{bmatrix}=\begin{bmatrix}r_1 & \cdots & r_n \end{bmatrix}^T$，其中$c_i$表示$A$的列向量，$r_i$表示$A$的行向量。根据上面的定义，问题可以表示如下:

$$
	f(x) = x^TAx = \sum_{i=1}^n{x_ix^Tc_i} = \sum_{i=1}^n{\sum_{j=1}^n{x_ix_jc_{ij}}}
$$

计算$\frac{\partial f(x)}{\partial x_k}$，只有当$i=k$或$j=k$的项保留，其他的都是常数，

$$
	\frac{\partial f(x)}{\partial x_k} = \sum_{j=1}^n{c_{kj}x_j} + \sum_{i=1}^n{c_{ik}x_i} 
	                                   = r_k^Tx + c_k^Tx = (r_k^T + c_k^T)x \qquad (2)
$$

将(2)导入(1)得到最后通解，


$$
\nabla f = \frac{\partial (x^TAx)}{\partial x} 
		 = \begin{bmatrix}
				(r_1^T + c_1^T)x \\
				\vdots \\
				(r_n^T + c_n^T)x \\
		   \end{bmatrix}
		  = \left( \begin{bmatrix} r_1^T \\ \vdots \\ r_n^T \end{bmatrix} + 
			\begin{bmatrix} c_1^T \\ \vdots \\ c_n^T \end{bmatrix} \right) x
		  = (A + A^T)x  \qquad(3)
$$

推导完毕！



## $b^TAx$向量求导

令$A=\begin{bmatrix} a_1 && \cdots && a_n\end{bmatrix}$，问题定义$f(x)$如下

$$
	f(x) = b^TAx = b^T\sum_{i=1}^n{a_ix_i}=\sum_{i=1}^n{b^Ta_ix_i}
$$

计算偏导

$$
	\frac{\partial (b^TAx)}{\partial x_k} = b^Ta_k \qquad (4)
$$

将(4)代入(1)，

$$
	\nabla f = \frac{\partial (b^TAx)}{\partial x} 
			 = \begin{bmatrix} b^Ta_1 \\ \vdots \\ b^Ta_n \end{bmatrix}
			 = \begin{bmatrix} a_1^Tb \\ \vdots \\ a_n^Tb \end{bmatrix}
			 = \begin{bmatrix} a_1^T \\ \vdots \\ a_n^T \end{bmatrix} b 
			 = A^Tb
$$

推导完毕！

## 总结
如果A是对称矩阵，即$A^T=A$，代入(3),有$\nabla f = 2Ax$。当A退化为标量时，结果与$ax^2$求导一致。仔细观察(4)，相当于将$x$的系数转置，当A退化为标量时，结果与$bax$求导一致。通过这两个矩阵求导，发现其实矩阵的多项式求导与常规多项式求导有一定的相似性，这一点值得好好体会。


## 相关资料
* [机器学习基石，线性回归相关章节](https://zh.coursera.org/course/ntumlone)
* [矩阵导数](https://ccjou.wordpress.com/2013/05/31/%E7%9F%A9%E9%99%A3%E5%B0%8E%E6%95%B8/)

 




