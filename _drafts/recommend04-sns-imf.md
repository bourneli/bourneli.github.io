---
layout: post
title: 推荐技术系列04：利用社交关系的隐式矩阵分解原理和实践
categories: [Recommendation,IMF,SNS]

---



## 问题背景

笔者的工作环境中有大量的关系链数据，但是在物品推荐时并没有充分利用这些数据，实在感觉可惜。所以希望找到一种方法可以充分利用关系链，以期望达到更好的推荐效果。之前笔者研究过[隐式矩阵分解(IMF)][imf]推荐技术，并且在实践中大规模的应用该方法，大部分场景上取得了不错的效果。该方法的主要优势是仅需用户物品交互数据即可快速上线，无需大量特征工程，并且可以使用廉价CPU进行大规模并行计算，轻松处理上亿数据。如果能够在保留这些特性的基础上再充分利用关系链，那岂不是一举两得。经过笔者的分析，IMF可以抽象为，

$$
\text{User} + \text{Item} \xrightarrow{\text{Embedding}} \text{Ranking}
$$

笔者尝试在此算法基础上融合关系链，整个算法的本质可以提炼为如下形式，

$$
\text{User} + \text{Item} + \text{Relation} \xrightarrow{\text{Embedding}} \text{Ranking}
$$

为什么融合关系链数据可以提升推荐效果？该方法主要受到生活中启发：物以类聚，人以群分。基本假设为：朋友之间有相同的特质，这种特质使得我们成为好友的可能性变大，同时也控制着我们对物品的喜好。那么通过关系链，将好友喜欢的物品传递给用户，是不是会有额外的效果。比如在现实中，你的好友给你推荐了几本书，你会发现有一种相见恨晚的感觉；你可能和你的朋友喜欢相同的一款奈雪果茶；你坐了你朋友的车后，你也买了同样的车型。



## 算法推导

经过对原始IMF的算法原理分析后，有两种可能的改进思路：

1. 社交关系融入目标函数
2. 社交关系融入Confidence



### 方向1：社交关系融入目标函数

目标函数的修改如下

$$
G(x_\star,y_\star) = \sum_{u,i} c_{ui} \left(p_{u,i} - \beta x_u^Ty_i- (1-\beta )\frac{\sum_{v \in n(u)} x_v^Ty_i s(x_u,x_v)}{\sum_{v \in n(u)} s(x_u,x_v)}  \right) ^2  + \lambda\left(\sum_u \Vert x_u \Vert^2 + \sum_i \Vert y_i \Vert^2 \right)  
 \qquad (1)
$$


IMF算法的符号与[这里][imf]保持一致。引入的新符号如下，

*  $n(u)$ 表示用户u的好友
*  $s(x_u, x_v)$ 表示用户u和v的关系权重 
*  $\beta$ 表示用户偏好和好友偏好的权重，介于0-1之间

虽然公式(1)考虑的好友的关系，但是这个算法计算复杂度非常高，他无法利用现有IMF的计算trick，无法在线性时间内完成计算，无法支撑大规模工业级别计算，所以放弃在此方向。



### 方向2：社交关系融入Confidence

首先回忆IMF中confidence的计算过程，

$$
c_{ui} = 1 + \alpha r_{ui} \qquad (2) 
$$

$$
c_{ui} = 1 + \alpha \log(1 + \frac{r_{ui}}{\epsilon}) \qquad (3) 
$$

(2)或(3)均可，根据实际效果显示，(3)的离线效果往往较好。 $r_{ui}$ 是用于u和物品i的交互，如果将该值考虑关系，那么可以仍然利用现有IMF的实现，即可融合关系数据。笔者给出一种改进方法作为示例，

$$
r^\prime_{ui} = r_{ui} + \delta \frac{\sum_{v \in n(u)} r_{vi} s(x_u,x_v)}{\sum_{v \in n(u)} s(x_v,x_v)}  \qquad(4)
$$

其中$\delta$是衰减系数，控制关系传递的confidence的显著程度。

$s(x_u, x_v)$最简单的方法是好友权重相同，即$s(x_u, x_v)=1$。但是，有些社交关系带权重，此时可以直接利用这些权重，即$s(x_u, x_v) = w_{uv}$。如果希望利用社交关系链本身的特征，并且关系链没有显示的权重，可以通过我们中心研发的[Personalized PageRank][ppr]算法计算出隐式的关系权重，该算法可以高效处理上百亿的关系链，即$s(x_u, x_v) = PPR(u,v)$。

此方法对原有IMF的计算框架无任何修改，保留了IMF的所有优点，仅需要在“特征”上将社交关系融如其中。笔者使用此方法分别进行了离线实验和线上验证，取得了不错的效果。



## 实验效果

*介绍一下数据类型，但是要隐去敏感信息，然后给出离线和在线效果，给出相对提升*



## 总结





## 参考资料

* [(2019)Distributed Algorithms for Fully Personalized PageRank on Large Graphs ][ppr]
*  [推荐技术系列02：隐式矩阵分解(IMF)详解][imf]









[ppr]:https://arxiv.org/pdf/1903.11749.pdf
[imf]:http://bourneli.github.io/imf/recommendation/2019/03/16/recommend02-imf.html