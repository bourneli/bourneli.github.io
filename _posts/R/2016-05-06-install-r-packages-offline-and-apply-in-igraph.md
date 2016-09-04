---
layout: post
title:  "离线安装R扩展包并应用到网络分析包igraph"
categories: [R,igraph]
---

## 背景
在有网络的情况下，安装R扩展包非常容易，只需要`install.packages('package_name')`，R就会下载最新版本并安装，如果有依赖，R会自动迭代的下载依赖并逐个安装，非常方便。但是，出于某些原因（主要是安全），计算机无法访问互联网，安装R扩展包就比较麻烦。比如igraph 1.0.1版依赖39个包，大多数不是常用的。如果手动到cran上去逐个下载、编译并安装，假设每个包需要10分钟，那么大概需要6个半小时。本文介绍一种方案，执行半自动化的离线安装R扩展包，并应用在igraph上，将整个过程时间缩短到半小时左右，前提是需要有一台可以访问互联网的计算机。

## 安装思路
在有互联网的计算机上，获取目标包的所有依赖库的名称，包括非直接依赖的包。然后，批量下载这些包并上传到那台不能连接互联网的机器上的指定目录中。给上传的所有包制作索引。最后，使用在`install.packages`安装。

## 安装igraph
最近的工作需要使用igraph包，下面就用上面的方法离线安装igraph，并且记录安装过程中遇到的几个坑以及解决方案。

{% highlight R%}
# 迭代获取所有依赖
getPackages <- function(packs){
  packages <- unlist(
    tools::package_dependencies(packs, available.packages(),
                                which=c("Depends", "Imports"), recursive=TRUE)
  )
  packages <- union(packs, packages)
  packages
}
packages <- getPackages(c("igraph"))
packages

# 批量下载所有依赖源代码
download.packages(packages, destdir="D:\\mnet\\igraph_dep", type="source")
{% endhighlight %}

依赖的包如下：

{% highlight raw %}
> packages
 [1] "igraph"       "methods"      "Matrix"       "magrittr"    
 [5] "NMF"          "irlba"        "stats"        "graphics"    
 [9] "grid"         "utils"        "lattice"      "pkgmaker"    
[13] "registry"     "rngtools"     "cluster"      "stringr"     
[17] "digest"       "grDevices"    "gridBase"     "colorspace"  
[21] "RColorBrewer" "foreach"      "doParallel"   "ggplot2"     
[25] "reshape2"     "iterators"    "parallel"     "codetools"   
[29] "gtable"       "MASS"         "plyr"         "scales"      
[33] "tools"        "xtable"       "Rcpp"         "stringi"     
[37] "dichromat"    "munsell"      "labeling"   
{% endhighlight %}

将下载的所有包的源代码上传到无法连接网络的机器上，并在该机器上执行下面的代码，用于生成索引，并安装。
{% highlight R %}
library(tools)
write_PACKAGES("/path/to/packages/")
install.packages("igraph", contriburl="file:///path/to/packages/")
{% endhighlight %}

P.S.: 上面**contriburl**参数中有三个“**/**”。

正常情况下，按照上面的过程，可以顺利完成包的依赖，但是igraph离线安装过程中遇到了两个坑，这里顺便记录一下，

### stringi安装时默认要求连接互联网
stringi是igraph的一个依赖包，在安装时，默认会到指定站点下载数据包icu52l.zip。由于服务器无法连接网络，自然无法下载，那么安装就会停止，后续所有的过程都无法执行。好在，可以通过参数指定stringi安装时，到指定路径获取icu52l.zip，所以你需要先手动下载icu52l.zip，然后上传到服务器指定地址即可，安装命令参考如下：

{% highlight R %}
install.packages("stringi", 
    contriburl="file:///path/to/packages/", 
    configure.vars="ICUDT_DIR=/path/to/data")  
{% endhighlight %}
这样，R会到执行路劲下获取icu52l.zip，而不会去互联网上下载。

### libxml2版本过低
igraph 1.0.1依赖libxml2 2.7.3版本及以上，如果版本过低，igraph编译会有语法错误通过，所以最好在系统级别重新安装符合版本要求的libxml2，之后可以顺利安装igraph。

希望以上过程对你有用！

## 参考资料
* [获取所有依赖包名称并批量下载](http://stackoverflow.com/questions/6281322/only-download-sources-of-a-package-and-all-dependencies/15650828#15650828)
* [设置本地R库](http://stackoverflow.com/questions/10807804/offline-install-of-r-package-and-dependencies/10841614#10841614)
* [stringi离线安装时手动设置数据源](http://stackoverflow.com/questions/27553452/how-to-install-stringi-library-from-archive-and-install-the-local-icu52l-zip/28530498#28530498)
* [libxml2版本低于igraph要求的最低版本](https://lists.nongnu.org/archive/html/igraph-help/2015-10/msg00022.html)