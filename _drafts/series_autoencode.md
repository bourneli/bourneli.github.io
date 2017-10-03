---
layout: post
title:  基于LSTM的序列自编码
categories: [autoencode,rnn,lstm,tensorflow]
---

## 背景

序列数据在工作中经常出现，比如

* 登陆某款APP的频率
* MOBA游戏中使用英雄的序列
* 日活跃用户的序列曲线
* 一段文本

最近流行的深度学习，可以很方便的处理这类数据，要么直接用序列数据作为输入，输出连续值或分类；要么将序列数据embedding成固定长度的向量，作为其他模型的输入。对于后者，我特别有兴趣，因为可以给现有的模型提供更丰富的特征，可以得到更好模型效果。本文将介绍基于LSTM的seq2seq序列自编码，使用tensorflow的python接口实现，主要参考了[(2014)Sequence to Sequence Learning with Neural Networks](https://arxiv.org/abs/1409.3215)。

## TensorFlow的工作流程简介

TensorFlow最近在深度学习社区异常火爆，因为有Google为其站台。不过说实话，其原生API不是特别友好，有一定门槛，无法快速开发原型，相比于Keras的易用性，差远了。好在TensorFlow将keras包括在内，所以可以直接在TensorFlow中使用Keras。不过，由于序列自编码的特殊性，我还是使用TensorFlow的原生API构建了整个架构，因为使用keras很难处理一些细节。

TensorFlow一开始需要构建计算图，构建完后，才能通过Batch SGD一点一点的逼近最优解。构建计算图的过程，可以类比为Spark RDD中的transform操作，执行Batch SGD可以类比为action操作。在构建计算图时，数据没有真实执行，只有在构建计算图后，通过特殊的优化tensor，使得Loss逐步收敛。本文主要介绍AutoEncode的计算图，求解过程就是顺水推舟了。

## 网络架构与实现

ppt绘制架构图

一步一步分解代码





## 实验效果

实验数据

实验方法介绍

实验结果





## 结论

* 可以有效编码不同周期的序列
* 对于曲线类似的序列，有一定的区分能力

还尝试了使用标签进行自编码，实际用途不是特别大，因为标签一般很难得到，得到了还需要编码吗？



