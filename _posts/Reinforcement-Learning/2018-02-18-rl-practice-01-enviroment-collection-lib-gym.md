---
layout: post
title:  强化学习实践01-环境库gym简介
categories: [Reinforcement-Learning]
---


前面经过[David Silver的强化学习课程](https://www.youtube.com/watch?v=2pWv7GOvuf0)的布道，对强化学习有了系统的认知。但是实践是检验真理的唯一标准，所以后面开启**强化学习实践**系列，分享相关内容。

本文主要介绍Open AI的[gym库](https://gym.openai.com/docs/)（[源代码](https://github.com/openai/gym)），该库是**环境**集合。环境虽然不是强化学习算法的核心，但是它非常重要。如果没有环境，任何强化学习的效果无法得知，它就像一个标尺，客观的评价不同强化学习算法的强弱。

个人理解，环境就是一些规则的集合。比如探测器月球登陆的环境，规则是物理定律，如重力加速度，空气阻力等。Atari游戏环境，规则是游戏玩法。月球登陆或是Atari游戏的规则比较容易的提取并封装在环境对象中，一个是物理定理，一个是人类设计的游戏玩法。但是，强化学习实践中有个很大的难点就是实现离线的环境，使其尽可能的与在线环境相似，这样才能降低强化学习使用成本，比如模拟在线用户行为的环境。不过这是另一个话题，希望后面有机会与大家分享。

建议使用源代码安装gym，因为gym在迭代更新，可能过一段时间就会添加新的环境，或者修改源代码。源代码安装方法如下


{% highlight shell %}
git clone https://github.com/openai/gym
cd gym
pip install -e . # minimal install
{%  endhighlight %}

推荐用python 3.5及以上版本，虽然也支持python 2.7，但是毕竟将来会被取代。


可以通过两种方法查看gym支持的环境，

1. 源代码，参考[这里](https://github.com/openai/gym/blob/master/gym/envs/__init__.py)
2. 代码调用，`print(envs.registry.all())`

PS: 有些环境需要安装第三方库，比如月球登陆环境（LunarLander-v2）需要二维物理引擎box2d。


在不使用任何强化学习算法的情况下，可以通过下面观察环境的表现，

{% highlight python %}
import gym
env = gym.make('CartPole-v0')
env.reset()
for _ in range(1000):
    env.render()
    env.step(env.action_space.sample()) # take a random action
{%  endhighlight %}

如果需要实现自定义环境，只需要继承环境基类[Env](https://github.com/openai/gym/blob/master/gym/core.py#L11)，然后实现下面几个方法既可以，

* action_space （必须），定义动作空间
* state_space （必须），定义状态空间
* reset （必须），定义初始化的状态
* step （必须），环境针对action的反馈
  * 返回四元组，包括**下一个状态, 奖励, 结束标记, 自定义信息**
* render （可选），每一步的渲染，用于可视化
* seed （可选），随机数种子
* close （可选），释放资源


按照gym的规则实现了自定义环境对象，或者使用现有的环境对象，可以直接以统一的方法与其他强化学习算法交互，如Q-Learning，Sarsa，DQN等，有兴趣的读者可以参考项目[bourneli/reinforcement-learning](https://github.com/bourneli/reinforcement-learning)。
