---
layout: post
title:  "Spark踩坑之App闭包Null Pointer问题"
categories: [scala,spark,pit]
---

昨天踩了[App子类闭包问题](http://stackoverflow.com/questions/31303827/spark-broadcasted-variable-returns-nullpointerexception-when-run-in-amazon-emr-c)，刚开始用Spark 2.1的DataSet相关API，误以为是使用的姿势不正确，定位问题的方向不对，浪费了好多时间调试。后来改回成DataFrame API，问题得到了快速定位。因为这个bug在DataSet闭包中，使用broadcast的value不会报错，程序可以顺利执行；而在DataFrame闭包中，调用broadcast的value，会抛出null pointer异常。

看看下面的例子，

{% highlight scala %}
object DemoBug extends App {
    val conf = new SparkConf()
    val sc = new SparkContext(conf)

    val rdd = sc.parallelize(List("A","B","C","D"))
    val str1 = "A"

    val rslt1 = rdd.filter(x => { x != "A" }).count
    val rslt2 = rdd.filter(x => { str1 != null && x != "A" }).count

    println("DemoBug: rslt1 = " + rslt1 + " rslt2 = " + rslt2)
}
{% endhighlight %}

输出内容

```
DemoBug: rslt1 = 3 rslt2 = 0
```

根据输出，说明变量**str1**并没有正确的传到**rdd**的闭包filter中。如果将App换成main，可以得到期望的结果。

{% highlight scala %}
object DemoBug {
    def main(args:Array[String]) = {
      val conf = new SparkConf()
      val sc = new SparkContext(conf)

      val rdd = sc.parallelize(List("A","B","C","D"))
      val str1 = "A"

      val rslt1 = rdd.filter(x => { x != "A" }).count
      val rslt2 = rdd.filter(x => { str1 != null && x != "A" }).count

      println("DemoBug: rslt1 = " + rslt1 + " rslt2 = " + rslt2)
    }
}

{% endhighlight %}

输出内容

```
DemoBug: rslt1 = 3 rslt2 = 3
```

根据[spark官方bug](https://issues.apache.org/jira/browse/SPARK-4170)反馈，此问题已经解决了，但是实际来看还是没有解决。所以还是乖乖使用main吧！
