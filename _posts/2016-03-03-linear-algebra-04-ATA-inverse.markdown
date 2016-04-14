---
layout: post
title:  "线代随笔04-$A^TA$可逆条件"
date:   2016-03-03 12:16:28 +0800
categories: [linear-algebra]
---

大多数矩阵不是方正，必然没有逆矩阵。但是，$A^TA$必然是方正，此时是否一定可逆呢？答案是，需要看情况 :-)。下面给出$A^TA$可逆的充分必要条件，并证明，希望读者可以体会到其中的美感！

$$
A^TA可逆 \Leftrightarrow A的列向量线性独立
$$


**证明：$A的列向量线性独立 \Rightarrow A^TA可逆$**

[$N(A^TA)=N(A)=0$](/linear-algebra/2016/04/14/linear-algebra-09-null-space-for-ata-and-a.html),证毕。

**证明：$A^TA可逆 \Rightarrow A的列向量线性独立$**

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





