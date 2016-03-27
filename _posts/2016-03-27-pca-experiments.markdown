---
layout: post
title:  PCA实验记要
categories: [linear-algebra, statistic, MVA, R]
---

PCA主要用于简化数据，去掉冗余部分（线性相关）。前提条件是数据中没有太多**异类数据**，否则会严重影响效果，因为异类数据会导致不相关的变量相关系数变高。使用之前，一般会剔除或限制异类数据。并且，原始数据中，需要有线性相关的部分，如果每个变量都是线性无关的，那么PCA基本上也没有什么作用。PCA简化后，可以用于数据可视化，方便数据解读。下面的试验，演示PCA的一些特性，便于理解，后面的试验全部基于R语言。


## 原始数据
随机生成1000个记录，每个记录有三个变量，都是基于正太分布随机生成，它们线性独立
{% highlight R %}
## 原始数据
n <- 1000
set.seed(4546576)
data <- data.frame(x1 = rnorm(n), 
                   x2 = rnorm(n, mean=12,sd=5), 
                   x3 = 3*rnorm(n, mean=-1,sd=3.5))
cor(data)
pairs(data,pch=19)
{% endhighlight %}
生成的散点图中，可以发现都是水平的椭圆，并且关联系数基本为0，说明变量之间是线性无关的。这三个变量也是数据的“主成份”。相关系数如下：
{% highlight raw %}
       x1      x2     x3
x1 1.0000  0.0027  0.015
x2 0.0027  1.0000 -0.028
x3 0.0148 -0.0283  1.000
{% endhighlight %}


## 衍生数据 
衍生数据基于主成份的线性组合，然后添加一些随机误差，避免完全的线性组合。可以发现，无论如何添加线性组合，只要进行适当处理，最后出来的结果都是前三个主成份可以表达99%+。

{% highlight R %}
## 原始数据
derive_data <- cbind(data,
                     x4=10*jitter(data$x1), 
                     x5=jitter(data$x2+0.6*data$x3),
                     x6=12*jitter(data$x1+data$x2),
                     x7=16*jitter(data$x1),
                     x8=7*jitter(data$x1))
cor(derive_data)
{% endhighlight%}
相关系数上可以看到一定的关系，
{% highlight raw %}
        x1      x2     x3     x4    x5     x6     x7     x8
x1 1.0000  0.0027  0.015 1.0000 0.014  0.202 1.0000 1.0000
x2 0.0027  1.0000 -0.028 0.0027 0.600  0.980 0.0027 0.0027
x3 0.0148 -0.0283  1.000 0.0148 0.783 -0.025 0.0148 0.0148
x4 1.0000  0.0027  0.015 1.0000 0.014  0.202 1.0000 1.0000
x5 0.0136  0.5997  0.783 0.0136 1.000  0.590 0.0136 0.0136
x6 0.2017  0.9800 -0.025 0.2017 0.590  1.000 0.2017 0.2017
x7 1.0000  0.0027  0.015 1.0000 0.014  0.202 1.0000 1.0000
x8 1.0000  0.0027  0.015 1.0000 0.014  0.202 1.0000 1.0000
{% endhighlight %}



## 实验1：直接PCA
PCA计算方法是将协方差矩阵对角化，原理是找到一个线性转换矩阵，将原始数据转换到一个新的线性空间，使得其方差最大。因为方差越大，信息越大。但是由于异类数据也会生成非常大的方差，所以一般需要剔除掉异类数据（比如异类值全部设置为最大/小限制）。
{% highlight R%}
m1 <- princomp(derive_data)
summary(m1)
plot(m1)
{% endhighlight%}
根据图像，可以发现主成份是前三个。PCA模型信息如下：
{% highlight raw %}
> summary(m1)
Importance of components:
                       Comp.1 Comp.2 Comp.3  Comp.4  Comp.5  Comp.6  Comp.7  Comp.8
Standard deviation      60.17 19.534 12.156 1.7e-03 1.4e-03 9.2e-04 6.7e-04 6.7e-05
Proportion of Variance   0.87  0.092  0.036 6.7e-10 4.7e-10 2.0e-10 1.1e-10 1.1e-12
Cumulative Proportion    0.87  0.964  1.000 1.0e+00 1.0e+00 1.0e+00 1.0e+00 1.0e+00
{% endhighlight %}


## 试验2：先将变量均值变化为0，然后PCA
{% highlight R%}
data_zero_mean <- as.data.frame(scale(derive_data, 
                                      center=T,
                                      scale=F))
model_zero_mean <- princomp(data_zero_mean)
plot(model_zero_mean)
summary(model_zero_mean)
{% endhighlight%}
根据之前的[协方差矩阵推导](linear-algebra/statistic/mva/2016/03/20/covariance-linear-algebra-expression.html)，发现协方差矩阵其实是使用等幂矩阵将数据矩阵X转换到一个均值为0的空间中，然后相乘得到。所以，我们的原始数据在PCA之前处理均值为0，得到的结果和直接使用PCA一致。PCA模型信息如下：
{% highlight raw %}
> summary(model_zero_mean)
Importance of components:
                       Comp.1 Comp.2 Comp.3  Comp.4  Comp.5  Comp.6  Comp.7  Comp.8
Standard deviation      60.17 19.534 12.156 1.7e-03 1.4e-03 9.2e-04 6.7e-04 6.7e-05
Proportion of Variance   0.87  0.092  0.036 6.7e-10 4.7e-10 2.0e-10 1.1e-10 1.1e-12
Cumulative Proportion    0.87  0.964  1.000 1.0e+00 1.0e+00 1.0e+00 1.0e+00 1.0e+00
{% endhighlight %}

## 实验3：先正规化，然后PCA
由于不同变量的单位不同，导致一些单位较大的变量会主导整个主成份分布，数值较小的独立变量会被掩盖，所以需要将每个变量处理成相同的单位，然后PCA，下面将所有变量转成标准正在分布的Z值，
{% highlight R%}
data_normal <- as.data.frame(scale(derive_data, 
                                  center=T,
                                  scale=T))
model_normal <- princomp(data_normal)
plot(model_normal)
summary(model_normal)
{% endhighlight%}
上面的PCA分布，可以发现第一主城分比之前低很多。

## 试验4：基于相关系数PCA
将数据处理成Z值后再PCA有个等价的简化处理，即直接对角化关联矩阵R，而不是协方差矩阵S，R与S的关系参考[这里](linear-algebra/statistic/mva/2016/03/20/covariance-linear-algebra-expression.html)。因为R是由方差为1的变量计算协方差矩阵得到，而z值的处理过程正是式每个变量的方差为1（减均值对PCA结果没有影响），所以两者效果等价，但是关联矩阵计算效率明显高于z值预处理。
{% highlight R%}
model_corr <- princomp(covmat = cor(derive_data))
summary(model_corr)
plot(model_corr)
{% endhighlight%}
可以发现，两者的效果完全一致。

## 实现5：先01正规化，然后PCA
有时候，用正规化z值处理不太适合，那么使用01正规化，也是一个不错的选择，或者log正规化也可以，这里演示01正规化
{% highlight R%}
minMaxScale <- function(data) {
  max_list <- apply(data,2,max)
  min_list <- apply(data,2,min)
  range_list <- max_list - min_list
  
  scale_data <- data
  for(i in 1:ncol(scale_data)) {
    scale_data[,i] <- (scale_data[,i] - min_list[i])/range_list[i]
  }
  return(scale_data)
}
scale_data_2 <- minMaxScale(derive_data)
m4 <- princomp(scale_data_2)
summary(m4)
plot(m4)
{% endhighlight%}
得到的分布与z值正规化略有不同，但是前三个成分仍然是主成份。


## 总结
PCA用于剔除线性依赖数据，但是计算之前，需要处理有异类数据和归一化变量单位。归一化方法有很多，比如01归一化，log，z-值。z-值归一化的等价方法是关联矩阵对角化，可以极大提高计算效率。


## 实验脚本
直接将下面数据放到R中即可执行，推荐使用RStudio。
{% highlight R%}
## 原始数据
n <- 1000
set.seed(4546576)
data <- data.frame(x1 = rnorm(n), 
                   x2 = rnorm(n, mean=12,sd=5), 
                   x3 = 3*rnorm(n, mean=-1,sd=3.5))
cor(data)
pairs(data,pch=19)


## 衍生数据
derive_data <- cbind(data,
                     x4=10*jitter(data$x1), 
                     x5=jitter(data$x2+0.6*data$x3),
                     x6=12*jitter(data$x1+data$x2),
                     x7=16*jitter(data$x1),
                     x8=7*jitter(data$x1))
cor(derive_data)



# 方法：直接pca
m1 <- princomp(derive_data)
summary(m1)
plot(m1)

# 方法：使用关联系数
model_corr <- princomp(covmat = cor(derive_data))
summary(model_corr)
plot(model_corr)

# 方法：标准正规化，效果与关联系数一致
data_normal <- as.data.frame(scale(derive_data, 
                                  center=T,
                                  scale=T))
model_normal <- princomp(data_normal)
plot(model_normal)
summary(model_normal)

# 方法：仅将平均设置为1 结果与方案1一致
data_zero_mean <- as.data.frame(scale(derive_data, 
                                      center=T,
                                      scale=F))
model_zero_mean <- princomp(data_zero_mean)
plot(model_zero_mean)
summary(model_zero_mean)

# 方案4 01正规化
minMaxScale <- function(data) {
  max_list <- apply(data,2,max)
  min_list <- apply(data,2,min)
  range_list <- max_list - min_list
  
  scale_data <- data
  for(i in 1:ncol(scale_data)) {
    scale_data[,i] <- (scale_data[,i] - min_list[i])/range_list[i]
  }
  return(scale_data)
}
scale_data_2 <- minMaxScale(derive_data)
m4 <- princomp(scale_data_2)
summary(m4)
plot(m4)
{% endhighlight %}

