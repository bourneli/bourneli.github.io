---
layout: post
title:  "R高效迭代data.frame"
categories: [R]
---

使用R时，常需要迭代data.frame的每行数据，如果操作不当，效率会大打折扣。看看下面的代码

{% highlight R %}
n <- 1e5
dd <- data.frame(x=seq(1.0,n,by=1),y=seq(1.0,n,by=1))

add_col <- function(dd) {
  sum1 <- 0.0
  sum2 <- 0.0
  for(i in 1:nrow(dd)) {
    sum1 <- sum1 + dd[i,1]
    sum2 <- sum2 + dd[i,2]
  }
  
  c(sum1, sum2)
}

# 向量化
system.time(c(sum(dd$x),sum(dd$y))) 
# 内置函数
system.time(apply(dd,2,sum)) 
# 逐行迭代
system.time(add_col(dd)) 
{% endhighlight %}

输出结果如下：

{% highlight raw %}
> system.time(c(sum(dd$x),sum(dd$y)))
用户 系统 流逝 
   0    0    0 
> system.time(apply(dd,2,sum))
用户 系统 流逝 
0.00 0.02 0.01 
> system.time(add_col(dd))
用户 系统 流逝 
4.39 0.01 4.51 
{% endhighlight%}

可以看到，逐行迭代效率最低，向量化效率最高，内置函数稍微比向量化方法差一点(可能由于函数调用会有一定耗时)。原因是R在逐行处理data.frame时，会产生[冗余的内存拷贝][1]，导致效率降低，而向量操作不会。所以，好的习惯是[能用向量操作，就不要逐行操作](2)，只有那些迭代次数不确定的时候，才需要逐行操作。


## 参考
* [Why is the time complexity of this loop non-linear?][1]
* [In R, how do you loop over the rows of a data frame really fast?][2]


<!-- 多余拷贝导致效率低 -->
[1]:http://stackoverflow.com/a/34826252/1114397
<!-- 向量操作习惯 -->
[2]:http://stackoverflow.com/questions/3337533/in-r-how-do-you-loop-over-the-rows-of-a-data-frame-really-fast/3337622#3337622