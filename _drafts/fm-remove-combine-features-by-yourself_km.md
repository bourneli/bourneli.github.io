
# 因子分解机FM-高效的组合高阶特征模型


## 背景

FM算法，全称[Factorization Machines][1],一般翻译为“因子分解机”。2010年，它由当时还在日本大阪大学的Steffen Rendle提出。此算法的主要作用是可以把所有特征进行高阶组合，减少人工参与特征组合的工作，工程师可以将精力集中在模型参数调优。FM只需要线性时间复杂度，可以应用于大规模机器学习。经过部分数据集试验，此算法在稀疏数据集合上的效果要明显好于SVM。


## 模型形式

对于特征集合$x = (x_1,x_2,\cdots,x_n)$和标签$y$。希望得到x与y的关系，最简单是建立线性回归模型，

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i \qquad (1)
$$

但是，一般线性模型无法学习到高阶组合特征，所以会将特征进行高阶组合，这里以二阶为例(理论上，FM可以组合任意高阶，但是由于计算复杂度，实际中常用二阶，后面也主要介绍二阶组合的内容)。模型形式为，

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n w_{ij}x_ix_j \qquad (2)
$$

相比于模型(1)而言，模型(2)多了$\frac{n(n-1)}{2}$参数。比如有(n=)1000个特征（连续变量离散化，并one-hot编码，特征很容易到达此量级），增加近50万个参数。

FM使用**近似矩阵分解**。将参数量级减少成线性量级。可以将所有$w_{ij}$组合成一个矩阵，

$$
  W = \begin{bmatrix}
    w_{11} & w_{12} & \cdots & w_{1n} \\
    w_{21}& w_{22}\\
     \vdots & \vdots & \ddots \\
    w_{n1} & w_{n2} & \cdots & w_{nn}
  \end{bmatrix}
$$

很明显，实数矩阵$W$是对称的。所以，实对称矩阵W正定（至少半正定，这里假设正定）。根据矩阵的性质，正定矩阵可以分解，而且形式非常优雅，

$$
  W = Q\Lambda Q^T \qquad (3)
$$

其中Q是正交单位矩阵，即$QQ^T=I$；$\Lambda$是对角矩阵，且对角线元素全部大于0。可以将其对角线元素从大到小排列，即$\lambda_1 \ge \lambda_2 \ge \cdots  \lambda_n > 0$，只要Q的行作对应调整，不影响等式(3)。

基于这些特性，可以分解$\Lambda= \sqrt{\Lambda}\sqrt{\Lambda^T}$，令$V = Q\sqrt{\Lambda}$，所以有

$$
  W=VV^T \qquad (4)
$$


理论上V应该是$n \times n$矩阵，但是使用**主成份近似**的思想，取$\sqrt{\Lambda}$最大的前f($\ll n$)个主对角元素，

$$
  W \approx V_fV_f^T \qquad(5)
$$

这样$V_f$就是$n \times f$矩阵了。$V_f$的形式如下，

$$
  V_f = \begin{bmatrix}
  v_{1,1} & v_{1,2} & \cdots & v_{1,f} \\
  v_{2,1} & v_{2,2} & \cdots & v_{2,f} \\
  \vdots  & \vdots  & \vdots & \vdots \\
  v_{n,1} & v_{n,2} & \cdots & v_{n,f}
  \end{bmatrix}^T
  = \begin{bmatrix}
  v_1  & v_2  & \cdots   & v_n   
  \end{bmatrix} \qquad (6)
$$

将(5)代入(2)

$$
  y(x) = w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n v_i^Tv_jx_ix_j \qquad (7)
$$

因此，交叉二阶参数从原来的$\frac{n(n-1)}{2}$降到$nf$。

公式(6)不但减少了二阶参数，同时降低了样本$X$的要求。公式(2)要求任意的特征$x_ix_j \ne 0$有足够的样本，才能学习到有意义的$w_{ij}$。但是，对于一个非常稀疏的数据集X，并不能保证任意$x_i \ne0$ 且 $x_j \ne 0$，更何况足够的样本！

FM模型中，$v_i$向量可以认为是特征$i$的隐式向量。对于任意特征$i$，样本应该是足够的，否则就没有必要添加该特征。所以，只要有足够样本学习到$v_i$与$v_j$，就可以学习到$w_{ij}$。

总的来说，计算效率得到质提升是因为使用近似计算，将参数复杂度从$O(n^2)$降到$O(n)$。之前工作中遇到的[介数估算](http://bourneli.github.io/graph/2017/05/19/summary-diamter-and-betweenness.html)，也使用类似套路，减少计算时间复杂度。


如果使用FM进行二元分类，需要将(5)放到sigmod函数$\theta$中，得到模型形式如下，

$$
  y_c(x) = \theta(y(x))= \frac{1}{1+e^{-(w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n v_i^Tv_jx_ix_j)}} \qquad (8)
$$

仔细观察(8)，其实是[逻辑回归](https://en.wikipedia.org/wiki/Logistic_regression)的基础上添加了全二阶组合。


## 学习过程

### 模型优化

上面提到了FM的形式和原因，接下来需要学习模型的参数，首先需要设定损失函数，

* **回归问题** 平方损失,$\frac{1}{2}(y-y(x))^2, y \in R,y(x) \in R$
* **二元分类问题** Logit Loss(也称作Cross-Entropy Error)，$\ln{(1+e^{-y y(x)})}, y\in \{-1,1\}, y(x) \in R $


最常用的是使用随机梯度下降求解参数，也可以使用L-BFGS，MCMC或ALS。使用随机梯度下降或L-BFGS均需要计算出损失函数的梯度。

最优化一般需要加上正规化项，避免参数过拟合，FM添加L2正规化，回归问题和二元分类问题的损失函数如下

**回归目标函数**

$$
  E_r(w,v)=\frac{1}{2m}\sum_{h=1}^m{\left(w_0 + \sum_{i=1}^nw_ix_i^{(h)} + \sum_{i=1}^n\sum_{j=i+1}^n v_iv_j^Tx_i^{(h)}x_j^{（h)} - y^{(h)}\right)^2} \\ + \frac{\lambda_0}{m}w_0^2 +  \frac{\lambda_1}{m}\sum_{i=1}^n{w_i^2} + \frac{\lambda_2}{m}\sum_{i=1}^nv_i^Tv_i
$$

**分类目标函数**

$$
E_c(w,v)=\frac{1}{m}\sum_{h=1}^m{\ln{\left(1+ e^{-y^{(h)}(w_0 + \sum_{i=1}^nw_ix_i + \sum_{i=1}^n\sum_{j=i+1}^n v_i^Tv_jx_ix_j)} \right)}} \\ +  \frac{\lambda_0}{m}w_0^2 +  \frac{\lambda_1}{m}\sum_{i=1}^n{w_i^2} + \frac{\lambda_2}{m}\sum_{i=1}^nv_i^Tv_i
$$

分类问题的标签$y^{h} \in \{-1,1\}$。

### 模型求导

平方损失或Logit Loss梯度非常容易计算，并且FM的线性部分损失函数也易得，下面只给出FM的二阶组合项梯度如下，

$$
  \frac{\partial{y(V)}}{\partial v_{i,f}} = \sum_{i=1}^n\sum_{j=i+1}^n v_{i,f}v_{j,f} x_ix_j \qquad (9)
$$


公式(8)的计算复杂度为$O(n^2)$，但经过一些数学变换，可以将复杂度降为$O(nf)$，首先，需要重新组织FM高阶的表现形式

$$
\begin{align}
  \sum_{i=1}^n\sum_{j=i+1}^n v_i v_j^T x_i x_j
  &= \frac{1}{2} \sum_{i=1}^n\sum_{j=1}^n v_i v_j^T x_i x_j - \frac{1}{2}\sum_{i=1}^n  v_iv_i^Tx_ix_i \\
  &= \frac{1}{2}\left( \sum_{i=1}^n\sum_{j=1}^n \sum_{k=1}^f v_{i,k}v_{j,k} x_i x_j - \sum_{i=1}^n  \sum_{k=1}^f v_{i,k}v_{i,k} x_ix_i  \right)  \\
  &= \frac{1}{2}\left( \sum_{k=1}^f \sum_{i=1}^n\sum_{j=1}^n v_{i,k}v_{j,k} x_i x_j   -  \sum_{k=1}^f \sum_{i=1}^n v_{i,k}^2 x_i^2  \right) \\
  &= \frac{1}{2} \sum_{k=1}^f{\left( \sum_{i=1}^n{v_{i,k}x_i}\sum_{j=1}^n{v_{j,k}x_j} - \sum_{i=1}^nv_{i,k}^2 x_i^2  \right)} \\
  &= \frac{1}{2} \sum_{k=1}^f{\left( \left(\sum_{i=1}^n{v_{i,k}x_i}\right)^2 - \sum_{i=1}^nv_{i,k}^2 x_i^2  \right)} \qquad (10)  \\
\end{align}
$$

对公式(10)计算偏导,

$$
  \frac{\partial{f(v_{i,k})}}{\partial v_{i,k}} =   x_i \sum_{j=1}^nv_{j,k}x_j - x_i^2v_{i,k}   \qquad(11)
$$

偏导(11)中，除了$\sum_{j=1}^nv_{j,k}x_j$，其他是常量复杂度$O(1)$，但是$\sum_{j=1}^nv_{j,k}x_j$可以复用，复杂度为O(n)，需要计算f个，总共复杂度为$O(nf)$。因为需要计算$nf$个偏导，所以最后整体的二项组合部分的复杂度仍为$O(nf)$。

到这里，优化目标函数和梯度全部演算完毕，放到SGD或者L-BFGS中既可以得到最优参数。


## FM实验效果

笔者在两个版本的FM实现上做过实验，

* [libFM][2]是由FM算法作者开发，优化器支持SGD，MCMC和ALS。
* [spark-libFM][3]是基于spark mllib库的框架开发，复用GradientDescent框架和L-BFGS框架，只需要实现梯度计算和梯度更新函数。后面可以考虑集成到[tmllib2](http://pub.code.oa.com/project/home?projectName=Tmllib2&comeFrom=121)中。

实验使用的数据集是一个生产环境的样本，共128521条记录，541个特征，正样本18296条，负样本110225条，正负比例约1:6，平均稀疏率为13%，二元分类问题。切分了80%作为训练，20%测试效果。实验结果如下，

算法 | AUC
---- | ---
逻辑回归LR | 0.643
随机森林 |  0.610
GBDT | 0.602
随机森林 + 逻辑回归 | 0.642
FM | 0.712

可以看到，FM效果最好，效果很明显比没有做高阶排序逻辑回归要好。LR比基于树模型（随机森林和GDBT）要好，可能是由于数据比较稀疏。FM的效果比LR要好，因为LR没有做高阶组合（维度太多，人工组合需要消耗一定的成本）。

实验结果虽然不能作为FM比LR等其他模型优秀的充分条件，但是至少可以给我们带来一些启示，后续应用中，可以尝试使用该模型，至少它不用手动组合二阶特征呀，懒人的福音，:-)。SNG使用FM在[企鹅FM推荐](http://km.oa.com/group/22605/articles/show/292186?kmref=search&from_page=1&no=2)在推荐上做了尝试。



## 相关扩展

FM有一个衍生算法FFM（Field-aware FM），大概思路是将特征进行分组，学习出更多的隐式向量V，FM可以看做FFM只有一个分组的特例。FFM复杂度比较高，比较适合高度稀疏数据；而FM可以应用于非稀疏数据，更加通用。

在工业界中，[GDBT+LR](http://bourneli.github.io/ml/2017/05/25/gdbt-lr-facebook-paper.html)套路得到广泛应用，但是也有其他同行使用[FM替代该套路](http://www.chinacloud.cn/show.aspx?id=21902&cid=6)也得到不错效果。更有甚者,使用[GDBT+FFM](https://github.com/guestwalk/kaggle-2014-criteo)获得了Kaggle CTR比赛的冠军。总而言之，掌握FM，可以给你的机器学习武器库中添加一个强有力的常规武器。


最后，感谢**easonpeng**带来的启发，引起了笔者对FM的兴趣；感谢**robinsonxu**同学提供的实验数据，手动点赞；感谢与**alcaidhuang**同学的讨论，学到了不少东西。



## 参考资料

* [1][Factorization Machines,Steffen Rendle,2010][1]
* [2][libFM,由FM作者Steffen Rendle开发，C++实现单机版][2]
* [3][spark-libFM，基于spark mllib框架实现的FM算法][3]
* [4][美团FFM应用][4]
* [5][新浪FM/FFM应用][5]

[1]:http://www.algo.uni-konstanz.de/members/rendle/pdf/Rendle2010FM.pdf
[2]:http://libfm.org/
[3]:https://github.com/zhengruifeng/spark-libFM
[4]:http://tech.meituan.com/deep-understanding-of-ffm-principles-and-practices.html
[5]:http://www.52caml.com/head_first_ml/ml-chapter9-factorization-family/#
