---
date: 2023-12-26
categories: 
    - 运维
tags: 
    - 网站建设
    - Git
readtime: 10
---

# 独立于 Github，更方便地管理自己的静态网站？来试试这套自托管 Git 仓库方案！

就在前几天，我成功地将我自己的网站由 wordpress 迁移为了静态网站。不过说是迁移，但是域名保持不变，本质上就是把将网站文件夹一整个换了遍。我选择的是 mkdocs 的网站框架，在一般情况下，我们选用 Github Pages 作为网站托管方案。但是这一次，我想仍然保留在自己的服务器上。但是我还是想要享受到     git 版本管理网站的快捷感，怎么办呢？这篇文章提供的 Git 自托管方案便可以为你解决这一烦恼！

<!-- more -->

## 服务器结构简介

这套方案可以实现一个非常方便的编写博客的方法。你可以在自己电脑上写好文章，生成网站，并且直接`git push`即可部署到自己的服务器上，一气呵成，一般情况下完全不需要登陆远程服务器。

在讲述方案之前，我先说明一下这套方案采用的服务器结构。

- 本地客户端：在本地，也就是我们用来写文章的 Windows 系统，**在下文，我称之为客户端**。客户端上，我们需要安装`Git`，并且我们要会用`git`提交文件。`git`的安装方法和使用方法在此不做赘述，相信各位试图搭建静态网站的读者，`git`的用法肯定是有所涉猎的。（当然，如果你的主力机是 Linux 系统的话也完全没有问题，将这篇文章涉及的 Windows 目录结构和命令使用翻译为 Linux 版本的，想必对你来说并不难。）
- 远程 Git 仓库：如果我们不想要将网站依托于 Github 这个网站，那么我们就得要自己建立一个 Git 服务器。通过这个 Git 服务器，我们接受来自客户端的 Git 请求来储存网站。
- 远程网站服务器：这个服务器就是实际上的 web 服务器。这一次，我采用的是`apache`服务器，通过虚拟主机方式，建立网络站点，并且实时同步 Git 服务器的内容，实现网站的更新。

通过这一套流程，我们就可以在本地写好文章，放在相应的网站位置，生成预览、提交文件，从而完成网站更新的一整套流程。

如下图，便为这一套系统的基本框架了，非常的简单。

![image-20231225225154806](https://repo.sxrhhh.top/image-20231225225154806.png)

## 在自己的 PC 上构建网站

构建网站的第一步，便是构建自己网站。要构建静态网站，有很多的解决方案，比较著名的就有`Hexo`,`Mkdocs`,`Hugo`,`Jekyll`等，甚至你还可以自己手写前端 CSS 和 JS 生成 html 文件。不管采用那种方案，最后的要求便是你要有一个`site`文件夹，或者叫其他名字，他的里面便是网站所有的被挂在前端的文件，包括 html 之类的玩意。我们的 Apache 服务器的文件目录便最终会指向它。
这次，我采用的 Mkdocs 由`python`编写而成，它的一个名为`mkdocs material`的主题显得非常简洁美观，所以我选择了它。和其他运行环境不同的是，这一次我们所有的网站环境全部会部署在本地，远程服务器**完全不需要**。这也很好理解，毕竟远程服务器只要展示网站文件就够了，生成网站的任务落在了客户端的头上。
Mkdocs 采用一个叫做`mkdocs build`的命令来构建网站文件夹，我们每一次用 git 提交文件之前都会进行一次这样的构建。现在，我们来建立那个 git 服务器。

## 建立远程 Git 服务器

要自由方便地管理网站文件，离不开的就是一个 Git 服务器。我们将会在远程服务器上搭建一个 Git 服务器。与想象中的不同，他不需要也没必要像 `GitLab` 那样配置复杂。与之相反的是，这件事出乎意料的简单。在此之前，我先带各位回忆一下 Github 上的 git 服务器长什么样。

### Github 的仓库方案

在 Github 上，如果我们需要克隆一个存储库，我们需要诸如以下命令：

```bash
git clone git@github.com:cycode0527/learngit.git
```

可以看到，这条命令采用 `ssh` 的方式链接的 Github 服务器，并且克隆了上面的 `cycode0527/learngit.git` 的仓库。但是这个克隆并不是一个简单的复制，在服务器上，它叫 `xxx.git` ，克隆到了本地，他就变成了一个叫做 `xxx` 的文件夹，里面有一个叫做 `.git`的隐藏文件夹。这其中的差别其实就是：**Git 服务器上的 git 仓库叫做裸仓库**。裸仓库没有“工作区”的概念，所有的数据全部保存在一个文件夹内，不需要直接操控其中的文件，只要干好 Git 服务器一件事就行。

再次看向上面那条 `clone` 命令，你可能还会注意到 `git@github.com`这一小节。只要你用过 `ssh` 这条命令，你应该就会清楚，这是指人家 Github 服务器上的一个叫做 git 的用户名。我们在克隆仓库时，就会连接到 github.com 的 git 用户，并且读取你（cycode0527）的 xxx.git 文件。搞清楚这一点，我们就清楚了，**在远程服务器上，有一个负责处理 git 请求和存储的用户，叫做 “git” 。**

### 建立 git 用户管理仓库

现在，我们就可以在远程服务端建立 git 用户了。

首先，我们要让服务器支持 git ，直接安装即可。

```bash
sudo yum install -y git	#(1)! 
```

1. 本条命令为 CentOS 使用，其他操作系统可自行更改

然后，我们建立 git 用户和用户组，用来运行 git 服务。

```bash
groupadd git
useradd git -g git	#(1)! 
```

1. 建立用户 git 并加入 git 用户组

用这种方式创建的用户，系统会默认创建好它的家目录。我们现在创建一个仓库吧。

```bash
cd /home/git
mkdir repo	#(1)!
cd repo
git init --bare website.git	#(2)!
```

1. 创建了一个叫做 repo 的文件夹,用于存放各种仓库
2. 建立一个名为 website 的仓库,--bare参数就是创建裸仓库。<br>
输出结果: 
```
Initialized empty Git repository in /home/git/repo/website.git
```

最后我们调整一下仓库的所有者，以防止你刚才创建 git 用户所用的 root 用户忘记切回来，导致 git 用户家目录下全是 root 的文件。

```bash
chown -R git:git /home/git/repo
```

现在，这个 git 服务器已经建好了，下一步要做的就是将客户端连接到这个服务器。

### 将客户端与 git 服务器同步

现在我们要做的就是，通过 `clone` 命令，将客户端的文件夹与远程 git 服务器建立映射。（当然，你也可以通过已有的网站目录，用 `git remote add` 的命令与新的服务器建立映射关系。）

为了在与远程服务器同步时不在输入密码，我们选用的是 `ssh` 的连接方式，但是在此之前，我们需要将自己客户端上的 ssh 公钥发给远程服务器的 git 用户，从此登陆 git 用户就不用密码了，更为方便的提交网站更改。

首先看一下自己的家目录下有没有 `.ssh` 文件夹，这个目录处在 `C:\Users\<你的用户名>\.ssh` 这里，如果有的话，跳转下一步；没有的话，请看如何生成一个 ssh 公钥。

现在打开你的 `windows` 终端，或者是 `cmd`、`powershell`、`git bash` 之类的玩意，输入如下命令：

```bash
ssh-keygen -t rsa
```

然后连按三个回车，ssh 密钥对就生成好了。现在再进入家目录下的 `.ssh` 文件夹，找到一个后缀名为 `.pub` 的文件，**不要双击打开，右键打开方式，找一个记事本或是 vscode 等能打开文本文件的软件打开这个文件**，将里面的内容尽数复制出来。

然后我们来到远程服务器，登录到 git 用户，建立 ssh 用户验证文件：

```bash
cd ~
mkdir .ssh
vim .ssh/authorized_keys
```

我们用 vim 打开了这个叫做 authorized_keys 的文件，然后把复制好的 ssh 公钥粘贴进去，保存并退出。~~（各位应该知道 vim 怎么操作吧……）~~

现在你可以试验一下，登录到 git 用户还需要密码不。

```bash
ssh git@www.sxrhhh.top
```

如果直接进去了，说明 ssh 已经不需要密码了，那么 git 的一切操作也可以不用密码快速进行。

在客户端下：

```bash
git clone git@www.sxrhhh.top:/home/git/repo/website.git
git clone git@www.sxrhhh.top:repo/website.git	#(1)!
```

1.  是的，冒号后面的路径就是相对于 git 用户家目录的相对路径

好了，如果你的电脑上出现了 `website` 文件夹并且里面还有 `.git` 隐藏文件，那么就成功了！

现在，你就可以将你的网站文件放进去，欢快地 `mkdocs build`, `git add .`,`git commit`, `git push`了。

!!! info "编写 `.gitignore` 文件"

    为了防止在写网站时一些乱七八糟的更改影响 git 的跟踪文件，我们要提前写好一个 `.gitignore` 文件，阻止它们进入 git 的跟踪。

    ```bash title=".gitignore"
    .vscode	#(1)! 
    *~	#(2)!
    ```

    1. 防止用 vscode 管理网站的时候工作区更改
    2. 防止用vim编辑网站文件时留下一堆储存文件

    后面我们还会往里面加一点东西，这个请读者留意一下。

## 建立 web 服务器

现在，我们已经搭建好了这一套管理系统的核心结构，那么最后一步，就是将上传的网站文件夹发给 Apache 服务器，让它帮忙共享这个文件夹。

我们知道，git 服务器上的文件是不能直接用的。我们还是需要用 `clone` 和 `pull` 的方法将文件提取出来。这一次我们操作的是 `root` 用户，这在权限管理上是不可取的，但是在这次的方案中，我们搭建的是静态网站，对 apache 用户的写入权限要求不高。另外，apache 是一个 "nologin" 的用户，他不能使用诸如 `git`  之类的命令，局限性过大。为了更好管理 web 文件夹，我们采用 root 用户登陆。当然，为了安全，你完全可以再创建一个名字类似于 `www-data` 的普通用户来给 apache 用。

现在我们就开始新建文件夹吧！

### 将 web 服务器连接到 Git 服务器

首先，我们要让 root 用户被 git 用户认证，原因和之前一样。即使在同一台机子上的 root 用户，想要 ssh 连接另一个用户照样需要认证。

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> /home/git/.ssh/authorized_keys #(1)!
```

1.  将root用户的ssh公钥加入到git用户的认证文件中。请确保在上一步中你已经创建了这个文件。



我们在根目录下创建一个文件夹，用来存储网站。当然你也可以在 `/var/www/` 下面建立，主要是我这个文件夹算是比较特殊，拎出来单独处理。

```bash
mkdir /www
cd /www
git clone git@localhost:repo/website.git	#(1)!
```

1. 哈哈,来自远程的localhost。如果你不确定，你可以把localhost改成你服务器域名。

如果一切顺利，你会有一个 `/www/website` 文件夹，在这个文件夹里面又有一个存放构建完毕的 `site` 文件夹。那么，`/www/website/site` 就是你的网站的文件系统路径了。记住它！

### 创建 Apache 虚拟主机

如果你先前没有见过网站，你完全可以通过修改 `DocumentRoot` 的参数来直接运行服务器。但是建立过网站的同学都知道，虚拟主机还是挺重要的。所用的方法我有另一篇博客写过了，网上也有很多教程和官方文档，这里长话短说。

这里以 `CentOS` 和 `fedora` 之类的使用 `rpm` 包管理器的系统为例，apache 的服务名字叫 `httpd`，配置文件在 `/etc/httpd/` 下，其他系统，譬如 `ubuntu` 下的 apache 可能被称作 `apache2` ，目录结构也不太一致。请各位同学自行查找相应配置方法，但是下面的配置文件内容不会有问题。（当然文件路径你也要根据自己实际情况修改）

```bash
cd /etc/httpd/conf.d
vim vhost.conf	#(1)!
```

1.  建立了一个叫做vhost.conf的文件，如果你有其他相同作用的文件，请打开它。


```conf title="vhost.conf"
<VirtualHost *:80>
    # 如果这个是你第一个虚拟主机，那么它也会成为你apache的默认共享位置
    ServerName www.sxrhhh.top	# 请自行更改为你自己的域名
    DocumentRoot "/www/website/site"
    <Directory /www/website/site>
    AllowOverride All
    Require all granted
    </Directory>
    
    # 下面的是用来强制跳转https用的,可以暂时不写。
    RewriteEngine on
    RewriteCond %{SERVER_PORT} !^443$
    RewriteRule ^(.*)$ https://%{SERVER_NAME}$1 [L,R]
</VirtualHost>
```

或者，我们去掉那些没用的，可以直接复制下面的：

```conf title="vhost.conf"
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot "/www/website/site"
    <Directory /www/website/site>
    AllowOverride All
    Require all granted
    </Directory>
</VirtualHost>
```

保存并退出，重启 Apache，看一下能否访问到自己的网站。

```bash
apachectl restart
```

如果一切安好，你现在就可以看到浏览器能访问到你自己的域名网站。

!!! info "注意"

    在此还是确认一下，在测试浏览器访问的时候，请确保你的域名服务商解析了你的域名到远程服务器上，或者你在本地客户端更改了 hosts 文件。否则你不可能用你的域名访问到服务器。

### 实时同步 web 网站目录

 我们在每一次用 `git push` 更新网站后还得要上 root 用户去 `git pull` 一下才能完成更新部署，太麻烦了。这时候，我们就要请出 `crontab` 这个自动任务管理工具了。

首先我们要写好一个更新脚本：

```bash
cd /www/website/
touch update update_log update_err
vim update	#(1)!
```

1. 这个就是更新脚本,坑爹的一个点就是文件名不能带小数点,要不然crontab会出问题。

```bash title="update"
#!/bin/bash
cd /www/website
/bin/git pull 1>>./update_log 2>>./update_err
# 上面那个 1>>./update_log 这一条是记录git pull的日志,如果更新频率够快,它会很快占用比你的网站还大。所以当你的网站已稳定运行后，你可以将这一段改为 1>/dev/null
```

我们建立了两个文件，叫 update_log 和 update_err 作为同步结果的日志收集。你现在可以直接运行来测试。

```bash
bash ./update
cat update_log
```

看一下 update_log 有没有多出一条回显信息。如果有，说明 pull 成功，已经完成了一次 Git 服务器到 web 服务器的同步。

现在我们要利用 `crontab` 实现实时同步。原理也很简单，就是每分钟运行一次 `update` 脚本。

```bash
crontab -e
```

在打开的页面中，我们在最后加入这么一条：

``` config title="crontab -e"
* * * * * /bin/bash /www/website/update > /dev/null 2>&1
```

上面这一条可以让我们每一分钟都执行一次 `update` 脚本。`>/dev/null` 后面的内容是用来防止 crontab 发一些奇怪的邮件的。

完成之后，你就可以时不时地看一眼你的 update_log 文件，看一眼它有没有每分钟都更新一次。

最后，我们将这几个文件加入到 `.gitignore` 文件里，确保它们只在本地运行。

```bash title=".gitignore"
.vscode
*~

./update*
```

## 验收成果

现在，我们的两个服务器和一个客户端上应该搞的东西都已经完成了，可以开始验收成果了！

首先，我们在客户端上写好网站内容，放置好博客文章，构建，发送！

```bash
mkdocs serve	#(1)!
mkdocs build
git add .
git commit -m "website updated"
git push
```

1. 通过虚拟服务器实时检验网站更改

如果一切顺利，你会在1分钟以内看到你的网站成功发生了改变。从此，你只要在 vscode 中操作好文章，vscode 就能做好从撰写文章、yaml 编写和网站上传等多种工作，怎么样，学会了吗？

## 网站维护碎碎念

在最后，我还是多提醒一些事项，防止有人在这里学习第一次建站。

首先，完成工作后，你的域名一定要解析到你的服务器，否则你的域名就没有用，而且 Apache 的虚拟主机策略也有可能让你的网站无法使用。

然后，记得使用 SSL 技术，也就是将你的网站升级为 `https` ，一方面是安全，另一方面是浏览器的一些自动跳转设置，尤其是以前用类似域名绑定过带 `https` 的服务器，可能会让你访问自己网站困难，一直跳转到带 `https` 的同名网址。

最后就是，如果你想多一份保障，不想完全脱离 Github ，只是网站不想建在那个上面，你还可以在 Github 上建立一个仓库，并且在 `update` 文件里面去 `git push` 它。相当于每分钟将网站仓库同步到 Github ，但是并不依赖于它，只是作为一个 Github 的备份策略。



---

作者：Sxrhhh

个人网站：[https://www.sxrhhh.top](https://www.sxrhhh.top/)

博客园：[Sxrhhh - 博客园 (cnblogs.com)](https://www.cnblogs.com/sxrhhh/)

转载请注明出处.

在个人网站持续更新中……
