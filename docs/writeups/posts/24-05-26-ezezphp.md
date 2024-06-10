---
date: 2024-05-26
categories: 
    - SWPUCTF
tags:
    - CTF
    - PHP
    - 文件包含
    - php伪协议
---
# 每日一题 —— [SWPUCTF 2022 新生赛]ez_ez_php

> 题目地址： <https://www.nssctf.cn/problem/2640>    

<!-- more -->

非常简单的一道题。首先盲猜 flag 在 `./flag.php` 下，使用 php 伪协议读取：

```
/?file=php://filter/convert.base64-encode/resource=flag.php
```

发现不少提示，甚至直接 `echo` 出来了。那么我们根据提示，直接访问 `/flag` 得到 flag. 非常简单和逆天的一道题。