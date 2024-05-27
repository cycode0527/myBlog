---
date: 2024-05-27
categories: 
    - SWPUCTF
tag:
    - CTF
    - PHP
---

# 每日一题 —— [SWPUCTF 2021 新生赛]jicao

> 题目地址： <https://www.nssctf.cn/problem/384>

进去给出了代码，可以看到要求传入两个参数：get 一个 `json` ; post 一个 `id`.

打开 HackBar， 直接 post 一个 `id=wllbNB`, 然后根据代码，我们对 `json` 参数需要有以下要求：

1. 是一个合法的 json 字符串
1. 该 json 对象内有 `"x": "wllm"` 这个键值对

所以我们直接构造 `json` 参数： 

```
?json={"x":"wllm"}
```

两个一起提交，最终得到 flag. 非常非常简单的一道题。

---

拓展：php 中存在一个叫做 `json_decode` 的函数，这个函数的作用就是解析 json 字符串。通常来讲这个函数有两个参数，即： `json_decode(string, bool)` 。第一个参数 string 类型，表示被解析的 json 字符串。而第二个参数表示解析成的 php 对象。 true 即转换成数组，而 false 就转换成普通的 php 对象。