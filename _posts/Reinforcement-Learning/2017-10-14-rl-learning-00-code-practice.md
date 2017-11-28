---
layout: post
title:  强化学习笔记-Talk is cheap, show me the code
categories: [Reinforcement-Learning]
---

这段时间自学强化学习相关的内容，发现有些概念，只是看教材和视频，仍然比较晦涩，所以希望在网上找些强化学习的库练手。感谢伟大的google，通过google发现了一个Google Brain Team的大牛[Denny Britz](https://twitter.com/dennybritz)开发的[Reinforcement Learning](https://github.com/dennybritz/reinforcement-learning)练习。

该练习基本与[Sutton](https://en.wikipedia.org/wiki/Richard_S._Sutton)的教材配套，每一个单元有相关知识概要和练习。如果系统的看了教材和David的课程，知识概要可以稍微略去。练习这块是干货，实现了Sutton教材中的一些关键例子，并且逐步引导我们实现相关算法，如Police Improvement, Value Evluation,MC Prediction等。这些算法虽然不难，但是如果不动手实践，部分细节无法理解透彻。每个算法练习均有solution，但是最好不要先看solution，那样就没有意思了。建议首先独立思考，反馈琢磨教材，至少实现一个版本，再看solution，这样也有不错的效果。

整个练习环境使用[Jupyter Notebook](https://jupyter.org/)和强化学习的框架为[Open AI/gym](https://github.com/openai/gym)。读者可以根据链接，按照官方指引安装这些python组件。
