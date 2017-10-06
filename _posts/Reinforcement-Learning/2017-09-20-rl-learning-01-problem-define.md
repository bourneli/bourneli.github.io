---
layout: post
title:  强化学习笔记01-问题定义
categories: [Reinforcement-Learning]
---


## 前言
由于工作需要，近期学习一些强化学习的内容。主要的参考资料是[David Silver](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Home.html)的讲课，

* [授课视频](https://www.youtube.com/watch?v=2pWv7GOvuf0)
* [同步课件](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching.html)
* 参考教材：An Introduction to Reinforcement Learning, Sutton and Barto, 1998


边写笔记，边学习的效率比较高，所以后面会开一个系列博客，发表我的学习笔记，由于还是这个方面的菜鸡，所以肯定有些地方领悟得不够透彻，还请大家多多包涵。



## 学习大纲

教材分为三个部分

* The Problem, Chapter 1~3
* Elementary Solution Methods, Chapter 4~6
* Unified View, Chapter 7~11



课本推荐的阅读策略

* Chapter 1 简短看下
* Chapter 2 看到2.2
* Chapter 3 略过3.4,3.5和3.9
* Chapter 4,5,6 按顺序阅读， Chapter 6最重要
* Chapter 8  如果关注机器学习和神经网络，看这章
* Chapter 9 如果关注AI或规划，看这章
* Chapter 10 最好看
* Chapter 11 例子，可以看看




**推荐阅读方法**

1. 由于教材，课件和授课视频同步，可以按章节阅读
2. 每个章节
   1. 粗略读教材对应章节，大概了解相关概念
   2. 阅读课件，记住不清楚的概念
   3. 详细观看授课视频，不清楚的地方可以反复观看，由于没有字幕，可能有点门槛，不过多看看，基本能看懂，附带对英语听力有一定提升。
   4. 第二次详细阅读课件+教材
   5. 整理笔记，梳理整个章节的内容，还有不明白的地方google检索相关概念。



## 第一讲笔记


非贪心算法，牺牲当前利益，获取长远利益

![core works](/img/rl_core_flow.png)

核心工作原理示意图。在每一步t，

Agent



  * 执行$A_t$
  * 接收$O_t$
  * 获取奖励$R_t$

环境



  * 接收$A_t$
  * 输出观察$O_{t+1}$
  * 输出奖励$R_{t+1}$



State是用于接下来该做什么的信息。比较难理解，下面的例子比较直观。


![state example](/img/state_example.png)

* Reward 电击或奶酪
* State 前面的序列
* Action 是否按把手


通过观察前面的历史序列，可以有两个State

1. 通过观察最后是三个序列
2. 通过计数亮灯次数

这两个state，可以得到不同的reward。


* **Policy** State到action的映射
* **Value Function** 未来reward的预估
* **Model** 预估环境下一步做什么，比如下一步状态或奖励


