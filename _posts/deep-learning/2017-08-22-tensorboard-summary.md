---
layout: post
title:  TensorBoard使用小结
categories: [deep-learning,tensorflow,tesorboard]
---


今天尝试使用Tensorboard调试模型，还是踩了一些坑，记录在这里，方便后面回顾。Tensorboard是Tensoflow配套的日志记录和分析工具。tensorflow提供api将日志写入到指定目录，然后启动tensorflow进程，即可通过浏览器显示模型执行时的各种参数，即使建模完毕，仍然可以观察整个过程，这一点非常赞。

在Windows的Anaconda环境上使用TensorBoard有个坑，启动TensorBoard的目录和模型目录必须在同一个磁盘分区，否则就无法找到日志信息。比如在**c盘**启动TensorBoard,但是日志却在**d盘**中的某个目录，那么即使日志成功写入，TensorBoard也只会展示下面的信息

![file not found](/img/tensorboard_not_found.png)


在tensorflow中使用TensorBoard api的步骤总结如下，

1. 定义summary变量，必须关联一个tensor，并且类型必须一致，scalar summary必须关联scalar tensor，否则会报错，血泪教训。
2. 使用with组织你的summary变量，方便在tensorboard中有组织的显示
3. merge变量，方便一起调用
4. 定义写入管道，一般定义train和test两个管道，可以方便记录loss在训练数据和验证数据上的学习曲线，方便分析模型学习程度。
5. 执行计算流。


当然还有很多细节没有交代，更多细节请参考[TensorBoard官方资料](https://www.tensorflow.org/get_started/monitors)。
