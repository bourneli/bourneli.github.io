---
layout: post
title: Linux安装R记要 II
categories: [R]
---

4年前，写了一篇博文[Linux安装R记要](http://www.cnblogs.com/bourneli/archive/2013/09/04/3300887.html)，最近在一台新机器安装R，发现按照之前的记录操作不灵了。只能感叹时代在变化，需要与时俱进。当时安装的版本是3.2.5,而这次由于需要装xbgoost库，需要更高版本，所以用了最新的3.4.1版本。在经过一段踩坑之后，需要把这些坑记录下拉，避免今后再次踩坑。


主要参考了下面几篇博客

* [1] [R-3.3.1源码安装](http://kuxingseng2016.blog.51cto.com/1374617/1846326)
* [2] [解决openssl: error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory错误](http://www.cnblogs.com/xyb930826/p/6077348.html)
* [3] [linux 安装boost  ](http://blog.163.com/zhangjie_0303/blog/static/9908270620131131102442926)


主要的操作按照[1]中的描述，大部分没有有问题。但是可能出现openssl的问题。解决这个问题，首先需要安装openssl。然后安装libcurl时，需要使用with-ssl选项，指定openssl的安装目录。接下来，编译R时，仍然会有库找不到的问题，此时，按照[2]作两个软连接就可以解决问题。这个问题比较隐晦，因为它不会报动态链接找不到，而是报curl版本过低。其实是由于curl无法启动，因为找不到动态库。这个必须手动启动curl才能发现这个问题，深坑一个！软连接安装如下

```
ln -s [openssl_home]/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -s [openssl_home]/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
```


最后，会出现问题
```
libbz2.a: could not read symbols: Bad value
```
这时按照[3]，重新编译bzip2包，修改makeFile如下

将
```
CFLAGS=-Wall -Winline -O2 -g $(BIGFILES)
```
改成
```
CFLAGS=-Wall -Winline -O2 -g $(BIGFILES) -fPIC
```

总结完毕！
