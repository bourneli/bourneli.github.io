---
layout: post
title:  "线代随笔02-$A\\vec{x} = \\vec{b}$的完整解"
date:   2016-02-23 17:36:11 +0800
categories: [linear-algebra]
---

## 完整解形式

$$ 
\vec{x} = \vec{x_p} + \sum_{i=1}^{n-r} c_i\vec{x_{n(i)}} 
$$

* 其中$\vec{x_p}$是任意特殊解，即$A\vec{x_p}=\vec{b}$;
* $r=rank(A)$;
* n是A的列数;
* $\vec{x_{n(i)}} \in N(A)$,即$A\vec{x_{n(i)}}=\vec{0}$。


## 证明：上面等式是Ax=b的解

$$
\begin{align}
	A(\vec{x_p} + \sum_{i=1}^{n-r} c_i\vec{x_{n(i)}}) 
	& =  A \vec{x_p}+ A \sum_{i=1}^{n-r} c_i \vec{x_{n(i)}} \\ 
	& =  A\vec{x_p}+\sum_{i=1}^{n-r} c_i A \vec{x_{n(i)}} \\
	& = \vec{b} + \sum_{i=1}^{n-r} c_i  \vec{0} \\
	&= \vec{b} \\
\end{align}
$$

证明完成。


## 证明：任何Ax=b的解都可以写成上面的形式

即证明

$$
	\forall  A\vec{x'}=\vec{b}, \exists  \vec{x'} = \vec{x_p} + \sum_{i=1}^{n-r} c_i\vec{x_{n(i)}}
$$


等价于

$$
	\forall \Delta{\vec{x}} = \vec{x'} - \vec{x_p}, \exists \Delta{\vec{x}} = \sum_{i=1}^{n-r} c_i\vec{x_{n(i)}}
$$


因为

$$
A\Delta{\vec{x}} = A\vec{x'} - A\vec{x_p} = \vec{b} - \vec{b} = \vec{0}
$$

所以$\Delta{\vec{x}} \in N(A)$, 所以上面等式成立。

证明完成。

