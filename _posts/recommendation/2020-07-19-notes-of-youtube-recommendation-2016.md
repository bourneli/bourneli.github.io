---
layout: post
title:  Notes of YouTube Recommendation 2016
categories: [Recommendation]
---



I have read the paper [(2016)Deep Neural Networks for YouTube Recommendations](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45530.pdf), which describes how YouTube made recommendations at that time. Although, it is a little out of period, but there are still many best practices that would work in your problems. In this post, I only record the points that is impressive to me and highly recommend readers to read the paper by yourselves.



# System Architecture

The recommendation system architecture demonstrates how the funnel retrieves and ranks the videos for each user. It's a two-stage approach. Firstly, it uses a candidate generation model to recall hundreds candidates from millions of videos. Then, it trains a ranking model to find the top-n videos from the candidates. It is an classic architecture, which is widely applied in recommendation system.

![](\img\youtube-2016\arch.png)



Let's take a look at the recalling model in details. It used word-2-vector to transform the videos and queries to embedding vectors. It averages the recently videos and queries vectors to trace the user's main actions. Then, it concatenates user's actions with geographic, age and gender to the first wider layer, followed by several layers of fully connected Rectified Linear Units(ReLU). 

The most impressive side is that it makes the model to solve a multi-class problem, so that it can get the embeddings of users and videos in the last layer. What's more, it doesn't use the model for online inference, but use an nearest neighbor index to approximately the top N candidates. This method is more efficient than online inferencing, because it avoids inferencing the millions of videos.   



![](\img\youtube-2016\arch-recall.png)



The ranking model is almost the same as the recalling one. However, it puts more efforts on feature engineering and applies the weighted logistic to put more attentions to the watch-time of each video. 

![](\img\youtube-2016\arch-rank.png)



# Inferencing watch-time

I think that the inference for watch-time is the most valuable practice in this paper. It uses weighted logistic regression to optimize the model and infer the rank in the form of  $e^{Wx+b}$, which takes the watch-time in consideration. Why does it work? The paper does not make it clear. I try to explain to it in Bayesian perspective. The form $e^{Wx+b}$ is odds, which is:
$$
\begin{align}
Odds &= \frac{p}{1-p} \\
	 &= \frac{P(y = 1\vert x)}{P(y=0 \vert x)} \\
	 &= \frac{\frac{P(x|y=1) P(y=1)}{P(x)}}{\frac{P(x|y=0)P(y=0)}{P(x)}} \\
     &= \frac{P(x \vert y=1)P(y=1)}{P(x \vert y=0) P(y=0)}
\end{align}
$$
Weighted logistic regression only changes the $P(y=1)$ to $wP(y=1)$ and $P(y=0) \ll P(y=1)$, so it almost keeps the other things the same and the new odds is $w$ times the original one. $w$ is proportional to watch-time, so the new odds is the expected watch-time. Whether one unit $w$ is equal to one second, or one minutes or other unit time depends on the tunning and the paper doesn't mention it. Different conversion means different sample weights, so you should tunning it in your own problem. You can also read this [article](https://zhuanlan.zhihu.com/p/61827629) to gain more information.

There are many best practices and I highly recommend you to read the paper in details.



# Reference

* [(2016)Deep Neural Networks for YouTube Recommendations](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45530.pdf)

* [重读Youtube深度学习推荐系统论文，字字珠玑，惊为神文](https://zhuanlan.zhihu.com/p/52169807)

* [YouTube深度学习推荐系统的十大工程问题](https://zhuanlan.zhihu.com/p/52504407)

* [揭开YouTube深度推荐系统模型Serving之谜](https://zhuanlan.zhihu.com/p/61827629)

* [Deep Neural Network for YouTube Recommendation论文精读](https://zhuanlan.zhihu.com/p/25343518)



