---
layout: post
title:  mermaid在markdown中生成gantt图
categories: [markdown,gantt]
---

最近工作中涉及到项目管理的内容。项目管理一般会用到gantt图。笔者不是专职PM，安装Project比较重，不够敏捷，所以尝试寻找轻量级解决方案。不负众望，果然找到标记语言[mermaid](https://mermaidjs.github.io/)，并且可无缝集成到typora。mermaid不仅支持gantt图，目前还支持flow chart和sequence chart，很适合展示模块之间的数据流或执行逻辑。

typora对memaid支持的非常好，具体操作参考[typora官方文档](https://support.typora.io/Draw-Diagrams-With-Markdown/)。代码示例如下

{% highlight raw%}
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram functionality to mermaid

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d

section Critical tasks
Completed task in the critical line :crit, done, 2014-01-06,24h
Implement parser and jison          :crit, done, after des1, 2d
Create tests for parser             :crit, active, 3d
Future task in critical line        :crit, 5d
Create tests for renderer           :2d
Add to mermaid                      :1d

section Documentation
Describe gantt syntax               :active, a1, after des1, 3d
Add gantt diagram to demo page      :after a1  , 20h
Add another diagram to demo page    :doc1, after a1  , 48h

section Last section
Describe gantt syntax               :after doc1, 3d
Add gantt diagram to demo page      : 20h
Add another diagram to demo page    : 48h
{% endhighlight %}



最开始是描述信息，定义图的类型，日期格式和标题。接下来，每个section就是一个项目。项目中，每一行就是一个任务。任务有名称和属性两部分组成，由英文冒号":"分开。属性意义：



* **done** 当前任务已经结束，如果没结束不用标记，必须为第一个。
* **crit** 当前任务很重要，用红色高亮显示，可用于标记里程碑，必须为第一个。
* **任务id** 必须全局唯一，可被后续任务引用，在状态后面（如果有状态描述）。此操作比较灵活，只需修改起始工作的时间，后置（**after**标记）任务时间全部会改变。如果没有后续依赖，也可省略任务id。
* 最后是任务时间，一般需要两个，开始和结束时间。可以是绝对时间或相对时间，非常灵活。



如果熟悉markdonw，对标记语言有一定了解，5分钟可以学会使用mermaid绘制gantt图管理项目。不过mermaid虽然敏捷，但是其gantt图还是有两个特性没有，



1. 无法按百分比显示当前项目进度。
2. 没有里程碑标记。



笔者个人理解mermaid是轻量级工具，gantt图的核心功能已实现，上面是非核心功能，可能后续会补足。



以上是使用memmaid绘制gantt图的一些理解，希望对读者的工作有帮助。



