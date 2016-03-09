---
layout: post
title:  "线代随笔06-正交矩阵Q及其应用"
categories: linear-algebra
---

如果矩阵的列向量互相正交，且长度为1，那么该矩阵称之为标准正交矩阵，不要求矩阵满秩。如果满秩，即Q是方正，称之为**正交矩阵(Orthogonal Matrix)**。标准正交矩阵有很多好的性质：

* $Q^TQ=I$，不要求Q为方阵。
* 如果$Q$为方阵，$Q^TQ=QQ^T=I \Rightarrow Q^T=Q^{-1}$。
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
	p & = \begin{bmatrix}q_1 \cdots q_2 \end{bmatrix} \begin{bmatrix} q_1^T \\ \vdots \\ q_2^T \end{bmatrix}b 
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
标准正交向量有这么好的性质，如何能正确的计算出来呢？我们知道，一个线性子空间的基是可以有无数组的，如果能够通过一特定线性子空间的任意特定基，找到一组等价标准正交基，那么岂不是很妙！Gram Schmidt算法就是用来处理这件事情，这里还是要感谢两位大神提供了这么好的算法。

该算法是迭代的，不能像之前的证明使用简单的公式表示。其核心思想是根据现有正交基，将新加入基正交化，确保每一次迭代，处理过的基都是正交的，直到处理完所有基。算法思路如下

1. 设$A=\begin{bmatrix} a_1 \cdots a_n \end{bmatrix}$，其中$a_i$线性独立
2. $Q=\begin{bmatrix}\end{bmatrix}$
3. $q_1= {a_1 \over \|\|a_1\|\|}$，将$q_1$放入$Q$，有$Q=\begin{bmatrix} q_1 \end{bmatrix}$
4. for $a_i$ in A and $i \ge 2$
5. * $分解a_i = aq_i + an_i,其中 aq_i \in C(Q) 且 an_i \in N(Q^T)$  分解就是减去投影
6. * $q_i = {an_i \over \|\| an_i \|\|}$,
7. * $Q=Q + q_i = \begin{bmatrix} q_1 \cdots q_i\end{bmatrix}$
7. end for 
8. 返回Q


## QR分解




