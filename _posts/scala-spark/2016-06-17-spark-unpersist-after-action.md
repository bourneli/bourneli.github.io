---
layout: post
title:  "Spark优化那些事(1)-请在action之后unpersisit!"
categories: [scala,spark]
---
## 背景
最近在使用spark做一些图算法方面的工作，遇到了一些spark性能优化方面的坑，折腾了好久，最后通过各方面的努力，包括与同事讨论，阅读spark相关的原始论文，stackoverflow提问，google检索等，解决了一些，这里开个系列，总结相关内容。本博文是该系列第一篇，分享一个之前一直没有注意的事情，cache/persist后的rdd，没有使用就unpersist，等于白干。下面看看示例代码，

{% highlight scala %}
val rdd1 = ... // 读取hdfs数据，加载成RDD
rdd1.cache

val rdd2 = rdd1.map(...)
val rdd3 = rdd1.filter(...)

rdd1.unpersist

rdd2.take(10).foreach(println)
rdd3.take(10).foreach(println)
{% endhighlight %}


上面代码的意图是：既然rdd1会被利用两次，那么就缓存起来，用完后释放内存。问题是，rdd1还没有被复用，就被“释放”了，导致rdd2,rdd3在执行take时，仍然需要从hdfs中加载rdd1,没有到达cache效果。


## 原理
这里要从RDD的操作谈起，RDD的操作分为两类：action和tranformation。区别是tranformation输入RDD，输出RDD，而action输入RDD，输出非RDD。transformation是缓释执行的，action是即刻执行的。上面的代码中，hdfs加载数据，map，filter都是transformation，take是action。所以当rdd1加载时，并没有被调用，直到take调用时，rdd1才会被真正的加载到内存。

cache和unpersisit两个操作比较特殊，他们既不是action也不是transformation。[cache会将标记需要缓存的rdd](https://github.com/apache/spark/blob/b0d884f044fea1c954da77073f3556cd9ab1e922/core/src/main/scala/org/apache/spark/SparkContext.scala#L1306)，真正缓存是在第一次被相关action调用后才缓存；[unpersisit是抹掉该标记，并且立刻释放内存](https://github.com/apache/spark/blob/b0d884f044fea1c954da77073f3556cd9ab1e922/core/src/main/scala/org/apache/spark/SparkContext.scala#L1313)。

所以，综合上面两点，可以发现，在rdd2的take执行之前，rdd1，rdd2均不在内存，但是rdd1被标记和剔除标记，等于没有标记。所以当rdd2执行take时，虽然加载了rdd1，但是并不会缓存。然后，当rdd3执行take时，需要重新加载rdd1，导致rdd1.cache并没有达到应该有的作用，所以，正确的做法是将take提前到unpersist之前，如下：

{% highlight scala %}
val rdd1 = ... // 读取hdfs数据，加载成RDD
rdd1.cache

val rdd2 = rdd1.map(...)
val rdd3 = rdd1.filter(...)

rdd2.take(10).foreach(println)
rdd3.take(10).foreach(println)

rdd1.unpersist
{% endhighlight %}

这样，rdd2执行take时，会先缓存rdd1，接下来直接rdd3执行take时，直接利用缓存的rdd1，最后，释放掉rdd1。

## 总结
上面的问题经过简化，剔除噪声，所以显得很简单。但是在实际工作中，当rdd在经过若干的if else, while后，很容易迷失方向。所以，使用RDD开发迭代算法时，需要时刻注意rdd的缓存和释放，确保rdd在unpersisit之前被加载，这里推荐[Graphx Pregel](https://github.com/apache/spark/blob/branch-1.6/graphx/src/main/scala/org/apache/spark/graphx/Pregel.scala)实现，很仔细的缓存和释放rdd，提高执行效率。



## 参考资料
* [Understanding Spark's caching](http://stackoverflow.com/questions/29903675/understanding-sparks-caching)
* [Spark源代码](https://github.com/apache/spark)



