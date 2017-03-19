---
layout: post
title:  "Spark优化那些事(4)-关于spark.driver.maxResultSize的疑惑"
categories: [scala,spark]
---

今天遇到了spark.driver.maxResultSize的异常，通过增大该值解决了，但是其运行机制不是很明白，先记录在这里，希望后面有机会可以明白背后的机制。

该异常会报如下的异常信息:

**Job aborted due to stage failure: Total size of serialized results of 3979 tasks (1024.2 MB) is bigger than spark.driver.maxResultSize (1024.0 MB)**

锁定了是spark.driver.maxResultSize引起的，[该参数][1]控制worker送回driver的数据大小，一旦操过该限制，driver会终止执行。所以，我加大了该参数，结果执行成功。

问题就是，代码里不涉及大规模数据回传，代码如下

{% highlight scala %}
... // 省略

// 加载原始数据
val srcData = client.tdwSql(srcDB)
	.table(srcTable, Array("p_" + curDateObj.toString(formatPattern)))
	.filter("iworldid in (%s)".format(worldIdList.mkString(",")))
	.repartition(dataPart)
	.persist(StorageLevel.MEMORY_AND_DISK)
println("Original Data =============================")
srcData.show(10, false)  // 数据加载成功，打印前10行数据

//  计算数据尺寸
val allSize = srcData.map(r => r.getString(3).size + 24).sum   // 此处发生上面的异常
val sizeInG = allSize / 1e9
println(
	s"""
	   |size in bytes : $allSize
	   |size in GB: $sizeInG
			""".stripMargin)

... // 省略
{% endhighlight %}

RDD.sum处发生的异常，但个人认为该action并不涉及大规模数据回传。走读了RDD代码，根据代码注释，该action会分别在每个partition计算sum的值，然后将该值回传给driver。设置了4000个分区，最多就4000个Long数据传回来(32KB)，不会操过1GB限制。原始数据有250G左右，所以重新分为了4000个分区，提高并发计算。这个问题在[Stackoverflow][3]上也有，但是目前没有可靠的答案。

使用的spark基础配置如下

	--num-executors 50
	--driver-memory 10G
	--executor-cores 2
	--executor-memory 10G

	spark.default.parallelism=200
	spark.storage.memoryFraction=0.8
	spark.network.timeout=600
	spark.driver.maxResultSize=10G


## 问题本质

*更新于2017-3-19*

其实上面已经提到了问题的本质，之前driver内存设置为1G，但是需要处理4000个分区，driver需要维护每个分区的状态，分区越多，消耗的driver内存越多，最终导致了driver的Out-Of-Memeory异常。日志里面说的很明白，所当将driver内存设置为10G后，问题迎刃而解。

Spark常见的两类OOM问题：Driver OOM和Executor OOM。如果发生在executor，可以通过增加分区数量，减少每个executor负载。但是此时，会增加driver的负载。所以，可能同时需要增加driver内存。定位问题时，一定要先判断是哪里出现了OOM，对症下药，才能事半功倍。




## 参考资料

* [Spark 1.6.1参数说明文档][1]
* [What is spark.driver.maxResultSize?][2]
* [On Spark 1.6.0, get org.apache.spark.SparkException related with spark.driver.maxResultSize][3]

[1]: http://spark.apache.org/docs/1.6.1/configuration.html
[2]: http://stackoverflow.com/questions/39087859/what-is-spark-driver-maxresultsize
[3]: http://stackoverflow.com/questions/36872618/on-spark-1-6-0-get-org-apache-spark-sparkexception-related-with-spark-driver-ma
