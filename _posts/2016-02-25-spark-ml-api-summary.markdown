---
layout: post
title:  Spark ML使用与改造经验分享
date:   2016-02-25 16:20:11 +0800
categories: spark
---

Spark从1.2版开始，引入了ML（ml包）框架。该框架用于规范机器学习算法接口，开发更高层API（如交叉验证，网格搜索），并且保持训练和预测过程中数据清洗的一致性。在1.2版本之前已经实现了一些机器学习算法(mllib包)，为了适应ML框架，避免重复造轮子，spark团队封装了部分1.2版本之前的算法，并集成到ML框架中，但是后续开发的新的算法，基本都是基于ML规范开发。
由于工作原因，从1.5版本开始，接触ML框架。当时ML功能还不太完善，笔者作了一些特定应用场景的改造，这篇文章主要记录笔者对ML框架的理解和改造经验，希望对读者有用。Spark ML框架官方文档参考[这里](http://spark.apache.org/docs/latest/ml-guide.html)。

## 核心概念
ML框架主要将机器学习过程抽象成下面几个对象，笔者感觉抽象得比较准确。其中核心概念是Pipeline(流水线)，

* **DataFrame** 带有schema的[RDD](http://spark.apache.org/docs/latest/programming-guide.html#resilient-distributed-datasets-rdds)，流水线中的数据流。 
* **Transformer** Pipeline中的处理模块，用于处理DataFrame。输入是DataFrame，输出也是DataFrame，比如预测模型。
* **Estimator** Pipeline中的计算模块，基于数据计算模型。输入是DataFrame，输出是Transfomrer，比如训练算法。
* **Pipeline** 串联不同的Transformer和Estimator。
* **Parameter** Transformer和Estimator模块参数的统一接口。


## Pipeline执行流程

ML核心对象组织方式使用了[组合设计模式](https://en.wikipedia.org/wiki/Composite_pattern)。Transformer和Estimator基类是[PipeLineStage](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/Pipeline.scala#L43)，Pipeline是PipelineStage的集合，Pipeline也是PipelineStage。

[Transformer::tranform](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/Transformer.scala#L68)函数输入DataFrame，输出DataFrame。[Estimator::fit](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/Estimator.scala#L65)函数输入DataFrame，输出[Model](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/Model.scala#L30)(带有父Estimator的Transformer)对象。

Pipeline的核心逻辑在[Pipeline::fit](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/Pipeline.scala#L126)函数中。该方法找到最后一个Estimator，然后执行之前的所有Transformer::transform和Estimator::fit(与之后的transform),新的PipelineModel中只有Transformer对象。根据笔者的工作经验，最后的Estimator一般都是分类算法，比如Gradient Boost Tree或Random Forest，而之前的Transformer是一些数据预处理过程，比如变量打包，添加元数据，过滤异常数据等。

## 算法评估改造
笔者工作中主要面对二元分类问题，而在目前1.6版的实现中，[BinaryClassificationEvaluator](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/evaluation/BinaryClassificationEvaluator.scala)只提供了areaUnderPR和areaUnderPR两个指标，而[MulticlassClassificationEvaluator](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/evaluation/MulticlassClassificationEvaluator.scala)却没有提供针对特定标签计算f1,precision,recall等评估指标。所以，基于这两点，笔者实现了一个二元评估对象，结合上面两个类，增加了基于特定的标签指标计算。代码如下：
{% highlight scala %}
class BinaryClassificationEvaluatorT(override val uid: String)
  extends Evaluator with HasPredictionCol with HasLabelCol {
  
  // ...

  override def evaluate(dataset: DataFrame): Double = {
    val schema = dataset.schema
    SchemaUtils.checkColumnType(schema, $(predictionCol), DoubleType)
    SchemaUtils.checkColumnType(schema, $(labelCol), DoubleType)

    val predictionAndLabels = dataset.select($(predictionCol), $(labelCol))
      .map {
        case Row(prediction: Double, label: Double) =>
          (prediction, label)
      }
    val metrics = new MulticlassMetrics(predictionAndLabels)
    val metric = $(metricName) match {
      case "fMeasure" => metrics.fMeasure(label = $(positiveLabel), $(betaForF))
      case "precision" => metrics.precision(label = $(positiveLabel))
      case "recall" => metrics.recall(label = $(positiveLabel))
    }
    metric
  }

   // ...
}
{% endhighlight %}

## 训练过程植入标签比例调整逻辑
由于工作中数据训练样本倾斜非常严重，直接使用原始样本分布，基本上无法得到理想的结果，需要在训练之前调整样本比例。ML提供了统一的接口，可以很容易将此过程封装成Transformer，方便不同场景中复用。这种做法确实可行，但是却违反了ML的接口规范，无法在ML框架中高级接口中执行。

因为截止到1.6.0版本，Transormer和Estimator是[Stateless](http://spark.apache.org/docs/1.6.0/ml-guide.html#properties-of-pipeline-components)(无状态，请在官网[首页](http://spark.apache.org/docs/1.6.0/ml-guide.html#properties-of-pipeline-components)搜索‘stateless’)。但是样本比例调整只需要训练过程中执行（Estimator::fit函数），预测过程无需执行。单独的transformer是不知道当前pipeline是训练状态还是预测状态，导致每次都会调整比例。而预测数据中是没有正负样本标签，导致此过无法执行预测过程。

也许可以根据DataFrame的schema信息，区别是训练状态还是预测状态。但是，如果是在交叉验证，验证的预测过程中，DataFrame是有标签的，此方案行不通。

所以，笔者的做法是将标签比例调整逻辑封装成到一个函数中，然后植入到特性分类器的训练过程中，这样就可以规避上面的问题。因为可以将比例调整过程当做训练算法的一部分，这样就可以完美适配Estimator无状态。缺点是需要修改源代码，成本较高。后面也许可以开发一个通用的适配器，低成本集成到不同分类算法中。

## 参数规范
ML中的Param规范主要将每一种类型的参数封装成对象，并且对常用参数开发了特定的类，比如[HasLabelCol](https://github.com/apache/spark/blob/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/param/shared/sharedParams.scala#L76)，该类用于设置DataFrame中标签列的名称，只需要使用**with**关键字继承该接口即可。每一个参数都可以有set，get和default方法，分别是设置，获取和默认值。参数具体使用细节，可以参考Spark ML中[feature包中的实现代码](https://github.com/apache/spark/tree/branch-1.6/mllib/src/main/scala/org/apache/spark/ml/feature)。

## 交叉验证
从1.2.0版本起，ML框架提供更高阶的K-fold交叉验证类[CrossValidator](https://github.com/apache/spark/blob/branch-1.5/mllib/src/main/scala/org/apache/spark/ml/tuning/CrossValidator.scala)，可能由于计算开销较大（需要计算K次），从1.5版本后，提供了1-fold交叉验证类[TrainValidationSplit](https://github.com/apache/spark/blob/branch-1.5/mllib/src/main/scala/org/apache/spark/ml/tuning/TrainValidationSplit.scala)。目前工作中，主要使用后者，开销较小，弊端是评估结果可能不太稳定。这两个高阶接口都使用网格搜索调优参数，这点非常赞，目前在工作中广泛使用。

## 小结
Spark引入ML框架后，接口的确规范了不少。后续工作中，会借鉴这套接口范接，开发基于业务特定的组件，避免重复造轮子。不过存在一定的潜在风险，如果后面spark ml接口标准改变了，可能会影响到已开发的组件和线上任务，需要谨慎！


 