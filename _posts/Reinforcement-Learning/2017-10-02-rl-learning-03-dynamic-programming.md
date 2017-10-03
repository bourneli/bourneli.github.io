---
layout: post
title:  强化学习笔记03-动态规划
categories: [Reinforcement-Learning]
---

你没看错，就是那个在ACM算法比赛中经常用到的**动态规划**。

### 动态规划与MDP的关系

动态规划是一种叫常见的求解最优解算法，一般问题具符合以下两个特性，可以使用动态规划求解

* 问题可以迭代表示，即问题可以分解为子问题
* 子问题可以被缓存和复用

MDP符合上面的特点

* Bellman公式是迭代形式，也就是可以用子问题表示
* 可以存储和复用值函数，

虽然动态规划可以接觉MDP问题，但是需要知道的内容太多，基本上要知道MDP的所有参数，$<S,A,P,R,\gamma>$。这在实际问题上，基本上不可能实现的。所以虽然动态规划的思路无法直接应用到实际问题，但是后续改进方法都是基于动态规划的思路，所以还是非常有必要了解如和使用动态规划求解MDP问题。

### 策略评估

给定一个策略，计算出所的值函数$v(s)$。同步的方法，步骤如下

* 在每次迭代k+1
* 对所有的状态$s \in S$
* 使用$v_k(s\prime)$更新$v_{k+1}(s)$
* $s^{\prime}$是$s$的后续状态

经过一段迭代后，该值会收敛。 数学形式如下，

$$
v_{k+1}(s)=\sum_{a \in A }\pi(a \vert s)\left(R_s^a + \gamma\sum_{s^{\prime} \in S} P_{ss^{\prime}}^av_k(s^{\prime}) \right) \\
v_{k+1} = R^\pi + \gamma P^\pi v_k (s^{\prime})   \qquad 矩阵表示
$$

在Small World Grid的例子中，由于上面的$R,P,\gamma,\pi$全部已知，所以在给定任意状态$v(s^\prime)$下，总可以评估$v(s)$。


### 策略改进

前面讲过，策略的优劣，是由对应的值函数反映，

$$
\pi^{\prime} \ge \pi, if \space  v_{\pi^\prime}(s) \ge v_{\pi}(s),\forall s \in S
$$

所以，上面知道了如何评估值函数，现在可以考虑得到更好的策略$\pi^\prime$。如何得到，方法很简单——贪心。对于确定性的策略，即$\pi(a|s)=1$，有$v_{\pi}(s) = q_{\pi}(s,a)$。使用贪心策略，

$$
\pi^\prime(s)=\arg \max_a q(s,a)
$$

这样，只需要一步，就可以完全确保新的值函数不比之前的值函数差，

$$
v_{\pi^\prime}(s)=q(s, \pi^\prime(s)) = \max_a q_\pi(a,s) \ge q_\pi(a,s) = v_\pi(s)
$$

最后一定会收敛，因为存在最优解，那么一定有上界，所以如果迭代如果一点点在变好，那么一定会达到最优解，可以通过下面的示例图形象的刻画整个过程。

<div style='display: block;margin: auto;width:80%'>
	<img src='/img/policy_evaluation_improvement.png' />
</div>

结合**策略评估**和**策略迭代**的过程，求解最优策略和最优值函数的方法称为**策略迭代**。

### 值迭代

此方法是另一种求解最优策略和值函数的方法。同步的计算方法如下

* 在迭代k+1过程
* 所有状态$s \ in S$
* 直接使用$v_k(s^{\prime}) $更新$v_{k+1}(s)$

相比于策略迭代，值迭代直接在原来的状态上收敛更快，存储要求更少，数学形式如下

$$
v_{k+1}(s) = \max_{a \in A}\left(R_s^a + \gamma \sum_{s^\prime \in S} P_{ss^\prime}^a v_k(s^\prime) \right) \\
v_{k+1} = \max_{a \in A} (R^a + \gamma P^av_k)
$$

在得到最优解后，最优策略可以直接根据贪心的方法得到。