---
layout: post
title:  Intellij上Scala的Maven项目打包本地第三方jar包
categories: [maven,scala]
---


使用Intellij+maven+scala开发spark程序，有时候需要引入不在maven库中的第三方jar包，也就是所谓的本地jar包。今天在导入时，还是遇到了一定的麻烦，这里记录一下。

## 主要解决方法参考[这里](http://stackoverflow.com/a/31023523/1114397)!

## 主要解决方法参考[这里](http://stackoverflow.com/a/31023523/1114397)!

## 主要解决方法参考[这里](http://stackoverflow.com/a/31023523/1114397)!

重要的事情说三篇。上面的主要思路是安装一个maven插件，然后将本地jar包放入maven本地库，这样最后打包时就无缝集成到maven原有的在线安装机制中了。当然，还需要Intellij知道这个第三方jar包，导航到**File-> Project Structure -> Modules -> Dependencies**，然后点击右边的“+”号，添加本地jar包即可。


今天遇到的一个主要问题就是将路径直接写在systemPath中，类似下面

	<dependency>
	    <groupId>com.loopj.android.http</groupId>
	    <artifactId>android-async-http</artifactId>
	    <version>1.3.2</version>
	    <type>jar</type>
	    <scope>system</scope>
	    <systemPath>${project.basedir}/libs/android-async-http-1.3.2.jar</systemPath>
	</dependency>

出现的问题就是最后生成的jar包中没有那个需要打包的jar包，等于没有打包。不过不会报错，比较难以发现，我是根据warning中的信息才顺藤摸瓜找到解决方案的。


## 更新第三jar包遇到的坑

如果第三方jar包有更新，需要更新本地库对应的jar包，也就是需要手动删除**~/.m2/repository**里面对应的jar包缓存，否则更新不会成功。官方文档描述如下

	The local repository is the local cache where all artifacts needed for the
	build are stored. By default, it is located within the user's home directory
	(~/.m2/repository) but the location can be configured in ~/.m2/settings.xml
	 using the <localRepository> element.

官方文档可以参考[这里](http://maven.apache.org/plugins/maven-install-plugin/)。