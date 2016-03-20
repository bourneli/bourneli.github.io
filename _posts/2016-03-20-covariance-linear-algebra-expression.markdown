---
layout: post
title:  协方差矩阵计算的矩阵推导
categories: [linear-algebra, statistic, a b]
---

协方差矩阵是多元统计分析(MVA, Multiple Variable Analysis)中使用得比较多的方法，主要分析变量之间的关联性。
该方法计算量很大，且很繁复，但是可以使用矩阵优美的演示整个过程，这里记录整个推导过程，作为备忘。

数据矩阵$X$维度为$n \times p$，n是数据量，p是每个数据的维度。协方差矩阵定义如下，

$$
	S = {1 \over{n-1}} \sum^n_{i=1}{(x_i-\bar{x})(x_i-\bar{x})^T}
$$

其中$\bar{x}的定义如下

$$
	\bar{x} = {1 \over n} \sum^n_{i=1}{x_i}
	        = {1 \over n} \begin{bmatrix}x_1 & \cdots & x_n\end{bmatrix}\begin{bmatrix}1 \\ \vdots \\ 1\end{bmatrix}
		    = {1 \over n} X^T 1_{n \times 1}
		    
$$

将S按照块矩阵展开

$$
	S = {1 \over{n-1}} \underbrace{\begin{bmatrix} x_1-\bar{x} & \cdots & x_n-\bar{x} \end{bmatrix}}_{L}
					   \underbrace{\begin{bmatrix} (x_1-\bar{x})^T \\ \vdots \\ (x_n-\bar{x})^T \end{bmatrix}}_{R}
$$

由于L与R部分内容几乎一致，所以下面只演算L，然后通过转置计算R。

$$
\begin{align}
	L &= \begin{bmatrix} x_1-\bar{x} & \cdots & x_n-\bar{x} \end{bmatrix} \\ 
	  &= \begin{bmatrix} x_1 & \cdots & x_n \end{bmatrix} - \begin{bmatrix} \bar{x} & \cdots & \bar{x} \end{bmatrix}  \\
	  &= X^T - \bar{x}\begin{bmatrix} 1 & \cdots & 1 \end{bmatrix} \\
	  &= X^T - \bar{x}1_{1 \times n} \\
	  &= X^T - {1 \over n} X^T 1_{n \times 1}1_{1 \times n} \\
	  &= X^T \underbrace{(I_{n \times n} - {1 \over n} 1_{n \times 1}1_{1 \times n})}_{H(n)} \\
	  &= X^TH(n)
\end{align}
$$

可以证明$H(n)^2=H(n)且H(n)=H(n)^T$且，这里先略去，所以，最后协方差矩阵S可写为如下

$$
	S = {1 \over n-1}X^TH(n)(X^TH(n))^T = {1 \over n-1}X^TH(n)H(n)^TX = {1 \over n-1}X^TH(n)X
$$

$H(n)$是有个很好的地方，它可以将$X$转到另外一个空间，使得均值向量为0。令$Y=H(n)X$，有

$$
\begin{align}
	\bar{y} &= {1 \over n} Y^T 1_{n \times 1} \\
			&= {1 \over n} (H(n)X)^T 1_{n \times 1} \\
			&= {1 \over n} X^TH(n)^T 1_{n \times 1} \\
			&= {1 \over n} X^T(I_{n \times n} - {1 \over n} 1_{n \times 1}1_{1 \times n}) 1_{n \times 1} \\
			&= {1 \over n} X^T(1_{n \times 1} - {1 \over n} 1_{n \times 1}1_{1 \times n}1_{n \times 1}) \\
			&= {1 \over n} X^T(1_{n \times 1} - 1_{n \times 1}) \\
			&= \vec{0}
\end{align}
$$

一般而言，由于每个维度的两个不同，协方差矩阵的值会去掉单位，得到相关系数(皮尔森系数)，其计算方法为$\rho_{ij} = {\sigma_{ij} \over \sigma_i\sigma_j}$，衍生到协方差矩阵上，得到如下形式，

$$
	R = D^{-1/2}SD^{-1/2}
$$

其中$D$是对角矩阵，对角为1到P个维度的方差${\sigma_i}^2$.
