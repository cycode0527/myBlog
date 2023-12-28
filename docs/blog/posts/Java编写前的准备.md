---
date: 2021-08-07 
slug: preparation-before-using-java
categories: 
    - 编程语言
tags: 
    - Java
    - 编程语言 
readtime: 5
---


# Java编写前的准备

!!!warning "黑历史提醒"
    本文系我当初在入坑编程学习后所作的第二篇博客，可能已经过时，请自行斟酌本文的信息准确性和时效性，本文仅为我从当初网站备份中考古并留存纪念。
    本文可能的过时信息：

    - 现在推荐学习和使用较新版本的 JDK，例如 JDK17
    - 现在基本不再使用 eclipse ，而基本改用 IntelliJ IDEA
    - 现在作者不再使用 Windows10 而改用 Windows11， 自行更改部分内容。

    本文雷点：

    - 图过多过杂
    - 删除线跟不要钱一样乱塞

<!-- more -->

## Java开发环境的搭建
### 了解JDK
首先，想要编写Java程序，必须要有Java对吧。所以我们需要安装一个Java。
重点来了，想要编写java程序我们需要JDK而不是JRE，平常我们想要玩Minecraft（我的世界）的时候要求下载Java，那个是JRE，两者之间的区别在于：**JRE是Java程序运行所需环境，就是说想玩Minecraft要靠它；JDK包括了JRE，编写并编译Java语言要靠它。同学们在下载时要多加小心。**



![JDK的构成](https://repo.sxrhhh.top/QQ%25E5%259B%25BE%25E7%2589%258720210807233129.png?imageSlim "JDK的构成")

### 下载JDK
JDK的版本有很多，~~最新的好像有JDK SE16之类的，这事我也没搞明白，反正选JDK8就是了~~，强烈推荐下载[JDK8](https://www.oracle.com/cn/java/technologies/javase/javase-jdk8-downloads.html "JDK8")，反正看准8就行。[迅雷](https://pan.xunlei.com/s/VMgWInVj09PjkS5J-zryiTjFA1 "迅雷")***提取码：ixqk***
### 安装JDK
好了，现在我们已经下载好了JDK，点开它，开始安装 ~~（正好电脑重装我也没有安装JDK，一起开始吧）~~。
大概要等个一分钟不到，我们进入了安装页面

![JDK安装页面](https://repo.sxrhhh.top/QQ%25E6%2588%25AA%25E5%259B%25BE20210808000222.png?imageSlim "JDK安装页面")

当然你也可以把“公共JRE那里叉掉（好像叉不掉？）”
一路下一步，那么就等待读条完成后，会蹦出一个窗口，那就是JRE安装页面，记住它安装的路径，一路下一步，就完成了。

![JRE安装界面](https://repo.sxrhhh.top/JRE%25E5%25AE%2589%25E8%25A3%2585%25E7%2595%258C%25E9%259D%25A2.png?imageSlim "JRE安装界面")

完事后你也可以单独安装我[迅雷](https://pan.xunlei.com/s/VMgWInVj09PjkS5J-zryiTjFA1 "迅雷")里的JRE，单独安装JRE似乎能解决一些不必要的麻烦。

### 配置JDK环境变量
~~肯定有人要问了啊：什么是环境变量了啊？~~，首先我要讲一讲环境变量的事。
你现在 ~~立刻马上~~ 打开cmd，输入javac和java -version看看效果

![cmd测试javac](https://repo.sxrhhh.top/cmd.png?imageSlim "cmd测试javac")

然后你会 ~~惊奇地~~ 发现：**javac没有什么用。javac是一个java语言编译命令，负责把java语言翻译成class语言让机器阅读。总之，你现在写了java程序也没用**。为什么会这样呢？首先我们先要了解到：**命令其实就是一个软件，一般就是exe文件**，找一找他在哪？

![javac位置](https://repo.sxrhhh.top/javac%25E4%25BD%258D%25E7%25BD%25AE.png?imageSlim "javac位置")

没错，**他就在java安装目录下面的bin文件夹里**。但是cmd不知道他在这，怎么办？我们要让这块地方被cmd所知道。这就是环境变量的作用。
我们先打开**“计算机”（或是“此电脑”）的属性**

![属性](https://repo.sxrhhh.top/%25E7%258E%25AF%25E5%25A2%2583%25E5%258F%2598%25E9%2587%258F1.png?imageSlim "属性")

然后打开环境变量。

![打开环境变量](https://repo.sxrhhh.top/%25E7%258E%25AF%25E5%25A2%2583%25E5%258F%2598%25E9%2587%258F2.png?imageSlim "打开环境变量")

然后新建JAVA_HOME变量

![新建环境变量](https://repo.sxrhhh.top/%25E6%2596%25B0%25E5%25BB%25BA.png?imageSlim "新建环境变量")

系统变量变量名 **JAVA_HOME**
系统变量变量值 **C:\Program Files\Java\jdk1.8.0_131**

![新建JAVA_HOME变量](https://repo.sxrhhh.top/JAVAHOME.png?imageSlim "新建JAVA_HOME变量")

这是最重要的一步，切记：**这个变量值是你安装JDK时的安装路径。**

然后在新建一个CLASSPATH变量

![CLASSPATH变量](https://repo.sxrhhh.top/CLASSPATH.png?imageSlim "CLASSPATH变量")

系统变量变量名 **CLASSPATH**
系统变量变量值 **.;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\lib\dt.jar;**
***注意不要漏掉最前面的小数点。***

然后就是编辑PATH变量了，双击Path变量，添加以下两条变量。
```
%JAVA_HOME%\bin
%JAVA_HOME%\jre\bin
```

![添加Path变量](https://repo.sxrhhh.top/Path%25E5%258F%2598%25E9%2587%258F.png?imageSlim "添加Path变量")

最后，**一路确定，关掉所有设置窗口**，打开cmd，再试试javac。

![javac完成](https://repo.sxrhhh.top/javac%25E5%25AE%258C%25E6%2588%2590.png?imageSlim "javac完成")

至此，JDK配置完成，你也完成了最烦的一步， ~~下次自己找攻略，别依赖我。~~

## 编程工具的准备
~~说实话的，用了IDE，前面的环境变量配不配置都无所谓了，但是我们还是要练练。~~
我们需要一样好用的IDE。那么什么是IDE呢，简单来说，就是你可以在里面 ~~傻瓜式的~~ 操作，你想要找到的东西（类和对象，这个我们以后再谈）他都能帮你找到，~~如果找不到那么就是你的程序出错了~~，而且java不像c++，他要的东西非常的散且多，不能像vscode那样保持项目的干净， ~~否则累死你~~。
那么，我推荐 ~~我自己用的~~ 一个IDE，就是[eclipse](https://www.eclipse.org/downloads/ "eclipse")，这个玩意你可以到管网下载，也可以到我[迅雷](https://pan.xunlei.com/s/VMgYjAen7XE37S3w9D2J4JiVA1 "迅雷")上下载，***提取码：d7dk***。我建议到我[迅雷](https://pan.xunlei.com/s/VMgYjAen7XE37S3w9D2J4JiVA1 "迅雷")上，为什么呢，这要涉及到eclipse新版本不支持老版本的JDK， ~~而作为程序员，尽量用低版本的java，毕竟不是每一个客户都会安装最高版本的java，到时候找你投诉就很难受。~~ 总之，高版本的java新增的东西我们也用不上，下载吧。
下载下来，解压后，把里面的文件夹放在一个你喜欢的位置

![我永远喜欢Program Files](https://repo.sxrhhh.top/eclipse%25E4%25BD%258D%25E7%25BD%25AE.png?imageSlim "我永远喜欢Program Files")

~~*我保证你到官网下载不到绿色版，即使官网提供*~~

![工作空间](https://repo.sxrhhh.top/%25E5%25B7%25A5%25E4%25BD%259C%25E7%25A9%25BA%25E9%2597%25B4.png?imageSlim "工作空间")

自己新建一个文件夹作为自己的一个工作空间。

