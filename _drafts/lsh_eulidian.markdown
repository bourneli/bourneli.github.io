---
layout: post
title:  欧式空间上LSH的工作原理
categories: [probability]
---

$$
	p(u) = Pr(h(p) = h(q)) = \int_{0}^{w}\frac{1}{u}f(\frac{t}{u})(1-\frac{t}{w})dt
$$

$w$是投影向量每个单位的宽度，$u$是向量$p$与$q$在原始空间距离。概率密度函数f在欧式空间中，使用标准正太分布概率密度函数。展开成只与u，w有关的概率函数

$$
\begin{align}
	p(u) = f(u,w) &= \int_{0}^{w}\frac{1}{u}f(\frac{t}{u})dt - 		\int_{0}^{w}\frac{1}{u}f(\frac{t}{u})\frac{t}{w}dt \\
	     &= \int_{0}^{w}f(\frac{t}{u})d\frac{t}{u} - \int_{0}^{w}\frac{1}{u\sqrt{2\pi}}e^{-\frac{t^2}{2u^2}}\frac{t}{w}dt \\
		 &= \int_{0}^{\frac{w}{u}}f(x)dx - \frac{-u}{\sqrt{2\pi}w}\int_{0}^{w}e^{-\frac{t^2}{2u^2}}d(-\frac{t^2}{2u^2}) \\
		 &= \frac{1}{2} - F(-\frac{w}{u}) + \frac{u}{\sqrt{2\pi}w}e^{-\frac{t^2}{2u^2}}|^w_0 \\
		 &= \frac{1}{2} - F(-\frac{w}{u}) + \frac{u}{\sqrt{2\pi}w}(e^{-\frac{w^2}{2u^2}}-1) \\
		 
\end{align}
$$

肯定有问题，如果u特别大，概率会大于1。全部概率，需要乘以2，对称。
 