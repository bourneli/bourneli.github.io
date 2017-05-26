---
layout: post
title:  GBDT特征转换+LR总结
categories: [ML]
---

最近拜读了Facebook 2014年的论文[Practical Lessons from Predicting Clicks on Ads at
Facebook](https://pdfs.semanticscholar.org/daf9/ed5dc6c6bad5367d7fd8561527da30e9b8dd.pdf)。推荐领域广为流传的GBDT+LR的套路出自这篇论文。但除此之外，论文中还有很多其他的技巧值得学习，本文简单记录这些技巧，方便后面回顾。本文同时记录了GDBT+LR的实验对比数据，作为参考。


## GBDT编码， LR建模

用LR做点击率预估时，需要做大量的特征工程。将连续特征离散化，并对离散化的特征进行One-Hot编码，最后对特征进行二阶或者三阶的特征组合，目的是为了得到非线性的特征。特征工程存在几个难题：

* 连续变量切分点如何选取？
* 离散化为多少份合理？
* 选择哪些特征交叉？
* 多少阶交叉，二阶，三阶或更多？

一般都是按照经验，不断尝试一些组合，然后根据线下评估选适当参数。

但是，使用GBDT编码，一举解决了上面的问题。确定切分点不在是凭主观经验，而是根据信息增益，客观的选取切分点和份数。每棵决策树从根节点到叶节点的路径，会经过不同的特征，此路径就是特征组合，而且包含了二阶，三阶甚至更多。

为什么不直接用GDBT，而非要用GDBT+LR呢？因为GDBT在线预测比较困难，而且训练时间复杂度高于LR。所以实际中，可以离线训练GDBT，然后将该模型作为在线ETL的一部分。

虽然Facebook论文提到GBDT+LR的效果是好于纯GBDT，甚至LR的性能也比GBDT要好。其实，我存怀疑态度的，所以我用R在本地做了一个实验，数据源是mlbench的5个数据包diabetes，satellite，sonar，vehicle和vowel，综合实验数据如下

<div align='center'>
<img src='/img/gbdt_lr_experment_stat.png' />
</div>

可以看到，除了召回率指标，其他所有指标均是gbdt > gbdt + lr > lr，这一点符合我之前的设想。当然，我的数据集也比较有限，不能以偏概全。但是从实验数据来看，这些算法在各项指标没有质的区别，所以实际工作中，找到重要的特征才是头等大事；算法方面，选择能够快速上线，够用就行，后面可以迭代优化。上述实验直接使用R xgboost的函数完成gbdt编码。之前没有发现该函数，基于gbm包的gbdt数，写了一个编码函数，重复造轮子，:-(。整个实验工程参考[这里](https://github.com/bourneli/data-mining-papers/blob/master/GBDT/gpdt-lr-exp/model_comparation.R)。




## 其他命题纪要

下面简单记录论文其他方面的主题，方便后面回顾

* 梯度下降学习率更新策略，本文介绍了常见的几种，并给出的试验数据
* 在线学习中，LR对BOPR。BOPR效果稍微好于LR，但是LR更为简单，所以最后还是选择了LR
* GDBT迭代轮数，大部分优化只需要500轮迭代GBDT模型可以完成。
* GDBT的数深度也不需要太深，2,3层一般满足要求。
* 特征也不是越多越好，重要性前Top 400特征就可以表现很好
* 历史特征比用户当前上下文特征更为重要，而且对预测的时间更为稳定。但是用户上下文数据可以有效的解决冷启动问题。
* 无需使用全部的样本，使用10%的训练数据的效果与100%没有太大差别
* 如果对负样本重采样，模型计算的概率，需要[重新修正](http://bourneli.github.io/machine-learning/prml/2016/12/19/compensating-for-class-priors.html)。修正公式为$q = \frac{p}{p+(1-p)/w}$，其中q是修正后的数据，p时模型预测的数据，w是负样本重采样比例。
