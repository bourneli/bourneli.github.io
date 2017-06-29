---
layout: post
title:  因子分解机FM-高效的全组合特征模型
categories: [ML,FM]
---

## 背景

FM算法，全称[Factorization Machines][1],中文一般翻译为“因子分解机”。2010年，它由当时还在日本大阪大学的Steffen Rendle提出的。此算法的主要作用是可以把特征进行全排列，不需根据领域知识，确定哪些特征需要组合高阶特征，这样可以大量特征工程的工作，我们可以将精力集中在模型优化中；更有意义的是，此过程只需要线性时间复杂度，可以应用于大规模机器学习应用。并且，经过试验，此算法在稀疏数据集合上的效果要明显好于SVM。


## 计算原理

前面提到了FM的这么多好处，我们来看看FM的具体形式。首先，对于特征集合$x = (x_1,x_2,\cdots,x_n)$和标签$y$。希望得到x与y的关系，可以使用线性回归模型，

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i \qquad (1)
$$

但是，一般线性模型无法学习到高阶行为，所以会将特征进行二阶组合，以期望学习到更为高阶的行为，这样的模型形式为

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n w_{ij}x_ix_j \qquad (2)
$$

相比于模型(1)而言，模型(2)多了$\frac{n(n-1)}{2}$参数，这可是参数爆炸性了。比如有1000个特征（将连续变量离散化后，经过one-hot编码后，特征上千不要太容易），那么就增加近50万参数！

FM使用了一个技巧——矩阵分解，将参数的数量减少成了线性量级。我们可以将$w_{ij}$看做一个矩阵，

$$
  W = \begin{bmatrix}
    w_{11} & w_{12} & \cdots & w_{1n} \\
    w_{21}& w_{22}\\
     \vdots & \vdots & \ddots \\
    w_{n1} & w_{n2} & \cdots & w_{nn}
  \end{bmatrix}
$$

实数矩阵$W$是对称的！所以实对称矩阵W正定（至少半正定，这里假设正定）！根据矩阵的知识，正定矩阵可以分解，而且形式非常简单，

$$
  W = Q\Lambda Q^T
$$

其中Q是正交单位矩阵，即$QQ^T=I$；$\Lambda$是对称矩阵，且对角线元素全部大于0，且可以将对角线元素从大到小排列，即$\lambda_1 \ge \lambda_2 \ge \cdots  \lambda_n > 0$。这些结构是不是非常优美！基于这些特性，可以分解$\Lambda= \sqrt{\Lambda}\sqrt{\Lambda^T}$，令$V = Q\sqrt{\Lambda}$，所以有$W=VV^T$。理论上V应该是$n \times n$矩阵，但是使用主成份的思想，取$\sqrt{\Lambda}$最大的前f($\ll n$)个主对角元素，

$$
  W \approx V_fV_f^T \qquad(3)
$$

这样$V_k$就是$n \times f$矩阵了。$V_f$的形式如下，

$$
  V_f = \begin{bmatrix}
  v_{1,1} & v_{1,2} & \cdots & v_{1,f} \\
  v_{2,1} & v_{2,2} & \cdots & v_{2,f} \\
  \vdots  & \vdots  & \vdots & \vdots \\
  v_{n,1} & v_{n,2} & \cdots & v_{n,f}
  \end{bmatrix}
  = \begin{bmatrix}
  v_1  \\
  v_2  \\
  \vdots   \\
  v_n   
  \end{bmatrix} \qquad (4)
$$

使用(3),(4)的形式表示$W$，代入(2)

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n v_iv_j^Tx_ix_j \qquad (5)
$$

这样，将需要计算的二阶参数从原来的$\frac{n(n-1)}{2}$降到$nf$个。

公式(5)不但减少了二阶参数，同时降低了样本的要求。公式(2)要求任意的特征$x_i$与$x_j$需要有足够的样本，才能学习到有意义的$w_{ij}$。但是，对于一个非常稀疏的数据集X，并不能能保证任意$x_i$与$x_j$都有样本，更何况足够的样本！

FM就不同，$v_i$向量可以认为是每个特征的隐式向量（命名原因）。对于每个特征，样本应该是足够的，否则就没有必要添加这个特征。所以，只要有足够样本学习到$v_i$与$v_j$，就可以学习到$w_{ij}$。




FM与SVD++,LR的关系


公式优化方法，
损失函数，
正规化

## 实现方法

基于一个github项目，添加了动量，计算验证数据的损失函数

## 后续展望

FFM， GBDT+FM

## 参考资料

* [1][Factorization Machines,Steffen Rendle,2010][1]
* [2]我的FM项目

[1]:http://www.algo.uni-konstanz.de/members/rendle/pdf/Rendle2010FM.pdf
