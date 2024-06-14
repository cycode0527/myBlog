---
date: 2024-06-14
categories: 
    - PolarCTF
tags:
    - CTF
    - 信息收集
---

# 每日一题 —— [PolarCTF]seek flag

> 题目地址：<https://www.polarctf.com/#/page/challenges>

<!-- more -->


题目打开页面很简单，没有前端。那么我们直接 `F12` 打开控制台查看源代码。题目提示 `爬虫要是爬到我的flag怎么办` 。可以知道，爬虫基本遵循 `robots.txt` 这个文件的内容。因此我们直接访问 `robots.txt` ，发现其中有一部分的 flag： `flag3:c0ad71dadd11}`。也就是说，我们需要找到剩下的两个 flag 片段。

首先我先使用 `dirsearch` ，试图扫描出一些目录，但是很遗憾两个字典都显示出只有一个 `robots.txt` 。

因为整个网站除了一个 `index.php` 和一个 `robots.txt` ，也没有别的什么信息了。那么我们最后把目光放在“网络”这一栏上。看到 `Set-cookie` 为 `id=0` ，那么我们尝试发包的时候，把 `Cookie` 中改为 `id=1` ，发现页面回显出了 flag1 。当然最后把 `id` 改为别的也没有什么用了。

最后，flag2 就在服务器发回的 response 头里找到了。没想到最容易找的，我是最后才找到。

