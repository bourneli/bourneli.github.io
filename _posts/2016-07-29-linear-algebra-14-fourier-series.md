---
layout: post
title:  "线代随笔14-正交向量到傅里叶级数"
categories: [linear-algebra]
---

## 正交基
傅里叶级数利用了正交基表示向量的思想，使用正交函数表示任意函数。首先回顾正交基，假设在$R^n$空间中，有n个正交向量$v_1,v_2,\cdots,v_n$，那么任意向量$x$可表示为：

$$
  x = c_1v_1 + c_2v_2 + \cdots + c_nv_n
$$

由于是正交基，计算系数$c_i$可以非常简单，上面等式两边同时乘以$v_i$，

$$
  v_i^Tx = c_iv_i^Tv_i \Rightarrow c_i = \frac{v_i^Tx }{v_i^Tv_i}
$$

如果正交基是是单位向量，那么可以简化为$c_i = v_i^Tx$。

## 正交函数

在介绍傅里叶级数之前，先介绍连续函数的内积。向量内积可以认为是离散函数上，对应元素乘积之和。连续函数的内积，概念类似，只是将求和换成积分，其他一致，是不是很优雅的衍生了一个新的定义，如函数$f(x),g(x)$,其点积如下:

$$
  (f(x), g(x)) = \int_{0}^{2\pi} f(x)g(x) dx
$$


后面的积分都在$[0, 2\pi]$之间讨论，因为三角函数式周期函数，傅里叶级数是使用三角函数对任意函数展开。同理，函数的模长平方定义如下：


$$
  (f(x), f(x)) = \int_{0}^{2\pi} \left(f(x)\right)^2 dx
$$


**例1** 正交三角函数

$$
  \int_{0}^{2\pi} \sin x \cos x dx
  = \int_{0}^{2\pi} \frac{1}{2} \sin 2x dx
  = \left[-\frac{1}{4}\cos 2x \right]_{0}^{2\pi}
  = 0
$$

上面例子说明$\sin x$与$\cos x$正交！


**例2** 通用正交三角函数

$$
  \int_{0}^{2\pi} \sin ax \cos bx dx \\
  \int_{0}^{2\pi} \sin ax \sin bx dx \\
  \int_{0}^{2\pi} \cos ax \cos bx dx
$$

首先，回顾几个相关的三角公式，根据[二角和差公式](http://baike.baidu.com/view/959840.htm)导出

$$
  \sin \alpha \cos \beta = \frac{\sin (\alpha + \beta) + \sin (\alpha - \beta)}{2} \\
  \sin \alpha \sin \beta = \frac{\cos (\alpha - \beta) - \cos(\alpha + \beta)}{2} \\
  \cos \alpha \cos \beta = \frac{\cos (\alpha - \beta) + \cos(\alpha + \beta)}{2}
$$

将三角公式代入上面的积分公式，可以将积变成和，方便积分，如下

$$
  \int_{0}^{2\pi} \frac{\sin (a+b)x + \sin (a-b)x}{2} dx 
  = -\frac{1}{2} \left[\frac{1}{a + b} \cos (a+b)x
   + \frac{1}{a - b} \cos (a-b)x \right]_{0}^{2\pi} \\
  \int_{0}^{2\pi} \frac{\cos(a-b)x - \cos(a+b)x}{2} dx 
  = \frac{1}{2} \left[\frac{1}{a - b} \sin (a-b)x
   - \frac{1}{a+b} \sin (a+b)x \right]_{0}^{2\pi} \\
  \int_{0}^{2\pi} \frac{\cos(a-b)x + \cos(a+b)x}{2} dx 
  = \frac{1}{2} \left[\frac{1}{a - b} \sin (a-b)x
   + \frac{1}{a+b} \sin (a+b)x \right]_{0}^{2\pi}
$$

当$a,b \in N$时，上面三个等式均为0，也就是此时，

$$
\sin ax与\sin bx正交 \\
\cos ax与\cos bx正交 \\
\sin ax与\cos bx正交
$$

## 傅里叶级数展开

根据基于向量的类似方法，在$[0,2\pi]$之间的可积（不一定连续）函数$f(x)$，可以展开为如下形式

$$
f(x) = a_0 + a_1 \cos x + b_1\sin x + a_2 \cos 2x + b_2 \sin 2x + \cdots
$$

 
接下来，可以按照向量的类似方法，计算系数，可以[参考三角函数积分](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E5%87%BD%E6%95%B0%E7%A7%AF%E5%88%86%E8%A1%A8)。首先，两边乘以$\cos nx$，并且在$[0,2\pi]$定积分，根据正交，有如下

$$
  \int_{0}^{2\pi} f(x)\cos(nx) dx= \int_{0}^{2\pi} a_n \cos^2(nx)dx=a_n\pi  
  \Rightarrow
  a_n = \frac{1}{\pi}\int_{0}^{2\pi} f(x)\cos(nx) dx
$$

然后两边乘以$\sin nx$，

$$
  \int_{0}^{2\pi} f(x)\sin(nx)dx= \int_{0}^{2\pi} a_n \sin^2(nx) dx=b_n\pi 
  \Rightarrow
  b_n = \frac{1}{\pi}\int_{0}^{2\pi} f(x)\sin(nx) dx
$$

当$x=0$时，计算系数$a_0$,

$$
  \int_{0}^{2\pi} a_0 dx= \int_{0}^{2\pi}f(x)dx \Rightarrow a_0=\frac{1}{2\pi} \int_{0}^{2\pi}f(x)dx
$$


## 方波的傅里叶级数展开

设$[0,2\pi]$之间的方波函数如下，

$$
  f(x) =
\begin{cases}
1,  & x \in [0,\pi) \\
-1, & x \in (\pi,2\pi]
\end{cases}
$$
 
根据上面公式，分别计算$a_0,a_n和b_n$,

$$
  a_0 = \frac{1}{2\pi}
  \left(\int_{0}^{\pi}1dx + \int_{\pi}^{2\pi}(-1)dx  \right) 
  = 0 \\
  \begin{align}
  a_n &= \frac{1}{\pi}\left(\int_{0}^{\pi}\cos(nx)dx + \int_{\pi}^{2\pi}(-1)\cos(nx)dx  \right) \\
  &=\frac{1}{\pi n}\left[\sin(nx)\right]_{0}^{\pi}-\frac{1}{\pi n} \left[\sin(nx)\right]_{\pi}^{2\pi} \\
  &= 0 \\
  \end{align} 
$$
$$
  \begin{align}
  b_n &= \frac{1}{\pi}\left(\int_{0}^{\pi}\sin(nx)dx + \int_{\pi}^{2\pi}(-1)\sin(nx)dx  \right) \\ 
  &=\frac{1}{\pi n}\left[-\cos(nx)\right]_{0}^{\pi}-\frac{1}{\pi n} \left[-\cos(nx)\right]_{\pi}^{2\pi} \\
  &= \frac{2}{\pi n}(1-\cos(n\pi))
  \end{align} 
$$

对于$b_n$，需要根据奇偶来计算具体值,

$$
  b_n = \begin{cases}
  \frac{4}{n\pi}, & n是奇数\\
  0, & n是偶数
  \end{cases}
$$


所以，最后展开结果为：

$$
f(x) = \frac{4}{\pi}(\frac{\sin(x)}{1} + \frac{\sin(3x)}{3} + \frac{\sin(5x)}{5} + \cdots)
$$


是不是很奇妙，一个普通函数，竟然可以用与他看似没有关系的三角函数表示，而且结果非常优美。升华一下上面思想，有时候万事万物就是这样，表面上看似没有关系的事情，背后可能有千丝万缕的联系。