---
layout: post
title:  如何在Windows上与TensorFlow愉快的玩耍？
categories: [deep-learning,tensorflow]
---

公司的集群有部署好的TensorFlow，但是使用起来不是特别方便，每次部署并等待调度非常耗时。对于初学者，由于很多概念不太熟悉，希望立刻输入就有反馈，这样的学习效率会比较高，所以还是在本地安装了TensorFlow。

我是在Windows Anaconda环境下安装的TensorFlow，这样可以避免影响本地的python环境。个人认为，Anaconda类似“虚拟机”，用的时候启动，该环境中的各种依赖都可以使用，不用的时候就关掉。同时，使用pycharm关联Anaconda环境下的TensorFlow，可以在本地执行很多TensorFlow官网的样例程序，非常赞。

### 安装步骤

具体安装步骤可以参考[官网指导](https://www.tensorflow.org/install/install_windows)，下面简要记录在Anaconda环境下的安装过程。

* 安装[Anaconda](https://www.continuum.io/downloads)。
* 制作Anacode TensorFlow环境（Anaconda可以同时支持多个环境，之前我就安装了cntk的环境），命令如下
`conda create -n tensorflow python=3.5`
整个过程需要在联网环境下进行，这是Anacoda会自动下载Python 3.5，并且独立于本地其它python版本。
* 安装好tensorflow环境后，此时TensorFlow还没有安装，只是对这个环境其了一个名字为“TensorFlow”。现在，可以用cmd启动该环境
{% highlight raw %}
C:> activate tensorflow
 (tensorflow)C:>  # Your prompt should change
{% endhighlight %}
* 安装Tensorflow，由于我的机器没有GPU，所以安装了CPU版本，
`(tensorflow)C:> pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/windows/cpu/tensorflow-1.3.0-cp35-cp35m-win_amd64.whl `
* 安装过程会下载一些依赖包，总体来说不是很多。最后检查一下安装的依赖包，使用`pip list`查看安装包。很好，没有那些注入木马的依赖包，可以放心的玩耍。
![pip list](/img/pip_list_tensorflow.png "TensorFlow依赖包")
* 检查安装是否成功。在上面的换进中，启动python交互环境，然后输入下面的代码

{% highlight raw %}
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
{% endhighlight %}


如果打印出了“Hello, TensorFlow!”，说明安装成功。

### pycharm关联Tensorflow

TensorFlow环境安装好了，接下来需要让IDE识别出TensorFlow的相关库，避免报错。我使用的是Intellij系列IDE---[pycharm](https://www.jetbrains.com/pycharm/download/)。风格与Intellij非常类似，但是只有python，所以比较轻量级。这个[问答](https://stackoverflow.com/questions/41767788/how-can-i-access-different-anaconda-environment-from-pycharm-on-windows-10)详细描述了如何让pycharm关联anaconda任意环境。主要步骤就是找到你的Anaconda TensorFlow环境中的python.exe，一般是`<ANACONDA_HOME>/envs/tensorflow/python.exe`。

### 启动Tensorboard

Tensorboard用来观察tensorflow执行时，各种指标的工具，作用十分类似spark web ui。最主要的使用场景是观察模型loss是否收敛。使用的示例程序参考这篇[官方文档](https://www.tensorflow.org/get_started/monitors)。一般在程序执行完后，如果指定了log，那么就可以通过tensorboard观察整个执行过程。我这里遇到了一个坑。官方文档里面没有指定port，但是我的本地在执行tensorboard命令后没有显示断后，所以需要我手动指定端口，才可以观察到下面的效果，具体命令如下。

```
tensorboard --logdir=log --port=8000
```

这样，才可已在[http://localhost:8000](http://localhost:8000)访问到下面的内容。

![tensorboard](/img/tensorboard_snapshot.png "TensorBoard例图")



环境搭建完毕，这样就可以愉快的与TensorFlow玩耍了！
