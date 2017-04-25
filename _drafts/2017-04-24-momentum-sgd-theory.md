---
layout: post
title:  应用动量提高梯度递减收敛速度
categories: [ML,SGD,momnetum,optimization]
---

最近一段时间在看了一个梯度递减的优化方法，使用物理中的动量概念(Momentum)降低梯度递减的收敛轮数。回忆一下高中物理，[动量](https://zh.wikipedia.org/wiki/%E5%8A%A8%E9%87%8F)就是物体速度与质量的乘积。从物理角度解决优化问题（模拟退火应该也属于这类），觉得还是挺有意思的，所以仔细的读完了全文，并且证明了论文中略过的推导，同时还做了一些实验，验证此理论。本文主要记录那些略过的推导，同时分享实验过程。


回忆梯度递减的优化方法，沿着梯度的反方向修改权重向量$w$

$$
  \Delta w_t = -\epsilon \nabla_w E(w_t)
$$

如果考虑动量，算法如下

$$
  \Delta w_t = -\epsilon \nabla_w E(w_t) + p\Delta w_{t-1}
$$

唯一的区别是每轮修改梯度，均考虑了上一轮梯度变化的情况$p\Delta w_{t-1}$。

论文从[阻力谐振子](https://zh.wikipedia.org/wiki/%E9%98%BB%E5%B0%BC)模型的角度进行类比，发现如果认为质量为0，此模型与梯度递减形式一致。

阻力谐振子描述的一个有重量，无体积的物体，在粘性介质中，受到弹性力的作用，在一维的情况下，其受力方程可以如下表示

$$
  m \ddot{x} = -c \dot{x} - kx \qquad (1)
$$

即质量与加速度乘积等于物体当前的受力情况。而物体在粘性介质中，并且受到弹簧牵引。粘性介质给质心阻力，其大小正比于当前速度，方向相反；弹簧弹力与位移正比，并且与位移反比。上面等式的c为粘性介质系数，k为弹簧弹性系数。

再回忆一下高中物理，[弹性势能](http://baike.baidu.com/item/%E5%BC%B9%E6%80%A7%E5%8A%BF%E8%83%BD)可以表示如下

$$
  E(x) = \frac{1}{2}kx^2
$$


如果对位移x求导，可以得到弹力大小，

$$
  \nabla_x E(x) = kx
$$

所以，如果将上面$x$全部替换为高纬度位移向量$w$，并代入受力方程(1),得到如下公式

$$

  m \frac{d^2 w}{d t^2} = - \mu\frac{dw}{dt} -\nabla_wE(w) \qquad (2)

$$


离散化2，得到相似形式。



## 参考资料

* [An overview of gradient descent optimization algorithms](http://sebastianruder.com/optimizing-gradient-descent/)
* On the momentum term in gradient decent learning, 1999, Ning Qian
