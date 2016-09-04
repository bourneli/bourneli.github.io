---
layout: post
title:  R glm函数模型系数NA问题
date:   2016-02-26 16:20:11 +0800
categories: R
---

今天用R glm做逻辑回归分析时，遇到模型系数为NA，效果如下

{% highlight text %}

Call:
glm(formula = is_lost ~ ., family = "binomial", data = data)

Deviance Residuals: 
    Min       1Q   Median       3Q      Max  
-1.4928  -1.2366   0.8919   0.9078   6.0179  

Coefficients: (1 not defined because of singularities)
                   Estimate Std. Error z value Pr(>|z|)    
(Intercept)        0.716540   0.011068  64.743  < 2e-16 ***
gold_num          -0.266506   0.052782  -5.049 4.44e-07 ***
ladder_single_num -0.054578   0.003099 -17.613  < 2e-16 ***
ladder_double_num -0.052016   0.009559  -5.442 5.28e-08 ***
room_num          -0.184745   0.008601 -21.480  < 2e-16 ***
robot_num         -0.073728   0.003479 -21.193  < 2e-16 ***
normal_num               NA         NA      NA       NA    
adventure_num     -0.043090   0.001246 -34.571  < 2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 71820  on 52243  degrees of freedom
Residual deviance: 63239  on 52237  degrees of freedom
AIC: 63253

Number of Fisher Scoring iterations: 6
{% endhighlight  %}

问题的原因是**列线性依赖,列线性依赖,列线性依赖**（重要的事情说三遍）。导致有无数种系数组合。R控制台会有**警告**信息，如下

{% highlight command %} 
> lr <- glm(is_lost~., data = data, family='binomial')
Warning message:
glm.fit:拟合機率算出来是数值零或一 
{% endhighlight %}

后来查ETL SQL，发现有个低级错误，导致两个列的数据一样，:-(。Cross Validate中也有同样的[问题和解答](http://stats.stackexchange.com/questions/25804/why-would-r-return-na-as-a-lm-coefficient)，供参考。
