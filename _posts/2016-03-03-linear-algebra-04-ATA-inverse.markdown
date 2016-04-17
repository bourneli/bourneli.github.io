---
layout: post
title:  "线代随笔04-$A^TA$可逆条件"
date:   2016-03-03 12:16:28 +0800
categories: [linear-algebra]
---

大多数矩阵不是方正，必然没有逆矩阵。但是，$A^TA$必然是方正，此时是否一定可逆呢？答案是：需要看情况 :-)。下面给出$A^TA$可逆的充分必要条件，并证明，希望读者可以体会到其中的美感！

$$
A^TA可逆 \Leftrightarrow A的列向量线性独立
$$

## 零空间性质
在证明之前，需要先证明一个性质:$N(A) = N(A^TA)$，即$A$的零空间与$A^TA$的零空间相同。

因为

$$Ax = 0 \Rightarrow A^TAx=0$$

所以$N(A) \subseteq N(A^TA)$

因为

$$
\begin{align}
	A^TAx =0 & \Rightarrow x^TA^TAx = 0 \\ 
	         & \Rightarrow |Ax|^2=0 \\ 
			 & \Rightarrow Ax=0
\end{align}
$$

所以$N(A^TA) \subseteq N(A)$

证毕！

## 原命题证明

利用上面零空间的性质，很容易证明原命题：

若A的列线性独立，那么$N(A^TA)=N(A)= {0}$，所以方正$A^TA$满秩，可逆。

若$A^TA$可逆，那么$N(A)=N(A^TA)= {0}$，所以A的列线性独立。

证毕！

## 结语
在线性代数的大部分应用中，都会遇到$A^TA$的形式，比如SVD，投影向量等。通过上面的证明，可以发现如果需要$A^TA$可逆，必须让A的列向量中没有多余的列，A必须是方形或者瘦长形(列数<=行数)。这种简洁的美感，可以用爱因斯坦的那句名言概括:

<p align='center'><strong> Everything should be made as simple as possible, but no simpler.</strong></p>





