---
layout: post
title:  AUC的数学解释
categories: [ML]
---



这两天中秋节，重温了[ROC][ROC wiki]的相关性质。AUC（全称AUROC）是ROC曲线下面的面积，其意义为

>The AUC of a classifier is equivalent to the probability that the classifier will rank a randomly chosen positive instance higher than a randomly chosen negative instance.

使用数学语言表示如下，
$$
AUC = P(s(x_1) > s(x_0)) \qquad (1)
$$


公式(1)的符合意义如下，

* $P()$表示概率；

* s()表示根据数据训练好的二元软分类器，比如逻辑回归，GBDT等；

* $x_1$表示从正样本中随机抽取的实例；

* $x_2$表示从负样本中随机抽取的实例。



但是公式(1)为什么成立呢？带着这个问题笔者google一番，终于找到了[其数学推导][AUC Interpretation]。

在正式推导之前，首先复习ROC的定义。ROC是一条曲线，该曲线上的任意一点由模型的TPR(True Positive Rate)和FPR(False Positive Rate)构成。TPR代表模型可以找到的正样本概率，俗称召回率，代表收益。FRP表示找到的错误的正样本的概率，表示模型的损失。所以，ROC上的任意一点表示模型在获得收益的同事，需要付出的代价。英语有句谚语--**No Pains，No Gains**，可以形象的形容ROC上的点。ROC曲线刻画模型在不同阈值$\tau$下，获取收益和损失的一种综合能力。但是ROC是一条二维曲线，不方便两个模型比较，所以一般都转为ROC下的面积，即AU(RO)C，来比较两个模型的能力。一旦模型训练完成，正负样本的得分分布就确定了，如下图，

![](\img\roc_demo.png)



正式推导之前，首先定义一些符号， 

* $f_1(s)$表示正样本的概率密度函数PDF，类似上面红色曲线；$F_1(s)=\int_{-\infty}^s f_1(x) dx$是其累计分布函数CDF。
* $f_0(s)$表示负样本的概率密度函数PDF，类似上面绿色曲线；$F_2(s)=\int_{-\infty}^s f_2(x) dx$是其累计分布函数CDF。
* $\tau$ 为任意选定的阈值
* 复用公式(1)中定义的符号

所以，根据上面的定义，可以容易计算出ROC的两个轴的表示，
$$
TPR = 1 - F_1(\tau) = y(\tau) \qquad (2) \\
FPR = 1 - F_0(\tau) = x(\tau) \qquad (3)
$$
直接使用微积分定义计算AUC的面积，
$$
AUC = \int_0^1 y(x) dx \qquad (4)
$$
归根到底，公式(4)是阈值$\tau$的积分,而阈值的范围是$(-\infty,+\infty)$，所以
$$
AUC = \int_0^1 y(x(\tau)) dx(\tau) = \int_{-\infty}^{+\infty} y(\tau)x^\prime(\tau) d \tau \qquad (5)
$$
将(2),(3)带入(5)，
$$
AUC = \int_{-\infty}^{+\infty} (1-F_1(\tau))(-f_0(\tau)) d\tau = \int_{+\infty}^{-\infty} (1-F_1(\tau)) f_0(\tau) d\tau \qquad (6)
$$
可以较为容易解读(6)。

其中$P(s(x_1) \ge \tau) = 1-F_1(\tau)$，即随机正样本模型打分$s(x_1) \ge \tau$的概率。$P(s(x_0) = \tau) = f_0(\tau) d\tau$,即随机负样本模型打分$s(x_0) = \tau$的概率，通过在整个阈值定义域积分，得到所有事件的概率。由于两个样本是随机从正样本和负样本选取，所以两个事件独立。所以AUC最终可以解读为**分别随机从正样本选取$x_1$，负样本选取$x_0$，AUC的值等于事件s(x_1)>=s(x_0)的概率**。




[AUC Interpretation]: https://stats.stackexchange.com/questions/180638/how-to-derive-the-probabilistic-interpretation-of-the-auc
[An introduction to ROC analysis]: http://people.inf.elte.hu/kiss/12dwhdm/roc.pdf
[ROC wiki]: https://zh.wikipedia.org/wiki/ROC%E6%9B%B2%E7%BA%BF