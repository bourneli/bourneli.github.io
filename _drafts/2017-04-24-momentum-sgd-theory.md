---
layout: post
title:  应用动量提高梯度递减收敛速度
categories: [ML,SGD,momnetum,optimization]
---

最近一段时间在看了一个梯度递减的优化方法，使用物理中的动量(Momentum)减少梯度递减的收敛轮数。回忆一下高中物理，[动量](https://zh.wikipedia.org/wiki/%E5%8A%A8%E9%87%8F)就是物体速度与质量的乘积。从物理角度解决优化问题（虽然不是第一次看见了，模拟退火应该也属于这类），觉得还是挺有意思的，所以仔细的读完了全文，并且证明了论文中略过的推导，同时还做了一些实验，验证此理论。本文主要记录那些略过的推导，同时分享实验过程。


## 参考资料

* [An overview of gradient descent optimization algorithms](http://sebastianruder.com/optimizing-gradient-descent/)
* On the momentum term in gradient decent learning, 1999, Ning Qian
