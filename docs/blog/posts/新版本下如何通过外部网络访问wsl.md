---
date: 2023-12-14
slug: connect-to-wsl
categories:
    - wsl
    - 服务器
---

# 新版本下如何通过外部网络访问wsl

 众所周知，wsl2是windows下的linux子系统，并且采用类似于虚拟机NAT的管理方式。一般情况下，外部网络很难直接访问到wsl上的服务，除非使用端口转发。而现在，微软更新了wsl 2.0.0，采用镜像网络配置，完美解决了所有网络上的问题。

<!-- more -->

## 研究起因

***[想直接看新版本解决方案的点这里](#1)***

由于在编译他人代码时需要linux环境，我就放在了wsl下编译运行，然后在本地，我就尝试用`ifconfig`得到的虚拟机ip，成功连接上了wsl服务器。但是，当我试图用局域网下其他设备连接时，很显然，根本不可能连接上。**这是因为wsl2采用了类似于NAT的网络模式，windows作为宿主机，隔离了局域网下其他设备和wsl的连接。**为了外部访问wsl，我也试过一些方法。

## 旧版本端口转发方案

在旧版本，常用的方法是端口转发，根据官方文档，在有管理员权限的powershell下，输入如下指令：

```powershell
netsh interface portproxy add v4tov4 listenport=<yourPortToForward> listenaddress=0.0.0.0 connectport=<yourPortToConnectToInWSL> connectaddress=(wsl hostname -I)
```

> 在此示例中，需要更新 `<yourPortToForward>` 到端口号，例如 `listenport=4000`。 `listenaddress=0.0.0.0` 表示将接受来自任何 IP 地址的传入请求。 侦听地址指定要侦听的 IPv4 地址，可以更改为以下值：IP 地址、计算机 NetBIOS 名称或计算机 DNS 名称。 如果未指定地址，则默认值为本地计算机。 需要将 `<yourPortToConnectToInWSL>` 值更新为希望 WSL 连接的端口号，例如 `connectport=4000`。 最后，`connectaddress` 值必须是通过 WSL 2 安装的 Linux 分发版的 IP 地址（WSL 2 VM 地址），可通过输入命令：`wsl.exe hostname -I` 找到。
>
> ---官方文档

然后，我将wsl中的`8303`端口映射在了windows下的`8304`端口下，我看到，在wsl中，服务器照常运行。

```powershell
netstat -ano | findstr 8304
```

也能看到8304端口正常监听。但是，在我访问wsl运行的游戏服务器~~（游戏名：DDraceNetwork）~~，客户端显示“udp疑似被拦截”。本来我认为这是端口转发失败了，但是我忽然注意到本地8304端口只有tcp。于是查询了有关资料，得到了`netsh`只支持tcp的结论。因此，netsh方案失败。

## 旧版本桥接模式方案

后来，我希望用hyper-v创建一个虚拟交换机，相当于一个可通外网的虚拟网卡，并让wsl连接。结果，出现了许多问题。

- 当前版本下，wsl2已不支持更改连接方式为`bridge`桥接模式，从根本上杜绝了这一方法
- 当我创建了虚拟交换机后并将其接入网桥，他直接**把我Windows下的网络干废了**。
- ~~试图用Vmware创建交换机，结果根本找不到选项~~

最后，我发现了官网给出了新方案！

<a id="1"></a>

## 新版本镜像网络模式方案

由于本人入坑wsl时间晚，在此之前，我对wsl1只是道听途说。据我了解，wsl1建立在兼容层上，与windows共存，因此wsl1的网络配置与windows一致，外部网络也可以很轻松地接入wsl。而wsl2则是基于hyper-v的虚拟机，采用的是新一套的NAT方案，较为独立。所以对于wsl2也要用和普通虚拟机一样的网络方案，如NAT转发和桥接。

而现在，在Windows`23H2`更新中，或是`22H2`中的insider，wsl2更新了镜像网络解决方案，这个方案将会解决几乎一切wsl上的网络问题。**而此方法，也将随着时间推移，成为wsl2的默认解决方案**。

在此贴出微软官方文档：

[使用 WSL 访问网络应用程序 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/networking)

### wsl版本检测

要启用镜像网络模式，首先要保证你的windows系统是23H2以上，或是加入了windows体验计划。除此之外，由于版本更新已有一段时间，如果你不确定是否可以使用，可以在cmd输入`wsl --version`查看wsl版本是否是`2.0.0`以上。如果是，那就可以。

### 配置wsl文件

在windows用户根目录下，新建个名为`.wslconfig`的文件，选择合适的编辑器打开它。

> 如果不知道自己的用户根目录，可以在cmd下输入`echo %USERPROFILE`，即可看到路径
>
> 经过实验，我甚至发现上面的命令只能在cmd命令提示符下执行，powershell还不行！！！

在文件中输入如下内容：

```
[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
```

然后，执行`wsl --shutdown`，等待大约8秒后重新启动wsl，即可成功改变网络策略。

你可以通过在wsl虚拟机内执行`ifconfig`，看到原本有的IP地址现在没了，即可证明mirrored模式启用成功。现在你可以用phpstudy等软件在windows下建立一个网站，然后用linux访问，会发现linux可以访问windows的服务。

比如说：我在windows下用80端口建立的网站，在linux下执行`curl localhost`，显示的是windows的网站。

再比如说：我在linux下用apache在81端口建立了个网站，在windows下用浏览器访问`http://localhost:81`，可以看到linux的网站。

那么，wsl的新版本网络配置也就成功了…………

…………

**吗？**

### 解决遗留的坑

很遗憾地告诉大家，我上面给出的配置文件**少了一句话**，而这句话才是本文的重点。为什么会单独拿出来讲，这是因为我在20分钟查找资料的过程，**没有一篇文章**讲到这个。现在我们先来看看问题在哪里。

首先，我们要先知道windows的IP地址，不要求上百度看公网IP，只要一个局域网内的外部IP，也就是你电脑直连路由器分配给你的IP，假设是192.168.1.101，那么我现在在浏览器访问`http://192.168.1.101`，可以成功看到宿主机创建的网站。

现在在让我们访问`http://192.168.1.101:81`，也就是linux下的网站端口，结果发现，**无法访问！**

也就是说：

- localhost:81	可以！
- 192.168.1.101:81	不行！

那就奇了怪了，我也是成功在官网找到响应的配置选项，只要在最后加上那么一行：`hostAddressLoopback=true`就行了。

最后的配置文件：

```
[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
hostAddressLoopback=true
```

重启wsl，问题解决，~~我的游戏服务器也能进入了，只是服务器列表不显示[滑稽]~~

## 尾声

我在这里贴出wsl2关于mirrored模式的一些其他配置选项，可以更好的帮助各位配置wsl。

| 设置名称                   | 值     | 默认值 | 说明                                                         |
| :------------------------- | :----- | :----- | :----------------------------------------------------------- |
| `useWindowsDnsCache`**     | bool   | false  | 仅当 `experimental.dnsTunneling` 设置为 true 时才适用。 如果此选项设置为 false，则从 Linux 隧道传输的 DNS 请求将绕过 Windows 中的缓存名称，以始终将请求放在网络上。 |
| `bestEffortDnsParsing`**   | bool   | false  | 仅当 `experimental.dnsTunneling` 设置为 true 时才适用。 如果设置为 true，Windows 将从 DNS 请求中提取问题并尝试解决该问题，从而忽略未知记录。 |
| `initialAutoProxyTimeout`* | string | 1000   | 仅当 `experimental.autoProxy` 设置为 true 时才适用。 配置启动 WSL 容器时，WSL 等待检索 HTTP 代理信息的时长（以毫秒为单位）。 如果代理设置在此时间之后解析，则必须重启 WSL 实例才能使用检索到的代理设置。 |
| `ignoredPorts`**           | string | Null   | 仅当 `experimental.networkingMode` 设置为 `mirrored` 时才适用。 指定 Linux 应用程序可以绑定到哪些端口（即使该端口已在 Windows 中使用）。 通过此设置，应用程序能够仅侦听 Linux 中的流量端口，因此即使该端口在 Windows 上用于其他用途，这些应用程序也不会被阻止。 例如，WSL 将允许绑定到 Linux for Docker Desktop 中的端口 53，因为它只侦听来自 Linux 容器中的请求。 应在逗号分隔列表中设置格式，例如：`3000,9000,9090` |
| `hostAddressLoopback`**    | bool   | false  | 仅当 `experimental.networkingMode` 设置为 `mirrored` 时才适用。 如果设置为 True，将会允许容器通过分配给主机的 IP 地址连接到主机，或允许主机通过此方式连接到容器。 请注意，始终可以使用 127.0.0.1 环回地址 - 此选项也允许使用所有额外分配的本地 IP 地址。 |

具有 `path` 值的条目必须是带有转义反斜杠的 Windows 路径，例如：`C:\\Temp\\myCustomKernel`

具有 `size` 值的条目后面必须跟上大小的单位，例如 `8GB` 或 `512MB`。

值类型后带有 * 的条目仅在 Windows 11 中可用。

值类型后显示 ** 的条目需要 [Windows 版本 22H2](https://blogs.windows.com/windows-insider/2023/09/14/releasing-windows-11-build-22621-2359-to-the-release-preview-channel/) 或更高版本。

---

作者：Sxrhhh

个人网站：[https://www.sxrhhh.top](https://www.sxrhhh.top/)

博客园：[Sxrhhh - 博客园 (cnblogs.com)](https://www.cnblogs.com/sxrhhh/)

转载请注明出处.

在个人网站持续更新中……
