---
layout: post
title: 【译】使用R和Caret包处理类别不平衡分类问题。
categories: [R,Translation]
---



*原文是[Handling Class Imbalance with R and Caret – An Introduction](https://www.r-bloggers.com/handling-class-imbalance-with-r-and-caret-an-introduction/)。该文章使用R的Caret包非常简洁的处理非平衡分类问题。这类题在笔者的工作中十分常见，所以记录下来，方便后面回顾。*



现实世界的分类任务中，处理高度不平衡的分类问题非常具有挑战性。本文由两部分组成，主要介绍一些R和caret相关技巧。这些技巧可以在高度不平衡的分类场景下提高模型预测性能。本文是第一部分，以通用视角介绍如何在实战中使用这些技巧。[第二部](https://www.r-bloggers.com/handling-class-imbalance-with-r-and-caret-caveats-when-using-the-auc/)分着重介绍这些技巧相关的“坑”，这些坑十分容易在实战中遇见，需要谨记。



## 分类评估指标

当训练完一个分类器后，你需要确定这个分类器是否有效。现存有很多指标用来评估分类器性能，可以大致将它们分类为两类：

1. 阈值相关：准确率(Accuracy)，精度(Precision),召回(Recall)和F1分(F1 score)全部属于此类。此类方法需要使用特定阈值，计算混淆矩阵，然后根据此矩阵计算上面所有的指标。这类指标在非平衡的问题上无法有效的评估模型性能，因为统计软件一般使用默认0.5作为阈值，这种不适当的设定会导致大多数样本被归纳到主要的类别中。
2. 阈值无关：ROC曲线下的面积(AUC)属于此类。它将真阳性看做是假阳性的函数，它们会随着切分阈值变化而变化。从概率的角度来看，此指标描述了**随机从正样本和负样本各取一个样本，然后使用模型对其打分，正样本得分大于负样本得分**的概率。(译者注:详细数学推导，可以参考笔者博文[AUC的数学解释](http://bourneli.github.io/ml/2018/09/24/roc-review.html))



## 提高非平衡数据下模型性能的方法

下面会介绍一些常用技巧处理非平衡分类问题的技巧，但是这个列表并不会详细描述所有细节。简短起见，本文仅提供简短的概要。如果希望了解这些技术背后的细节，作者强烈建议阅读[这篇博文](https://www.svds.com/learning-imbalanced-classes/)。

* 样本加权：将更多的权重放到更少的类别的样本上，这样当模型在少量类别上犯错后，会得到更多惩罚
* 向下采样：在主要类别的样本中，随机移除一些样本。
* 向上采样：在较少类别的样本中，随机重复一些样本。
* SMOTE：向下采样，同时在较少的样本中，使用插值的方法随机生成新的样本。

需要着重指出的是，这类加权和抽样技巧对阈值依赖的指标有非常显著的影响，因为他们人工的将阈值朝着最优的地方移动。阈值无关的指标仍然会被这类方法改进，但是效果并没有声称的那样显著。



## 模拟设置

使用caret包中的**twoClassSim**函数模拟类别不平衡。我们将训练集和预测集分开模拟，每个数据集有5000个样本。除此之外，我们添加了20个有意义的变量，以及10个噪音变量。参数intercept控制整体不平衡的程度，最终数据集的正负样本比例为50:1。

{% highlight R %}

library(dplyr) # for data manipulation
library(caret) # for model-building
library(DMwR) # for smote implementation
library(purrr) # for functional programming (map)
library(pROC) # for AUC calculations

set.seed(2969)

imbal_train <- twoClassSim(5000,
​                           intercept = -25,
​                           linearVars = 20,
​                           noiseVars = 10)

imbal_test  <- twoClassSim(5000,
​                           intercept = -25,
​                           linearVars = 20,
​                           noiseVars = 10)

prop.table(table(imbal_train$Class))

## 
## Class1 Class2 
## 0.9796 0.0204

{% endhighlight %}



## 初始结果

gbm模型用于给这些数据建模，因为它可以处理模拟数据中潜在的交互和非线性部分。模型超参数，在训练数据上使用5折交叉验证调节。为了避免确定最后的阈值，AUC被用于评估模型的效果。因为需要交叉验证，下面的代码可能需要一点执行时间，所以可以减少参数repeats来加速实验过程，或者使用函数trainControl中的参数**verboseIter=TRUE**来追踪整个过程。



{% highlight R %}


# Set up control function for training

ctrl <- trainControl(method = "repeatedcv",
​                     number = 10,
​                     repeats = 5,
​                     summaryFunction = twoClassSummary,
​                     classProbs = TRUE)

# Build a standard classifier using a gradient boosted machine

set.seed(5627)

orig_fit <- train(Class ~ .,
​                  data = imbal_train,
​                  method = "gbm",
​                  verbose = FALSE,
​                  metric = "ROC",
​                  trControl = ctrl)

# Build custom AUC function to extract AUC
# from the caret model object

test_roc <- function(model, data) {

  roc(data$Class,
​      predict(model, data, type = "prob")[, "Class2"])

}

orig_fit %>%
  test_roc(data = imbal_test) %>%
  auc()
## Area under the curve: 0.9575

{% endhighlight %}

整体而言，此模型的AUC为0.96，效果还不错。我们可以使用上面提到的技巧提高AUC吗？



## 使用加权或采样方法处理非平衡分类

加权和采样方法在caret包中使用非常简单。使用train函数中的weights参数，即可将权重加到建模过程中（此方法必须支持weights参数，完整列表参考[这里](https://redirect.viglink.com/?format=go&jsonp=vglnk_154149705618827&key=949efb41171ac6ec1bf7f206d57e90b8&libId=jo5glaug01021u9s000DAvjb6gjc04wa7&loc=https%3A%2F%2Fwww.r-bloggers.com%2Fhandling-class-imbalance-with-r-and-caret-an-introduction%2F&v=1&out=https%3A%2F%2Ftopepo.github.io%2Fcaret%2Ftrain-models-by-tag.html%23Accepts_Case_Weights&title=Handling%20Class%20Imbalance%20with%20R%20and%20Caret%20%E2%80%93%20An%20Introduction%20%7C%20R-bloggers&txt=here)）。采样方法可以使用trainControl函数中的sampling参数设置。为了得到一致的结果，seeds必须保持不变。

需要注意，使用采样方法时，只有训练数据需要采样，测试数据不能采样。这意味着需要在交叉验证过层中使用采样技巧。caret的作者Max Kuhn给出了一个非常好的例子，描述如果不注意这个细节将会发生什么，具体可以参考[这里](https://redirect.viglink.com/?format=go&jsonp=vglnk_154149749080530&key=949efb41171ac6ec1bf7f206d57e90b8&libId=jo5glaug01021u9s000DAvjb6gjc04wa7&loc=https%3A%2F%2Fwww.r-bloggers.com%2Fhandling-class-imbalance-with-r-and-caret-an-introduction%2F&v=1&out=https%3A%2F%2Ftopepo.github.io%2Fcaret%2Fsubsampling-for-class-imbalances.html&title=Handling%20Class%20Imbalance%20with%20R%20and%20Caret%20%E2%80%93%20An%20Introduction%20%7C%20R-bloggers&txt=this%20caret%20documentation)。下面代码演示如何正确使用采样技巧。



{% highlight R %}

# Create model weights (they sum to one)

model_weights <- ifelse(imbal_train$Class == "Class1",
​                        (1/table(imbal_train$Class)[1]) * 0.5,
​                        (1/table(imbal_train$Class)[2]) * 0.5)

# Use the same seed to ensure same cross-validation splits

ctrl$seeds <- orig_fit$control$seeds

# Build weighted model

weighted_fit <- train(Class ~ .,
​                      data = imbal_train,
​                      method = "gbm",
​                      verbose = FALSE,
​                      weights = model_weights,
​                      metric = "ROC",
​                      trControl = ctrl)

# Build down-sampled model

ctrl$sampling <- "down"

down_fit <- train(Class ~ .,
​                  data = imbal_train,
​                  method = "gbm",
​                  verbose = FALSE,
​                  metric = "ROC",
​                  trControl = ctrl)

# Build up-sampled model

ctrl$sampling <- "up"

up_fit <- train(Class ~ .,
​                data = imbal_train,
​                method = "gbm",
​                verbose = FALSE,
​                metric = "ROC",
​                trControl = ctrl)

# Build smote model

ctrl$sampling <- "smote"

smote_fit <- train(Class ~ .,
​                   data = imbal_train,
​                   method = "gbm",
​                   verbose = FALSE,
​                   metric = "ROC",
​                   trControl = ctrl)

{% endhighlight %}

测试集的AUC表明，相比于原始方法，权重方法或采样方法有明显的提升。

{% highlight R %}
# Examine results for test set

model_list <- list(original = orig_fit,
​                   weighted = weighted_fit,
​                   down = down_fit,
​                   up = up_fit,
​                   SMOTE = smote_fit)

model_list_roc <- model_list %>%
  map(test_roc, data = imbal_test)

model_list_roc %>%
  map(auc)

## $original
## Area under the curve: 0.9575
## 
## $weighted
## Area under the curve: 0.9804
## 
## $down
## Area under the curve: 0.9705
## 
## $up
## Area under the curve: 0.9759
## 
## $SMOTE
## Area under the curve: 0.976
{% endhighlight %}

我们可以通过ROC曲线来观察，加权和采样方法在哪些地方优于原始方法。我们可以看出加权模型在整个ROC曲线上都优于其他方法，同时原始方法的假阳率在0~0.25之间明显比其他方法差。这表明其他模型在早期有更好的检索数字。即当样本被模型计算为较少类型的高概率样本时，其他算法可以更准确的确定真阳样本。

{% highlight R %}

results_list_roc <- list(NA)
num_mod <- 1

for(the_roc in model_list_roc){

  results_list_roc[[num_mod]] <- 
​    data_frame(tpr = the_roc$sensitivities,
​               fpr = 1 - the_roc$specificities,
​               model = names(model_list)[num_mod])

  num_mod <- num_mod + 1

}

results_df_roc <- bind_rows(results_list_roc)

# Plot ROC curve for all 5 models

custom_col <- c("#000000", "#009E73", "#0072B2", "#D55E00", "#CC79A7")

ggplot(aes(x = fpr,  y = tpr, group = model), data = results_df_roc) +
  geom_line(aes(color = model), size = 1) +
  scale_color_manual(values = custom_col) +
  geom_abline(intercept = 0, slope = 1, color = "gray", size = 1) +
  theme_bw(base_size = 18)

{% endhighlight %}



![](F:\my.git\github\bourneli.github.io\img\roc-auc-in-r.png)



## 最后的思考

上面的文章中，我总结出了一些步骤用于改进非平衡分类问题的性能。虽然在模拟数据上，加权方法优于采样方法，但是这并不表名在所有数据下都是这样。因此，在你的数据集上，必须尝试不同的方法确认最优的方法。我发现在很多不同的非平衡数据集上，采样或加权方法并没有显著的AUC提升。[下偏博文](https://www.r-bloggers.com/handling-class-imbalance-with-r-and-caret-caveats-when-using-the-auc/)，我将介绍一些使用AUC的坑，以及其他一些更有意义的指标。请继续收看！