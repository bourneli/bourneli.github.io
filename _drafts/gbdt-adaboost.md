---
layout: post
title:  GBDT前世今生(I)---AdaBoost
categories: [ML,GDBT]
---


AdaBoost解决二元分类问题，模型形式为叠加模型，即最终模型是由一个个基础模型叠加的形式组合起来，形式如下

$$
  G(x) = sign \left( \sum_{m=1}^M{\alpha_m g_m(x)} \right) \qquad(1)
$$

$g_m$是浅层决策树，甚至可以是最简单的决策树---只有一个分割点，两个叶子。





Boosting是一种逐步增强的方法，通过将基础模型叠加到现有模型上，可以获得非常好的效果，通用形式如下


$$
  f(x) =  \sum_{m=1}^M{\beta_m b(x;\gamma_m)} \qquad(2)
$$

为了学习(2)的所有参数$\beta_m,\gamma_m$，一般会选取损失函数$L$，然后根据现有数据，转成下面的目标函数

$$
  \min_{\{\beta_m,\gamma_m\}_1^M} {\sum_{n=1}^N L\left(y_i, \sum_{m=1}^M{\beta_m b(x_i;\gamma_m)} \right)} \qquad(3)
$$

对于大多数的损失函数$L$和基础函数$b$，优化(3)都是需要消耗巨大的计算资源和优化技巧，因为有m个基础函数，不同基础函数的系数还需要不断协调才能达到最优。但是，学习单一的基础函数是比较简单的，

$$
  \min_{\{\beta_m,\gamma_m\}_1^M} {\sum_{n=1}^N L\left(y_i, \beta b(x_i;\gamma) \right)} \qquad(4)
$$


所以，一种基于贪心策略的简化近似算法诞生了，称为**前项逐步迭代建模（Foward Stagewise Additive Modeling**。它的核心思想是逐步学习基础函数$b_m(x,\gamma_m)$，并添加到现有模型中，但是不会修改之前模型的参数和系数,令$f_m(x) = \sum_{m=1}^M{\beta_m b(x_i;\gamma_m)}$。算法($I$)：

---
1. Initialize $b_0(x,\gamma_0) = 0$
2. For m = 1 to M
  (a) Compute
      $$
        (\beta_m,\gamma_m) =  \arg \min_{\beta, \gamma} \sum_{n=1}^N L(y_i, f_{m-1}(x_i) + \beta b(x_i,\gamma) )
      $$
  (b) Set $f_m(x) = f_{m-1}(x) + \beta_m b(x;\gamma_m)$
---


举个例子，损失函数为平方错误

$$
  L(y,f(x)) = (y-f(x))^2 \qquad (5)
$$

将(5)代入算法($I$)的2(a)中，

$$
  \begin{align}
    L(y,f_{m-1}(x_i) + \beta_m b(x_i;\gamma_m)) &= (y-f_{m-1}(x_i) - \beta_m b(x_i;\gamma_m))^2 \\
    &= (r_{m,i} - \beta_m b(x_i;\gamma_m))^2
  \end{align}
$$

可以看到，其实基础模型$\beta_m b(x;\gamma_m)$就是学习之前模型$f_{m-1}(x)$的残差。



为什么需要基础函数b比随机稍微好一点，在AdaBoost的计算过程中，可以清楚看出来。它可以确保$\beta > 0$，同时却保了减少当前的错误的方向是最优的。

AdaBoost使用指数损失函数，形式如下

$$
  L(y, f(x)) = e^{-yf(x)}, \quad y \in \{-1,1\}
$$

将指数损失函数代入算法（I）2a中，有

$$
  (\beta_m,G_m) =  \arg \min_{\beta, G} \sum_{n=1}^N e^{-y_i(f_{m-1}(x_i) + \beta G(x_i)) }
$$

令$w_i^{(m)} = e^{-y_if_{m-1}(x_i)}$，可以理解为样本$i$在低$m$轮的权重,并且该权重只与1到m-1轮函数以及$x_i,y_i$有关，与第m轮函数无关，那么即使一个常量，

$$
  (\beta_m,G_m) =  \arg \min_{\beta, G} \sum_{n=1}^N  w_i^{(m)} e^{-\beta y_i G(x_i) }, G(x) \in \{-1,1\} \qquad (10.9)
$$

目标函数有了，将目标函数变形，方便求解


$$
  \begin{align}
  \sum_{n=1}^N  w_i^{(m)} e^{-\beta y_i G(x_i) }
    &= e^{-\beta} \sum_{y_i = G(x_i)} w_i^{(m)} +   e^{\beta} \sum_{y_i \ne G(x_i)}w_i^{(m)} \\
    &= e^{-\beta} \left(\sum_{i=1}^N w_i^{(m)} - \sum_{i=1}^N w_i^{(m)}I(y_i \ne G(z_i)) \right)  
       + e^{\beta} \sum_{i=1}^N w_i^{(m)} I(y_i \ne G(x_i))
    \\
    &= e^{-\beta}  \sum_{i=1}^N w_i^{(m)} - e^{-\beta} \sum_{i=1}^N w_i^{(m)}I(y_i \ne G(z_i))  
       + e^{\beta} \sum_{i=1}^N w_i^{(m)} I(y_i \ne G(x_i)) \\
    &= e^{-\beta}  \sum_{i=1}^N w_i^{(m)} + (e^{\beta} - e^{-\beta}) \sum_{i=1}^N w_i^{(m)}I(y_i \ne G(z_i)) \qquad (10.11)
  \end{align}
$$

目标函数(10.11)是关于标量$\beta$与函数$G$的函数,可以求偏导，计算最优值。
