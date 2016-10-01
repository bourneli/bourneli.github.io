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

## 参考代码

下面的实现是离线计算版本，在线计算版本需要后台服务器开发，无法使用spark实现。但是如果掌握了整个LSH原理，在线版本不会太困难。

{% highlight scala %}
import org.apache.spark.rdd.RDD
import org.apache.spark.storage.StorageLevel
import breeze.linalg.{Vector, DenseVector, norm}
import breeze.stats.distributions.{Gaussian, Uniform}
import org.apache.spark.HashPartitioner
import scala.reflect.ClassTag
import scala.util.Random
import scala.collection.mutable.ArrayBuffer

/**
  * @param lshWidth     映射后的桶的宽度
  * @param bucketWidth  桶内，相同lsh的个数
  * @param bucketSize   桶的个数
  * @param storageLevel 缓存策略
  * @param parts        分区数量
  * @param lshUpBound   lsh上界，没个key中的候选对象个数高于上界，随机选取
  * @param lshRandom    随机选取的数量
  * @param mergeSize    LSH表格缓存个数
  */
class LSHForE2(
                  distance: Double,
                  lshWidth: Double,
                  bucketWidth: Int = 10,
                  bucketSize: Int = 200,
                  storageLevel: StorageLevel = StorageLevel.MEMORY_AND_DISK,
                  parts: Int = 1000,
                  lshUpBound: Int = 1000,
                  lshRandom: Int = 10,
                  mergeSize: Int = 8) extends Serializable {

    assert(lshWidth > 0, s"Width = $lshWidth must be great than 0")
    assert(bucketWidth > 0, s"k = $bucketWidth must be great than 0")
    assert(bucketSize > 0, s"l = $bucketSize must be great than 0")
    assert(lshRandom < lshUpBound,
        s"lshRansom $lshRandom should be less than lshUpBound $lshUpBound")

    /**
      *
      * @param raw
      * @return
      */
    def lsh[K: ClassTag](raw: RDD[(K, Vector[Double])]): RDD[(K, K, Double)] = {

        // 分区加缓存，用于后面的join
        val data = raw.repartition(parts).persist(storageLevel)
        val nf = java.text.NumberFormat.getIntegerInstance

        // 特征长度
        val vectorWidth = data.first._2.length
        assert(vectorWidth > 0, s"Vector width is $vectorWidth")

        // 生成所有的hash参数
        val normal = Gaussian(0, 1) // 生成随机投影向量
        val uniform = Uniform(0, lshWidth) // 生成随机偏差
        val hashSeq: IndexedSeq[(Vector[Double], Double)] = for (i <- 0 until bucketWidth * bucketSize)
                yield (DenseVector(normal.sample(vectorWidth).toArray), uniform.sample)
        val hashFunctionsBC = data.sparkContext.broadcast(hashSeq)

        // 计算lsh值
        val lshRDD = data.map({ case (id, vector) =>
            val hashFunctions = hashFunctionsBC.value
            val hashValues = hashFunctions.map({
                case (project: Vector[Double], offset: Double) =>
                    (((project dot vector) + offset) / lshWidth).floor.toLong
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
        val bucketBuffer = ArrayBuffer[RDD[(K, K)]]()
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
            val rst = idGroupRDD.flatMap(neighbors => {
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
                        val randomRhs = (for (i <- 0 until lshRandom)
                            yield neighbors(rand.nextInt(neighbors.length)))
                            .distinct.map(x => (key, x))
                        allRandomPairs.appendAll(randomRhs)
                    }
                    allRandomPairs.toArray[(K, K)]
                }
            }).partitionBy(new HashPartitioner(parts)).persist(storageLevel)

            val currentResultSize = rst.count
            val end = System.currentTimeMillis
            val timeCost = (end - begin) / 1000d
            println(s"=======Rount $i, Result Size = %s,Time Cost: $timeCost s"
                .format(nf.format(currentResultSize)))
            idGroupRDD.unpersist(false)
            bucketBuffer.append(rst)

            // 合并中间表格，节省空间
            if ((bucketBuffer.length >= mergeSize
                || i == bucketSize - 1)
                && bucketBuffer.length > 1) {
                val mergeBegin = System.currentTimeMillis

                val merged = data.sparkContext.union(bucketBuffer)
                    .distinct(parts).partitionBy(new HashPartitioner(parts)).persist(storageLevel)
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
        val distinctLshPairs = bucketBuffer(0).persist(storageLevel)
        bucketBuffer.clear
        val distinctLshParisSize = distinctLshPairs.count
        println(s"Distinct Lsh Pair Size %s, rate = %.3f".format(
            nf.format(distinctLshParisSize), distinctLshParisSize / allPairSize))

        // 关联原始数据，计算相似度
        val finalResult = distinctLshPairs.join(data)
            .map({ case (leftId, (rightId, leftValue)) => (rightId, (leftId, leftValue)) })
            .partitionBy(new HashPartitioner(parts))
            .join(data)
            .map({ case (rightId, ((leftId, leftValue), rightValue)) =>
                (leftId, rightId, norm(leftValue - rightValue, 2))
            })
            .filter(_._3 <= distance)
            .persist(storageLevel)
        val finalResultCount = finalResult.count
        println(s"final result count %s, rate = %.3f".format(
            nf.format(finalResultCount), finalResultCount / allPairSize))
        distinctLshPairs.unpersist(false)

        finalResult
    }
}
{% endhighlight %}

## 结语

这次编写LSH系列博文，对该技术的系统总结。此过程学到了很多，包括论文阅读技巧，基础概率与微积分应用，spark优化等。虽然蛋痛过，但是觉得很值得。该方法应该可以在实际场景中应用到，比如相似用户计算，相似文本识别等，这也是驱使我投入精力来学习和实现该算法的主要动力，希望后面有机会应用到工作中。


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