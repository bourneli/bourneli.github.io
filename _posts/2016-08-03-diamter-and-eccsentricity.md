---
layout: post
title:  "图直径与离心率(eccentricity)相关推论"
categories: [graph]
---

计算图的直径，如果按照定义直接计算，复杂度会是$O(n^3)$。实际通常采取估算的方法，估算过程会涉及到一个重要的概念——离心率(eccentricity)。离心率与直径有一些推论，这里集中证明，作为备忘。以下推论适用全联通图，无向/有向,无权/有权均可，但是权重需要非负。

**定义1** 离心率 

对任意点$v$的离心率是该点到图中其他点最短距离的最大值，

$$\epsilon(v) = \max_{w \in V} dist(v,w)$$

其中$dist(v,w)$是点$v$到$w$最短距离，$V$是节点集合。

**定义2** 直径

直径是两点最短距离中的最大值，根据离心率定义，可以如下表示

$$
	\Delta(G) = \max_{v \in V}\epsilon(v)
$$

根据上面定义，离心率和直径建立了联系。

**推论1** 开阔的直径上下界

$$
 \epsilon(v) \le \Delta(G) \le 2*\epsilon(v) 
$$

证明:

下界根据定义容易证明，省略。上界证明使用反正法，设直径由点$x$和$y$确定，根据离心率定义，容易推得如下

$$
\Delta(G) = dist(x,y) \le dist(x,v)+dist(v,y) \le  2  *\epsilon(v)
$$

可以构造一个路径$x \rightarrow v \rightarrow y$，得到一个新的直径，小于当前直径，得到矛盾。

证毕！


**推论2** 离心率上下界 

$\epsilon(v)$是特定点$v$离心率，$w(\in V)$是任意其他点(包括$v$),$v$相对于$w$固定

$$
	\max(\epsilon(v) - dist(v,w), dist(v,w)) \le \epsilon(w) \le dist(v,w) + \epsilon(v)
$$

证明：

证明上界， 根据离心率展开，设$x$是$w$离心率的另一个端点

$$
	\epsilon(w) \le dist(w,v)+dist(v,x) \le d(v,w) + \epsilon(v) 
$$

小于当前离心率，得到矛盾，所以假设不成立。

证明下界，其中$dist(v,w)$与$epsilon(w)$的关系易得，略去。$\epsilon(v)-dist(v,w)$与$epsilon(w)$关系，可以将$-dist(v,w)$移到不等式另外一边，得到形式与上届一致，证明方法一致，省略。

证毕！

**定义3** $\epsilon_L(v)$为当前点离心率下界，$\epsilon_L(v)$为离心率上界。

**推论3** 紧凑的直径上下界

$$
	\max_{v \in V}\epsilon_L(v)\le \Delta(G) \le \min(\max_{v \in V}\epsilon_U(v), 2*\min_{v \in V}\epsilon_L(v))
$$


证明：

根据$推论1$，很容易证明，省略。

证毕！

