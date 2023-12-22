---
date: 2022-04-29
slug: how-to-generate-javadoc
categories:
    - 编程语言
tags:
    - Java
    - 写作
---

# 如何生成一个java文档

众所周知，一个程序给别人看可能可以看懂，几万行程序就不一定了。在更多的时候，我们并不需要让别人知道我们的程序是怎么写的，只需要告诉他们怎么用的。那么，api文档就发挥了它的作用。

<!-- more -->

-----

## 1. 什么是api文档？

顾名思义，文档是给人看的，那么api文档就是告诉别人我的程序要怎么用。一个最典型的例子就是JDK8的帮助文档，如图：[JDK8文档链接)](https://docs.oracle.com/javase/8/docs/api/)

![image-20220427122437430](https://repo.sxrhhh.top/2854299-20220427122444981-1551499804.png)

一看：一目了然，想找什么都有，极大地方便了我们这种**使用**JDK的人。

## 2. 写好java文档注释

既然要生成文档，我们就需要一个写好文档注释，在类和方法前使用/** xxx */来进行文档注释

| 标签 | 描述 | 实例|
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| @author       | 标识一个类的作者                                       | @author description                                          |
| @deprecated   | 指名一个过期的类或成员                                 | @deprecated description                                      |
| {@docRoot}    | 指明当前文档根目录的路径                               | Directory Path                                               |
| @exception    | 标志一个类抛出的异常                                   | @exception exception-name explanation                        |
| {@inheritDoc} | 从直接父类继承的注释                                   | Inherits a comment from the immediate surperclass.           |
| {@link}       | 插入一个到另一个主题的链接                             | {@link name text}                                            |
| {@linkplain}  | 插入一个到另一个主题的链接，但是该链接显示纯文本字体   | Inserts an in-line link to another topic.                    |
| @param        | 说明一个方法的参数                                     | @param parameter-name explanation                            |
| @return       | 说明返回值类型                                         | @return explanation                                          |
| @see          | 指定一个到另一个主题的链接                             | @see anchor                                                  |
| @serial       | 说明一个序列化属性                                     | @serial description                                          |
| @serialData   | 说明通过writeObject( ) 和 writeExternal( )方法写的数据 | @serialData description                                      |
| @serialField  | 说明一个ObjectStreamField组件                          | @serialField name type description                           |
| @since        | 标记当引入一个特定的变化时                             | @since release                                               |
| @throws       | 和 @exception标签一样.                                 | The @throws tag has the same meaning as the @exception tag.  |
| {@value}      | 显示常量的值，该常量必须是static属性。                 | Displays the value of a constant, which must be a static field. |
| @version      | 指定类的版本                                           | @version info                                                |

例如：

```java
/** 这个类绘制一个条形图
* @author runoob
* @version 1.2
*/
```

以及我自己的：

![image-20220427131244224](https://repo.sxrhhh.top/2854299-20220427131248367-618323699.png)



这样，就写好了文档注释。



## 3. 利用命令行制作一个api文档

打开cmd，来到项目目录，一个命令`javadoc`极大地方便了我们

```shell
javadoc -encoding UTF-8 -charset UTF-8 Doc.java
```

**注意：一定要输入`-encoding UTF-8 -charset UTF-8 `这个东西，否则你写的中文注释全是乱码。**

然后，你就会看到当前目录下多出来了一堆文件：

![image-20220427125012970](https://repo.sxrhhh.top/2854299-20220427125015777-492194320.png)

重点是`index.html`。如果你们有学习或了解过网站组成的话，就会知道index.html是网站目录下的索引，打开它相当于打开了一个页面。

打开它，一个熟悉的窗口：

![image-20220427125146050](https://repo.sxrhhh.top/2854299-20220427125149045-1326341225.png)

## 4. 使用IDEA生成doc文档

不知道你们有没有开始用IDEA来写java程序，反正我最近是转IDEA了，确实舒服。那我还是扯两句如何生成doc：

![image-20220427131957916](https://repo.sxrhhh.top/2854299-20220427132002172-1928773838.png)

![image-20220427133404529](https://repo.sxrhhh.top/2854299-20220427133407525-379996067.png)

-----------

最后感谢哔哩哔哩up朱遇见狂神说的视频

[遇见狂神说的个人空间_哔哩哔哩_bilibili](https://space.bilibili.com/95256449?spm_id_from=333.788.b_765f7570696e666f.1)
