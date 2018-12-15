---
layout: post
title:  LSH在欧式空间的应用(4)--算法实现与优化总结
categories: [probability,LSH]
---

此博文是本主题的最后一篇，主要是记录实现该算法时，所遇到的性能问题，以及解决方法，并且在最后给出实现代码，希望对读者有用。

实现的工具是scala/spark，spark的api使得实现迭代的算法非常容易。但是如果不优化，在大的数据集上运行效率非常堪忧。好在经过一定摸索，$9千万\times 15$的数据集，用了13个小时计算完成，最后得到了138亿的相似对。Spark的参数配置如下：

    num-executors 100
    driver-memory 1G
    executor-cores 2
    executor-memory 10G
    spark.default.parallelism=200
    spark.storage.memoryFraction=0.8
    spark.yarn.allocation.executor.maxMemory=35G
    spark.driver.maxResultSize=4G

## 优化1-分布合并hash桶

LSH算法需要合并L个hash桶。最开始是先将L个桶计算完后，然后合并去重。这样需要同时将L个桶的数据保存在内存中，非常消耗空间。所以，优化的方法是按每n(<L)个桶合并，这样最多也只需要同事保留n个hash桶，节省了空间。


## 优化2-巨片随机化

在LSH过程中，如果数据分布非常集中，那么必然导致hash桶中一个hash key上聚集非常多的数据。比如在我的试验数据中，300万的数据，有一个hash key上聚集了1.7万的数据，称此现象为巨片。如果对这些数据进行排列组合，那么单个partition的突破spark 2GB限制，导致计算异常结束。解决方法是设置一个阀值，如果单个key聚集的对象数量高于这个阀值，就随机取样少量相识对象。这样效果不会太差，因为能够聚到一起的，说明本来就很相似。但是效率却得到了极大提升。

## 优化3-数据id变成整型

有些数据的id是字符串型，该数据十分消耗内存，建议通过hash的方法将其转成整型。比如我的试验数据集合，原始id是32个字符串，通过hash变成Long后，只有8个字节，空间节省了75%。虽然hash过程中可能存在一定冲撞，但应是小概率时间，可以忽略。


## 优化4-减少频繁Iterable转IndexedSeq

这个地方是没有注意的细节，修改后效率极大提升，所以还是记录于此。GroupByKey后得到的对象是Iterable，无法随机访问，必须转成IndexedSeq对象。修改之前，每次都在随机访问时转成IndexedSeq，相当消耗新能，尤其是部分hash key特别密集时，部分倾斜的分区计算相当滞后。修改后，只转换一次IndexedSeq，后续访问重复利用，性能得到了极大的提升。

## 优化5-手动Hash分区

这个问题可以参考[StackoverFlow中问题][4]。在**优化1-分布合并hash桶**时，结果需要调用partitionBy(new HashPartitioner(parts))，将结果转成ShuffledRDD。因为distinct操作可以最大化利用ShuffledRDD的，减少不必要的重新排序和网络传输。

## 优化6-提前过滤 (2017-5-23更新)

由于多个rdd merge时会很多内存和时间，所以每次计算一个桶rdd即过滤。此过程的开销是每个rdd与原始数据join两次。但是，适当做好partition（参考[这里](http://bourneli.github.io/scala/spark/2017/03/19/spark-job-stage-task-smart-join.html)），只有一次join需要shuffle，另外一次不需要。最终性能得到了巨大提升。原来1亿乘30的数据计算时会OOM，优化后同样配置可以稳定在6小时完成。2千万行乘以30列的数据，优化前需要3小时，优化后只需要1小时。spark 2.1中添加了欧式距离LSH[BucketedRandomProjectionLSH](https://github.com/apache/spark/blob/branch-2.1/mllib/src/main/scala/org/apache/spark/ml/feature/BucketedRandomProjectionLSH.scala)，但是没有做太多优化，同样配置，该算计算上面提到的1亿乘以30的数据集时，出现OOM异常。

## 参考代码 (2018-12-15更新)

下面的实现是离线计算版本，在线计算版本需要后台服务器开发，无法使用spark实现。但是如果掌握了整个LSH原理，在线版本不会太困难。下面是源代码，有兴趣的同学可以自取。

{% highlight scala %}
import org.apache.spark.ml.linalg.{Vectors, Vector}
import org.apache.spark.rdd.RDD
import org.apache.spark.storage.StorageLevel
import breeze.stats.distributions.{Gaussian, Uniform}
import org.apache.spark.HashPartitioner
import scala.reflect.ClassTag
import scala.util.Random
import scala.collection.mutable.ArrayBuffer

/**
  * 实现欧式空间的LSH算法，添加了大量优化，详细信息，可以参考下面的文献
    *
  * http://bourneli.github.io/probability/lsh/2016/09/15/lsh_eulidian_1.html
  * http://bourneli.github.io/probability/lsh/2016/09/22/lsh_eulidian_2.html
  * http://bourneli.github.io/probability/lsh/2016/09/23/lsh_eulidian_3.html
  * http://bourneli.github.io/probability/lsh/2016/10/01/lsh_eulidian_4.html
    *
  * @param distance     每个相似对的最大距离小于等于distance
  * @param lshWidth     映射后的桶的宽度，文献中对应w
  * @param bucketWidth  桶内，相同lsh的个数，文献中对应k
  * @param bucketSize   桶的个数,文献中对应L
  * @param storageLevel 缓存策略，数据量小为Memory,数据量大为Memeory_and_Disk
  * @param parts          RDD分区数量,用于设置并发
  * @param lshUpBound     单个桶样本上限，一旦突破此上限，采取随机取样策略
  * @param lshRandom      随机选取的数量，随机的个数
  * @param mergeSize      LSH表格缓存个数
  * @param maxSize        每个样本，最多保留maxSize个相似对
    */
class LSHForE2(distance: Double, lshWidth: Double, bucketWidth: Int = 10, bucketSize: Int = 200,
      ​         storageLevel: StorageLevel = StorageLevel.MEMORY_AND_DISK, parts: Int = 1000, lshUpBound: Int = 1000,
      ​         lshRandom: Int = 10, mergeSize: Int = 8, maxSize: Int = 12) extends Serializable {

  assert(distance > 0, s"distance = $distance must be greater than 0")
  assert(lshWidth > 0, s"Width = $lshWidth must be great than 0")
  assert(bucketWidth > 0, s"k = $bucketWidth must be great than 0")
  assert(bucketSize > 0, s"l = $bucketSize must be great than 0")
  assert(lshRandom < lshUpBound, s"lshRansom $lshRandom should be less than lshUpBound $lshUpBound")

  /**
​    * 计算lsh
​    *
​    * @param raw 需要计算的数据
​    * @tparam K id类型，可以使string或long，建议为long，节省内存
​    * @return 每一对id以及距离
​    */
  def lsh[K: ClassTag](raw: RDD[(K, Vector)]): RDD[(K, K, Double)] = {

    // 分区加缓存，用于后面的join
    val data = raw.partitionBy(new HashPartitioner(parts)).persist(storageLevel)
    val nf = java.text.NumberFormat.getIntegerInstance
    
    // 特征长度
    val vectorWidth = data.first._2.size
    assert(vectorWidth > 0, s"Vector width is $vectorWidth")
    
    // 生成所有的hash参数
    val normal = Gaussian(0, 1) // 生成随机投影向量
    val uniform = Uniform(0, lshWidth) // 生成随机偏差
    val hashSeq: IndexedSeq[(Vector, Double)] = for (i <- 0 until bucketWidth * bucketSize)
      yield (Vectors.dense(normal.sample(vectorWidth).toArray), uniform.sample)
    val hashFunctionsBC = data.sparkContext.broadcast(hashSeq)
    
    // 计算lsh值
    val lshRDD = data.map({ case (id, vector) =>
      val hashFunctions = hashFunctionsBC.value
      val hashValues = hashFunctions.map({
        case (project: Vector, offset: Double) =>
          val dotProduct = project.toArray.zip(vector.toArray).map(x => x._1 * x._2).sum
          ((dotProduct + offset) / lshWidth).floor.toLong
      })
      (id, hashValues.toArray.grouped(bucketWidth).toArray)
    }).persist(storageLevel)
    println("LSH Tag")
    lshRDD.take(10).map(x =>
      "%s -> %s".format(x._1, x._2.map(_.mkString(" ")).mkString(",")))
      .foreach(println)
    val dataSize = lshRDD.count
    val allPairSize = dataSize * (dataSize - 1) / 2d
    println("LSH Size:" + nf.format(dataSize))
    
    // 计算所有的LSH桶
    val bucketBuffer = ArrayBuffer[RDD[(K, (K, Double))]]()
    (0 until bucketSize).map(i => {
      val begin = System.currentTimeMillis
    
      // 统计lsh后每个id的聚集的数据量
      val idGroupRDD = lshRDD.map(x => (x._2(i).mkString(","), x._1))
        .groupByKey(parts)
        .map(_._2.toArray) // 只需要id,并且转成array，方便后面随机选取，否则非常消耗性能。
        .filter(_.length > 1) // 过滤一个id的情况
        .persist(storageLevel)
      val meanGroup = idGroupRDD.map(_.length).mean
      val stdevGroup = idGroupRDD.map(_.length).stdev
      val maxGroup = idGroupRDD.map(_.length).max
      val pairsCount = idGroupRDD.map(_.length).map(x => x * (x - 1)).sum
      println(
        s"""
           |Mean Group Size: $meanGroup
           |Standard Deviation: $stdevGroup
           |Max Group Size: $maxGroup
           |Pairs Count: $pairsCount
                """.stripMargin)
    
      // 组合相似对
      val similarPairs = idGroupRDD.flatMap(neighbors => {
        if (neighbors.length <= lshUpBound) {
          // 候选集不多，排列组合任意两个id
          val pairs = neighbors.combinations(2)
            .map(x => (x(0), x(1))).toArray
          pairs ++ pairs.map(_.swap)
        } else {
          // 候选集过多，随机选取，避免排列组合爆炸
          val rand = new Random(System.currentTimeMillis())
          val allRandomPairs = ArrayBuffer[(K, K)]()
          for (key <- neighbors) {
            val randomRhs = (for (_ <- 0 until lshRandom)
              yield neighbors(rand.nextInt(neighbors.length)))
              .distinct.map(x => (key, x))
            allRandomPairs.appendAll(randomRhs)
          }
          allRandomPairs.toArray[(K, K)]
        }
      }).partitionBy(new HashPartitioner(parts)).persist(storageLevel)
    
      // 剔除过多数据
      val reduceSimilarPairs = similarPairs.join(data)
        .map({ case (srcKey, (dstKey, srcVec)) => dstKey -> (srcKey, srcVec) })
        .partitionBy(new HashPartitioner(parts))
        .join(data)
        .map({ case (dstKey, ((srcKey, srcVec), dstVec)) =>
          val distance = scala.math.sqrt((srcVec.toArray zip dstVec.toArray)
            .map(x => scala.math.pow(x._1 - x._2, 2)).sum)
          srcKey -> (dstKey, distance) // TODO: 此处可能可优化
        })
        .groupByKey(parts)
        .flatMap({ case (srcKey, distList) => distList.toArray
          .filter(_._2 < distance)
          .sortBy(_._2)
          .slice(0, maxSize)
          .map({ case (dstKey, dist) => (srcKey, (dstKey, dist)) })
        }).partitionBy(new HashPartitioner(parts)).persist(storageLevel)
    
      val currentResultSize = reduceSimilarPairs.count
      val end = System.currentTimeMillis
      val timeCost = (end - begin) / 1000d
      println(s"=======Rount $i, Result Size = %s,Time Cost: $timeCost s"
        .format(nf.format(currentResultSize)))
      idGroupRDD.unpersist(false)
      bucketBuffer.append(reduceSimilarPairs)
    
      // 合并中间表格，节省空间
      if ((bucketBuffer.length >= mergeSize
        || i == bucketSize - 1)
        && bucketBuffer.length > 1) {
        val mergeBegin = System.currentTimeMillis
    
        val merged = data.sparkContext.union(bucketBuffer)
          .groupByKey(parts)
          .flatMap({ case (srcKey, dstList) => {
            dstList.toArray
              .distinct
              .sortBy(_._2)
              .slice(0, maxSize)
              .map({ case (dstKey, dist) => (srcKey, (dstKey, dist)) })
          }
          }).partitionBy(new HashPartitioner(parts)).persist(storageLevel)
    
        val mergedCount = merged.count
        println(s"Merged Count : %s".format(nf.format(mergedCount)))
    
        bucketBuffer.foreach(_.unpersist(false))
        bucketBuffer.clear
        bucketBuffer.append(merged)
    
        val mergeEnd = System.currentTimeMillis
        val timeCost = (mergeEnd - mergeBegin) / 1000d
        println(s"Merge Time Cost: $timeCost s")
      }
    })
    
    assert(1 == bucketBuffer.size, "Bucket Size is great than 1")
    val finalResult = bucketBuffer(0)
      .map({ case (srcKey, (dstKey, dist)) => (srcKey, dstKey, dist) })
      .persist(storageLevel)
    bucketBuffer.clear
    val distinctLshParisSize = finalResult.count
    println(s"Distinct Lsh Pair Size %s, rate = %.3f".format(
      nf.format(distinctLshParisSize), distinctLshParisSize / allPairSize))
    
    finalResult
  }
}
{% endhighlight %}



## 算法参数说明（2018-12-15更新）

在使用此算法的时候，有同学反馈参数太多，不会配置，下面笔者简单的介绍一下参数的意义，以及相关资料。参数主要分为几类，

- 核心算法参数，这些参数与算法精度和召回相关，建议了解算法原理后进行设置效果更佳，
  - **lshWidth** 映射后的桶的宽度，文献中对应w，该值越大，可能导致所有样本映射到一个桶中，可以参考[这篇文章](http://bourneli.github.io/probability/lsh/2016/09/23/lsh_eulidian_3.html)进行设置该值。
  - **bucketWidth** 桶内相同lsh的个数，文献中对应k，该值越大，精度越高。
  - **bucketSize** 桶的个数,文献中对应L，该值越大，召回越大，可以参考[这篇文章](http://bourneli.github.io/probability/lsh/2016/09/23/lsh_eulidian_3.html)进行设置。
  - **lshUpBound** 单个桶样本上限，一旦突破此上限，采取随机取样策略，具体意义见本文[优化2-巨片随机化][优化2-巨片随机化]
  - **lshRandom** 随机选取的数量，随机的个数，具体意义见本文[优化2-巨片随机化][优化2-巨片随机化]

- 性能优化参数，这些参数如果不太清楚，可以先使用默认值，然后逐步在实验中调整，对结果精度影响不大
  - **storageLevel** 缓存策略，数据量小为Memory,数据量大为Memeory_and_Disk
  - **parts** RDD分区数量,用于设置并发
  - **mergeSize** LSH表格缓存个数
- 应用相关参数，这些参数与应用逻辑强相关，
  - **distance** 每个相似对的最大距离小于等于distance
  - **maxSize** 每个样本，最多保留maxSize个相似对



## 结语

这次编写LSH系列博文，对该技术的系统总结。此过程学到了很多，包括论文阅读技巧，基础概率与微积分应用，spark优化等。虽然蛋痛过，但是觉得很值得。该方法应该可以在实际场景中应用到，比如相似用户计算，相似文本识别等，这也是驱使我投入精力来学习和实现该算法的主要动力，希望后面有机会应用到工作中。

## 更新于2017-05-23

最近半年来，应用LSH在好友推荐中，主要用于推荐陌生人，该算法推荐的用户在后续一起活跃上，明显优于对照组。


## 参考文献

* [LSH在欧式空间的应用(1)--碰撞概率分析][1]
* [LSH在欧式空间的应用(2)--工作原理][2]
* [LSH在欧式空间的应用(3)--参数选择][3]
* [StackOverflow spark: How to merge shuffled rdd efficiently?][4]
* [Mining of Massive Datasets,第二版， 3.6.3节](http://www.mmds.org/)
* $E^2$LSH 0.1 User Manual, Alexandr Andoni, Piotr Indyk, June 21, 2005, Section 3.5.2
* (2004)Locality-Sensitive Hashing Scheme Based on p-Stable.pdf
* (2008)Locality-Sensitive Hashing for Finding Nearest Neighbors

[1]: /probability/lsh/2016/09/15/lsh_eulidian_1.html
[2]: /probability/lsh/2016/09/22/lsh_eulidian_2.html
[3]: /probability/lsh/2016/09/23/lsh_eulidian_3.html
[4]: http://stackoverflow.com/a/39806633/1114397
