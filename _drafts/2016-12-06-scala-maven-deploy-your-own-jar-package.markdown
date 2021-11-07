---
layout: post
title:  在maven库中上传你自己开发的第三方包
categories: [maven,scala]
---

将开发的通用库上传到mvn库中，其他同学使用会比较方便。本文记录相关配置，以及此过程中遇到的坑。


## 设置全局setting.xml
settingsx.xml需要添加的记录

	<servers>
		<server>
			 <id>snapshots</id>
			 <username>[your_name]</username>
			 <password>[your_password]</password>
		</server>
		<server>
			 <id>releases</id>
			 <username>[your_name]</username>
			 <password>[your_password]</password>
		</server>
	</servers>

主要是设定两个server的id和账号，snapshots用于调试，releases用于发布。前者版本可以覆盖，后者版本发布后无法覆盖，版本号必须变化。

## 设置组件pom.xml
组件pom.xml需要添加的记录如下

	<!-- 配置远程发布到私服，mvn deploy -->
	<distributionManagement>
		<repository>
			<id>releases</id>
			<url>http://[repo_host]/nexus/content/repositories/thirdparty</url>
		</repository>
		<snapshotRepository>
			<id>snapshots</id>
			<url>http://[repo_host]/nexus/content/repositories/thirdparty-snapshots</url>
		</snapshotRepository>
	</distributionManagement>

这里的**release**和**snapshots**与setting.xml对应的。这里的组件pom.xml就是要上传到maven库供其他人使用的pom.xml。设置上面的配置有两个目的：一个是通过id找到账号，第二个是告诉mvn上传的具体目录。

## 设置调用组件的项目pom.xml

现在就可以类似引用其他项目一样，添加下面的dependency

	<dependency>
      <groupId>your.package.id</groupId>
      <artifactId>your_package_name</artifactId>
      <version>0.0.1</version>
    </dependency>

但是，还不够，还需要指定第三方maven库的配置，否则还是找不到，如下

	<repositories>
        <repository>
            <id>release-repo</id>
            <url>http://[repo_host]/nexus/content/repositories/thirdparty</url>
        </repository>
        <repository>
            <id>snapshot-repo</id>
            <url>http://[repo_host]/nexus/content/repositories/thirdparty-snapshots</url>
        </repository>
    </repositories>

其实，上面的repository也可以设置在setting.xml中。上面两个pom.xml中的url需要保持一致。


## 部署snapshots包

使用mvn命令部署，需要将本地maven配置path路径，这样可以在命令行中执行mvn命令。可直接用Intellij目录下plugin中的maven3/bin目录即可。

上传本地snapshots包到maven服务器，命令如下

	mvn deploy:deploy-file -e -DgroupId=your.package.id -DartifactId=your_package_name -Dversion=1.0.0-SNAPSHOT -Dpackaging=jar -Dfile=target/your_package_name-1.0.0-SNAPSHOT.jar -DrepositoryId=snapshots -Durl=http://[repo_host]/nexus/content/repositories/thirdparty-snapshots
	
我将上面的命令放到一个脚本中，并且将脚本放到pom.xml同级目录。

## 部署releases包

如果是上传release版本，可以使用类似下面的命令，

	mvn deploy:deploy-file -e -DgroupId=your.package.id -DartifactId=your_package_name -Dversion=0.0.1 -Dpackaging=jar -Dfile=target/your_package_name-0.0.1.jar -DrepositoryId=releases -Durl=http://[repo_host]/nexus/content/repositories/thirdparty

配置时，参数**-DrepositoryId**写成了**release**，少写了一个**s**，导致一直报401错误，这个错误比较有迷惑性。之前一直以为是密码错误，所以一直没有找到问题的根源，后来在yunshandi同学的帮助下，发现原来是打错了id。releases版本，一个版本号只能对应一个jar包，不能覆盖，这样保证版本的稳定性，如果尝试覆盖releases版本，会报400错误。


上传完毕后，可以在maven站点上搜素到相关的包。并且可以在其他工程中应用，非常方便。


最后，感谢[黄博gotoli](www.algorithmdog.com)和TEG同事yunshandi。