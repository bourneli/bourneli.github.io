---
layout: post
title: 推荐技术系列03：Word2Vector的原理和推荐中的应用
categories: [Recommendation,Word2Vector,NLP,Embedding]

---



## 技术背景

Word2Vector自从2013年被Google的Mikolov及其团队提出，目前被广泛的应用在工业界。虽然它是NLP领域的技术，但是也可以被用来解决物品推荐相关的问题。经实践证明，无论是直接使用Word2Vector做推荐，还是用于计算物品特征，均可以得到比较好的效果。本文主要介绍Word2Vector的原理，包括主要思路，优化以及应用案例。



## 工作原理

Word2Vector的主要作用是将语料中的单词（word）映射到一个固定低维度的稠密向量空间中，使得相近的词(如orange和fruit，就比orange和house要近)在这个空间中的距离较近。由于词之间具有距离相似性，可以用来做聚类，分类，统计分析，可视化，所以word2vector属于一种预训练技术。

它的做法是解决一个“假”(fake)的相邻词查找任务，在解决完这个任务的同时，得到“副产物”就是所有的词向量。另一个类似的技术是Auto Encode，通过编码将原始数据压缩到一个低维的向量空间，然后通过解码将其还原为原始数据，但是最终只需要中间的隐藏向量。

这个“假”任务有两种形式:Skip-Gram和CBOW。这两个形式原理比较类似，但是Skip-Gram的形式更为简单，所以主要在Skip-Gram的形式下介绍相关原理和tricks。



### Skip-Gram架构

Skip-Gram架构解决相邻词预测问题。具体做法是在语料中任意找到一个词，然后预测其周围词的概率，相邻范围由Window Size设定，下面取Window Size=2为例,

![](\img\w2v_img\training_data.png)

根据上面的语料，可以生成样本(the,quick)，(quick, brown)，(fox,over)等。有了这些样本后，需要构建一个函数，接收词的One Hot编码，输出所有的词的概率$p_i$。Word2Vector本质上是将无监督学习转成有监督问题。有很多方法可以求解这个问题，比如深度神经网络叠加100层MLP，或者用浅层逻辑回归，又或者Wide & Deep等。但是Google的工程师使用了相对非常简单的网络结构，如下：

![](\img\w2v_img\skip_gram_net_arch.png)

从网络结构来看，就只有一个隐藏层，甚至没使用激活函数，然后接一个Softmax层，将输出归一化为概率。笔者感觉就是“As simple as possible, but no simpler.”。而我们的词向量就是中间这个隐藏层的参数，

![](\img\w2v_img\word2vec_weight_matrix_lookup_table.png)

这个隐藏层可以看做一个矩阵$\matrix{M}$（如上图左边矩阵），将词的One Hot编码看做是行向量$\vec{x}$。$\vec{x}\matrix{M}$得到该词的向量。如果从上往下看（如上图右边矩阵），向量$\vec{x}$只有在对应词的地方是1，其他全是0，所以词向量$\vec{x}\matrix{M}$就是矩阵$\matrix{M}$对应的行向量。



### 优化Tricks

为什么使用如此简单的网络结构？假设所有的词集合为$V$,词的个数为$\vert V \vert = 10000$，词的隐含维度为300，那么中间隐含层$\matrix{M}$需要学习的参数为$10000\times 300=3,000,000$，同理softmax层的参数数量也为$3,000,000$，所以整体需要学习的参数数量为**6,000,000**。如果使用100层的MLP，需要学习的参数会达到上亿的数量级。如此多的参数，需要非常大的样本才能够得到比较好的效果，所以必须使用一些优化的技巧，否则很难得到好的词向量。

Word2Vector的作者们使用了一些采样技巧，在急剧的减少计算量的同时，提升了词向量的质量效果，总结如下：

1. 对频繁出现的词进行欠采样，降低训练样本数量；
2. 上下文窗口尺寸随机变化，对临近的词给予更高的权重；
3. 修改目标函数，使用“负采样”（Negative Sampling）使得每个训练样本只更新模型中的一小部分参数。



#### Trick 1: 频繁词欠采样

回顾输入的样本，

![](\img\w2v_img\training_data.png)

上面样本的主要问题就是词“the”出现得太多了，而且诸如("fox", "the")的这类样本意义不是特别大，所以需要将这类样本从整体的训练样本中剔除。Word2Vector通过给每个词设定一个概率来随机剔除这个词，这个词的概率与词的频率有关，


$$
P(w_i) = 1- \sqrt{\frac{t}{f(w_i)}}
$$


其中$f(w_i)$词$w_i$的频率，$t$是词频的阈值，大于这个阈值的词需要欠采样，原始论文中给的值是$10^{-5}$。这个公式保留了词的频率和采样概率的正相关性，但是对词频大于$t$的词给出了非常大的惩罚，词频小于t的词概全部保留。这虽然是启发式的方法，但实际效果非常好。



#### Trick 2: 随机窗口尺寸

Word2Vector设计了一个巧妙且简单的方法，对较近的词给予较高权重，同时减少训练样量，提升计算效率。通过随机减少上下文窗口长度来实现此目的。比如上下文窗口window_size=5，那么每次计算样本时，随机在window_size=1到5之前选取，通过这种方法，较近词被选取的权重高，较远词被选取的权重低，示意图如下，

![](\img\w2v_img\shrunking_context_size.png)

上下文内的词被选取为训练样本的概率分布类似一个钟型。这种方法不需要修改任何网络架构或者数学定义即可达到目的，非常优雅。



#### Trick 3: 负采样

训练样本的量级可能达到十几亿，而参数为几百万，使用梯度下降优化方法每处理一个样本需要计算几百万参数的梯度，训练完所有的样本需要更新的参数量是天文数字（十几亿乘以几百万），即使用随机梯度递减，计算的量级也很大（几百万乘以几百万）。

“负采样”优化方法通过将问题转成二分类方法，转换后的目标函数每次迭代只需要更新相关的词的向量，其他词向量无需更新，所以减少了每一个更新内部的操作，再加上随机梯度下降，双管齐下，使得整个训练变得可行。

我们采用[斯坦福深度自然语言处理CS224N课程](https://www.bilibili.com/video/av41393758/?p=3)第2,3讲中的符号进行演示。原始问题的目标函数如下（极大似然，负的log），


$$
J(\theta) = -\frac{1}{T} \sum_{t=1}^T \sum_{-m\le j \le m\text{ & } m j \ne 0} \log{p(w_{t+j} \vert w_t)}
$$


将里面的$p(w_{t+j} \vert w_t)$单独提取出来，


$$
p(o \vert c) = \frac{\exp{(u_o^Tv_c)}}{\sum_{w=1}^V \exp{(u_w^Tv_c)}}
$$


其中c表示center，即中心词，o表示outside，即中心词之外被预测的词，V是整个词的集合，同一个词此作为c（目标词向量）或o（softmax层对应的向量）时需要学习的向量不同。通过一顿操作（可参考Stanford CS224n课程第二讲），该函数梯度为


$$
\frac{\partial p(o|c)}{\partial v_c}=u_o - \sum_{x=1}^V p(x\vert c) u_x
$$


该梯度的计算量显然是$O(V)$，难以接受。

原始问题既然是预测中心词c的情况下出现词o的概率，其实就是预测o和c共现的概率，所以原始问题可以变成二元分类问题，即将问题$p(o \vert c,\theta)$转成$p(D=1 \vert o, c;\theta)$，正样本是o和c共现，负样本就是与o不共现的样本，由于词语很多，所以随机抽k的样本作为负样本（此思路训练集只有正样本，随机在未知的数据抽取负样本的方法异曲同工）。更改后的单个样本的目标函数变为如下，


$$
J(\theta) = \log \sigma(u_o^Tv_c) + \sum_{i=1}^k \mathbb{E}_{j\sim P(w)} [\log{\sigma(-u_j^Tv_c)}]
$$


可以看到，这个目标函数最多涉及到$k+2$个词，大大减少了计算的时间复杂度。负采样方法通过问题转换，使得内部计算的量级大幅度减少。



Word2Vector还有一种形式为CBOW（Continuous Bag-Of-Word），它解决的fake问题和Skip-Gram恰好相反，即给出多个上下文的词，预测中间词的概率，主要使用了Hierarchical SoftMax和Huffman Tree来加速求导过程，避免求导倒时遍历整个词集V，有兴趣的读者可以参考[这里](http://building-babylon.net/2017/08/01/hierarchical-softmax/)。




## 应用案例

Word2Vector虽然是NLP技术，但是近几年来被广泛应用与物品推荐领域。其背后的思想很简单，word2vector学出来的词向量可以将类似的词映射到相近的空间中。将用户和物品交互序列看做句子，物品本身看做词，应该也可以将相近的道具映射到一起，这样即可以直接使用kNN对这些做道具推荐，也可以放到下游中利用监督学习做物品推荐。下面列举几个工业界的案例。



### 案例1：音乐推荐

 Anghami是中东地区的音乐流媒体公司（可以理解为中东的QQ音乐）。他们从用户的收听列表中学习歌曲的向量，然后根据每个用户的收藏列表的中的歌曲向量平均后得到用户的（Taste）口味向量，可以将口味向量周围还没有被用户收藏或收听的歌曲推荐给用户，此[动画](https://cdn-images-1.medium.com/max/1400/1*xbNM_CnEIWQtGbsLmZtE-A.gif)形象的演示了整个过程，有兴趣的读者可以看看。这是原始博客[Using Word2vec for Music Recommendations](https://towardsdatascience.com/using-word2vec-for-music-recommendations-bb9649ac2484)。

 

### 案例2：Airbnb的listing推荐

Airbnb修改了Word2Vector中的Skip Gram的目标函数，加入了短租房相关的逻辑进行训练，结果取得了很好的做法，并且将其实践发表成论文[Real-time Personalization using Embeddings for Search Ranking at Airbnb](https://medium.com/airbnb-engineering/listing-embeddings-for-similar-listing-recommendations-and-real-time-personalization-in-search-601172f7603e)，该论文获得2018 KDD最佳论文。其主要思想是在原有Negative Sampling基础上添加了成功预订的listing作为正样本，并且在中心listing地理附近没有出现的listing作为负样本，同时还用地理位置附近的listing的平均值作为新加入的listing的向量，解决冷启动问题，更多细节可以参考上面的论文，或者阅读机器之声的文章[从KDD 2018最佳论文看Airbnb实时搜索排序中的Embedding技巧](https://www.jiqizhixin.com/articles/2019-01-24-20)。



### 案例3：雅虎邮件产品推荐

雅虎通过用户邮箱中的购物收据，获取用户和物品的交互列表，然后生成物品向量。其创新在于将用户的物品向量进行聚类，然后为了避免推荐相似的物品，他们推荐相领聚类中的相似物品，示意图如下，

![](\img\w2v_img\cluster_recommendations.png)

更多细节，可以参考雅虎在2016年发表的论文[(2016)E-commerce in Your Inbox: Product Recommendations at Scale](https://arxiv.org/pdf/1606.07154.pdf)。



### 案例4：搜索和广告推荐

这次是雅虎的搜索，他们将用户的query，广告和点击的连接作为序列进行学习，这样将query和广告映射到相同的空间，就可以推荐出更加相关的广告。他们面临的主要挑战是查询，广告和点击的量级是亿级别，而不是NLP中的万级别。所以，他们开发了分布式的word2vector算法用于训练如此大的样本空间。同时，他们开发了一个类似Facebook Fassis的在线向量检索系统[Nearest](http://www.nearist.ai/)。更多细节，可以阅读他们2016年发表的论文[Scalable Semantic Matching of Queries to Ads in Sponsored Search Advertising](https://arxiv.org/abs/1607.01869)。



## 总结

Word2Vector之所以被广泛的应用，笔者认为它解决了推荐中的核心问题--物品特征。物品低维度稠密向量的距离特征，可以很好的被现有的推荐算法利用，给出较好的排序结果。其他方法也可以计算物品的特征，但是计算效率和实现成本远远不及Word2Vecor，所以此技术基本上成为各公司推荐团队的标准配置，掌握此技术很有必要。

上面简单的介绍了Word2Vector的原理，以及应用案例，希望能够降低读者入门门槛，如果希望了解更多细节，可以阅读**参考资料**中相关的文献。



## 参考资料

* 原始论文1 (2013)Efficient Estimation of Word Representations in Vector Space
* 原始论文2 (2013)Distributed Representations of Words and Phrases and their Compositionality
* [Word2Vec Tutorial - The Skip-Gram Model](http://mccormickml.com/2016/04/19/word2vec-tutorial-the-skip-gram-model/)
* [Word2Vec Tutorial Part 2 - Negative Sampling](http://mccormickml.com/2017/01/11/word2vec-tutorial-part-2-negative-sampling/)
* [Applying word2vec to Recommenders and Advertising](http://mccormickml.com/2018/06/15/applying-word2vec-to-recommenders-and-advertising/)
* [斯坦福深度自然语言处理CS224N课程](https://www.bilibili.com/video/av41393758/?p=3)第2,3讲
* (2015)word2vec Explained Deriving Mikolov  Negative-Sampling Word-Embedding Method
* [Hierarchical Softmax for CBOW](http://building-babylon.net/2017/08/01/hierarchical-softmax/)
* [What is hierarchical softmax?](https://www.quora.com/What-is-hierarchical-softmax)
* (2018)Real-time Personalization using Embeddings for Search Ranking at Airbnb. 2018 KDD最佳论文。
* [(2016)E-commerce in Your Inbox: Product Recommendations at Scale](https://arxiv.org/pdf/1606.07154.pdf)
* [(2016)Scalable Semantic Matching of Queries to Ads in Sponsored Search Advertising](https://arxiv.org/abs/1607.01869)

