---
layout: post
title:  Automated Machine Learning调研
categories: [automl]
---

## 缘起

Automated Machine Learning(后面简称**AutoML**)这项技术在上个世纪90年代就开始在工业界出现，可以参考文章[Automated Machine Learning: A Short History](https://blog.datarobot.com/automated-machine-learning-short-history),不过打那之后就没有什么太大动静，可能是因为都是商业化软件，不利于交流。直到最近一两年，google借着深度学习的春风在AutoML上发力，先后发表了多篇相关paper以及blog，

* [Using Machine Learning to Explore Neural Network Architecture](http://ai.googleblog.com/2017/05/using-machine-learning-to-explore.html)（2017.5）

* [AutoML for large scale image classification and object detection](http://ai.googleblog.com/2017/11/automl-for-large-scale-image.html)（2017.11）

* [Using Evolutionary AutoML to Discover Neural Network Architectures](http://ai.googleblog.com/2018/03/using-evolutionary-automl-to-discover.html)（2018.3）

这些都是使用AutoML技术自动设计深度学习网络架构。经过google的助推，AutoML在近期又被AI从业者广泛关注（墙外的同学可以通过Google Trends检索“AutoML”体会一下）。



## AutoML是什么

AutoML是什么，可以参考[Why AutoML Is Set To Become The Future Of Artificial Intelligence](https://www.forbes.com/sites/janakirammsv/2018/04/15/why-automl-is-set-to-become-the-future-of-artificial-intelligence/#f10cf67780ae)这篇文章。笔者认为在学术界和工业界，有两种不同的诠释，

* **学术界** 根据[wiki定义](https://en.wikipedia.org/wiki/Automated_machine_learning)，一种自动化的端到端的机器学习过程（包括数据处理，特征选择，模型选择和调参）的应用，解决实际问题。
* **工业界** 降低机器学习使用门槛，更大限度利用机器学习技术创造价值。

更加接地气的分享，可以参考2018年6月[第四范式涂威威：AutoML技术现状与未来展望](https://link.zhihu.com/?target=https%3A//v.qq.com/x/page/y071886cmu6.html)的分享。虽然没有谈论很多技术细节，但是可以高屋建瓴的了解当前Automated Machine Learning研究现状。映像比较深的有以下几点，

* 自动机器学习，本质上就是机器学习。
* 强化学习解决AutoML问题，使用更难的技术解决相对简单的问题。可能不是一个很好的方向，虽然Google在尝试。
* 对数据本身做特征，以及配置做特征，然后训练预估模型的效果。模型效果的模型。
* AutoML应该解决问题动态变化的场景。
* 图像领域的应用，网络结构的搜索。
* 研究方向：效率和泛华

更多细节建议观看视频，50分钟不到，有些地方可以快进。



## 学术和工业界动向

目前工业界的一些主流AutoML应用，

* [Google Cloud AutoML](https://cloud.google.com/automl/) 截止2018年7月，仅发布了AutoML Vision，这些内容与上面的blog以及paper有着密不可分的联系。
* [Microsoft Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/) 微软针对视觉识别，机器翻译，情感识别等问题的解决方案及服务。
* [Clarifai's image recognition service](https://www.clarifai.com/)  提供多种场景的视觉识别解决方案。
* [第四范式先知系统](https://www.4paradigm.com/product/prophet) 由数据核心、算法核心、生产核心三大模块组成，覆盖了人工智能在生产中的各应用环节，使用AutoML技术降低AI开发门槛，帮助企业更加高效地在AI时代从战略、策略到执行全面智能化。 

通过上面几家主流AutoML厂商，笔者觉得AutoML的商业模式的是将**AI技术云端化**，类似当年云计算将计算能力云端化一样。类似Google开源TensorFlow这样的案例，在AutoML领域应该比较少见。 



目前主流开源学术项目(参考[Automated Machine Learing Wiki定义](https://en.wikipedia.org/wiki/Automated_machine_learning))，

* 超参数优化和模型选择

	* [H2O AutoML](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/automl.html) provides automated data preparation, hyperparameter tuning via random search, and stacked ensembles in a distributed machine learning platform.
	* [mlr](https://github.com/mlr-org/mlr) is a [R](https://en.wikipedia.org/wiki/R) package that contains several hyperparameter optimization techniques for machine learning problems.

* 全栈优化

	* [Auto-WEKA](http://www.cs.ubc.ca/labs/beta/Projects/autoweka/) is a Bayesian hyperparameter optimization layer on top of [WEKA](https://en.wikipedia.org/wiki/Weka_(machine_learning)).
	* [Auto-sklearn](https://automl.github.io/auto-sklearn/stable/) is a Bayesian hyperparameter optimization layer on top of [scikit-learn](https://en.wikipedia.org/wiki/Scikit-learn).
	* [TPOT](https://github.com/rhiever/tpot) is a [Python](https://en.wikipedia.org/wiki/Python_(programming_language)) library that automatically creates and optimizes full machine learning pipelines using [genetic programming](https://en.wikipedia.org/wiki/Genetic_programming).

* 深度神经网络架构搜索
	* [devol](https://github.com/joeddav/devol) is a Python package that performs Deep Neural Network architecture search using [genetic programming](https://en.wikipedia.org/wiki/Genetic_programming).
	* [Google AutoML](https://research.googleblog.com/2017/05/using-machine-learning-to-explore.html) for deep learning model architecture selection.



最后推荐[德国Freiburg大学AutoML实验室](https://www.ml4aad.org/),他们拿到了AutoML Challenge第一届(2015-2016)和第二届(2017-2018)的冠军，应该可以代表目前AutoML学术界的顶尖水平。



## 写在最后

Automated Machine Learning的技术目前不是特别成熟，所以孕育着机会，并伴随着挑战。但其本质仍然是机器学习问题，所以作为机器学习从业者有必要了解该技术，欢迎有兴趣的同学与笔者一起交流和讨论。