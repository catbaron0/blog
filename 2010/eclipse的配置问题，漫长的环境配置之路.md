---
title: "eclipse的配置问题，漫长的环境配置之路"
date: "2010-07-15"
categories: 
  - "技"
tags: 
  - "eclipse"
  - "java"
  - "jsp"
  - "linux"
---

因为要和老师做jsp的开发，所以这两天就做了一些功课。win7下面的环境倒是还算顺利，一路配置下来，无非就是sdk，mysql，eclipse，tomcat。这些网上的教程都是大堆大堆的，这里就不罗嗦了。  
但是，最近一直尝试过渡到linux下面，所以就在ubuntu找了以下相关软件。  
需要的软件，navicat，eclipse，mysql等等都有linux版。（navicat的linux版是免费的，win下面是收费的还要破解……）不过中间出了一些问题。这里贴出来，希望那些通过谷歌找到这篇文章的同学能有所帮助。  
先说一下，我主要想说的是eclipse中出现java.sql无法引用的问题。  
关于这个问题，网上的说法很多都是说环境变量`CLASSPATH`配置的问题。从原理上来说，确实是有这个可能，但是我的不是这个问题。如果又遇到相同问题的同学，而且通过修改`classpath`没有解决，可以直接跳过一，从二开始。  

## 一.sdk的配置

因为要用到java和jsp，所以首先要装sdk。sdk去sdk的官方网站就可以下载。  
网上又终端的安装方法：

```bash
sudo apt-get install sun-java6-sdk
```

但是我使用的结果是失败。应该是源的问题。  
sdk的安装不是我们讨论的重点。我这里重点说一下环境变量的配置。  
网上所说的方法有很多，关于配置环境变量，我总结了以下，大概是修改这三个文件都可以成功（修改改一个就可以了）。分别是

```
～/.bashrc
/etc/profile
/etc/environment
```

修改的内容，主要是配置`PATH`，`CLASSPATH` ，`JAVA_HOME` 这三个变量。设定`JAVA_HOME` 主要是为了修改方便。真正要用到的应该只有`PATH`，和`CALSSPATH`。但是后两个要用到`JAVA_HOME`。  
`PATH`：这个类似于windows里面的`windows/system32`之类的路径。就是，当你在终端中输入一个命令，系统会去`PATH`中的路径找这个命令。linux的`PATH`本来应该包括`/bin`，`/sbin`之类的路径。而当你安装完sdk之后，在编译java文件的时候要用到`javac`这种`java`命令。所以我们要告诉系统去哪里找这些命令。  
我们假设你的sdk装在`～/sdk1.6`这里。那么我们的`javac`命令应该在`sdk1.6/bin`目录里面。所以我们要把它添加到`PATH`里面。  
在上面提到的三个文件里面最后面添加上

```
PATH="xxx：/home/sdk1.6/bin"
```

前面的`xxx`是系统本来的PATH设置，只要在后面加上我们自己的就 可以了。  
注意，在windows里面，系统环境变量的不同值之间用分号隔开，linux里面用冒号隔开。  
`CLASSPATH`：这个路径中包含了你在java中生成的类和java自带的类。如果设置失败可能会导致一些类的方法无法调用。  
一般我们要包括`～/sdk1.6/lib/tools.jar`，`～/sdk1.6/jre/lib/dt.jar`等文件。这些文件里面包含了我们常用的一些类，比如`java.sql`等。所以我们要在环境文件中添加

```
CLASSPATH=.:～/sdk1.6/lib/tools.jar：～/sdk1.6/jre/lib/dt.jar：$CLASSPATH
```

上面这句话里面要注意  
1\. 等号后面有一个“.”，作为第一个变量值，表示当前目录。后面分别是刚才提到的两个文件。  
2\. 最后面的`$CLASSPATH`是因为，防止已经有了这个`CLASSPATH`，如果我们不佳最后的`$CLASSPATH`，以前的变量会被这次的内容覆盖。加上最后这个符号，则表示添加内容。  
最后说一下`JAVA_HOME`。这个其实有点像是宏定义。因为我们可以看到，刚才配置的两更变量，中间路径用到了很多相同的路径`～/sdk1.6/`。所以我们就把它定义成`JAVA_HOME`，使用起来比较方便。这样再顶一以上的变量的时候，就可以通过`%JAVA_HOME%`来引用`～/sdk1.6/`了。而且一旦以后要修改环境变量，我们直接修改`JAVA_HOME`就行了。

```
JAVA_HOME=～/sdk1.6/
```

## 二.eclipse的错误

eclipse安装过程我就不罗嗦了。就是解压，改权限而已。  
我吧一个工程考过来，然后设置eclipse的服务器tomcat。配完之后，选中`index.jsp`文件，选择`run as...`\-->`run on server`。在语句“import=java.sql.”提示错误

```
The import java.sql cannot be resolved
```

说明这个包引入失败  
csdn里面有人遇到类似的问题，说是sdk没配置好导致的。于是我才看了大量的sdk的配置的东西……  
然后查了资料，发现`java.sql`是在`rt.jar`里面。所以更加坚信了，是`CLASSPATH`的配置有了问题。  
结果，我就离真相越来越远。  
真相在[这里](http://blog.csdn.net/JeamKing/archive/2010/04/30/5544896.aspx)：  
其实很简单，只是当时我找错了方向而已。不过作为补偿，倒是了解了不少sdk和jdbc的原理。  
关于jdbc，mysql驱动，我会再写总结的。

## 三.Eclipse中的中文乱码。

关于这个乱码，其实大多数是在windows下面是好的，到了ubuntu就成乱码了。这是因为，ubuntu默认的中文编码是utf8，而非gbk或者gb18030。所以我们要手动位ubuntu添加gbk编码。编辑`/var/lib/locales/supported.d/local`文件，在后面添加

```
zh_CN.GBK GBK
zh_CN.GB18030 GB18030
```

然后终端运行

```
sudo locale-gen
```

之后，ubuntu环境就可以支持gbk编码了  
我们转到eclipse下面，对文件或者堆工程进行操作  
选中文件活工程，右击，选择properties（首选项）->resource（资源）->text file encoding，编码选项里面没有gbk和gb18030，需要自己填写。填上需要的编码，apply应用之后，应该乱码就正常了。  
好了，电脑也快没电了  
总之，从昨天开始，面对这些错误，不停地谷歌谷歌谷歌，却一直得不到解决，确实很打击我。但是最终还是找到了解决方法。  
我有自己的野心，必须面对自己。  
希望志同道合的同学，一起进步吧。  
以上。
