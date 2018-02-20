---
layout: post
title:  强化学习实践02-特征工程仍然很重要
categories: [Reinforcement-Learning]
---

特征工程在传统机器学习中的作用毋庸置疑，在强化学习中仍然非常重要，本文通过一个简单的实验，观察特征工程的效果。

使用的环境是gym库中的小车爬坡，编号**MountainCar-v0**。该环境的状态有两个，小车位置和小车速度。动作有三个，左加速，右加速和不加速。所以属于连续状态离散动作的强化学习问题。使用[线性拟合的q-learning](https://github.com/bourneli/rl-practice/blob/master/py-proj/lib/q_learning.py)处理这个问题，方法原理可以参考下面几篇文章

* [强化学习笔记05-无模型(Model Free)策略求解](http://bourneli.github.io/reinforcement-learning/2017/10/17/rl-learning-05-model-free-mc-td-control.html)
* [强化学习笔记06-函数逼近](http://bourneli.github.io/reinforcement-learning/2017/11/11/rl-learning-06-function-approximation.html)

[特征处理](https://github.com/bourneli/rl-practice/blob/master/py-proj/lib/features.py)使用如下方法，

* 不处理，作为对照组
* 标准化，通过均值和方差对每个特征标准化
* RBF，粗糙编码的连续版本，计算复杂度高，效果好，适合低纬度编码，编码之前先标准化。
* 二阶特征交叉，先标准化特征

实验代码参考[这里](https://github.com/bourneli/rl-practice/blob/master/py-proj/experiment/q_learning_car_mountain.py)，实验结果如下（原始数据比较分散，为了更好的观察趋势，使用了均值拟合），

![](/img/rl/learning-rate-car-mountain-longer.png)

可以发现如果不做任何特征处理，效果非常差，智能体根本无法学习出有价值的行为；使用了标准化或者二阶交叉特征特征处理后，智能体行为有明显改进，可以较早的学习到一个较高的reward，但是无法持续太久；使用RBF做特征工程的效果最好，智能体明显学习到了爬坡的诀窍。但是，无论使用哪个特征处理，到最后的效果都有变差的趋势。

个人理解，本质上值行为函数的估计通过梯度递减求解，学习率没有作适当的设置，导致达到最优点附近后，又震荡到了其他地方，类似下面红色曲线，

![](https://qph.ec.quoracdn.net/main-qimg-f3972c89625c0451f0ea92a4c0ea0728.webp)

实验中使用的学习率是0.01，当调整到0.001后，得到的学习曲线如下，果然是学习率影响了整个学习曲线的形状。

![](/img/rl/learning-rate-car-mountain-longer-smaller-learning-rate.png)


这次分享就到这里，还是那句老话“**纸上得来终觉浅，绝知此事要躬行**”，后面仍需不断实践，才能加深强化学习的理解。
