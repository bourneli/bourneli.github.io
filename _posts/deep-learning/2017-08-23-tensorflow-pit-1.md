---
layout: post
title:  TensorFlow踩坑记要(1)
categories: [tensorflow]
---

最近在学习tensorflow，学习过程中遇到一些问题，在这里开一个系列，记录遇到的坑，解决方法和技巧。今天要记录的内容总结如下：

* 避免在初始化变量后构建tensor;
* 尽可能在session.run执行完可以执行的tensor，可提高计算效率;
* 常用二元分类度量tensor;
* tf.gather切分tensor的例子;

## 避免在初始化变量后构建tensor

在计算auc时，需要使用tf.metrics.auc这个函数构建tensor。Tensorflow先构建计算图，然后再开始计算，一旦开始计算，就不能修改图，否则会报运行时错误，

{% highlight python %}
sess.run(tf.global_variables_initializer())
sess.run(tf.local_variables_initializer())

# ... 建模过程，这里省略

auc = tf.metrics.auc(...)
sess.run(auc,[...])
{% endhighlight %}

代码执行时，总是抛出异常，导致意思是部分内部tensor没有初始化，在网上找到了[解决方案](https://stackoverflow.com/questions/39435341/how-to-calculate-auc-with-tensorflow)，该方案确实没有问题，但是我的问题不是这里，正确的解决方法是将auc的定义提前init前，这样才能够被初始化，代码如下，



{% highlight python %}
auc = tf.metrics.auc(...) # 定义提前

sess.run(tf.global_variables_initializer())
sess.run(tf.local_variables_initializer())

# ... 建模过程，这里省略

sess.run(auc,[...])
{% endhighlight %}

修改后，问题迎刃而解。感谢[xiaoxiwang(王晓曦)](https://github.com/vshallc)同学指点迷津，尽快走出了这个坑，才有时间王者上钻石。



## 尽可能在session.run执行完可以执行的tensor

假如有多个tensor，复用同一个feed_dict，建议一次性批量执行run，而不是要分开run，这样可以提高执行效率，如下示例

{% highlight python %}

# Block 1
t1_rst, t2_rst, t3_rst = sess.run([t1,t2,t3], feed_dict = some_dict)  # 快

# Block 2
t1_rst = sess.run(t1, feed_dict = some_dict)  
t2_rst = sess.run(t2, feed_dict = some_dict)
t3_rst = sess.run(t3, feed_dict = some_dict)  

{% endhighlight %}

Block 1与Block 2的效果一样，但是Block 1块很多，亲测有效。



## 常用二元分类度量tensor

* tf.metrics.recall
* tf.metrics.precision
* tf.metrics.auc
* tf.metrics.accuracy

这些tesnfor的输入基本上可以通用，非常方便。



## tf.gather切分tensor的例子

tf.gather用于切分tensor，取出需要的部分，是非常好用的函数。假如有如下tensor

{% highlight python %}

t = tf.Variable([[1, 1.1],
                 [2, 2.2],
                 [3, 3.3],
                 [4, 4.4]])

tf.gather(t, axis = 1, indices = 0) # 第一列 [1, 2, 3, 4]
tf.gather(t, axis = 1, indices = 1) # 第二列 [1.1, 2.2, 3.3, 4.4]

{% endhighlight %}

tensor t的shape=(4,2)，使用axis指定是4的维度还是2的维度，所以如果取列，axis = 1，如果取行，axis = 0。


ok，今天就记录这么多。
