---
layout: post
title:  "线代随笔09-矩阵乘法与秩"
categories: [linear-algebra]
---


**矩阵乘法不会增加矩阵的秩**。形式化表示如下

$$
	rank(AB) \le min(rank(A), rank(B))
$$

证明：

A乘以B，可以看作是对B的行做线性组合，而线性组合是不会对行空间R(B)有扩充，所以$rank(AB) \le rank(B)$。

利用上面的性质，有$rank(AB)=rank(B^TA^T)\le rank(A^T) = rank(A)$。

证毕！


**若A,B的列均线性独立，秩均为r**。根据之前的[随笔](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)，$B^TB$与$A^TA$均可逆，所以有

$$
	r = rank(A^TAB^TB) \le rank(AB^TB) \le  rank(AB^T) \le rank(B^T) = rank(B) = r
$$

所以

$$
	rank(AB^T)=rank(A)=rank(B)=r
$$

也就是说，按照上面形式相乘的两个矩阵，**秩无变化**。




