---
layout: post
title:  "线代随笔09-$A^TA$与A的零空间相同"
categories: [linear-algebra]
---

$A^TA$矩阵在线性代数应用中经常会用到，这次讨论的内容是证明$N(A)=N(A^TA)$。需要证明两点：

1. $\forall x \in N(A), x \in N(A^TA)$
2. $\forall x \in N(A^TA), x \in N(A)$

第一点，显而易见，$Ax = 0 \Rightarrow A^TAx=0$。

关键是第二点，当$A^TAx=0$时，是否存在$Ax \ne 0$?这里使用反正法。假设,

$$ \exists x, A^TAx=0, Ax \ne 0 $$

经过一番折腾，发现宏观上，无论怎么变换，也找不到破绽，所以必须从微观上观察，也就是$A^TAx$的结构，先定义$A^T$

$$ 
	A^T = \begin{bmatrix} a_1^T \\ \vdots \\ a_n^T \end{bmatrix}
$$
 
所以有

$$
	A^TAx = \begin{bmatrix} a_1^TAx \\ \vdots \\ a_n^TAx \end{bmatrix} = 0		  
$$

对任意$i \in (1, \cdots, n) $,有 $a_i^TAx=0$。因为$Ax \ne 0$,所以只有两种情况

1. $a_i = 0$
2. $a_i \ne 0, a_i \perp Ax$

对于情况2，因为$a_i, Ax \in C(A)$且$Ax \ne 0, a_i \ne 0$,所以$a_i^TAx \ne 0$，所以只有一种情况，

$$\forall i \in (1, \cdots, n), a_i \equiv 0$$

得到$A \equiv 0$，与原命题矛盾。所以，$ \exists x, Ax \ne 0, A^TAx=0 $不成立，所以原命题成立，证毕！

