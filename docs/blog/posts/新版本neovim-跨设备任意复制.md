---
date: 2024-06-06
slug: neovim-copy-anywhere
categories:
    - 运维
tags:
    - wsl
    - Neovim
    - vim
    - OSC-52
    - Lua
---
# 穿透 wsl 和 ssh, 新版本 neovim 跨设备任意复制，copy anywhere!

## 1. 创作动机

最近一个星期，我入坑了 neovim, 然后开始配置各种插件。同一个时间点，我入手了一台 surface go2, 这是个 Windows 平板，我在上面也是装好了各种软件，配置了 wsl2, 并且配置了 ssh。然后我发现当我 ssh 连接到宿舍的高性能笔记本的时候，我打开 neovim 时候无法进行复制粘贴。严格地说，是无法进行复制，因为你无法使用 `ctrl+shift+C` 进行复制。所以我们通常在本机配置 `opt.clipboard:append("unnamedplus")` 来直接让 neovim 和系统剪贴板打通。但是这一次我们通过 ssh 传输数据，傻眼了。

因此，寻找一个可以通过 ssh 传输剪贴板的方案，~~迫在眉睫~~.

<!-- more -->

## 2. 原理介绍

首先，我们要介绍一下 nvim 传输剪贴板的方式。

第一步，当 neovim 执行 `"*y` 或者 `"+y` 的时候，就会把选中的内容写入到系统剪贴板中。这一步，并不是直接写入系统剪贴板的。neovim 会按照一定的优先级调用程序，而当某个程序符合要求的时候，neovim 就会调用这个程序。在以 X11 驱动的 Linux 桌面上，这个程序是 `xclip`。也就是说只要有 `xclip` 这个程序，neovim 就可以把内容写入到剪贴板。Wayland 、MacOS 、Windows 等平台都是有对应的剪贴板程序的，我们可以在 neovim 中使用 `:help clipboard-tool` 来查看对应的程序。

还是第一步，如果 neovim 是使用 ssh 远程连接的，那么远程的我们肯定也是无法访问到系统剪贴板的。所以我们可以通过设置 `g:clipboard` 这个变量，让 neovim 执行 `"*y` 或者 `"+y` 的时候，把内容作为传入到 `g:clipboard` 这个变量定义的函数中。这一步我们可以实现将复制的内容利用 `OSC-52` 传输到远程的机器上。

第二步，我们在本地的机器上操控远程的 neovim 时，会接收到传来的 `OSC-52` 内容。如果你的终端模拟器能支持 `OSC-52` 的话，它就会帮你自动的把传来的内容写入到系统剪贴板中。但是反过来的话却不一定，这个是安全原因，因为如果你的终端想要发送剪贴板的内容，必然需要读取系统剪贴板。这个是有风险的。如果开启，我们可以在远程上直接使用 `p` 命令粘贴。但是不开启的话，我们仍然可以进入 `insert` 模式，然后使用 `ctrl+shift+v` 粘贴。

第三步，我们配置好 `g:clipboard` 变量之后，就可以直接使用 `"*y` 或者 `"+y` 进行复制。但是如果我们想要更方便，不想使用寄存器，直接使用 `y` `p` 命令，那么我们需要让 neovim 默认使用无名寄存器，配置加入这么一条就行了: `opt.clipboard:append("unnamedplus")`。

如图就是这一套剪贴板内容传输的流程图。当你配置完成之后，你就可以不需要多余的操作而直接复制粘贴了。

![](https://repo.sxrhhh.top/undefined20240605015014.png)

下面就是具体的各种情况的配置过程。

## 3. 配置 ssh 传输

### 3.1 理论基础

ssh 传输剪贴板是目前用处最广的, 配置它我们就需要利用好 `OSC-52` 协议。

首先，我们需要确认好我们目前使用的终端是否支持 `OSC-52` 协议。网上可以找到一张主流终端模拟器是否对 `OSC-52` 协议的支持表。这里我就贴下来了。

- Windows平台 - Windows Terminal：支持
- Mac 平台 - iTerm2：支持
- Mac 平台 - terminal.app：不支持
- Ubuntu 平台 - Gnome Terminal：不支持
- Chromebook - hterm：支持
- 跨平台 - alacritty：支持
- 跨平台 - kitty：支持
- 终端复用 - tmux：支持
- 终端复用 - screen：支持

也就是说，当你使用 Windows Terminal 作为你连接和使用远程 linux 上的 neovim 时，你就可以直接使用 `OSC-52` 协议进行剪贴板传输，请放心使用。

然后，我们需要在远程的 neovim 中配置 `g:clipboard` 变量。这个变量分为两个部分，一部分是 copy，另一部分是 paste。copy 部分是当执行 `"*y` 或者 `"+y` 的时候，把选中的内容作为参数传给 `g:clipboard.copy，`paste 部分是当执行 `"*p` 的时候，把远程的 neovim 的剪贴板内容作为参数传给 neovim 来直接粘贴。然而通常，paste 会导致终端读取我们本地的剪贴板，经过网络传输，不太安全。所以一般大部分终端模拟器的 `OSC-52` 协议是**不可读取**的。这包括了 **Windows Terminal**
 
打开 neovim， 输入 `:help clipboard-osc52` 就可以看到 `OSC-52` 的帮助文档。这说明 neovim 已经原生支持了 `OSC-52` 协议。网上似乎说这一条的毕竟少，毕竟 neovim 0.10 是三个星期前刚发布的，之前用的人多了就没有关注这个新配置。而我正好是最近入坑的，查帮助文档看到了这个。在此之前人们基本上都是推荐 `vim.oscyank` 这个插件来支持 `OSC-52` 协议的。既然有原生的支持，我们就**不需要**再安装这个插件了。

### 3.2 按照官方文档配置 

我们先在 neovim 里面执行 `:help clipboard-osc52` 看一下官方文档。如果你之前正常配置过 neovim ，就应该会有一个自己的配置文件目录，这里不再赘述，所以我们直接贴出官方的配置文档。

```lua
    vim.g.clipboard = {
      name = 'OSC 52',
      copy = {
        ['+'] = require('vim.ui.clipboard.osc52').copy('+'),
        ['*'] = require('vim.ui.clipboard.osc52').copy('*'),
      },
      paste = {
        ['+'] = require('vim.ui.clipboard.osc52').paste('+'),
        ['*'] = require('vim.ui.clipboard.osc52').paste('*'),
      },
    }
```

我们只需要把这一段配置复制到自己的配置文件中，然后重启 neovim 即可。

~~如果配置到这里就结束了，那么也不失为美好的结果。但是既然是个新东西，又怎么会让你如愿呢？~~

之前说过，Windows Terminal 是不支持读取剪贴板的，所以当远程的 neovim 尝试执行 `"*p` 的时候，会试图通过 `OSC-52` 协议读取本地剪贴板的内容，然后传给远程的 neovim。但是 Windows Terminal 是不支持读取剪贴板的，所以这个过程会失败。我们在 neovim 中执行 `:reg` 来查看寄存器，可以看到 neovim 弹出了报错：

```
Waiting for OSC 52 response from the terminal. Press Ctrl-C to interrupt ...
```

正因如此，每当我们执行 `"*p` 的时候，都会弹出一个报错。我们必须要手动按下一个回车键，才能正常粘贴，而且粘贴的还是普通的 `""` 临时寄存器。这非常的影响体验。因为我们确实只需要复制功能，粘贴功能我们只需要进入 `INSERT` 模式，然后使用 `ctrl+shift+v` 粘贴就可以了。现在我们来重新配置一下，让它不再报错。

### 3.3 配置不报错

上文说到，如果本地终端不支持剪贴板的读取，那么远程的 neovim 会尝试通过 `OSC-52` 协议读取本地剪贴板的内容，然后失败，影响写作体验。

在 github 上有这么一个 [issue](https://github.com/neovim/neovim/issues/28611)，提出了一个解决方案如下：

```lua
function no_paste(reg)
    return function(lines)
        -- Do nothing! We can't paste with OSC52
    end
end

vim.g.clipboard = {
    name = "OSC 52",
    copy = {
         ["+"] = require("vim.ui.clipboard.osc52").copy("+"),
         ["*"] = require("vim.ui.clipboard.osc52").copy("*"),
    },
    paste = {
        ["+"] = no_paste("+"), -- Pasting disabled
        ["*"] = no_paste("*"), -- Pasting disabled
    }
}
```

可以看到，这个方案是直接把 `paste` 设置为一个空函数。这样当 neovim 执行 `"*p` 的时候，就不会再试图去利用 `OSC-52` 协议读取本地剪贴板的内容了。

但是这个方法有一个问题，就是说当进行粘贴的时候，因为函数返回的格式并不符合官方文档的要求，那么 neovim 就会报错。

```
clipboard: provider returned invalid data
```

这个报错每一次粘贴都会出现，还是红字非常刺眼。我对此很难受，所以还需要想办法解决这个问题。解决思路就是：**因为我们预期的粘贴命令是粘贴 neovim 自己的 `y` 复制，所以我们只需要让 paste 可以返回 `""` 临时寄存器的内容就可以了。**

以下就是我的配置，把上面的配置替换成下面这个：

```lua
function my_paste(reg)
    return function(lines)

        --[ 返回 “” 寄存器的内容，用来作为 p 操作符的粘贴物 ]
        local content = vim.fn.getreg('"')
        return vim.split(content, '\n')
        
    end
end

if (os.getenv('SSH_TTY') == nil)
then
    --[ 当前环境为本地环境，也包括 wsl ]
    opt.clipboard:append("unnamedplus")
else
    opt.clipboard:append("unnamedplus")
    vim.g.clipboard = {
      name = 'OSC 52',
      copy = {
        ['+'] = require('vim.ui.clipboard.osc52').copy('+'),
        ['*'] = require('vim.ui.clipboard.osc52').copy('*'),
      },
      paste = {
        --[ 小括号里面的内容可能是毫无意义的，但是保持原样可能看起来更好一点 ]
        ["+"] = my_paste("+"),
        ["*"] = my_paste("*"),


    },
}
end
```

在这一段 lua 配置中，可以看到我用了一个 if 条件判断，检测 `$SSH_TTY` 环境变量来检测是否为本地环境。如果当前环境是本地环境，那么除了使用 `unnamedplus` 寄存器以外什么也不做，否则就使用 `OSC-52` 协议传输剪贴板。

关于本地环境，因为没有设置 `g:clipboard` 变量，所以 neovim 就会根据 `:help clipboard-tool` 文档来选择默认的剪贴板。因为我们 linux 机器上面安装了 `xclip` 这个软件，所以 neovim 就会把内容复制到本地的系统剪贴板。

而远程环境，因为设置了 `g:clipboard` 变量，所以 neovim 会使用 `OSC-52` 协议来传输它打算复制到系统剪贴板的内容。而粘贴的时候，则会调用我写的 `my_paste` 函数，这个函数会返回 `""` 寄存器的内容。因此，我成功的把 `p` 的功能给修复到了正常的状态。

## 4. 配置 tmux (可选)

如果你使用的是 tmux，那么你可以通过配置 tmux 来跳过步骤 3。我并没有使用 tmux 的打算，但是如果你使用 tmux，那么你需要配置一下。网上关于 tmux 开启 `OSC-52` 协议的配置很多，这里我不再赘述，我也暂时没有配置的打算。我只讲一下 tmux 代理的原理。

其实也非常好懂，当你使用 tmux 的时候，neovim 不会认为你的终端是来自远程 ssh 连接的。他只知道将内容发送到 tmux 进程，然后由 tmux 转发给远程的 neovim。

根据[官方文档](https://github.com/tmux/tmux/wiki/Clipboard)，tmux 只需要这么设置就可以开启 `OSC-52` 协议了。

在 `.tmux.conf` 文件中添加如下配置：
```bash
set -s set-cliplboard on
# 好像还有一些，为了避免误人子弟还是建议自己去查一下
```     

这样，tmux 就会把内容发送给本地终端。

!!!warning "勘误"
     上文说到，neovim 不会认为你的终端是来自远程 ssh 连接的。其实这句话是不对的，`$SSH_TTY` 环境变量是否设置，取决于你的 tmux 实在什么环境下开启的。那么如果你是 tmux 和直连混用的人，你需要这么配置

    首先先 `echo $TERM` 检查一下你的终端类型。通常情况下，当我直连终端时，这个值为 `xterm-256color`，当使用 tmux 的时候，这个值会变成 `tmux-256color`。

     然后，改写我之前的 lua 配置，利用 `if (os.getenv('TERM') == 'tmux-256color')` 判断。具体的配置教程可以参考 `:help g:clipboard`, 也可以参考网上教程。

## 5. 配置 wsl

到了这里，我很高兴的告诉你，在我的设备上(wsl 2.1.0, neovim 0.10.0)，只需要一个 `opt clipboard:append('unnamedplus')`，wsl 就会自动和系统剪贴板交换，虽然使用 `win+V` 快捷键查看系统剪贴板可能不一定能看见。

如果你的配置还是不行，建议参考一下 `:help clipboard-wsl`。

## 6. 总结

我本来是不愿意来 neovim 的，所以自从接触 vim 之后也是过了大半年才入了 neovim 的坑。正因为来晚了，我才看到官方已经原生支持 `OSC-52` 协议了。当然对于各位早已入坑的 neovim 玩家来说，也完全没必要去折腾。我希望这篇文章能带给和我一样的 neovim 新手，让他们能够在被 neovim 的剪贴板折磨的时候，能够通过搜索引擎，搜索到这么一篇 2024 年的教程。也希望大家和我多多交流，互相学习。个人网站下方的评论区已开放，欢迎大家来到博客园、知乎、个人网站的评论区一起交流。


---

作者：Sxrhhh 

个人网站：[https://www.sxrhhh.top](https://www.sxrhhh.top/)  

博客园：[Sxrhhh - 博客园 (cnblogs.com)](https://www.cnblogs.com/sxrhhh/) 

转载请注明出处. 

在个人网站持续更新中……
