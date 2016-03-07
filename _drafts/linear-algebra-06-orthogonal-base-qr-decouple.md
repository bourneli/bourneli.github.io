---
layout: post
title:  "线代随笔06-正交基与QR分解"
categories: linear-algebra
---

## 标准正交矩阵Q
如果矩阵的列向量互相正交，且长度为1，那么该矩阵称之为标准正交矩阵，不要求矩阵满秩。如果满秩，即Q是方正，称之为正交矩阵(Orthogonal Matrix)。标准正交矩阵有很多好的性质，这就是为什么常用的原因。

* $Q^TQ=I$，不要求Q为方阵。
* $Q^T=Q^{-1}$，需要Q为方阵。

* Qx不改变x的长度。$\|Qx\|^{2}=(Qx)^TQx=x^TQ^TQx=x^Tx=\|x\|^2$
* Q不改变向量点积。$Qx \cdot Qy = (Qx)^TQy=x^TQ^TQy=x^Ty$ 


## 投影中，正交矩阵Q取代A
* 如果A=Q，投影矩阵可简化为$P=Q(Q^TQ)^{-1}Q^T=QQ^T$，如果Q满秩，进一步化简为$P=I$。根据简化的投影矩阵，Q中单列$q_i$的投影矩阵$P_i=q_iq^T_i$。

## Gram Schmidt算法
可以通过单向量投影，证明gram schmidt方法计算出来基同样span整个空间

## QR分解




