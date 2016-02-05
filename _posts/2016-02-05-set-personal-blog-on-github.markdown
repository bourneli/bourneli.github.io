---
layout: post
title:  "配置github个人主页踩过的坑"
date:   2016-02-05 10:31:28 +0800
categories:
---

最近年末，终于有时间配置自己的github个人主页。踩了一些坑，总结一下，希望对其他同学有帮组。

# 环境要求
* 可以畅通访问github等国外网站，大环境是这样，你懂的 :)
* 注册github账号
* 建议在mac配置本地环境，使用windows也可以（我就是win 8），因为ruby支持类unix系统比windows好太多

# 大致过程
github提供静态blog所需的一切，包括流量，空间，服务器托管。github官方建议使用ruby下的jekyll模块生成静态站点，用于维护css，header，等公共部分。编写markdown格式的blog（这也是为什么转投github的一个原因）。

# 参考文档
* [如何创建github规范的博客仓库](https://pages.github.com/)
* [Windwos上安装ruby](http://jekyll-windows.juthilo.com/)
* [手把手安装jekyll等依赖](https://help.github.com/articles/using-jekyll-with-pages/)

# 踩过的“坑”
* 安装了*github-pages*包后，无需单独安装*jekyll*，
* 目录下需要有Gemfile文件，内容为[手把手安装jekyll等依赖](https://help.github.com/articles/using-jekyll-with-pages/)中对应。
* 创建了jekyll的框架后，手动cp到你的本地仓库中，然后add,commit,push到github上，就可以访问了。
* 启动命令 ```bundle exec jekyll serve```,本地url[http://localhost:4000](http://localhost:4000)

希望这些内容对你有用。