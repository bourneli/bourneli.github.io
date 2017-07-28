---
layout: post
title:  GBDT前世今生(I)---AdaBoost
categories: [ML,GDBT]
---

详细探讨GDBT之前，不得不提AdaBoost，因为GDBT是根据AdaBoost的算法抽象总结得到。AdaBoost是一个迭代模型，用于二元分类，标签取值1或-1。其主要思想是每一轮，通过不断调整错误样本的权重，将上一轮错误的样本权重提高，这样不断的从错误中学习，最后得到一个比较好的模型。十分类似人类学习过程----根据过去错误的经验，不断总结，然后做得更好。形式化的给出AdaBoost算法，

---
Input: N数据量，M迭代轮数
1. 初始化每个数据的权重$w_i=\frac{1}{N},i = 1,2,\cdots,N$
2. For m = 1 to M:
  (a) 使用权重$w_i$抽样,得到数据集$D_m$;使用$D_m$学习模型$G_m(x)$
  (b) 计算
    $$
      err_m = \frac{\sum_{i=1}^N w_i I(y_i \ne G_m(x_i))}{\sum_{i=1}^N w_i}
    $$
  (c) 计算$\alpha_m = \log((1-err_m)/err_m)$
  (d) 更新$w_i \leftarrow w_i e^{\alpha_m \bullet I(y_i \ne G_m(x_i))}, i = 1,2,\cdots,N$
3. 最后模型$G(x) = sign\left[ \sum_{m=1}^M \alpha_m G_m(x)   \right]$
---

AdaBoost算法中，2(c)和2(d)是不是很神奇，为什么这样计算权重和单个模型系数。


这里2017-7-24



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
  \min_{\beta,\gamma} {\sum_{n=1}^N L\left(y_i, \beta b(x_i;\gamma) \right)} \qquad(4)
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

令$w_i^{(m)} = e^{-y_if_{m-1}(x_i)}$，可以理解为样本$i$在第$m$轮的权重,并且该权重只与1到m-1轮函数以及$x_i,y_i$有关，与第m轮函数无关，

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
    &= (e^{\beta} - e^{-\beta}) \sum_{i=1}^N w_i^{(m)}I(y_i \ne G(x_i)) + e^{-\beta}  \sum_{i=1}^N w_i^{(m)}  \qquad (10.11)
  \end{align}
$$

目标函数(10.11)是关于标量$\beta$与函数$G$的函数,可以求偏导，计算最优值。

对于变量$G$而言，10.11是一个线性函数，只需要下面的值最小即可

$$
  G_m = \arg \min_{G} \sum_{i=1}^N w_i^{(m)}I(y_i \ne G(x_i)) \qquad (10.10)
$$

但是需要系数$e^{\beta} - e^{-\beta}$大于0,即$\beta > 0$，

对于变量$\beta$而言，对(10.11)求偏导，并且等于0，

$$
  \beta_m = \frac{1}{2}\log \frac{1-err_m}{err_m},其中err_m = \frac{\sum_{i=1}^Nw_i^{(m)}I(y_i \ne G_m(x_i))}{\sum_{i=1}^Nw_i^{(m)}}
$$

对于任何一个$G$,随机分类器的错误率为0.5，所以可以和容易保证$err_m < 0.5$，这样保证$\beta_m > 0$了。这里有鸡生蛋诞生鸡的问题，到底是谁保证了谁，其实是互相保证了，是不是有点类似“死锁”

$$
  err_m < 0.5 \Leftrightarrow \beta_m > 0
$$

后面继续进行演化权重weight，就可以完全推导AdaBoost的形式了。


exp loss和squared loss分别对应分类和回归问题，而且可以基于他们推导出漂亮的adaboost算法，但是他们对大的$yf(x)$惩罚力度太大，导致对噪声非常铭感，而在实际应用中不太好用。
