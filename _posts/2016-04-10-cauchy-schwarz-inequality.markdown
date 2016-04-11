---
layout: post
title:  柯西-施瓦茨不等式推广到向量空间$\Bbb R^m$
categories: [inequality]
---

实数域$\Bbb R^n$下[柯西不等式](https://en.wikipedia.org/wiki/Cauchy%E2%80%93Schwarz_inequality)如下

$$
|\langle x,y\rangle|^2 \le \langle x,x\rangle \cdot \langle y,y\rangle,\qquad \forall x,y \in R^n
$$

将其变换，可以得到如下等价形式

$$
	\left|\sum_{i=1}^nx_i\cdot y_i\right|^2 \le \sum_{i=1}^n|x_i|^2\sum_{i=1}^n|y_i|^2,\qquad \forall  x_i,y_i \in \Bbb R
$$

令$y_i = 1$，上述不等式化简为

$$
	\left|\sum_{i=1}^nx_i\right|^2 \le n\sum_{i=1}^n|x_i|^2,\qquad \forall  x_i \in \Bbb R
$$

**现在，如果将上面的定理推广到向量空间$\Bbb R^m$下，即$ x_i \in \Bbb R^m$，是否仍然成立呢?**

$$
	\left|\sum_{i=1}^nx_i\right|^2 \le n\sum_{i=1}^n|x_i|^2,\qquad \forall  x_i \in \Bbb R^m
$$

据说，[柯西-施瓦茨不等式在任何内积空间都成立](http://math.stackexchange.com/questions/1731819/proof-of-analogue-of-the-cauchy-schwarz-inequality-to-vector)（这一点有待后续研究），可导出上面的等式成立。由于内积空间的论断本人未经证实。所以下面，使用数学归纳法证明上面的不等式的正确性！

证明：

当n=1时，化简为$x_1^2 = x_1^2$,成立

当n=2时，化简为($x_1-x_2)^2 \ge 0$,成立

假设当n=k时，下面不等式成立

$$
	\left|\sum_{i=1}^kx_i\right|^2 \le k\sum_{i=1}^k|x_i|^2,\qquad \forall  x_i \in \Bbb R^m
$$

当n=k+1时，设

$$
	\Delta_{k+1} = \left|\sum_{i=1}^{k+1}x_i\right|^2 - (k+1)\sum_{i=1}^{k+1}|x_i|^2
$$

即证明在$\Delta_k \ge 0$时，$\Delta_{k+1} \ge 0$成立。将上面公式展开，化简，有

$$
	\begin{align}
	\Delta &= kx_{k+1}^2 + \sum_{i=1}^kx_i^2 - 2x_{k+1}\sum_{i=1}^kx_i 
			+ \bigg(\left|\sum_{i=1}^kx_i\right|^2 - k\sum_{i=1}^k|x_i|^2\bigg) \\
	       &= \sum_{i=1}^k(x_{k+1}-x_i)^2 + \Delta_k \ge 0
	\end{align}
$$

证毕。

其实，上面的等式可以进一步化简

$$
	\Delta_n = n \sum_{i = 1}^{n} |x_i|^2  - \bigg | \sum_{i = 1}^{n} x_i \bigg|^2 
			 = {1 \over 2} \sum_{i = 1}^{n} \sum_{j = 1}^{n} (x_i - x_j)^2 \ge 0
$$

有兴趣的读者可以使用数学归纳法证明。

## 参考
* [Cauchy-Schwars inequality Wiki](https://en.wikipedia.org/wiki/Cauchy%E2%80%93Schwarz_inequality)
* [What is the difference between square of sum and sum of square?](http://math.stackexchange.com/a/439238/261790)
* [Proof of analogue of the Cauchy-Schwarz inequality to vector](http://math.stackexchange.com/questions/1731819/proof-of-analogue-of-the-cauchy-schwarz-inequality-to-vector)

<!-- UY BEGIN -->
<div id="uyan_frame"></div>
<script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=2094661"></script>
<!-- UY END -->
