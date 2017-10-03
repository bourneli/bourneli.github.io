---
layout: post
title:  强化学习笔记02-马尔科夫决策过程MDP
categories: [Reinforcement-Learning]
---



这一讲主要介绍MDP，通过搭积木的方式，一点点累加，最终得到马尔科夫决策过层以及相关概念。



### **Markov Process** 马尔科夫过程

核心思想是下一个动作只与当前动作有关，与历史无关，无记忆。由$<S,P>$构成

* S，状态集合
* P，状态转移矩阵， $P_{ss^{\prime}}=P[S_{t+1}=s^{\prime}\vert S_t = s]$



### **Markov Reward Process** 马尔科夫奖励过程

MP基础上添加了奖励。由$<S,P,R,\gamma>$

* S，状态集合
* P，状态转移矩阵， $P_{ss^{\prime}}=P[S_{t+1}=s^{\prime} \vert S_t = s]$
* R 是奖励函数，与状态有关，$R_s=E[R_{t+1} \vert  S_t=s]$
* $\gamma$是打折系数，约束未来奖励的预期，$\gamma \in [0,1]$



### **Return** 回报

从时刻t开始的折扣回报

$$
	G_t=R_{t+1} + \gamma R_{t+2} + \cdots = \sum_{k=0}^{\infty}\gamma^k R_{t+k+1}
$$


为什么要设计为上面的样子



* 数据上易于操作

* 避免奖励无限循环

* 未来既然不可知，那么就应该少期望一点

* 在经济学上，即时激励可以获取更多的利息

* 人们更喜欢即时激励

* 如果序列有限，那么可以设置$\gamma=1$，不打折。





###  **Value Function** 值函数

这个概念很重要，MDP基本上就是为了最大化这个指标。在t时刻，处于状态s时，回报的期望，表示如下

$$
v(s)=E(G_t \vert S_t=s)
$$

随着状态变化，这个期望值是在变化的。

Bellman等式，可以将值函数分解如下，


$$
\begin{align}
v(s)  &= E[G_t \vert S_t = s] \\
        &= E[R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots \vert S_t = s] \\
        & = E[R_{t+1} + \gamma (R_{t+2} + \gamma R_{t+3} + \cdots) \vert S_t = s]  \\
        & = E[R_{t+1} + \gamma G_{t+1} \vert S_t = s] \\
        & = E[(R_{t+1} + \gamma v(S_{t+1}) \vert S_t = s] \\
        & = R_s + \gamma \sum_{s\prime \in S}P_{ss^{\prime}} v(s^{\prime})
\end{align}
$$


当前状态的值函数，是即时激励加上折扣乘以下个状态值函数期望。


$$
v=R+\gamma Pv
$$



上面还可以用矩阵表示，


$$
\begin{bmatrix}
v(1) \\
v(2) \\
\vdots \\
v(n)
\end{bmatrix} = 
\begin{bmatrix}
R_1 \\
R_2 \\
\vdots \\
R_n
\end{bmatrix} + 
\gamma \begin{bmatrix}
P_{11} & P_{12} & \cdots & P_{1n} \\
P_{21} & P_{22} & \cdots & P_{2n} \\
\vdots & \vdots  & \ddots & \vdots \\
P_{n1} & P_{n2} & \cdots & P_{nn}
\end{bmatrix}
\begin{bmatrix}
v(1) \\
v(2) \\
\vdots \\
v(n)
\end{bmatrix}
$$


$R,\gamma,P$已知，存在解析解


$$
v=(I-\gamma P)^{-1}R
$$



解此方程复杂度为$O(n^3)$，有几个常用迭代近似解法，



* 动态规划 Dynamic Programming

* 蒙特卡洛评估 Mento-Carlo evaluation

* 时分学习 Temporal-Difference Learning




### **Markov Decision Process**

MDP是在MRP基础上，添加的动作集合A，由$<S,A,P,R,\gamma>$表示。现在，状态转移需要根据动作来决定，

- S，状态集合
- A，动作集合
- P，状态转移矩阵， $P_{ss^{\prime}}^a=P[S_{t+1}=s^{\prime} \vert S_t = s, A_t = a]$
- R 是奖励函数，与状态有关，$R_s^a=E[R_{t+1} \vert S_t=s, A_t=a]$
- $\gamma$是打折系数，约束未来奖励的预期



###  **Policy** 策略

直观解释，就是状态与动作的映射的概率分布，是**概率分布**，


$$
\pi(a|s)= P[A_t = a \vert S_t = s]
$$

策略完全决定了智能体的行为，如果该分布确定，那么智能体的行为就确定了，可以认为是模型。后面的工作就是使用各种方法，找到最优策略。下面，将策略加入奖励和转移方程，可以认为是期望，


$$
P_{ss^{\prime}}^{\pi} = \sum_{a \in A} \pi(a \vert s)P_{ss^{\prime}}^a \\
R_s^{\pi} = \sum_{a \in A}\pi(a \vert s)R_s^a
$$

值函数也可以引入策略，


$$
v_{\pi}(s) = E_{\pi}[G_t \vert S_t = s]
$$

值函数可以进一步按动作拆分，称为行为值函数（Action-Value Function）


$$
q_{\pi}(s,a) = E_{\pi}[G_t \vert S_t = s, A_t = a]
$$

值函数与动作值函数的关系，


$$
v_{\pi}(s) = \sum_{a \in A} \pi(a \vert s)q_{\pi}(s,a)
$$

Bellman等式应用于策略值函数和行为值函数


$$
v_{\pi}(s) =  E_{\pi}[R_{t+1} + \gamma v_{\pi}(S_{t+1}) \vert S_t = s] \\
q_{\pi}(s,a) = E_{\pi}[R_{t+1} + \gamma q_{\pi}(S_{t +1},A_{t+1}) \vert S_t = s, A_t = a]
$$

Bellman公式的最大好处就找到了值函数的**递归关系**。将递归形式按照期望定义展开，


$$
q_{\pi}(s,a) 
= R_s^a + \gamma \sum_{s^{\prime} \in S} P^a_{ss^{\prime}} v_{\pi}(s^{\prime}) 
= R_s^a + \gamma  \sum_{s^{\prime} \in S} P^a_{ss^{\prime}} \sum_{a^{\prime} \in A} \pi(a^{\prime} \vert s^{\prime}) q_{\pi}(s^{\prime},a^{\prime}) 
\\
v_{\pi}(s) = \sum_{a \in A} \pi(a \vert s) q_{\pi}(s,a) = \sum_{a \in A} \pi(a \vert s)\left (R_s^a + \gamma \sum_{s^{\prime} \in S} P^a_{ss^{\prime}} v_{\pi}(s^{\prime})\right)
$$

上面的值函数，仍然可以写成矩阵形式，这次添加了策略因素


$$
v_{\pi} = R^{\pi} + \gamma P^{\pi}v_{\pi} => v_{\pi} = (I-\gamma P^{\pi})^{-1}R^{\pi}
$$

矩阵形式，so beautiful!



###  **最优值函数** Optimal Value Function

RL问题据就是求解这个最优值函数，找到最优的策略$\pi$，定义如下



$$
v_*(s) = \max_{\pi} v_{\pi}(s),\\
q_*(s,a) = \max_{\pi} q_{\pi}(s,a)
$$

最优策略的Bellman公式展开，


$$
v_*(s) = \max_{a} \left ( 
	R_s^a + 
	\gamma \sum_{s^{\prime} \in S} P^a_{ss^{\prime}} v_{\pi}(s^{\prime})
\right) \\

q_*(s,a) 
= R_s^a + \gamma \sum_{s^{\prime} \in S} P^a_{ss^{\prime}} \max_{a^{\prime}}{q_*(s^{\prime},a^{\prime})}
$$

朴素的贪心思想，无处不在。没有解析解，但是有很多迭代方法，

* Value Iteration
* Police Iteration
* Q-learning
* Sarsa



