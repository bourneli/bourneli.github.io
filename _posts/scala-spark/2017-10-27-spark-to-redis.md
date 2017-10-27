---
layout: post
title:  "Spark写Redis实践总结"
categories: [scala,spark,pit]
---

[Redis](https://zh.wikipedia.org/zh-cn/Redis)是一个高性能键值数据库，最近几年非常流行（[Google趋势](https://trends.google.com/trends/explore?date=all&q=redis)或[百度指数](https://zhishu.baidu.com/?tpl=trend&word=redis)）。笔者所在的团队也在大规模的使用Redis作为后台数据存储解决方案。Redis作为机器学习算法与后台服务器的媒介，算法计算用户数据并写入Redis；后台服务器读取Redis，并为前端提供实时接口。本文主要介绍Spark写如Redis的实践，同时记录一些坑，方便后面回顾。

笔者使用Spark 2.0的scala API，使用[jedis](https://github.com/xetorthio/jedis/wiki/Getting-started)客户端API，dependency如下

{% highlight xml %}
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
  <type>jar</type>
</dependency>
{% endhighlight %}

写Redis的代码如下

{% highlight scala %}
 // 写Redis
sampleData.repartition(500).foreachPartition(rows => {
  val rc = new Jedis(redisHost, redisPort)
  rc.auth(redisPassword)
  val pipe = rc.pipelined

  rows.foreach(r => {
    val redisKey = r.getAs[String]("key")
    val redisValue = r.getAs[String]("value")
    pipe.set(redisKey, redisValue)
    pipe.expire(redisKey, expireDays * 3600 * 24)
  })

  pipe.sync()
})
{% endhighlight %}


### 实践1：控制客户端对象数量

sampleData是一个DataSet，每一行有两个数据：key和value。由于构建Jedis客户端会有一定开销，所以一定不要用map，而是mapPartition或foreachPartition。这样，这个开销只会与parition数量相关，与数据总量无关。试想如果sampleData有1亿行，在map中将会构建1亿个Jedis对象。

### 实践2：批量插入数据

笔者使用了pipe进行批量插入，而不是逐条插入，批量插入效率与逐条插入效率差异[参考这里](http://www.cnblogs.com/ivictor/p/5446503.html)。但是批量插入**有个非常大的坑**。上面的代码中，一次性批量插入了整个partition的数据，所以如果单个partition的数据量太多，会导致Redis内存溢出，导致服务不可用！

解决方法是在foreachPartition之前，repartition整个DateSet，确保每个分区的数据不要太大。推荐控制在1千左右。正如上面的列子，笔者将sampleData分为500个分区，每个分区1000条，那么sampleData的总数为50万左右。但是，如果数据总量太大，单个分区过小，会导致分区数过大，这样需要提高driver的内存，否则会导致driver内存溢出。

### 实践3：控制在线更新并发

Redis一般提供在线服务，在更新Redis的同时，它可能在前端提供服务。所以在写Redis时，不能使用太多executor。否则会使得QPS过高，影响在线服务响应，甚至导致Redis瘫痪。推荐的实践方法是提高数据的分区数量，确保每个partition的数量较小，然后逐步提高并发数量（executor数量）。观察在不同数量executor下，并发写入Redis的QPS，直到QPS达到一个可以接受的范围。


### 最后
实践是检验真理的唯一标准。上面的几点实践在不同Redis集群下，具体数值可能不一样，但原理不变。希望这些总结对你有用。如果自己想搭建spark+Redis环境，推荐VPS供应商[Vultr](https://www.vultr.com/?ref=7245986)，无需购买服务器，随时随地可用，物美价廉。









