---
layout: post
title:  矩阵分解技术应用于在线游戏道具推荐场景的调研
categories: [Recommendation,Matrix-Factorization]
---



## 问题背景

在线游戏中，道具售卖是业务主要收入来源，如何高效的售卖道具，直接决定了游戏的收入。但是，相比于被广泛研究的电影推荐，商品推荐等场景，游戏道具推荐有其独特性，

* 道具范围有限。大部分游戏道具数量在100~1000的范围内，长尾效果不明显，基于热门销售的推荐策略往往非常有效。
* 道具使用与游戏设计强绑定，相比传统推荐场景，更依赖专家规则推荐道具。
* 道具特征不足。缺乏道具的结构化描述信息。静态道具图片对于数值型（真气丹，经验书等）道具基本没有作用；对于装饰型道具有部分效果，但是这类道具一般动画特效，静态图片无法描述特效。
* 道具没有显示反馈，比如喜欢，讨厌这类程度的数据。
* 人工维护道具特征成本高昂。

归根结底，游戏道具特征的缺乏，用户对道具显示反馈的缺失等问题对道具推荐产生比较大的阻碍。所以笔者希望找到一种方法，不需借助道具特征，就可以推荐道具，并且线上效果需要强于专家规则和热门销售。



## 矩阵分解推荐技术调研

矩阵分解推荐技术是协同过滤推荐技术中一类。协同过滤推荐算法不需要用户特征和道具特征，仅需要用户和道具的交互数据，所以被工业界广泛的使用。据2015年一篇[矩阵分解推荐技术](https://www.sciencedirect.com/science/article/pii/S1877050915007462)的综述显示，目前主要的矩阵分解推荐方法为,

- SVD，基于矩阵SVD分解思想，计算用户和item隐式向量，推导用户与其他item的得分。
- PCA，基于降为的矩阵分解技术，具体paper没看，应该与SVD类似。
- PMF，矩阵分解中融合的概率分布，将得分看做是正太分布，利用贝叶斯方法推导目标函数，然后SGD求解。
- NMF，非负矩阵分解，将目标矩阵和分解后的矩阵均中的元素均要求为正数，最后通过约束优化求解。

上述几类矩阵分解技术中，使用和研究最广泛的是SVD推荐技术。但是此SVD不是直接使用数学中的矩阵SVD分解技术，而是借助矩阵SVD分解思想，经过改良的SVD推荐技术，作者是Simon Funk，所以也称为[FunkSVD](http://sifter.org/simon/journal/20061211.html)。该技术在2006年Netflix推荐大奖中得到了第三名，由于其形式优美，后来又被广范使用和研究，衍生出了多个版本，比如[SVD++](http://www.cs.rochester.edu/twiki/pub/Main/HarpSeminar/Factorization_Meets_the_Neighborhood-_a_Multifaceted_Collaborative_Filtering_Model.pdf)等。



## 基于隐式反馈的矩阵分解算法

上面提到的这些算法应用的数据集主要是显示反馈---即用户对商品（或电影）的喜好程度，如讨厌，一般，喜欢，非常喜欢等表示程度的数据。在游戏道具推荐场景中，用户的显示反馈（Explicit Feedback）极其匮乏。但是，隐式反馈(Implicit Feedback)却非常丰富，比如用户浏览、使用、购买道具的历史记录都是可以非常轻松获取。所以，在游戏道具推荐场景下，传统的FunkSVD以及衍生算法不能直接应用。如要强行套用，虽然技术上可以，但是原理上无法解释，有点类似使用线性回归解决二分类问题，实际效果可想而知。

为了解决隐式反馈无法兼容显示反馈的问题，YiFan Hu在2008年发表了一篇文章[[2]](http://yifanhu.net/PUB/cf.pdf)解决此问题。主要思想框架仍然是FunkSVD，但是设计了启发式公式将隐式反馈转成显示反馈，然后修改目标函数。同时，考虑隐式反馈的本质，需要计算用户对所有道具的相关性，而不是FunkSVD中仅计算用户与道具有交互的相关项，所以使用了ALS而不是SGD作为最终优化方法。此算法最近获得了**2017 IEEE ICDM 10-Year Highest-Impact Paper Award**，引用量1500+，可见其影响力和实际效果。

虽然此算法影响力较大，但是邓爷爷说过，**实践是检验真理的唯一标准**。后续，笔者打算在多个游戏道具推荐场景应用此算法，进行线下和线上实验，验证其有效性。



## 参考资料

1. [A Gentle Introduction to Recommender Systems with Implicit Feedback](https://jessesw.com/Rec-System/)
2. [(2008)Collaborative Filtering for Implicit Feedback Datasets](http://yifanhu.net/PUB/cf.pdf)，YiFan Hu 
3. (2008)Collaborative Filtering for Implicit Feedback Datasets文章的[Spark实现](http://spark.apache.org/docs/latest/mllib-collaborative-filtering.html)。
4. [(2008)Factorization Meets the Neighborhood- a Multifaceted Collaborative Filtering Model](https://github.com/gpfvic/IRR/blob/master/Factorization%20meets%20the%20neighborhood-%20a%20multifaceted%20collaborative%20filtering%20model.pdf)SVD++原始论文
5. [lightFM](https://github.com/lyst/lightfm)在矩阵分解的基础上，可以融合item或user标签特征。
6. [PMF:概率矩阵分解](https://zhuanlan.zhihu.com/p/27399967)
7. [SVD++ Netflix大奖解决方案paper](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf)
8. [MF Tutorial,总共4篇](http://nicolas-hug.com/blog/matrix_facto_1)，第4篇后面有很多refernce，很值得借鉴

