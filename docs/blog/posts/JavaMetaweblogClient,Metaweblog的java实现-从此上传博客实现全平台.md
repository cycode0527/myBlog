---
date: 2022-05-19
slug: javametaweblogclient
categories:
    - 程序开发
tags:
    - Java
    - Metaweblog
    - API
---

# JavaMetaweblogClient,Metaweblog的java实现-从此上传博客实现全平台

不知你是否会遇到下面这样的情况:想要通过java上传博客,结果却发现api的使用有些复杂。没关系，这里帮你解决了api的问题。在使用Metaweblog的时候，只需要调用网页上同名的方法就好了，一键发送命令，感受调用api的快感。

<!-- more -->

## 1. 什么是Metaweblog?

Metaweblog是一个webservice，也就是在网络上使用的api。它基于xml-rpc实现。对于现存的博客平台，例如博客园、开源中国、wordpress等都实现了metaweblog。通过这个api，你可以不登录网页，直接通过一些程序来增删查改你的博客，上传媒体文件。

## 2. Metaweblog的应用

metaweblog的适用范围不太广，但是绝对能满足于博客管理的需要。通常，面对下面情况，你就可以使用metaweblog：

* 你已经编辑好了一篇博客，保存为markdown文件，想要快速上传博客；
* 你想删除一篇博客；
* 你想更新一篇博客，但又不想复制并粘贴全部；
* 你用离线编辑器写博客，但是图片又不想保存在本地，想直接把图片上传到博客服务器里面；

因此，metaweblog可以满足你的需要

## 3. 如何使用Metaweblog

在使用本项目前，你首先需要了解一下metaweblog的原理以及如何使用。

目前，有关metaweblog的官网已经崩了，但是你仍然可以[在这里](https://rpc.cnblogs.com/metaweblog/example)查到它的api。

![image-20220519123223269](https://repo.sxrhhh.top/2854299-20220519123224856-94407514.png)

如图所示，有很多的方法（函数）可供调用。显然，你可以按照它的指示调用方法，实现博客的增删查改和媒体文件的上传。

**在这张图里，我们看到的是博客园的api调用界面，而别的网站则不一定有这样的界面，但是你可以自己尝试它们的api是否开放。下面是各大博客网站的api地址**

* 博客园：<https://rpc.cnblogs.com/metaweblog/你的用户名>,你可以在你的[后台设置](https://i.cnblogs.com/Configure.aspx)里查看有关设置
* 开源中国：<https://my.oschina.net/action/xmlrpc>
* 51cto：<http://imguowei.blog.51cto.com/xmlrpc.php>
* 网易（163）：<http://os.blog.163.com/word/>
* typecho 博客：<http://xx.com/action/xmlrpc>，其中 xx.com 为你的博客网址。如果 typecho 还没有进行 urlrewrite，则为 <http://xx.com/index.php/action/xmlrpc>
* 新浪博客： <http://upload.move.blog.sina.com.cn/blog_rebuild/blog/xmlrpc.php> 新浪博客的 API 已关闭了

* CSDN: <http://write.blog.csdn.net/xmlrpc/index>,可惜CSDN的api也关闭了
* 自建wordpress博客: <http://www.example.com/xmlrpc.php>,其中`www.example.com`是你的网站地址域名.

## 4. 本项目介绍

**然而, 对于各种语言, metaweblog的实现也不一样**.因此,你需要去寻找各种语言的实现方法.在网上,你可以查到python如何实现metaweblog，C#更是有专门的api，极大地方便了使用者调用。那么，java呢？很抱歉，网上有关java实现metaweblog的文章少之又少。为了解决这种情况，笔者做出了名为JavaMetaweblogClient的api，方便java爱好者去调用，去实现。

### 4.1 metaweblog与java之间的关系映射

如果你看了api的介绍，你就会知道，这个api是给多个语言使用的，所以很多的数据类型java都没有。那么我们就需要一个映射表。本项目通过`apache` 的`xmlrpc`实现，所以我们可以查看他的[官方文档](https://ws.apache.org/xmlrpc/types.html)

![image-20220519125713720](https://repo.sxrhhh.top/2854299-20220519125715384-654727151.png)

这些就是你要去了解的。**其中，`struct`的类型对应到java里面是`Map<String,Object>`。但是你并不需要如此，在本项目中，我已经将struct打包成了一个类对象，例如Post，里面的成员变量就是`dateTime` `description` `title` `categories`四个**.需要用的时候就依次把变量填入即可。

这时，你在使用这些方法时就会更加的得心应手。

### 4.2 使用JavaMetaweblogClient

本项目本质上只实现了Client，但是很明显我们只需要client，服务器的事由博客方来干。那么我们就要来了解一下如何使用api。

打开本项目的java文档，你会看见Client类上有教程，但是在这里我还是会再打一遍。

使用方法大致分为以下三步：(以新建一个博客文章为例) 

1. 你要创建一个Client对象
2. 准备好参数
3. 上传命令,并处理异常

这里我们以newPost方法为例：

```java
public static void newPostTest() {
   // 准备好命令所需参数(新建Post对象)
    Post post = new Post(new Date(), "# This is a post\n> You can see the Post\n", "Test");
    // 准备好返回值(自己看方法注释的返回类型)
    String result = null;
    // 创建连接客户端
    try {   // 自己解决抛出的异常
        Client client = new Client("https://www.cycode.club/xmlrpc.php");
        result = client.newPost("default", "S*******u", "******", post, false);
    } catch (MalformedURLException e) {
        e.printStackTrace();    // 一般为URL格式错误
    } catch (XmlRpcException e) {
        e.printStackTrace();    // 一般为参数不全、服务器错误、URL输入错误
    }
    // 输出结果
    System.out.println(result);
    }
```

其中的核心代码只有一条：

```java
result = client.newPost("default", "S*****u", "*****", post, false);
```

至于其他的代码，通常IDE会自己生成，以及提醒你参数的填入。其中Post参数需要直接填入本项目已创建好的`Post`类，而不是自己写一个`Map<String,Object>`.

## 5. 最后的话

作为新手程序员和他的第一个api,有很多的信息都在javadoc文档里。如果有什么问题，尽量去查看docs文档，有很多位置都可以查看文档。

1. 下载使用的jar包内的docs文件夹
2. github上的[项目地址](https://github.com/cycode0527/JavaMetaweblogClient)中的docs文件夹
3. ~~docs文件夹的[托管地址](https://www.sxrhhh.top/JavaMetaweblogClient/docs)~~**更方便,推荐**

## FAQ

1. **Q：目前程序有什么已知的问题吗？**

    **A**：本项目就是apache的xmlrpc实现的套壳，如果有什么问题，大多是xmlrpc的问题，毕竟现在用xmlrpc的人已经很少了。

    不过，我依旧发现了如下问题：

    * getPost方法与wordpress上的editor.md插件冲突

2. **Q：我想知道哪里有详细的教程。**

   **A**：本人是新手程序员，管理教程会非常的麻烦，但我大量的帮助文档都写死在程序里了，查看javadoc文档获得的信息会比在这里大得多。可以查看本文档第五项查看javadoc地址。



## 联系我

* email: <hhhsxr@qq.com>
* QQ: 1914913486
* 博客园: <https://www.cnblogs.com/sxrhhh>
* github: <https://github.com/cycode0527>
* 个人网站: <https://www.sxrhhh.top>(还在实验中)