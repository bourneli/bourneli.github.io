---
layout: post
title: 【译】使用R和Caret包处理类别不平衡分类问题。
categories: [R,Translation]
---



*此文翻译文章[Handling Class Imbalance with R and Caret – An Introduction](https://www.r-bloggers.com/handling-class-imbalance-with-r-and-caret-an-introduction/)。该文章使用R的Caret包，非常简洁的处理非平衡分类问题。这类题在笔者的工作中十分常见，所以记录下来，方便后面回顾。下面是翻译全文。*



现实世界的分类任务中，处理高度不平衡的分类问题非常具有挑战性。本文由两部分，主要介绍一些R和caret相关技巧。这些技巧可以在高度不平衡的分类场景下提高模型预测性能。本文是第一部分，以通用视角介绍如何在实战中使用这些技巧。第二部分着重介绍这些技巧相关的“坑”，这些坑十分容易在实战中遇见，需要谨记。



## 分类评估指标

当训练完一个分类器后，你需要确定这个分类器是否有效。现存有很多指标用来评估分类器性能，可以大致将它们分类为两类：

1. 阈值相关：准确率(Accuracy)，精度(Precision),召回(Recall)和F1分(F1 score)全部属于此类。此类方法需要使用特定阈值，计算混淆矩阵，然后根据此矩阵计算上面所有的指标。这类指标在非平衡的问题上无法有效的评估模型性能，因为统计软件一般使用默认0.5作为阈值，这种不适当的设定会导致大多数样本被归纳到主要的类别中。
2. 阈值无关：ROC曲线下的面积(AUC)属于此类。它将真阳性看做是假阳性的函数，它们会随着切分阈值变化而变化。从概率的角度来看，此指标描述了**随机从正样本和负样本各取一个样本，然后使用模型对其打分，正样本得分大于负样本得分**的概率。(译者注:详细数学推导，可以参考笔者博文[AUC的数学解释](http://bourneli.github.io/ml/2018/09/24/roc-review.html))



## 提高非平衡数据下模型性能的方法

下面会介绍一些常用技巧处理非平衡分类问题的技巧，但是这个列表并不会详细描述所有细节。简短起见，本文仅提供简短的概要。如果希望了解这些技术背后的细节，作者强烈建议阅读[这篇博文](https://www.svds.com/learning-imbalanced-classes/)。

* 类权重:将更多的权重放到更少的类别的样本上，这样当模型在少量类别上犯错后，会得到更多惩罚
* 向下采样：在主要类别的样本中，随机移除一些样本。
* 向上采样：在较少类别的样本中，随机重复一些样本。
* SMOTE：向下采样，同时在较少的样本中，使用插值的方法随机生成新的样本。

需要着重指出的是，这类权重和抽样技巧对阈值依赖的指标有非常显著的影响，因为他们人工的将阈值朝着最优的地方移动。阈值无关的指标仍然会被这类方法改进，但是效果并没有声称的那样显著。



## 模拟设置

使用caret包中的**twoClassSim**函数模拟类别不平衡。我们将训练集和预测集分开模拟，每个集和5000个样本。除此之外，我们添加了20个有意义的变量，以及10个噪音变量。参数intercept控制整体不平衡的程度，最终数据集的正负样本比例为50:1。

{% highlight R %}

library(dplyr) # for data manipulation
library(caret) # for model-building
library(DMwR) # for smote implementation
library(purrr) # for functional programming (map)
library(pROC) # for AUC calculations

set.seed(2969)

imbal_train <- twoClassSim(5000,
​                           intercept = -25,
​                           linearVars = 20,
​                           noiseVars = 10)

imbal_test  <- twoClassSim(5000,
​                           intercept = -25,
​                           linearVars = 20,
​                           noiseVars = 10)

prop.table(table(imbal_train$Class))

## 
## Class1 Class2 
## 0.9796 0.0204

{% endhighlight %}















