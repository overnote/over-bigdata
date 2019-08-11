## 一 Scala简介

Scala是一种多范式的编程语言，其设计的初衷是要集成面向对象编程和函数式编程的各种特性。Scala运行于Java平台（Java虚拟机），并兼容现有的Java程序。  

Scala特色：
- 1.优雅：这是框架设计师第一个要考虑的问题，框架的用户是应用开发程序员，API是否优雅直接影响用户体验。
- 2.速度快：Scala语言表达能力强，一行代码抵得上Java多行，开发速度快；Scala是静态编译的，所以和JRuby,Groovy比起来速度会快很多。
- 3.能融合到Hadoop生态圈：Hadoop现在是大数据事实标准，Spark并不是要取代Hadoop，而是要完善Hadoop生态。JVM语言大部分可能会想到Java，但Java做出来的API太丑，或者想实现一个优雅的API太费劲。 

## 二 Scala安装

#### 2.0 安装前提

因为Scala是运行在JVM平台上的，所以安装Scala之前要安装JDK，开发工具推荐使用IDEA+Scala插件。  

#### 2.1 win/mac安装

访问Scala官网http://www.scala-lang.org/下载Scala编译器安装包，推荐下载 scala-2.10 版本。  

#### 2.2 linux安装

下载Scala地址http://downloads.typesafe.com/scala/2.10.6/scala-2.10.6.tgz
```
# 解压
tar -zxvf scala-2.10.6.tgz -C /usr/local

# 配置环境变量，将scala加入到PATH中
vim /etc/profile
export JAVA_HOME=/usr/java/jdk1.7.0_45
export PATH=$PATH:$JAVA_HOME/bin:/usr/localscala-2.10.6/bin
```
