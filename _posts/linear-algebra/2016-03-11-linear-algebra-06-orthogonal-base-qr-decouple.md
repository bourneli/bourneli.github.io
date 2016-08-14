---
layout: post
title:  "线代随笔06-正交矩阵Q及其应用"
categories: [linear-algebra]
---

如果矩阵的列向量互相正交，且长度为1，那么该矩阵称之为标准正交矩阵，不要求矩阵满秩。如果满秩，即Q是方正，称之为**正交矩阵(Orthogonal Matrix)**。标准正交矩阵有很多好的性质：

* $Q^TQ=I$，不要求Q为方阵。
* 如果$Q$为方阵，$Q^TQ=QQ^T=I \Rightarrow Q^T=Q^{-1}$ (利用投影$P=QQ^T=I$证明)。
* $Qx$不改变x的长度。$\|Qx\|^{2}=(Qx)^TQx=x^TQ^TQx=x^Tx=\|x\|^2$
* $Q$不改变向量点积。$Qx \cdot Qy = (Qx)^TQy=x^TQ^TQy=x^Ty$ 


## 投影中，使用标准正交矩阵Q取代A
在上一篇博文[线代随笔05-向量投影与线性回归](/linear-algebra/2016/03/05/linear-algebra-05-projection-and-linear-regression.html)中，推导出了向量投影公式，里面有$A^TA$形式，如果将Q取代A，那么重新推导相关结论，

* 投影系数 $\hat{x}=(Q^TQ)^{-1}Q^Tb=Q^Tb$。
* 投影矩阵 $P=Q(Q^TQ)^{-1}Q^T=QQ^T$。如果$Q$只有一列$q$，$P=qq^T$（**下面会用到**）。
* 投影向量 $p=Q(Q^TQ)^{-1}Q^Tb=QQ^Tb$。

根据简化后的投影向量p，可以进一步观察p的组成结构，假设Q的列为n

$$
\begin{align}
	p & = QQ^Tb = \begin{bmatrix}q_1 \cdots q_2 \end{bmatrix} \begin{bmatrix} q_1^T \\ \vdots \\ q_2^T \end{bmatrix}b 
	    = \begin{bmatrix}q_1 \cdots q_2 \end{bmatrix} \begin{bmatrix} q_1^Tb \\ \vdots \\ q_2^Tb \end{bmatrix} \\
	  & = \sum_{i=1}^n (q_1^Tb)q_i = \sum_{i=1}^n q_i(q_1^Tb) = \sum_{i=1}^n (q_iq_i^T)b
\end{align}
$$

通过最终形式，可以发现向量$b$到$C(Q)$的投影，本质上是b到每个**正交向量$\vec{q_i}$的投影**的和,并且这些投影分量正交，设$\vec{q_i},\vec{q_j}$为$Q$中任意两列向量，

$$
\begin{align}
	(q_iq_i^T)b \cdot (q_jq_j^T)b &= ((q_iq_i^T)b)^T(q_jq_j^T)b \\
								  &= b^Tq_iq_i^Tq_jq_j^Tb \\
								  &= b^Tq_i(q_i^Tq_j)q_j^Tb \\
								  &= b^Tq_i(\vec{0})q_j^Tb \\
								  &= \vec{0}
\end{align}
$$

标准正交矩阵$Q$是一个非常优美的矩阵，它可将$b$投影到每个列向量，并且彼此之间正交，没有冗余。

## 计算标准正交向量---Gram Schmidt算法
标准正交向量有这么好的性质，如何能正确的计算出来呢？我们知道，一个线性子空间的基是可以有无数组的，如果能够通过一特定线性子空间的任意特定基，找到一组等价标准正交基，岂不妙哉！Gram Schmidt算法用来处理这件事情，这里还是要感谢两位大神提供了这么好的算法。

该算法是迭代的，不能像之前使用简单的公式表示。其核心思想是根据现有正交基，通过投影，将新加入基正交化，确保每一次迭代，处理过的基都是正交的，直到处理完所有基。算法思路如下

1. 设$A=\begin{bmatrix} a_1 \cdots a_n \end{bmatrix}$，其中$a_i$**线性独立**(否则算法无法执行)
2. $Q=\begin{bmatrix}\end{bmatrix}$，最开始是空的
3. $q_1= {a_1 \over \|\|a_1\|\|}$，将$q_1$放入$Q$，有$Q=\begin{bmatrix} q_1 \end{bmatrix}$
4. for $a_i$ in A and $i \ge 2$
5. * $n_i = a_i - QQ^Ta_i = (I-QQ^T)a_i$
6. * $q_i = {n_i \over \|\| n_i \|\|}$，线性独立确保$\|\|n_i\|\| \ne 0$
7. * $Q=Q + q_i = \begin{bmatrix} q_1 \cdots q_i\end{bmatrix}$
7. end for 
8. 返回Q

## 矩阵分解：A=QR
在上面的过程中，将$a_i$分解为正交基的线性组合

$a_1 = (q_1^Ta_1)q_1$

$a_2 = (q_1^Ta_2)q_1+(q_2^Ta_2)q_2 $ 

$a_3 = (q_1^Ta_3)q_1+(q_2^Ta_3)q_2+(q_3^Ta_3)q_3 $

$\vdots$ 

$a_n = (q_1^Ta_n)q_1 + (q_2^Ta_n)q_2 + \cdots + (q_n^Ta_n)q_n$

将上面的公式总结为矩阵形式，

$$
	A = \begin{bmatrix} a_1 & a_2 & \cdots & a_n\end{bmatrix} 
	  =  \underbrace{\begin{bmatrix} q_1 & q_2 & \cdots & q_n\end{bmatrix}}_Q
		 \underbrace{
			\begin{bmatrix}
				q_1^Ta_1 & q_1^Ta_2 & q_1^Ta_3 & \cdots & q_1^Ta_n \\
				0		 & q_2^Ta_2 & q_2^Ta_3 & \cdots & q_2^Ta_n \\
				0        & 0        & q_3^Ta_3 & \cdots & q_3^Ta_n \\
				\vdots	 & \vdots   & \vdots   & \ddots & \vdots    \\
				0        & 0        & 0        & \cdots & q_n^Ta_n
			\end{bmatrix}
		}_R = QR
$$

上面就是A=QR分解，是不是很简洁，完美诠释了Gram-Schmidt算法的整个过程。

R的每列表示的是$a_i$在$q_i$上的投影系数，由于$a_i$只投影到$q_1 \cdots q_i$上，所以R是一个上三角矩阵。Q不一定可逆，但R必须可逆。$A^TA=R^TQ^TQR=R^TR \Rightarrow x^TR^TRx={\|\|Rx\|\|}^2 > 0, 当x \ne \vec{0}$，所以$A^TA$**正定**。

## 总结
本文主要描述了正交矩阵的定义与性质，尤其是正交矩阵应用在矩阵投影中，有着十分简洁和优美的特性。然后介绍了正交矩阵计算方法--Gram Schmidt算法，进而抽象成矩阵分解A=QR。希望读者喜欢。
