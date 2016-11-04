---
layout: post
title:  "线代随笔17-奇异值分解(SVD)推导"
categories: [linear-algebra]
---

奇异值分解可以说是基础线性代数的高潮部分，它将很多基础线性代数的内容串联起来，比如正交矩阵，基，正定，矩阵的秩，特征值，特征向量，矩阵的逆等等。奇异值分解的理论非常漂亮，它可以对任意形状的矩阵进行分解，而且得到分解形式也很特殊。本博文以一种倒叙的方式介绍奇异值分解，从中可以领会到数学美感。

假设矩阵A为任意维度，A的秩为$r$，如果不为方正，那么自然是得不到特征值和特征向量的。但是$A^TA$与$AA^T$却是对称的，而且根据正定性，它们是半正定矩阵。所以，可以计算相关的特征值和特征向量。那么，这两个矩阵的特征向量是否有什么关系呢？

## 大胆假设

我们来做一些大胆的设想，假设下面条件统统成立：

1. $v_1,\cdots,v_r \in R(A)$,$u_1,\cdots,u_r \in C(A)$
2. $v_1,\cdots.v_r是正交单位矩阵,u_1,\cdots,u_r是正交单位矩阵$
2. $Av_1 = \rho_1u_1, Av_2 = \rho_2u_2,\cdots , Av_r = \rho_ru_r$
3. $\rho_i > 0, i = 1,2,\cdots,r$

读者可能不禁要想，对于任意矩阵A，这种假设是不是太过苛刻！先不管那么多，假设成立，可以得到

$$
	A\begin{bmatrix}v_1 & \cdots & v_r\end{bmatrix} = \begin{bmatrix}u_1 & \cdots & u_r\end{bmatrix}
	\begin{bmatrix}
		\sigma_1 \\ 
         &\ddots & \\
         &&\sigma_r
    \end{bmatrix}
$$

由于$v_i,u_i$均是正交的，所以可以使用Gram-Schimidt方法将其扩展到整个行空间和列空间，得到下面

$$
	A\underbrace{\begin{bmatrix}v_1 & \cdots & v_r  & v_{r+1} & \cdots & v_n\end{bmatrix}}_V
    = \underbrace{\begin{bmatrix}u_1 & \cdots & u_r & u_{r+1} & \cdots & u_m \end{bmatrix}}_U 
	\underbrace{
        \begin{bmatrix}
            \sigma_1 \\ 
             & \ddots \\
             && \sigma_r  \\
             &&& 0 \\
             &&&& \ddots \\
             &&&&& 0 \\
        \end{bmatrix}
    }_{\Sigma} 
$$

简化上面的表达式

$$
    AV = U\Sigma \Rightarrow A = U\Sigma V^T
$$

也就是说，如果矩阵A满足上面这些条件，就可以将A分解成两个正交矩阵和一个对角矩阵，是不是非常优美！由于$\Sigma$后面对角线元素都为0，所以可以进一步简化：

$$
    \underbrace{A}_{m \times n} = \underbrace{U_r}_{m \times r} \underbrace{\Sigma_r}_{r \times r} \underbrace{V^T_r}_{r \times n}
$$

$U_r$是U保留前r列，$V^T_r$是$V^T$保留前r行，$\Sigma_r$是保留前r行与r列。


## 小心求证

首先从$A^TA$开始，由于半正定性，所以可以将其特征值写成如下形式

$$
	A^TAv_i=\sigma_i^2v_i \qquad (1)
$$

由$A^TA$半正定性，可得$v_i$标准正交，且令$\sigma_i \gt 0(i = 1,2, \cdots, r$)称之为**奇异值**(注意:不要与特征值混淆)。对公式(1)两边乘以$v_i^T$得到

$$
	v_i^TA^TAv_i=v_i^T\sigma_i^2v_i \Rightarrow \|Av_i\|^2 = \sigma_i^2 \Rightarrow  \sigma_i  = \|Av_i\| \qquad (2)

$$

从公式(2)可以了解到，奇异值$\sigma_i$是$A^TA$单位特征向量$v_i$在矩阵A列空间的投影向量的模长（很绕口，看看就好，不用太在意）。然后，继续对公式(1)两边乘以A,构造出$AA^T$

$$
	AA^TAv_i=A\sigma_i^2v_i \Rightarrow AA^T(Av_i)=\sigma_i^2(Av_i) \qquad (3)
$$

线性代数中有很多重要证明利用了括号变化，上面又是一个重要的例子。根据公式(3),可以发现$AA^T$与$A^TA$具有相同的特征值$\sigma_i^2$。并且特征向量具有关系，令$u_i$是$AA^T$的特征向量，明显有$u_i$正交，且由公式(3),$u_i=Av_i$。但是正交特征矩阵一般需要为单位矩阵，而且由公式(1)可以容易得到模长，所以最终令$u_i=\frac{Av_i}{\sigma_i}$。上面的假设得到了证明。

事实证明上面的假设并不苛刻，对于任意A都可以找到这些向量和奇异值将其分解，不得不佩服发现SVD的数学家们。

## 计算策略

由于$A^TA$与$AA^T$的**特征值相同**，计算策略显而易见，计算维度较小的那个矩阵的特征值。因为特征值的复杂度是$O(n^3)$,n是矩阵的行数，当行数很高时，根本无法计算。spark中的SVD也是按这种策略实现。

## 奇异值的权重

奇异值都是大于0的实数，可以认为是权重，所以将奇异值按照从大到小的方式延对角线排列，可以得到唯一的分解。将SVD分解按代数形式展开，如下

$$
	A = \begin{bmatrix}u_1 & \cdots & u_r\end{bmatrix}
		\begin{bmatrix}
			\sigma_1 \\ 
	         &\ddots & \\
	         &&\sigma_r
	    \end{bmatrix}
  		\begin{bmatrix}v_1^T \\ \vdots \\ v_r^T\end{bmatrix}
	  = \sum_{i=1}^{r}\sigma_i u_i v_i^T
$$

A是r个秩为1的矩阵的线性组合，且每个矩阵室由单位向量乘法构成模，奇异值是系数。如果最后的奇异值比较小，可以忽略，对A的整体影响比较小，这样可以做有损压缩。
