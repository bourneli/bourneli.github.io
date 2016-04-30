---
layout: post
title:  "线代随笔05-向量投影"
categories: [linear-algebra]
---

向量投影是线性代数中很重要的应用，它用于找到向量到目标投影空间中的投影向量。以三维空间为例，目标投影空间可以是线，也可以是面。线性回归是常用的数据统计分析手段，用于分析自变量与因变量的关系。线性回归求解过程与统计基本上没有关系，可以用线性代数的向量投影计算系数。当变量的系数计算完后，系数的[显著性检验](http://stats.stackexchange.com/q/148803/31830)与统计相关。

## 投影矩阵推导
设向量$\vec{b}$投影到$C(A)$，这里很有必要假设$A$的列向量线性独立，因为如果$A$的列向量线性相关，投影的结果实质上是不受影响的，但是线性依赖的列向量会很大程度的影响计算，所以假设$A$的列向量线性依赖，后面会看到为什么需要这样假设。

设$\vec{p}=P\vec{b}=A\hat{x}$是向量$\vec{b}$在$C(A)$中的投影,$P$是投影向量,$\hat{x}$是$\vec{b}$投影到$C(A)$中的A列向量的线性组合。那么$\vec{e}=\vec{b}-A\hat{x}$属于$C(A)$的正交补里面,也就是属于$N(A^T)$，所以

$$
\begin{align}
	A^T\vec{e}=\vec{0} & \Rightarrow A^T(\vec{b}-A\hat{x}) = \vec{0} \\
					   & \Rightarrow A^T\vec{b}-A^TA\hat{x} = \vec{0} \\
					   & \Rightarrow A^T\vec{b}=A^TA\hat{x} \\
					   & \Rightarrow (A^TA)^{-1}A^T\vec{b}=\hat{x}  \\
					   & \Rightarrow \vec{p}=A(A^TA)^{-1}A^T\vec{b}=A\hat{x} 
	
\end{align}
$$

令投影向量$P=A(A^TA)^{-1}A^T$，此公式中包含$(A^TA)^{-1}$，由于之前已假设A的列向量线性独立，所以[$A^TA$的逆必存在](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)。

## 投影矩阵性质

* 如果A是方正，且满秩，那么$P=I$。推导：$P=A(A^TA)^{-1}A^T=(AA^{-1})((A^T)^{-1}A^T)=I$。如果C(A)可以支持整个空间，那么任何向量自身就是C(A)的投影。
* $P^2=P$。推导：$P^2=A(A^TA)^{-1}A^TA(A^TA)^{-1}A^T=A(A^TA)^{-1}(A^TA)(A^TA)^{-1}A^T=A(A^TA)^{-1}A^T=P$。投影一次后，再次投影不会有加成效果。
* $P^T=P$,对称。推导：直接利用转置与逆的互换性质。




