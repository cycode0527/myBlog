---
date: 2023-06-21
slug: fever-in-ttrss
categories:
    - 运维
tags:
    - RSS
    - 网站建设
    - Tiny Tiny RSS
---

# 解决ttrss(Tiny Tiny RSS)中fever无法使用的问题

## 问题描述

在ttrss刚搭建好的时候，进行了如下操作：

<!-- more -->

![image-20230621135321319](https://repo.sxrhhh.top/picgoimage-20230621135321319.png)

随后键入了密码（fever密码）

![image-20230621135630945](https://repo.sxrhhh.top/picgoimage-20230621135630945.png)

最后，按照官方给的提示，在Fluent Reader中测试，弹出如下错误信息：

![image-20230621135725039](https://repo.sxrhhh.top/picgoimage-20230621135725039.png)

## 解决方案

复制官方给的链接，删除其中的“.local”

尾部记得加上"/"

示例：

```
https://www.example.com/plugins.local/fever/	#错误
https://www.example.com/plugins/fever	#错误
https://www.example.com/plugins/fever/	#正确
```



完美解决。

---

作者：[Sxrhhh](https://www.sxrhhh.top)

转载请注明出处.