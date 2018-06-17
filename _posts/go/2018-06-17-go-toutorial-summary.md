---
layout: post
title:  Go之语法特性总结
categories: [Go]
---



最近抽了几个小时快速浏览了Go的特性，参考其官方指引[A Tour of Go](https://tour.golang.org/list)，如果本地安装了go环境，也可以通过命令`go tool tour`直接访问本地版本。总结一下我的感受，

* 简洁
* 为服务开发而生

官方指引总将Go的特性总结为5块，

* [Packages, variables, and functions.](https://tour.golang.org/basics) 介绍基础数据类型，变量，包和函数等特性。
* [Flow control statements: for, if, else, switch and defer.](https://tour.golang.org/flowcontrol) 逻辑控制，与主流语言基本相同，但是却将一些可有可无的内容进行简化，比如G的循环没有while，全是for。
* [More types: structs, slices, and maps.](https://tour.golang.org/moretypes) 基本容器以及相关操作。
* [Methods and interfaces.](https://tour.golang.org/methods)Go中没有类，而是将类拆解为Method，Interface和Struct，可能绝对OO太重。基于性能考虑，Go中显示支持指针，开发者可以自己决定是否需要值拷贝。由于指针直接操作内存，容易出错，除了C/C++，主流语言很少支持指针。一向以简介而著称的Go的竟然支持此特性，侧面说明Go十分注重性能。 
* [Concurrency.](https://tour.golang.org/concurrency) 此特性足以说明Go为服务开发而生。Go将多线程，管道通信，锁等服务器开发基础特征直接作为内置特性支持，而不是想其他语言那样作为扩展部分。

所以，从目前来看，Go的简洁和服务器开发的特性，看似可以解决我们的燃眉之急。实践是检验真理的唯一标准，希望几个月回头看这篇博客时，我的脸上是挂着微笑。

