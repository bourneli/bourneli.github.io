---
layout: post
title: Bayesian is Very Interesting
categories: [Bayesian, Statistics]
---



I haven't post a blog since last August. Recently, I have been learning Bayesian Statistics and think it very interesting. So, I decide to record something. 

There are two dominate thoughts in statistics -- Bayesian VS Frequentist. The most significant difference is that Bayesian requires a prior distribution, but Frequentist not. From Bayesian view, they think the parameters of distributions also follow some prior distributions.

Most of us begin learning statistics with Frequentist, so it's difficult to accept the concept of prior distribution at first. Once we choose the prior, we can combine the likelihood and prior to get the posterior probability. The prior distributions are the knowledge outside of the current problem. **Bayesian uses a rigorous mathematical method to merge the current problem (likelihood) and previous knowledge (prior) to obtains a better result**.

Let me give an example.


$$
p(\theta \vert D) = \frac{p(D \vert \theta) p(\theta)}{p(D)} = \frac{p(D \vert \theta)p(\theta)}{\int p(D \vert \theta) p(\theta) d\theta}
$$

* $D$ stands for data. 
* $\theta$ means the parameter you want to inference, such as the head rate of a coin. 
* $p(D \vert \theta)$ is the likelihood, which is the probability of Data D when the $\theta$ is constant. 
* $p(\theta)$ is the prior distribution for $\theta$, such as normal distribution, Bernoulli distribution and so on.
* $p(\theta \vert D)$ is the posterior distribution that we want to obtain.

From the formula, $p(D)$ is complex and difficult to compute in most cases. However, it is   a constant, so we can safely ignore it and still get the right posterior distribution. As a result, we only need to compute the value of $p(D \vert \theta)p(\theta)$, which is easy to obtain.   

There are many topic related to Bayesian Statistics, such as 

* Conjugate of prior and posterior distributions
* Markov Chain Monte Carlo 
* Generalized Linear Models
* Hierarchical Models

* Bayesian Machine Learning

Bayesian is very interesting, and I will continue to learn it to extend my knowledge edges. 

Today is traditional lunar Qingming Festival, and this article is dedicated to the memory of my grandfather, K.D. Li(李克德), 1922~2002. 

