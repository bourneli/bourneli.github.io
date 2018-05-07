---
layout: post
title: R中的函数编程包purrr
categories: [R]
---



最近由于工作内容有些变动，需要一段时间调整，所以好久没有更新博客了。不过这个不能作为借口，手动捂脸！最近需要用R做一些快速的试验，但是数据是时序化的，使用R进行ETL比较麻烦（原先这么认为）。本打算使用python ETL，然后R分析建模。Python虽然灵活，但是这样与R割裂开，后面实验的效率不高，因为需要来回切换工具。磨刀不误砍柴工，为什么不尝试找找R中是否有相关的特性呢，说不定打开了另外一扇大门。

功夫不负有心人，终于google到了这个R包--**purrr**，其官方介绍如下：


purrr enhances R's functional programming (FP) toolkit by providing a complete and consistent set of tools for working with functions and vectors. If you've never heard of FP before, the best place to start is the family of map() functions which allow you to replace many for loops with code that is both more succinct and easier to read. The best place to learn about the map() functions is the iteration chapter in R for data science.

purrr可以在R上使用[函数编程](https://en.wikipedia.org/wiki/Functional_programming)的特性，非常类似工作中常用的scala/spark。如果具有一定函数编程经验，建议直接阅读并执行官方例子，并且看官方文档，

{% highlight R %}
require(purrr)

# NOT RUN {
1:10 %>%
  map(rnorm, n = 10) %>%
  map_dbl(mean)

# Or use an anonymous function
1:10 %>%
  map(function(x) rnorm(10, x))

# Or a formula
1:10 %>%
  map(~ rnorm(10, .x))

# Extract by name or position
# .default specifies value for elements that are missing or NULL
l1 <- list(list(a = 1L), list(a = NULL, b = 2L), list(b = 3L))
l1 %>% map("a", .default = "???")
l1 %>% map_int("b", .default = NA)
l1 %>% map_int(2, .default = NA)

# Supply multiple values to index deeply into a list
l2 <- list(
  list(num = 1:3,     letters[1:3]),
  list(num = 101:103, letters[4:6]),
  list()
)
l2 %>% map(c(2, 2))

# Use a list to build an extractor that mixes numeric indices and names,
# and .default to provide a default value if the element does not exist
l2 %>% map(list("num", 3))
l2 %>% map_int(list("num", 3), .default = NA)

# A more realistic example: split a data frame into pieces, fit a
# model to each piece, summarise and extract R^2
mtcars %>%
  split(.$cyl) %>%
  map(~ lm(mpg ~ wt, data = .x)) %>%
  map(summary) %>%
  map_dbl("r.squared")

# Use map_lgl(), map_dbl(), etc to reduce to a vector.
# * list
mtcars %>% map(sum)
# * vector
mtcars %>% map_dbl(sum)

# If each element of the output is a data frame, use
# map_dfr to row-bind them together:
mtcars %>%
  split(.$cyl) %>%
  map(~ lm(mpg ~ wt, data = .x)) %>%
  map_dfr(~ as.data.frame(t(as.matrix(coef(.)))))
# (if you also want to preserve the variable names see
# the broom package)
# }
{% endhighlight %}



总结一些常用函数

* %>% 管道符号，用于将操作串联起来
* map 函数编程中最常用的遍历方法
* split 类似groupBy,非常好用
* keep 类似filter，可以过滤掉无用数据，有个反操作discard
* map_dfr 此函数对于我当前工作非常重要，它可以输入data.frame输出仍为data.frame，而且每个单元计算出的结果也可以多行，类似flatmap。常用R库plyr中的没有类似函数。









