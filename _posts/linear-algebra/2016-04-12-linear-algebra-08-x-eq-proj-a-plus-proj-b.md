---
layout: post
title:  "线代随笔08-任何向量都可以表示为任意两个正交补空间投影的和"
categories: [linear-algebra]
---

在$\Bbb R^m$空间中，任意向量$x$可以由其在两个正交补空间的投影相加得到。直观的理解，正交补子空间将$\Bbb R^m$严格的切分成了两个部分，这两个部分唯一的交集是$\Bbb 0$向量。下面来证明这个论断。

假设A，B矩阵，列向量线性独立，且$C(A)^{\perp}=C(B)$。那么$B^TA=0$，且[$A^TA,B^TB$均可逆](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)。任意向量$x \in \Bbb R^m$在$C(A),C(B)$的[投影](/linear-algebra/2016/03/05/linear-algebra-05-projection-and-linear-regression.html)如下

$$ p_a = A(A^TA)^{-1}A^Tx$$

$$ p_b = B(B^TB)^{-1}B^Tx$$

现在即证明 

$$
	x = A(A^TA)^{-1}A^Tx + B(B^TB)^{-1}B^Tx = (A(A^TA)^{-1}A^T + B(B^TB)^{-1}B^T)x
$$

将x左边括号内部单独提取出来，记作$S$,

$$
	S = A(A^TA)^{-1}A^T + B(B^TB)^{-1}B^T
$$

由于$B^TA=0$，所以为了利用此条件，等式两边分别乘以$B^T$

$$
	\begin{align}
	B^TS &= B^TA(A^TA)^{-1}A^T + B^TB(B^TB)^{-1}B^T \\
	     &= 0(A^TA)^{-1}A^T + (B^TB)(B^TB)^{-1}B^T \\
		 &= B^T \\
	\end{align}
$$

是不是很简单了，同理两边乘以$A^T$，

$$
	\begin{align}
	A^TS &= A^TA(A^TA)^{-1}A^T + A^TB(B^TB)^{-1}B^T \\
	     &= (A^TA)(A^TA)^{-1}A^T + 0(B^TB)^{-1}B^T \\
		 &= A^T \\
	\end{align}
$$

目前利用了$C(A) \perp C(B)$条件，但是仅仅这样是不够的，无法约束$S=I$恒成立。下面利用C(A)与C(B)的互补性，也就是$\begin{bmatrix}B & A\end{bmatrix}$可逆,其转置$$\begin{bmatrix}B^T \\ A^T\end{bmatrix}$$也可逆。那么，结合上面的条件，

$$
	\begin{bmatrix}B^T \\ A^T\end{bmatrix}S=\begin{bmatrix}B^TS \\ A^TS\end{bmatrix} = \begin{bmatrix}B^T \\ A^T\end{bmatrix}
$$

两边同时乘以$$\begin{bmatrix}B^T \\ A^T\end{bmatrix}^{-1}$$，得到$S=I$恒成立，所以原始等式恒成立，证毕。


 
