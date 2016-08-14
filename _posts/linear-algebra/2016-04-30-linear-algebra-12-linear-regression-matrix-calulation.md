---
layout: post
title:  "线代随笔12-线性回归的矩阵推导"
categories: [linear-algebra,calculus]
---

线性回归的计算方法很多，比如最小二乘，梯度下降，今天分享一种矩阵求导计算方法，并且将其与投影联系，可以更加感性的了解线性回归的计算原理。

## 计算推导

令矩阵$A$是系数矩阵,且A的列线性独立，一般$A$的第一列或最后一列是1，用于表示截距，但是这对我们的推导没有任何影响。$y$是我们的目标结果。现在需要计算一个权重向量$w$，使得差的平方错误最小，记作:

$$
\begin{equation*}
\begin{aligned}
& \underset{x}{\text{min}} && E(w) \\
& \text{s.t.} && E(w)=|Aw-y|^2
\end{aligned}
\end{equation*}
$$

对$E(x)$做相关变化

$$
\begin{align}
	E(w) &= (Aw-y)^T(Aw-y) \\ 
		 &= (w^TA^T-y^T)(Aw-y) \\ 
		 &= w^TA^TAw - w^TA^Ty - y^TAw + y^Ty \\
		 &= w^TA^TAw - y^TAw - y^TAw + y^Ty \\
		 &= w^TA^TAw - 2y^TAw + y^Ty \\
\end{align}	
$$

$E(w)$是个凸函数，最小值在所有偏导为0的地方，

$$
	\frac{\partial E(w)}{\partial w} = 2A^TAw - 2A^Ty = 0
$$

上面的计算使用了向量求导的相关计算，参考[线代随笔11-线性回归相关的向量求导](/linear-algebra/calculus/2016/04/28/linear-algebra-11-derivate-of-linear-regression.html)。由于A的列线性独立，所以[$A^TA$可逆](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)，化简上述公式，

$$
\begin{align}
	& 2A^TAw - 2A^Ty = 0 \\
	& \Rightarrow A^TAw = A^Ty \\ 
	& \Rightarrow w = (A^TA)^{-1}A^Ty \\
\end{align}
$$

推导完毕！

## 线性回归与投影的关系

上述结果与[投影系数](/linear-algebra/2016/03/05/linear-algebra-05-projection-and-linear-regression.html)的计算公式一致。这不是巧合，线性回归的本质是找到一个线性组合$w$，使得因变量$y$被由自变量$A$的列的线性组合表示。但实际情况，绝大多数是无法找到这种完美的解。那么采取$C(A)$中与$b$最近的向量作为其近似解。这个最近的向量，通过上面的推导，就是投影系数。可以想象一下三维空间中，直线投影到平面，通过三角关系，可以发现最近的向量是垂直的投影向量。 

## 参考
* [线代随笔04-$A^TA$可逆条件](/linear-algebra/2016/03/03/linear-algebra-04-ATA-inverse.html)
* [线代随笔05-向量投影](/linear-algebra/2016/03/05/linear-algebra-05-projection-and-linear-regression.html) 
* [线代随笔11-线性回归相关的向量求导](/linear-algebra/calculus/2016/04/28/linear-algebra-11-derivate-of-linear-regression.html)
