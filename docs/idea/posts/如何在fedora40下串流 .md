--- 
date: 2024-05-25
slug: how-to-stream-sunshine-or-steamlink-in-fedora40
categories:
    - 想法
tag:
    - Fedora
    - Sunshine
    - Linux
    - Steam Link
readtime: 1
--- 


# 如何在 fedora 40 下畅快地串流 Sunshine 或者 Steam Link

省流：回滚到 X11 。

```
sudo dnf install plasma-workspace-x11
```

哈哈。。。

---

不是开玩笑，今天折腾了半天，wayland 下真的莫名其妙的权限管理，让各种串流软件失效，而且解决步骤及其麻烦，至少在我的电脑上行不通，可能是我菜吧。但是每个软件都麻烦的搞一次，今天是 Sunshine, 明天是 Steam Link, 后天估计就是 OpenVNC 了。不想冒这个险，就回滚了。挺好的，X11 先用着吧。

这 blog 实在是太没有营养了，就归类这里吧（笑
