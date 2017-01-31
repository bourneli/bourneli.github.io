---
layout: post
title:  论文笔记-(2014)Do we need hunders of Classier to Solve Real World Classication Problems?
categories: [ML,papers]
---

今天读了一篇2014年的分类器论文--Do we Need Hundreds of Classiers to Solve Real World Classication Problems?该论文是西班牙圣地亚哥波斯特拉大学和google的工程师联合发表。

该论文主要测试17类共**179**个分类器进行性能。使用的数据是UCI的公共数据源，共**121**个，这些数据以现在的观点而言都是小数据集，最多也就是10万左右，上百列已经算多。使用的评价指标是准确率Accuracy。读此论文时，还回顾了[Boost和bagging](http://stats.stackexchange.com/questions/18891/bagging-boosting-and-stacking-in-machine-learning)。这是两类ensemble模型，Boost通过迭代，学习错误降低bias；Bagging通过随机取样降低Variace。

最终的统计结论是**随机森林**的平均准确率最高，并且在许多指标中都排名靠前。但是也只是在10%左右的数据集第一。虽然随机森林准确率第一，但是具体问题具体看到，有些问题可能SVM，神经网络等的表现会更好。而且，这些测试数据集都是在小数据上，当数据量级上千万甚至更多时，可能产生不一样的结论。

结合工作经验，现实问题如果需要分类器，可以先尝试大家普遍使用的方法，如逻辑回归，随机森林，GBDT，SVM，朴素贝叶斯，神经网络等，给出初步试验报告。然后在结合实际情况，比如生成环境限制，应用场景，模型时间复杂度，项目进度等，选择其中一个相对较好的分类器作为落地方案，并且不断调试参数和选取特征，逐步提高模型性能。当落地方案ok，并且已上线应用，可以在尝试A/B test,逐步试验其它分类器。
