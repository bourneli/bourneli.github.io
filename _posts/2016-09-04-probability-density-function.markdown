---
layout: post
title:  常系数随机变量与对应概率密度函数
categories: [probability]
---


最近在看LSH(Locality Sensitive Hashing)相关的资料，里面有一个欧式空间的分析，涉及到概率密度的线性变化。之前百思不得其解，后来google了相关概率密度的内容后，得到了正确的解法。主要思想就是概率密度函数之间的关系，需要通过分布函数来转换，因为连续概率密度函数上的任一点，概率均为0。


假设随机变量$X \sim D$,D是一个特定分布,其概率密度函数$f_X(x)$。那么随机变量$Y=cX$(c为常数且$c \gt 0$)的概率密度函数如何用$f_X(x)$表示。既然$\frac{Y}{c}$与X同分布，那么是否有$f_Y=f_X(\frac{Y}{c})$。这个的想法是**错误的**，之前就在这里卡壳。需要考虑到分布函数，也就是真实的概率，而不是直接套用概率密度函数。

假设X的分为函数为$F_X(x)$,那么分布函数与概率密度的关系：

$$
	F_X(x)=Pr(X \le x)=\int_{-\infty}^x f_X(t)dt 
	\Rightarrow f_X(x) = \frac{d F_X(x)}{dx} \qquad(1)
$$


所以参考(1)，对于Y的分布函数，有如下

$$
	F_Y(y) = Pr(Y \le y) 
           = Pr(cX \le y) 
           = Pr(X \le \frac{y}{c}) 
           = F_X(\frac{y}{c})  \qquad (2)
$$

对(2)求导，得到Y的概率密度函数（需要利用复合函数求导），

$$
	f_Y(y) = \frac{d F_X(\frac{y}{c})}{dy} =  \frac{1}{c} f_X(\frac{y}{c}) \qquad(3)
$$

最后(3)就是随机变量$Y=cX$的概率密度函数。

**小结**

1. 概率密度的关系需要借助分布函数
2. 概率密度函数上任意一点概率为0
3. 常系数随机变量的概率密度关系为公式(3)


**参考资料**

* [一个网友的概率公式证明](http://blog.sina.com.cn/s/blog_67914f2901019p3v.html)
* [概率密度乘法Wiki](https://en.wikipedia.org/wiki/Probability_density_function)
* [Stack Exchange先关问题](http://math.stackexchange.com/a/275668/261790)