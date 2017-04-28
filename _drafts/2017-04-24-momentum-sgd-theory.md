---
layout: post
title:  应用动量提高梯度递减收敛速度
categories: [ML,SGD,momnetum,optimization]
---

最近一段时间在看了一个梯度递减的优化方法，使用物理中的动量概念(Momentum)降低梯度递减的收敛轮数。回忆一下高中物理，[动量](https://zh.wikipedia.org/wiki/%E5%8A%A8%E9%87%8F)就是物体速度与质量的乘积。从物理角度解决优化问题（模拟退火应该也属于这类），觉得还是挺有意思的，所以仔细的读完了全文，并且证明了论文中略过的推导，同时还做了一些实验，验证此理论。本文主要记录那些略过的推导，同时分享实验过程。

物理的角度类比梯度下降有其合理性。使用物理场景类比，梯度下降可以想象为一个小石子，从较高的山坡往下滚动，当达到比较低的地方就停下来了，此时重力势能降到最低，动能为0。开销函数的初始值，可以认为是最开始的重力势能。

我们下面要讨论是用**阻力谐振子**的物理模型分析梯度下降，虽然没有上面小石子的例子直观，但是他们的运动方程非常类似。

## 阻力谐振子模型与梯度递减的关系

回忆梯度递减的优化方法，沿着梯度的反方向修改权重向量$w$

$$
  \Delta w_t = -\epsilon \nabla_w E(w_t) \qquad (1)
$$

其中 $\epsilon$为学习率，$w$为权重向量。如果考虑动量，算法修改如下

$$
  \Delta w_t = -\epsilon \nabla_w E(w_t) + p\Delta w_{t-1} \quad (2)
$$

唯一的区别是每轮修改考虑了上一轮梯度变化的情况$p\Delta w_{t-1}$。

论文从[阻力谐振子](https://zh.wikipedia.org/wiki/%E9%98%BB%E5%B0%BC)模型的角度进行类比，发现如果振子质量为0，此模型与梯度递减形式一致。

阻力谐振子描述的一个有重量，无体积的物体，在粘性介质中，受到弹性力的作用，在一维的情况下，其受力方程可以如下表示

$$
  m \ddot{x} = -c \dot{x} - kx
$$

即质量($m$)与加速度($\ddot{x}$)乘积等于物体当前的受力情况。而物体在粘性介质中，并且受到弹簧牵引。粘性介质给质心阻力，其大小正比于当前速度($\dot{x}$)，方向相反；弹簧弹力与位移($x$)正比，并且与位移反比。上面等式的$c$为粘性介质系数，$k$为弹簧弹性系数。

再回忆一下高中物理，[弹性势能](http://baike.baidu.com/item/%E5%BC%B9%E6%80%A7%E5%8A%BF%E8%83%BD)可以表示如下

$$
  E(x) = \int_{x}^0 -kx dx = \frac{1}{2}kx^2
$$


如果对位移x求导，可以得到弹力大小，

$$
  \nabla_x E(x) = kx
$$

所以，如果将上面$x$全部替换为高纬度位移向量$w$，并代入上面的受力方程,得到如下公式

$$

  m \frac{d^2 w}{d t^2} = - \mu\frac{dw}{dt} -\nabla_wE(w) \qquad (4)

$$

将速度和加速度离散化，速度的离散形式

$$
  \frac{dw}{dt} = \frac{w_{t+\Delta t} - w_t}{\Delta t}
$$  

加速度的离散形式

$$
  \frac{d^2w}{dt^2} = \frac{v_t - v_{t-1}}{\Delta t}
      = \frac{w_{t+\Delta t}+ w_{t-\Delta t} - 2w_t}{(\Delta t) ^2}

$$

将离散化的变量代入(4)

$$
m   \frac{w_{t+\Delta t}+ w_{t-\Delta t} - 2w_t}{(\Delta t) ^2} = - \mu \frac{w_{t+\Delta t} - w_t}{\Delta t} -\nabla_wE(w) \qquad (5)
$$

整理(5)，如下

$$
  w_{t+\Delta t} - w_t =
    - \frac{(\Delta t)^2}{m+\mu \Delta t} \nabla_w E(w)
    + \frac{m}{m+\mu\Delta t}(w_t - w_{t-\Delta t}) \qquad(6)
$$

将上面比较复杂的系数，用其他符号替代，

$$
  \epsilon = \frac{(\Delta t)^2}{m+\mu \Delta t} \quad(7) \\
  p = \frac{m}{m+\mu\Delta t} \quad (8)
$$

将(7),(8)代入(6)，得到方程(2)。说明，**阻力谐振子的模型等价于动量梯度递减**。如果考虑特殊情况，物体的质量$m=0$，那么此时**等价常规梯度递减**。


## 稳定性与收敛分析

如果从能量守恒的角度，可以发现损失函数与机械能等价。机械能表示如下

$$
  E_T = \frac{1}{2}m\frac{dw^T}{dt}\frac{dw}{dt} + E(w) \qquad (9)
$$

公式(9)中 $\frac{1}{2}m\frac{dw^T}{dt}\frac{dw}{dt}$ 为动能，E(w)为势能。

当振子达到局部最低点的时候，势能为0，动能为0。

在局部最优点$w_0$的地方对受力方程(4)[泰勒展开](https://zh.wikipedia.org/wiki/%E6%B3%B0%E5%8B%92%E7%BA%A7%E6%95%B0),线性估算得到如下方程，

$$
  m \frac{d^2 w}{d t^2} + \mu\frac{dw}{dt} \approx -H(w-w_0) \qquad (10)
$$

其中H为海森矩阵，对称且正定，每个元素为：

$$
  h_{i,j}=\frac{\partial^2{E(W)}}{\partial (w_i) \partial (w_j)} \rvert_{w_0} \qquad(11)
$$

为了方便计算，且不是去一般性，可以认为$w_0=0$,因为$w_0$仅为坐标，可以任意定义。看到正定矩阵，心中一阵狂喜。正定矩阵可以分解，且分解形式非常优美。将H分解如下，其中Q为标准正交矩阵，

$$
  H = QKQ^T,QQ^T = I \qquad (12)
$$

K为对称矩阵，且每个元素均大于0，
$$
  K = \begin{bmatrix} k_1\\ & k_2 \\ & & \ddots \\ & & &k_n \end{bmatrix} \qquad (k_i > 0) \qquad(13)
$$


可以对w进行线性转换，

$$
  w^{\prime} = Q^Tw \qquad (14)
$$


将(12)~(14)代入(10)

$$
  m \frac{d^2 w^{\prime}}{d t^2} + \mu\frac{dw}{dt} \approx -Kw^{\prime} \qquad (15)
$$

通过矩阵分解，(15)中的向量每个元素的关系分解开，可以单独考虑，

$$
  m \frac{d^2 w_i^{\prime}}{d t^2} + \mu\frac{dw_i}{dt} \approx -k_iw_i^{\prime} \qquad(16)
$$

这里的$k_i$就是弹簧弹性系数。公式(16)是线性微分方程，可以转成矩阵形式求解特征值求解，相关解法可以参考[线代随笔15-常系数线性矩阵微分方程](algebra)。

当不考虑质量时（$m=0$）,期解为

$$
  w^{\prime}_i(t) = ce^{\lambda_{i,0}t} \qquad(17)
$$

其中c为常量，

$$
  \lambda_{i,0} = -\frac{k_i}{\mu} \qquad (18)
$$

如果考虑质量($m \ne 0$), 解如下

$$
  w^{\prime}_i(t) = c_1e^{\lambda_{i,1}t} + c_2e^{\lambda_{i,2}t} \qquad(19)
$$

其中$c_1,c_2$为常量。


## 参考资料

* [An overview of gradient descent optimization algorithms](http://sebastianruder.com/optimizing-gradient-descent/)
* On the momentum term in gradient decent learning, 1999, Ning Qian
