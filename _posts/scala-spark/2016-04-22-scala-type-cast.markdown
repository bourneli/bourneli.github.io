---
layout: post
title:  "Scala数值类型转换"
categories: [scala,spark]
---

最近在使用scala开发一些spark的扩展，遇到一个常见问题就是将各种数值类型(Int,Short,Float等)对象转成Double。在网上找了一些资料，最后得到两套解决方法:

* **静态方案** 利用scala泛型中的隐式转换。
* **动态方案** 利用java数值基类java.lang.Number作为scala所有数值的基类，动态匹配类型，然后强行转换。

最后采取了动态转换方案，因为类型是运行时才知道的，所以静态方案不适用于此场景，下面贴出两个方案：

{% highlight scala %}
object test  {
    // 静态
    def toDoubleStatic[T](a:T)(implicit n:Numeric[T]) = n.toDouble(a)
    toDoubleStatic(1.1)
    toDoubleStatic(2)

    // 动态
    def toDoubleDynamic(x: Any) = x match {
        case s: String => s.toDouble
        case jn: java.lang.Number => jn.doubleValue()
        case _ => throw new ClassCastException("cannot cast to double")
    }
    toDoubleDynamic(1.1)
    toDoubleDynamic(2)
    toDoubleDynamic("2")
}
{% endhighlight %}

对于数组类型的转换也是经常遇到的问题。因为spark api常用的接口是 **org.apache.spark.mllib.linalg.Vector**，所有数组都需要转成Vector。
scala的集合类型主要分为Seq，Set和Map。目的是需要将所有Seq都转成Vector。但是scala 2.8以后，Array是不属于scala集合框架的，它是为了与java组数兼容而设计的一个集合类，也就是[Seq不是Array的基类](http://stackoverflow.com/a/6165595/1114397)，那么怎么去匹配呢？仍然利用泛型的语法，但是运行时匹配，使用Array[T]作为所有Array的基类，解决方案如下：

{% highlight scala %}
import org.apache.spark.mllib.linalg.{Vector, Vectors}
object test  {

    // 动态
    def toDoubleDynamic(x: Any) = x match {
        case s: String => s.toDouble
        case jn: java.lang.Number => jn.doubleValue()
        case _ => throw new ClassCastException("cannot cast to double")
    }

    def anySeqToSparkVector[T](x: Any) = x match {
        case a: Array[T] => Vectors.dense(a.map(toDoubleDynamic))
        case s: Seq[Any] => Vectors.dense(s.toArray.map(toDoubleDynamic))
        case v: Vector => v
        case _ => throw new ClassCastException("unsupported class")
    }
	
    anySeqToSparkVector(Array(1,2.3,3))
    anySeqToSparkVector(Array("1",2.3,3))
    anySeqToSparkVector(Seq("1",2.3,3))
}

{% endhighlight %}

scala语法博大精深，后面有空还是需要整块学习一下，好不容在网上找到了这些解决方案，记录于此，作为备忘。