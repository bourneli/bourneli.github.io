---
layout: post
title:  LSH在欧式空间的应用(3)--参数选择
categories: [probability,LSH]
---

参数选择在LSH中非常重要，直接影响**计算性能**和**准确率**。利用[上篇博文][2]的理论，现在可以轻易解决参数设定的问题。本文通过一个简单的例子，描述如何选择参数，并且提供参考代码。

首先，假设数据源D的每行数据$x\in R^100, 1 \ge x_i \le 0$]。说明这个数据中，最大的距离是$10(=\underbrace{\sqrt{1+1+\cdots+1}}_{100})$。可以任务，$r_1=0.5$是一个比较近的范围，$r_2=9.5$是一个比较远的范围。希望用大于等于0.7的概率使在$r_1$内的数据投射到一起，0.05的概率使相距$r_2$以外的数据不投射到一起。使用下面代码，进行尝试：

{% highlight R %}
# 碰撞函数
root_fun <- function(target) {
  function(c) {
    1-2*pnorm(-1/c) + (2*c/sqrt(2*pi))*(exp(-1/(2*c^2))-1) - target
  }
}

# 碰撞函数反函数
g_i <- function(p) {
  uniroot(root_fun(p),lower = 1e-10, upper = 10, tol = 1e-5)$root
}


# max D = 10
# 设置w
r1 <- 0.5
r2 <- 9.5

p1 <- 0.75
p2_too_small <- 0.05 
{% endhighlight %}



## 参考文献

* [LSH在欧式空间的应用(1)--碰撞概率分析][1]
* [LSH在欧式空间的应用(2)--工作原理][2]
* [Mining of Massive Datasets,第二版， 3.6.3节](http://www.mmds.org/)
* $E^2$LSH 0.1 User Manual, Alexandr Andoni, Piotr Indyk, June 21, 2005, Section 3.5.2
* (2004)Locality-Sensitive Hashing Scheme Based on p-Stable.pdf
* (2008)Locality-Sensitive Hashing for Finding Nearest Neighbors

[1]: lsh_eulidian_1.html
[2]: lsh_eulidian_2.html