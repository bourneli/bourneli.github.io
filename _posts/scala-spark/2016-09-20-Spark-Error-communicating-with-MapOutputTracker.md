---
layout: post
title:  "Spark优化那些事(3)-Spark Error communicating with MapOutputTracker异常"
categories: [scala,spark]
---

今天碰到了MapOutputTracker异常，通过提高Driver Memory解决了，这里记录一下，作为备忘。问题的起因是解析数据源有200GB+,4亿条记录，每一条是一个二进制的blob字符串。起初设置如下

* num-executors = 100
* executor-cores = 2
* executor-memory = 10 GB
* driver-memory = 1 GB

手动repartion了RDD为100个分区。由于spark每个partition有[2GB的限制](1)，而数据总量是200GB+,那么至少有一个分区超过该限制，执行时抛出**Fetch Block**异常。

所以，只有将分区设置多些，后来将RDD分区数设置为2000。但是执行中抛出了异常消息“**Spark Error communicating with MapOutputTracker**”。经过一番google，并且与同事讨论，最后确定可能是分区太多，导致driver需要维护的分区信息太多，超出了driver的内存范围（当时设置为1GB）。所以，最后将driver-memory设置为**10GB**，问题得到解决！

上面是两个问题串联起来的，如果单独看，比较难梳理出问题源头。只有将两个问题联系一起，才能梳理出问题的真正原因。


## 参考资料

* [Spark分区2GB限制][1]
* [Stack Overflow上的相关问题][2]

[1]: http://www.cnblogs.com/bourneli/p/4456109.html
[2]: http://stackoverflow.com/questions/32487147/spark-error-communicating-with-mapoutputtracker


