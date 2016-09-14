---
layout: post
title:  LSH在欧式空间的应用(1)--误差分析
categories: [probability,LSH]
---

最近中秋休假，终于有时间静下来心来研究欧式空间的LSH(Locality Sensitive Hashing)的误差分析。打算基于欧式空间LSH，发表一个系列博文，详细描述工作原理和实践经验。误差分析这块，需要的数学相对集中，虽然都是大学工科数学，但是如果长期不用，捡起来还是比较费时间。笔者将尽力详细介绍整个推导过程，希望读者可以领会其中数学的美感，并在实践中验证相关理论。

## LSH做什么

LSH是一种最近邻居的估算方法，主要思路是设计一些hash函数，将那些比较类似的对象hash到一个单位，距离较大的hash到不同的单位里。hash过程相当于一种过滤机制，缩小检索范围。最近邻居蛮力计算方法需要的时间复杂度$O(n^2)$，而使用LSH，可以在保证错误率很低(比如5%以下)的情况下，时间复杂度降到$O(n)$。在不需要准确最近邻居的应用场景，可以大规模应用，比如$n$达到亿级甚至更多。笔者的工作中，此类场景比较常见，这也是为什么值得投入时间研究该技术的主要动力。


## 欧式空间的LSH

LSH的技术比较成熟，应用最多还是在集合上，结合minhash算法，得到漂亮的理论推导和不错应用效果。但是笔者常见的应用场景主要的数据对象在欧式空间中，所以本系列博文着重介绍欧式空间的LSH相关的内容。欧式空间中，LSH的hash算法的思路是将n维向量随机射到一个向量，使用向量点乘，由于投射向量不是单位向量，所以严格意义上不能称之为投影。投射hash算法如下:

$$
	h(v) = \left\lfloor \frac{a \bullet v + b}{w} \right\rfloor
$$

其中$b(\in [0,w])$是随机量，$a (\in R^n,a_i \sim N(0,1))$，是被投射的向量。投射完后，需要设置一个固定长度为$w$的参数，将向量严格的划分为不同的单位，投射到相同单位的向量就认为比较近。所以，$w$的设置十分重要，如果设置太大，比较远的对象也设hash到一个单位里，无法做到过滤的效果；如果太小，即使很近的对象也到不了一个桶里面，导致找不到相领的对象。

## 概率分析

设向量$p,q \in R^n$，并且$u=\lVert p-q\rVert_2$，p与q投射到任意向量$a$的概率如下，

$$
	p(u) = Pr(h(p) = h(q)) = 2\int_{0}^{w}\frac{1}{u}f(\frac{t}{u})(1-\frac{t}{w})dt
$$

上面概率公式很突然，先别慌是怎么过来的，后面慢慢道来，我们先直奔主题--得到该概率的解析解。这样就可以观察w,u与概率的关系，控制误差。$f(x)$是稳定分布的概率密度函数，该分布只有在欧式距离和曼哈顿距离才有解析解，否则没有。好在我们关心的是欧式空间，所以可以得到解析解。欧式空间中，$f(x)(=\frac{1}{\sqrt{2\pi}}e^{-\frac{x^2}{2}})$是标准正在分布的概率密度函数。完整推导如下，


$$
\begin{align}
	p(u) = f(u,w) &= 2(\int_{0}^{w}\frac{1}{u}f(\frac{t}{u})dt - \int_{0}^{w}\frac{1}{u}f(\frac{t}{u})\frac{t}{w}dt) \\
				  &= 2(\int_{0}^{w}f(\frac{t}{u})d\frac{t}{u} - \int_{0}^{w}\frac{1}{u\sqrt{2\pi}}e^{-\frac{t^2}{2u^2}}\frac{t}{w}dt) \\
				  &= 2(\int_{0}^{\frac{w}{u}}f(x)dx - \frac{-u}{\sqrt{2\pi}w}\int_{0}^{w}e^{-\frac{t^2}{2u^2}}d(-\frac{t^2}{2u^2})) \\
				  &= 2(\frac{1}{2} - F(-\frac{w}{u}) + \frac{u}{\sqrt{2\pi}w}e^{-\frac{t^2}{2u^2}}|^w_0) \\
				  &= 2(\frac{1}{2} - F(-\frac{w}{u}) + \frac{u}{\sqrt{2\pi}w}(e^{-\frac{w^2}{2u^2}}-1)) \\
		 
\end{align}
$$
 
上面的概率只与$u$，$w$的比例有关，令$c=\frac{u}{w}$，
 
$$
	g(c) = 1 - 2F(-\frac{1}{c}) + \sqrt{\frac{2}{\pi}}c(e^{-\frac{1}{2c^2}}-1) 
$$

上面推导的结果只与w,u的比例有关，这一点很重要！这样无需关系数据真实的单位，比如某个特征度量在线时长，单位用"秒"；另外一个特征衡量付费能力，单位使用“元”。都可以将这些特征归一化到0-1之间，改变u的取值范围，然后使用相对比例设置w。使用R观察曲线趋势，

{% highlight R %}
pr <- function(c) {
  1-2*pnorm(-1/c) + (2*c/sqrt(2*pi))*(exp(-1/(2*c^2))-1)
}

x <- seq(0,10,by=0.01)
d <- data.frame(x=x,y=sapply(x,pr))

require(ggplot2)
text_size <- element_text(size = 19)
(qplot(x,100*y,data=d, geom="line") + xlab("C") + ylab("概率%") + ggtitle("") 
+ theme(axis.text = text_size,  
        axis.title = text_size))
{% endhighlight %}

<div align='center'>
	<img src="\img\prob_with_c_lsh.png"/>
</div>


根据曲线，上述函数使个减函数，也就是比例越大，聚到一起的概率越低，理论与直觉一致。可以通过设定u，然后通过曲线，得到对应w。上述通过实践角度观察，有点不太严谨，所以下面还是通过计算概率，计算导数的解析解：

$$
	\begin{align}
		g\prime(c) &= -2f(-\frac{1}{c})(-1)(-1)c^{-2} + \sqrt{\frac{2}{\pi}}(e^{-\frac{1}{2c^2}}-1) + \sqrt{\frac{2}{\pi}}c(e^{-\frac{1}{2c^2}}(-\frac{1}{2})(-2)c^{-3}) \\
				   &= -\frac{2}{c^2}f(-\frac{1}{c}) + \sqrt{\frac{2}{\pi}}(e^{-\frac{1}{2c^2}}-1) + \sqrt{\frac{2}{\pi}}e^{-\frac{1}{2c^2}}c^{-2} \\
				   &= -\frac{2}{c^2}\frac{1}{\sqrt{2\pi}}e^{-\frac{1}{2c^2}} + \sqrt{\frac{2}{\pi}}(e^{-\frac{1}{2c^2}}-1) + \sqrt{\frac{2}{\pi}}e^{-\frac{1}{2c^2}}c^{-2} \\
				   &= \sqrt{\frac{2}{\pi}}(e^{-\frac{1}{2c^2}}-1) < 0
	\end{align}
$$

其导数严格小于0，证实了上述曲线确实严格下降。

## 概率公式推导过程

稳定分布是什么
期望做类比
