---
date: 2024-01-16
categories:
    - 运维
tags: 
    - Linux
    - 笔记本
    - Sunshine
slug: sunshine-moonlight-to-create-a-virtual-monitor
readtime: 7

--- 

# Sunshine + Moonlight 纯软件实现全平台设备作 Linux 副屏

最近，我想要通过视频学习一些技术知识，作为笔记本用户，没有外接屏幕显然是十分痛苦的，需要不断切换窗口，并且还会互相遮挡。于是我便萌生了使用身边的平板和手机作为副屏的想法。经过一番查找，发现各种千奇百怪的方法，有的付费，有的卡顿，最主要的就是：大部分方案是基于 Windows 系统的。在其中，我发现 Moonlight 串流方案基于全平台。在此基础上，我解决了一些问题，成功完成副屏延伸。

在本文中，Moonlight 只是一种串流方案，其实并非本文的重点。想知道重点在哪里吗？那就接着往下看！

<!-- more -->

##  初识 Moonlight

在完成一系列副屏投屏操作前，我还是先简单介绍一下 Moonlight，这是一个串流软件，可以接收电脑屏幕上的内容和声音，并且可以反过来操控电脑。首先先要了解一点的是：**Moonlight 是一个客户端软件**，换句话说就是，Moonlight 应该安装在手机平板之类的“主控机”上，而不是像我电脑那样的“受控机”。那么显然，想要接收到电脑发出的串流信息，电脑本身必须要有一个服务端软件来发送串流。下面就是一个大坑。

我第一次接触到 Moonlight 的时候，是在研究 Steamlink 的平替，那个时候，我想要一个不同于 Steamlink 的方案，用来手机连接电脑玩一些 steam 上的视觉小说游戏 ~~(嗯，也就是 galgame)~~，而网上有关方案，大部分都推荐 Moonlight，但是所有这类文章，无一不陈述一个所谓的建议：**建议使用 N卡 的用户使用 Moonlight 串流，并且开启 Nvidia Geforce 上的一个 SHIELD 功能**，利用那个玩意作为电脑上的串流服务端，配套上如今 SHIELD 功能消失的解决方法，可谓井井有条，方案详细。一言以蔽之：**脱裤子放屁**。

原谅我话说那么难听，但是铺天盖地的 Nvidia Shield 的方案不仅难用，而且缺少了国内的支持，最主要的是我当时没有成功串流，只好放弃 Moonlight 方案，继续 Steamlink。如此方案，混淆了我的视听，让我没能深究背后的那个**备用方案**。直到我更换了 Fedora 这个 Linux 发行版作为主力操作系统，我在很清楚 Linux 上面没有 Nvidia Geforce Experience 这个软件的情况下，排除了一切有关这个的方案继续探究 Moonlight，这才找到 Moonlight 是有一个配套的服务端 —— Sunshine 的。

很显然，作为一个连接串流用的软件，有一个专用的配套的服务端，才更加符合我们的常识。事实也证明了，**局域网下 Sunshine + Moonlight 方案，拥有无异于物理显示屏的体验**。

## 部署 Sunshine 服务端与 Moonlight 客户端

想要完成副屏的连接，第一步就是要完成手机与电脑间最最最基本的串流，也就是同屏串流。其基本效果和 Steamlink 一致，都是能将电脑的视频音频转发到手机上，并且手机上也有办法操控电脑。现在可以在我们的 Linux 系统上部署 Sunshine 服务端了。

首先我们来到 Sunshine 的 Github 官网：<https://github.com/LizardByte/Sunshine/releases>，可以看到有几十种不同的发行包，支持 Linux 各大发行版和 Windows 版本，不确定支不支持 MacOS。但最重要的是，无论是 Sunshine 还是 Moonlight，都提供 AppImage 版本的软件包，不同于各大发行版上的包管理器，这种软件包是真正意义上的跨发行版，全 Linux 统一使用。因为我使用的是 Fedora 39, 不确定用官方提供的 38 版本的包会不会出问题，就统一采用了 AppImage 包。现在，下载下来改个名字放在你用户的 bin 目录下。

```bash
mv ~/下载/sunshine.AppImage ~/bin/sunshine
chmod 755 ~/bin/sunshine	#(1)! 
```

1. AppImage 程序可以直接终端运行，但是先给个执行权限。很惭愧，我自己为了图省事，全部给的是777权限。

本例中的 ` ~/bin/` 目录应当是一个你方便管理的，并且处在 PATH 环境变量下，方便你直接终端输入运行。在我的电脑上，我设置为 `~/bin` 而不是 `~/bin/.local/bin`来方便我管理。如果你采用的是 zsh 作为 默认 shell, 你可能已经把这个目录作为执行目录了，请自行 `echo $PATH` 查看。

接下来，终端运行 `sunshine` 命令，你就会发现你桌面右下角的系统托盘上多了一处地方，可以右键打开 web 管理器。为什么要这么描述？因为在我的电脑上，sunshine 在系统托盘上根本没有图标。

<figure markdown>   ![sunshine 任务栏](https://repo.sxrhhh.top/undefined7330e753652fd1e765ccb6a14b3cc4d2.jpg)   <figcaption>刚换 Linux, 这上面的 qq 截图死活截不到右键菜单，无奈，请选择你的拍屏导师.jpg</figcaption> </figure>

点击任务栏上面的那个 Open Sunshine，你就会被跳转到 Sunshine 的web 管理页面，会要求你新建一个账户和密码。这一套账号密码是用来以后登陆后台管理的，与 Moonlight 主控机那边无关。毕竟串流相当于全权交出电脑管理权限，web 页面的端口号要是爆了，能访问到你电脑的陌生设备可就能来帮你管理电脑了，所以设置账号密码也很重要。

!!!tip "小贴士"
    你要是忘了账号密码，可以直接终端输入 `sunshine --creds` 重置新的账号密码。

其实挺好玩，毕竟这一套帐密系统说是有账号密码，其实就是要你依次输入两个密码，我看不到账号的作用在哪里。

然后就是 Moonlight 主控端了，在任意一个你想要作为副屏的设备，比如手机、安卓平板、ipad、另一台电脑、甚至是 Steamdeck 和 Switch，都可以下载安装 Moonlight。各个平台不再赘述，操作方法大同小异。在安装完成 Moonlight 后，**先确保设备和电脑连接在同一个路由器下**，然后，打开 Moonlight ，等待软件找到你的电脑 Host。如果找不到，直接 IP 连接也是一个好主意。

!!!info "吐槽"
    如果你在确认同一个网段下，确实无法连接到电脑，可以选择检查你电脑的防火墙设置。和 Windows 下的防火墙不一样，Linux 下的防火墙是基于端口和 tcp/udp 协议的。毕竟个人桌面电脑不会跑什么公网服务，为了图省事，我直接把 firewalld 给 disable 了。

当你第一次连接到电脑时，Moonlight 会给出一个 PIN 码要求服务端确认。你可以点击右下角的弹窗提醒，也可以手动进入到 web 后台输入 PIN 码。

![image-20240115215240866](https://repo.sxrhhh.top/undefinedimage-20240115215240866.png)

根据终端命令给出的提示，你似乎还可以通过 `sunshine -0 xxxx` 这样的方式在终端输入 PIN 码来连接，但是我还没有测试。

如果一切顺利，你应该能够看到手机和电脑的画面同步了，并且手机上滑动屏幕还能操控鼠标移动。如果手机或平板有外接键盘，你甚至可以用那个键盘来打字。

## 创建虚拟显示屏

如果你是试图在 Linux 上串流游戏的，那么到这也就结束了。但我们目的并不停滞于此，我们想要的是一个副屏，能够把手机平板当作电脑显示屏的方案。很明显，我们现在只能手机电脑同步画面，想要一个新的显示屏用来串流，我们采用什么方法呢？**这才是本文的重点**。

在网上一部分的教程中，有人提出使用 HDMI 欺骗器用来创建一个屏幕。作为 Linux 系统，几乎所有设备都可以虚拟一个，那么还要连接一个欺骗器的方案肯定不符合我们的预期，我们需要一个纯软件的方案。一部分教程给出了一些第三方虚拟屏幕软件，基本都是 Windows 下的，既然是设备层面地去虚拟一个显示器，用 wine 去兼容的方案肯定不符合预期。还有一个方案提出下载一个叫 virtscreen 的应用，它有 AppImage 的包，很可惜它无法在我的设备上运行，即使我的系统完全符合要求，更何况这个软件局限于 Xorg 的 X11 桌面，不支持 Wayland。

最后，我在一个叫做 deskreen 的副屏方案的 issue 区找到了一个建立副屏的方案：<https://github.com/pavlobu/deskreen/issues/42>。他采用了 Linux 下 X 桌面原生的屏幕管理器 `xrandr`

。虽然同样不支持 Wayland，但是我相信 Wayland 下一定有一个对位的管理器可以用类似的方法实现本文方案。

!!!tip "小贴士"
    据 [千雪的咖啡厅](https://blog.chyk.ink/2022/07/17/linux-virtual-display/) 博客下描述，Wayland 用户应该可以用一个叫做 `krfb-virtualmonitor` 的命令行工具来创建虚拟显示屏，有可能比 X11 还要快。

这里提到了 deskreen 这个方案，但是我没有使用，可能是因为我不太喜欢用 web 这样的形式来扩展副屏，显得怪怪的。

现在我们打开终端，输入 `xrandr` ，就可以看到 X11 桌面下，你目前已经连接了几个显示屏。在正常情况下，你一般只会有一个显示屏显示 `connected` 。

![image-20240116132622658](https://repo.sxrhhh.top/undefinedimage-20240116132622658.png)

在我的电脑上，显示已经连接了一个叫做 `eDP` 的屏幕，紧跟着一堆 `DisplayPort` ，因为我的电脑是笔记本，所以是这个样子。根据电脑不同，这些名字还可能会有诸如 `HDMI-0` 、`DVI-D-0` 之类的设备名。

在图中，我们还能看到 `DisplayPort-0` 下面已经有了一个类似分辨率的东西，这是因为我已经配置完了，否则这一项会和下面的 1-7 完全一样。在这里你也能看到 `165.00` 旁边有一个 `*` 号，这表示这块屏幕已经向外输出视频了。

!!!tip "小贴士"
    在这里，我们采用的是现有的未连接通道，如果你不想要如此使用，而是自己虚拟一个显示屏，又或者是确实没有空余的未连接通道，你可以移步至千雪大佬的博客：[千雪的咖啡厅](https://blog.chyk.ink/2022/07/17/linux-virtual-display/)，那里会有新建一个显示屏的方法

```bash
xrandr --addmode DisplayPort-0 1920x1080 
xrandr --output DisplayPort-0 --mode 1920x1080 --left-of eDP
```

核心命令就是这么两行，**不要直接复制粘贴，请先往下看，然后根据自己的电脑情况修改参数**。

其中第一行表示给 `DisplayPort-0` 添加一个分辨率 mode，然后第二行就是要求这块屏幕输出内容了。那么这块屏幕应当与主屏幕连接以作为副屏使用，我们需要为它安排一个位置，注意最后两个参数 `--left-of eDP` ，很好理解，第一个参数就是表明我们创建的新屏幕应当放在第二个参数（我们主屏幕）的哪一个方位。本例中就是把副屏放在主屏幕的左边。这个方位一般有四种：`--left-of`、`--right-of`、`--above`、`--below`，这四个参数顾名思义，不多解释。

当你敲完第二个命令的时候，你的屏幕应该会闪烁一下，就跟 Windows 下调整完显示设置那样，整个桌面都刷新了。加入你设置的参数是 `--left-of` ，那么你把鼠标往最左边移动，就会发现左边的屏幕打通了，可以多往左移动一大段距离，再移回来。很明显，已经成功创建了一个虚拟屏幕。

如果你想关掉这个屏幕，也就是停止输出，可以执行：

```
xrandr --output DisplayPort-0 --off
```



### 写一个创建屏幕的脚本（可选）

这里的一切关于屏幕的改动都不是持久的，也就是说，电脑重启，甚至是桌面重启都会导致设置丢失，新屏幕消失不见。为了方便我们动态管理屏幕，我们不采用持久化的处理，而是写一个方便自己管理的脚本。于是，我利用我学习 shell 时长不到 10min 的技术，勉强写了一个能用的脚本，但确实方便很多。

```bash title="vmonitor.sh" linenums="1"
#!/bin/bash
xrandr --addmode DisplayPort-0 1920x1080 #(1)! 
if [ $# != 1 ]
then
        vmonitor --help
elif [ $1 == "below" ]
then
        xrandr --output DisplayPort-0 --mode 1920x1080 --below eDP
elif [ $1 == "left" ]
then
        xrandr --output DisplayPort-0 --mode 1920x1080 --left-of eDP
elif [ $1 == "right" ]
then
        xrandr --output DisplayPort-0 --mode 1920x1080 --right-of eDP
elif [ $1 == "above" ]
then
        xrandr --output DisplayPort-0 --mode 1920x1080 --above eDP
elif [ $1 == "off" ]
then
        xrandr --output DisplayPort-0 --off
elif [ $1 == "--help" ]
then
        echo "  参数："
        echo "  below           在主屏幕下方创建一个虚拟屏幕"
        echo "  above           在主屏幕上方创建一个虚拟屏幕"
        echo "  left            在主屏幕左方创建一个虚拟屏幕"
        echo "  right           在主屏幕右方创建一个虚拟屏幕"
        echo "  off             关闭虚拟屏幕"
else
        vmonitor --help
fi
```

1. 该脚本下所有命令重复执行都是安全的，所以我把这句话直接放在了第一行

然后我再把这个脚本软连接到 `~/bin` 下，这样一来，我就可以方便的用类似于终端命令的方法去快速地创建虚拟屏幕了。比如 `vmonitor left` 或者是 `vmonitor off`。各位可以自己去自定义。

## 将副屏进行串流

现在便是我们的最后一步。我们创建好了一个副屏，鼠标也是顺利地进入了那块地方，但是我们却没有办法看到。现在，拿出平板放在一边，我们要进入 Sunshine 去设置串流位置，也就是说让平板能够显示我们的副屏。

首先是在终端打开 Sunshine, 观察它的输出信息，可以看见有一列的显示屏编号：

![image-20240116192259024](https://repo.sxrhhh.top/undefinedimage-20240116192259024.png)

可以看到，我们的 `DisplayPort-0` 在终端上显示为 1 号，记录一下，这将是我们串流的默认通道。

然后进入到 Sunshine 的 Web 后台，找到显示器设置，将你的显示器编号输入进去。

<figure markdown>  ![image-20240116192851250](https://repo.sxrhhh.top/undefinedimage-20240116192851250.png) <figcaption>可以看到红框部分也是提示了我们，显示器编号应该去哪里找</figcaption> </figure>

现在，重启平板上的 Moonlight，重新连接电脑，你就会发现一件奇怪的事——平板黑屏了。如果你把鼠标移过去倒也能看到鼠标，但是就是没有画面。

其实这个问题倒也不是问题，如果你曾经因为桌面卡死而执行过 `killall plasmashell` 这种命令就会知道，其实只是没有桌面系统，也就是任务栏、桌面图片和一些控件，但是已经开启了的应用是可以在上面运行的。换句话讲，你现在把你主屏幕上的一些应用直接拖到左边副屏上，就已经可以使用了！

大功告成！！！

## 已知问题

在测试途中，倒也发现了一些问题。

首先就是在 Moonlight 连接状态时，如果你像我一样写好了 `vmonitor` 脚本，并且依次执行 `vmonitor off` 、`vmonitor left` 就会发现副屏切换回主屏幕界面，然后再切回副屏界面，所有应用保存在副屏。这个很符合预期。

**但是**，如果我们在已经关闭了副屏的基础上，也就是 `vmonitor off` 的情况下，在终端运行了 `sunshine` ，很有可能会造成 KDE 卡死，除了鼠标所有东西都不动了。我的解决方案是：切换到 TTY 下，运行 `sudo systemctl restart sddm` 来重启窗口管理器解决问题，代价就是一些已经开好了的桌面应用需要重新开启。

因此，每次开启电脑或是重启桌面后，必须先要运行 `xrandr` 命令，或是像我那样的 `vmonitor` 脚本，再运行 Sunshine, 防止桌面卡死。

第二点就是在 Moonlight 串流成功后，电脑会首先将声音扬声器转移到 Moonlight 控制端，也就是平板发出了声音。如果我再在任务栏将声音通道转回来，我的电脑会恢复发出声音的情况，但是我的平板并不会停止发出声音。我在声音设置下将除了耳机以外所有音频通道全部静音也无济于事。我的解决方案便是将平板在物理层面上静音，也就是设置设备的扬声器。



---

作者：Sxrhhh 

个人网站：[https://www.sxrhhh.top](https://www.sxrhhh.top/)  

博客园：[Sxrhhh - 博客园 (cnblogs.com)](https://www.cnblogs.com/sxrhhh/) 

转载请注明出处. 

在个人网站持续更新中……
