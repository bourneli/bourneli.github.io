---
layout: post
title:  "线代随笔13-行列式及其相关性质"
categories: [linear-algebra]
---

本博文主要回顾行列式的定义及其相关性质，介绍行列式与矩阵联系最紧的相关部分，大体结构如下：

* 行列式定义
* 3个基础性质
* 7个衍生性质

## 行列式的定义
矩阵A是n阶方阵，其行列式定义如下，

$$
	|A| = \sum_{j=1}^na_{1j}(-1)^{1+j}M_{1j} = \sum_{j=1}^na_{ij}(-1)^{i+j}M_{ij}
$$

其中，$M_{ij}$表示A去掉第i行和第j列后的矩阵的行列式，$(-1)^{i+j}M_{ij}$称为代数余子式。对任意行展开，得到的结果相同（可以通过数学归纳法证明，有点繁琐，这里略去）。行列式是一种计算规则，将n*n个实数映射为一个实数。如果按照基础定义，计算复杂度是$O(n!)$。所以，直接计算，显然不现实，下面通过定义，推导出一些性质，使得行列式的计算变得简单。

## 3个基础性质
根据定义，可以推导出三个行列式的基础性质，通过它们又可以推导出更多有用的衍生性质，下面先推导出这三个基本性质。

### 1.归一性

$$|I_n|=\prod_{i=1}^n1=1$$


直接按照定义展开即可。

### 2.行交换

任意两行交换，行列式值为原来的值乘以-1。使用数学归纳法证明，

证明：

当n=2的时候，很容易证明。

假设n=k（k>2）成立，

当$n=k+1$时，令第$i$行与第$j$行交换，$i \ne j$。选取第$k$行展开，$m \ne i$且m!=j。令$D$为原行列式值，而$D’$为换行后的值，有

$$
	D = \sum_{q=1}^na_{kq}(-1)^{k+q}M_{kq}
$$

$$
	D' = \sum_{q=1}^na_{kq}(-1)^{k+q}M_{kq}'
$$


经观察，$M_{kq}$ 和$M_{kq}'$是k阶行列式，且两者对应i，j列交换，根据假设有$M_{kq}'=-M_{kq}$，所以
 
$$
	D' = \sum_{q=1}^na_{kq}(-1)^{k+q}M_{kq}' = \sum_{q=1}^na_{kq}(-1)^{k+q}(-M_{kq}) = -\sum_{q=1}^na_{kq}(-1)^{k+q}M_{kq} = -D
$$ 
 
 
证毕！

### 3.线性计算


线性计算是线性代数的核心，也就是向量加法与常数乘法，详细代数也有类似性质。

**加法性**

$$
	D = \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_{k1} + b_{k1} & \cdots & a_{kn} + b_{kn} \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix}
	  = \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_{k1} & \cdots & a_{kn} \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix}
	  + \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	b_{k1} & \cdots & b_{kn} \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix}
	  = D_a + D_b
$$ 
 
证明：
直接按照定义，将第k行展开，
 
$$
	D = \sum_{q=1}^n(a_{kq}+b_{kq})(-1)^{k+q}M_{kq} 
	  = \left(\sum_{q=1}^na_{kq}(-1)^{k+q}M_{kq}\right) + \left(\sum_{q=1}^nb_{kq}(-1)^{k+q}M_{kq}\right)
	  = D_a + D_b
$$


证毕！

**标量乘法**

$$
	\begin{vmatrix}
	\cdots & \cdots & \cdots \\
	ca_{k1} & \cdots & ca_{kn} \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix} 
	= c\begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_{k1} & \cdots & a_{kn} \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix}
$$

 
证明：
直接按照定义，将第k行展开，

$$
	\sum_{q=1}^n ca_{kq} (-1)^{k+q} M_{kq} 
	= c\sum_{q=1}^n a_{kq} (-1)^{k+q} M_{kq} = cD
$$
 
证毕！


## 7个衍生性质

### 4.行相同
假设矩阵A第i,j行一样，那么按照第i行展开，得到行列式D，然后交换第i行与第j行，仍然按照第i行展开，由于两行一样，所以行列式的值仍为D，但是根据行交换，行列式反号定理，D=-D，最后得到D=0.

### 5.行减不改变行列式
令行列式D如下，k为任意常量
 
$$
	D = \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_1 & \cdots & a_n \\
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix} 
$$


那么第a行减去k乘第b行，结果不变，如下推导

$$
	\begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_1-kb_1 & \cdots & a_n-kb_n \\
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix} =
    \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	a_1 & \cdots & a_n \\
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix} +
    \begin{vmatrix}
	\cdots & \cdots & \cdots \\
	-kb_1 & \cdots & -kb_n \\
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix}	
	= D - k\begin{vmatrix}
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	b_1 & \cdots & b_n \\
	\cdots & \cdots & \cdots \\
	\end{vmatrix} = D - k0 = D	
$$

证毕！

### 6.全0行
如果矩阵存在某一行全部为0，那么行列式为0。直接按照0行展开，即可得到结果。

### 7.三角矩阵的行列式
值为对角线元素乘积。使用定义，直接展开即可。

### 8.矩阵可逆 <=> 行列式不为0
证明：
如果矩阵可逆，那么根据消元（不改变行列式），可以得到上三角矩阵，且对角元素不为0，那么行列式不为0。
如果行列式不为0，仍然可以通过消元得到等价三角矩阵，由于行列式不为0，所以对角元素均不为0，所以矩阵可逆。
证毕！

### 9.矩阵乘法的行列式

证明：
基础矩阵：消元E，乘法D，替换P与B的效果如下。

$$
	|EB|=|B|
$$
 
$$
	|DB|=\prod_{i=1}^nd_iB
$$

$$
	|PB|=-|B|
$$

如果A可逆，那么$A=X_1 X_2…X_n$,其中X_i是基础矩阵，
那么
$$
	|AB|=|X_1||X_2 \cdots X_nB|=|X_1X_2||X_3 \cdots X_nB| = |X_1 \cdots X_n||B| = |A||B|
$$


如果A不可逆，那么A奇异，那么AB也是奇异的，通过消元，必然可以得到全0行，那么$\|A\|=\|AB\|=0$。

证毕！


### 10.转置矩阵的行列式

转置矩阵的行列式与原行列式相等，即
$$
	|A|=|A^T|
$$

证明：

如果$A$奇异（不可逆），那么$A^T$必然奇异，所以
$$
 |A^T|=|A|=0
$$ 。

若$A$可逆，那么$A$可以$LU$分解，即
$$
	|A|=|LU|=|L||U|=|U|=\prod_{i=1}^nu_{ii}
$$ ，$L$是下三角矩阵，对角线为1，所以
$$
	|L|=1
$$。

同理，
$$
	|A^T|=|U^T L^T|=|U^T ||L^T |=|U^T |=\prod_{i=1}^nu_ii=|U|=|A|
$$。

证毕！


## 参考资料
* Introduction to Linear Algebra, 4th Edition, by Gilbert Strang, Chapter 5 Determinant
* [行列式乘法证明](http://www.math.lsa.umich.edu/~speyer/417/DetMult.pdf)
