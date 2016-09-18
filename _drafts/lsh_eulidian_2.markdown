---
layout: post
title:  LSH在欧式空间的应用(2)--工作原理
categories: [probability,LSH]
---

## 核心概念：$(d_1,d_2,p_1,p_2)-sensitive$

LSH的核心工作原理就是找到一组hash函数，它们必须符合$(r_1,r_2,p_1,p_2)-sensitive$条件。设一类函数$H=\{h: S \rightarrow U\}$，对于任意一种距离度量方法D，任意向量$q,v \in S$

$$
	如果 v \in B(q, r1) \\
	如果 v \notin B(q, r1) \\
$$


<div align='center'>
	<img src='/img/lsh_create_table.png' />
</div>

<div align='center'>
	<img src='/img/lsh_query.png' />
</div>

