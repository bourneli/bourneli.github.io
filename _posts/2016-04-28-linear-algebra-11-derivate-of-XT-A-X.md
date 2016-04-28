---
layout: post
title:  "线代随笔11-$x^TAx$求导"
categories: [linear-algebra,calculus]
---

这次探讨的主题是向量平方的求导，即$\frac{\partial (x^TAx)}{\partial x}$。注意，这是对向量求导，而不是常见的标量。

首先，定义标量对向量求导，令$f(x_1,\cdots,x_n)$为多元可导函数，记作$f(x)$，其中$x=(x_1,\cdots,x_n)^T$。$f$对$x$的倒导数定义为下面的n维向量：

$$
\nabla f = \frac{\partial f(x)}{\partial x} 
		 = \begin{bmatrix}
				\frac{\partial f(x)}{\partial x_1} \\	
				\vdots \\
				\frac{\partial f(x)}{\partial x_n} \\
		   \end{bmatrix} \qquad (1)
$$

令$A=\begin{bmatrix}c_1 & \cdots & c_n \end{bmatrix}=\begin{bmatrix}r_1 & \cdots & r_n \end{bmatrix}^T$，其中$c_i$表示$A$的列向量，$r_i$表示$A$的行向量。根据上面的定义，问题可以表示如下:

$$
	f(x) = x^TAx = \sum_{i=1}^n{x_ix^Tc_i} = \sum_{i=1}^n{\sum_{j=1}^n{x_ix_jc_{ij}}}
$$

现在计算$\frac{\partial f(x)}{\partial x_k}$，就是一般的偏导计算。需要寻找规律，只有当$k=i$或$k=j$的项保留，其他的都是常数，

$$
	\frac{\partial f(x)}{\partial x_k} = \sum_{j=1}^n{x_jc_{kj}} + \sum_{i=1}^n{x_ic_{ik}} 
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

如果A是对称矩阵，即$A^T=A$，代入(3),有$\nabla f = 2Ax$。当A退化为标量时，即维度为$1 \times 1$，结果与$ax^2$求导一致。



