---
layout: post
title:  强化学习笔记05-无模型(Model Free)策略求解
categories: [Reinforcement-Learning]
---

这一讲合并了Sutton教材5,6,7章的后半部分。上一讲主要介绍MC，TD的值函数计算。即给定policy，估计值函数$V(s)$。这一讲，介绍求解最优policy以及值函数$Q(s,a)$。主要介绍的内容如下

* MC $\epsilon$-贪心求解策略
* SARSA: TD(0) On-Policy $\epsilon$贪心求解策略
* Q-Learing: TD(0) Off-Policy $\epsilon$贪心求解策略
* SARSA和Q-Learning的通用版本



### On/Off-Policy的区别

* On-Policy学习：直接通过目标策略$\pi$采样，学习目标策略$\pi$
* Off-Policy学习：通过行为策略$\mu$采样，学习目标策略$\pi$

### $\epsilon-$贪心探索

核心思想是让m个行为均有机会得到尝试，而不是总是尝试当前最优的行为。设置参数$\epsilon \in [0,1]$，以概率$1-\epsilon$使用最优行为（即贪心），或以概率$\epsilon$随机选择一个行为，形式的表示如下

$$
\begin{equation}
  \pi(a\vert s)=\left\{
  \begin{array}{@{}ll@{}}
   \epsilon/m + 1-\epsilon & \text{if}\ a^*=\arg \max_{a \in A(S)} Q(s,a) \\
    \epsilon/m & \text{otherwise}
  \end{array}\right.
\end{equation}
$$

根据上面定义，可以发现$\sum_{a \in  A(s)} \pi(a \vert s) = 1$ 。

### 加权平均小于等于最大值

后面证明$\epsilon$-贪心策略可以确保策略更新时会用到**加权平均小于等于最大值**。假设数组$X={x_1,x_2,\cdots, x_n}, \  x_i \in R$，证明

$$
\max(X) \ge \sum_{i=1}^n w_ix_i \qquad (1) \\
s.t.  w_i \ge 0  且 \sum_{i=1}^n w_i = 1
$$

对$w_i$进行通分，转成如下形式

$$
w_i = \frac{m_i}{M} \qquad (2) \\
s.t. \sum_{i=1}^n m_i = M, M \gt 0, m_i >= 0
$$

任意小数是可以转成分数形式，不断通分，可得到(2)。将(2)代入(1),

$$
\max(X) \ge \sum_{i=1}^n \frac{m_i}{M}x_i  \\
\Leftrightarrow M \max(X) \ge \sum_{i=1}^n m_ix_i \qquad (3)
$$

(3)不等式的左边有M个最大值，有边有m个值，但是他们都小于等于$max(X)$，所以(3)不等式恒成立，证毕。



### $\epsilon-$贪心策略提升

贪心策略是可以稳定提升值函数值，

$$
\begin{align}
q_\pi(s, \pi^{\prime}(s))
	&= \sum_{a \in A(s)} \pi^{\prime}(a \vert s) q_\pi(s,a)  \qquad Line \ 1\\
	&= \epsilon/m \sum_{a \in A(s)}q_\pi(s,a) + (1-\epsilon) \max_{a \in A(s)} q_\pi(s,a)  \qquad Line \ 2 \\
	&\ge \epsilon/m \sum_{a \in A(s)}q_\pi(s,a)  + (1-\epsilon) \sum_{a \in A(s)} \frac{\pi(a \vert s) - \epsilon/m}{1-\epsilon} q_\pi (s,a)  \qquad Line \ 3 \\
	&= \sum_{a \in A(s)} \pi(a \vert s) q_\pi(s,a) = v_\pi(s)  \qquad Line \ 4\\

\end{align}
$$

第二行是根据$\epsilon-$贪心策略定义得到，第三行中$ \sum_{a \in A(s)} \frac{\pi(a \vert s) - \epsilon/m}{1-\epsilon} = 1$，所以该项是加权平均数系数。第二行到第三行使用了上面证明的定理。后面的证明顺水推舟。证明的关键是构造这个加权平均的系数，非常漂亮。

### SARSA

TD(0)的On-Policy形式，为什么称为“SARSA”，难道发明它的是个妹纸？看看更新图，

![](/img/sarsa_back_up_diagram.png)

更新步骤需要5个样本序列$S_t,A_t,R_t,S_{t+1},A_{t+1}$，简称SARSAR，更新公式为

$$
Q(S,A) \leftarrow Q(S,A) + \alpha(R + \gamma Q(S^\prime, A^\prime) - Q(S,A))
$$

算法伪代码如下，

![](/img/sarsa_algo.png)

为什么说是TD(0)？因为只看了后面一步既可以更新Q函数。为什么是On-Policy？因为完全通过策略$\pi$的样本，没有使用其他策略样本。Sarsa可以通用化扩展为$Sarsa(\lambda)$,方法类似$TD(\lambda)$，先定义$q^{(n)}_t$和$q_t^{\lambda}$，然后使用后向视角逐步更新，具体细节这里略去，详情可以参考教材和幻灯片对应章节。



### Q-learning

在讲具体算法之前，需要先了解[Importance Sampling](https://www.youtube.com/watch?v=S3LAOZxGcnk)，主要思想是用已知的分布p，去估算未知或复杂的分布q的特性的方法，比如q的期望，方差，概率积分等。[为什么Q-Learning是On-Policy](https://groups.google.com/forum/#!topic/rl-list/4Efnr0gXhAU0) ,因为它用行为**$\epsilon$-贪心策略**去更新目标**贪心策略**，其更新图如下，

![](/img/q_learning_back_up_diagram.png)

其中，S,A,R是根据$\epsilon$贪心策略生成，$S^\prime,A^\prime$用来更新贪心策略。其更新公式为

$$
Q(S,A) \leftarrow Q(S,A) + \alpha(R+\gamma \max_{a^\prime}Q(S^\prime,a^\prime) - Q(S,A))
$$

算法伪代码如下，

![](/img/q_learning_algo.png)

使用[强化学习笔记-Talk is cheap, show me the code](http://bourneli.github.io/reinforcement-learning/2017/10/14/rl-learning-00-code-practice.html)提到的Cliff实验显示，Q-Learning的收敛速率的确比Sarsa(0)要快。$Q-Learning(\lambda)$形式大致方法与之前类似，只是在部分细节上做了适当修改，这里省略。



### 动态规划DP与时分TD的比较

为什么没有提到MC，MC是$TD(1)$，所以不单独拿出来。下面是更新图和更新方法的区别，作为总结。

![](/img/dp_td_back_up.png "更新策略示意图")

![](/img/dp_td_update.png "更新算法对比")



目前，我们处理的问题都是离散的，即Q的状态空间是十分有限的，如果遇到状态空间是连续的，或者范围很大，怎么办？这也是后面的章节将要处理的问题。
