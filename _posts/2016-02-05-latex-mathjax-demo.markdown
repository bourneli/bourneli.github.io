---
layout: post
title:  "LaTeX和MathJax在博客中集成数学公式"
date:   2016-02-05 10:31:28 +0800
categories: "LaTeX"
---

## 方案
使用TeX(LaTeX)编写公式，使用MathJax在线实时渲染公式。网上可以找到很多TeX资料，在学术界已经流行了几十年，所以可以很方便上手，下面给出了一些TeX资料。MathJax是一个在线的公式渲染引擎，集成非常方便，只需要在header中加入一段js代码即可，免费（如果用的好，可以捐点），由MIT和美国数学委员会资助，技术和维护应该不成问题。所以选取这套方案。

## 资料

* [TeX快速常用示例](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)
* [TeX问答](http://tex.stackexchange.com/)
* [MathJax官方使用文档](https://docs.mathjax.org/en/v2.6-latest/start.html#putting-mathematics-in-a-web-page)

## 我的示例

### 行内inline

When $a \ne 0$, there are two solutions to $\(ax^2 + bx + c = 0\)$ and they are
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

### 矩阵和向量

$$
\begin{bmatrix}
a & b \\ c & d
\end{bmatrix}
\vec{x} = \vec{b}
$$
