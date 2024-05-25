---
date: 2024-05-25
categories:
    - HDCTF
tags:
    - CTF
    - SSTI
    - PHP
    - RCE
---
# 每日一题 —— [HDCTF 2023]SearchMaster

> 题目地址：<https://www.nssctf.cn/problem/439>

<!-- more -->

进入网页，可以看到右边有提示：you can post me a data. 于是，我们利用 POST 请求提交一个变量为 `data` 的参数。可以看到，我们给 `data` 赋的值，会在网页正中央回显。

再看到右边提示：模板都一样我很抱歉，说明这道题考点有可能是 SSTI 模板注入。我进入到了左边作者的博客中，并没有发现什么有用的提示。通过插件，我发现题目网页是 php 项目。于是 bing 搜索关键词：php ssti,发现有几种框架可能会有 SSTI，分别是 twig 和 Smarty. 我们给 `data` 传入参数 `{{123+123}}` ，发现网页报错了，泄漏了网页的框架为 `Smarty` 。

现在，我们根据网上资料，先给 `data` 赋值为 `{$smarty.version}` ，发现回显为 `4.1.0`。然后，我们继续百度 Sweaty 4.1.0 RCE，得到 payload 的格式为：`{if system('cat /flag_13_searchmaster')}{/if}`。其中，第一个 if 后面的语句可以任意替换，完成 RCE 漏洞。

---

ps: 如果根据作者博客的框架，搜索 hexo 的模板注入，我们会发现一个 payload: `{% include_code ../../../../../../../../etc/passwd %}` , 然而当我们将此 payload 提交后，会发现作者手动过滤了此注入。

```
no way! but you can see my blog~:Boogipop's Blog
```

说明我们需要另寻他路。

---

ps2: 其实我也想通过这方法出一道题，来推广一下自己的博客。撞车了（bushi