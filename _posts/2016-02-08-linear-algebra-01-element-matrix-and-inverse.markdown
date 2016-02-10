---
layout: post
title:  "线代随笔01-基础消元矩阵与逆矩阵"
date:   2016-02-10 22:32:28 +0800
categories: linear algebra
---

## 序言
系统回顾线性代数已经有大半年时间，现在沉淀相关知识，方便后续回顾。使用的资料是[MIT的线性代数公开课和其配套教材](http://ocw.mit.edu/courses/mathematics/18-06-linear-algebra-spring-2010/video-lectures/)。视频与教材同步学习，完成课后练习效果更佳（**强烈推荐**）。利用业余时间（每天晚上和部分周末时间），回顾一遍大概需要半年左右时间。线性代数有一种美感，它将一些复杂的概念用及其简洁的符号描绘。这系列博客我称之为“**线代随笔**”，主要记录一些比较有趣和实用的线性代数概念。本文是开篇，回顾基础消元矩阵与逆矩阵的关系。

## 基础消元矩阵
主要有三类基础消元矩阵

* 消元矩阵：消除另一行（或列）的元素，使其为0，简化矩阵，例如

$$ E =
\begin{bmatrix}
1 & 0 \\ -e & 1
\end{bmatrix}
$$

e为任意不为0的数。

* 除法矩阵：对任意一行进行除法操作，使矩阵中的一个轴元素(pivot value)化简为1，例如

$$ D =
\begin{bmatrix}
1 & 0 \\ 0 & d
\end{bmatrix}
$$

d为非零数，对矩阵第二行（或列）全部除以d。


* 排练矩阵：交换单位矩阵的任意两行（或列）

$$ P =
\begin{bmatrix}
0 & 1 \\ 1 & 0
\end{bmatrix}
$$

有兴趣的读者可以根据[矩阵乘法](https://en.wikipedia.org/wiki/Matrix_multiplication)定义，验证上面三类基础矩阵的效果。

上面三类基础矩阵的共性

* 可逆。
* 当在乘法**左**边时，作用与右矩阵的对应**行**。
* 当在乘法**右**边时，作用与做矩阵的对应**列**。


## 使用基础消元矩阵求逆

**逆矩阵是基础消元矩阵的连续相乘的积。**举个例子，设一系列矩阵如下

$$ A = 
\begin{bmatrix}
1 & 1 \\ 2 & 1
\end{bmatrix},
E_{21} = 
\begin{bmatrix}
1 & 0 \\ -2 & 1
\end{bmatrix},
E_{12} = 
\begin{bmatrix}
1 & 1 \\ 0 & 1
\end{bmatrix},
D_2 = 
\begin{bmatrix}
1 & 0 \\ 0 & -1
\end{bmatrix}
$$

我们有下面的结果（有兴趣的读者可以手动计算），

$$
D_2E_{12}E_{12}A=I
$$

等式两边右乘$A^{-1}$，

$$
D_2E_{12}E_{12}AA^{-1}=IA^{-1}，即 D_2E_{12}E_{12}=A^{-1}
$$

## 高斯乔丹法
上述方法就是[高斯乔丹求逆法](https://en.wikipedia.org/wiki/Gaussian_elimination#Finding_the_inverse_of_a_matrix)的核心思想。形式化，对任意可逆矩阵$A$，将其放入下面块矩阵中，

$$
B = \begin{bmatrix}
A & I
\end{bmatrix}
$$

使用一些列基础消元矩阵左乘$B$，将$A$变成$I$，同时$I$变成$A^{-1}$，形式如下

$$
\begin{align}
	D_1 D_2 \cdots P_1 P_2 \cdots E_1 E_2 \cdots
	\begin{bmatrix}
	A & I
	\end{bmatrix} 
	& =  \begin{bmatrix}
	D_1 D_2 \cdots P_1 P_2 \cdots E_1 E_2 \cdots A & D_1 D_2 \cdots P_1 P_2 \cdots E_1 E_2 \cdots I
	\end{bmatrix} \\
	& = \begin{bmatrix}
	I & D_1 D_2 \cdots P_1 P_2 \cdots E_1 E_2 \cdots
	\end{bmatrix} \\ 
	& = \begin{bmatrix}
	I & A^{-1}
	\end{bmatrix} \\ 
\end{align}
$$

## 总结
所以，我们的逆就是一些列基础消元矩阵的乘积，上述过程清楚地记录整个过程。在消元的过程中，会渐渐发现一些矩阵的性质，比如矩阵的基，维度，线性无关等，这些需要引入线性空间的概念，后续会有更详细的记录，敬请期待。



