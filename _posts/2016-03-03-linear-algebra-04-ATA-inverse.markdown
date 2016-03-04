---
layout: post
title:  "线代随笔04-$A^TA$可逆条件"
date:   2016-03-03 12:16:28 +0800
categories: linear algebra
---

大多数矩阵不是方正，必然没有逆矩阵。但是，$A^TA$必然是方正，此时是否一定可逆呢？答案是，需要看情况 :-)。下面给出$A^TA$可逆的充分必要条件，并证明，希望读者可以体会到其中的美感！

$$
A^TA可逆 \Leftrightarrow A的列向量线性独立
$$

**证明：$A^TA可逆 \Rightarrow A的列向量线性独立$**

使用反正，假设A的列线性依赖，那么有

$$
\exists{x \neq 0},Ax=0
$$

两边同时乘以$A^T$，x保持不变，进而有

$$
\exists{x \neq 0},A^TAx=A^T0=0
$$

因为$A^TA$可逆，所以

$$
\begin{align}
\exists{x \neq 0}, A^TAx=0 
& \Rightarrow (A^TA)^{-1}(A^TA)x=0 \\
& \Rightarrow x=0, x必须为0
\end{align}
$$

与假设矛盾，所以假设不成立，所以A的列向量线性独立。




**证明：$A的列向量线性独立 \Rightarrow A^TA可逆$**

计算$A^TA$的列空间，

$$
\begin{align}
A^TAx=0 
& \Rightarrow x^TA^TAx=x^T0=0 \\
& \Rightarrow |Ax|^2=0  \\
& \Rightarrow Ax=0 \\
& \Rightarrow A的列向量线性独立 \\ 
& \Rightarrow x=0 \\
& \Rightarrow 方阵A^TA的零空间为空，说明满秩，必可逆
\end{align}
$$

证毕。上面命题的否命题也成立，

$$
A^TA不可逆 \Leftrightarrow A的列向量线性依赖
$$

在线性代数的大部分应用中，都会遇到$A^TA$的形式，比如SVD，投影向量等。通过上面的证明，可以发现如果需要$A^TA$可逆，必须让A的列向量中没有多余的列，A必须是方形或者瘦长形(列数<=行数)。这种简洁的美感，可以用爱因斯坦的那句名言概括:

<p align='center'><strong> Everything should be made as simple as possible, but no simpler.</strong></p>





