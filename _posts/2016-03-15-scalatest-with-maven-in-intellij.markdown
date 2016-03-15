---
layout: post
title:  Scalatest集成到Intellij
categories: [scala]
---

[scalatest maven插件](http://www.scalatest.org/user_guide/using_the_scalatest_maven_plugin)需要至少java 1.8版本，但是公司的开发环境java版本只到1.7，所以无法使用。使用maven自带的单元测试插接[maven-surefire-plugin](https://maven.apache.org/surefire/maven-surefire-plugin/usage.html)，其实已经足够，关键是需要定义好后缀，参考下面的配置片段。

{% highlight raw %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.13</version>
    <configuration>
        <useFile>false</useFile>
        <disableXmlReport>true</disableXmlReport>
        <!-- If you have classpath issue like NoDefClassError,... -->
        <!-- useManifestOnlyJar>false</useManifestOnlyJar -->
        <includes>
            <include>**/*Test.*</include>
            <include>**/*Suite.*</include>
            <include>**/*Demo.*</include>
        </includes>
    </configuration>
</plugin>
{% endhighlight %}

同时，要添加scalatest依赖，否则编译无法通过，如下

{% highlight raw %}
<dependency>
    <groupId>org.scalatest</groupId>
    <artifactId>scalatest_2.10</artifactId>
    <version>2.2.6</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}

最后，也是最关键的一点，在每个类上面添加[JUnitRunner执行器标志](http://stackoverflow.com/a/4663014/1114397)，并且使用Scalatest框架类（各种scalatest风格随便使用），而不是junit框架类，否则无法执行，如下

{% highlight raw %}
import org.scalatest._
import org.junit.runner.RunWith
import org.scalatest.junit.JUnitRunner

@RunWith(classOf[JUnitRunner])
class DenseVectorSuite extends FunSuite {
    val dv1 = DenseVector(1.0f, 2.0f, 3.0f)
    val dv2 = DenseVector(4.0f, 5.0f, 6.0f)

    test("norm") {
        assert(6.0 == 6.0f)
    }
}
{% endhighlight %}

虽然有点费解，juint和scalatest混合在一起，搞得有点晕，但是这是无法使用scalatest maven插件的情况下，可以正常执行scalatest框架的权益方法。
按照上面的配置，使用“mvn test”命令即可自动寻找相关类，然后执行test()方法中的测试用例。

输出效果
{% highlight raw %}
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.apache.spark.mllib.util.tencent.DenseVector2Demo
Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.852 sec
Running org.apache.spark.mllib.util.tencent.DenseVectorSuite
Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.013 sec
Running org.apache.spark.mllib.util.tencent.TestMainSuite
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 sec

Results :

Tests run: 11, Failures: 0, Errors: 0, Skipped: 0
{% endhighlight %}

步骤总结

* 使用maven-surefire-plugin作为单元测试插件，并添加相关配置
* 添加scalatest依赖
* 使用scalatest框架开发测试用例，如继承FunSuite和test方法中写测试用例
* 测试类的后缀与maven-surefire-plugin中的配置保持一致
* `mvn test`执行测试