---
date: 2023-07-14
slug: raspberrypi-omv-nas
categories:
    - 运维
tags: 
    - 树莓派
    - NAS
---


# 通过树莓派+OMV建立自己的NAS

最近看到家中吃灰的树莓派，我突发奇想：不如把它做成NAS。虽然肯定不如专业的NAS稳定，容量也才可怜的32G，但是这样让我练练尝尝鲜也未尝不可。说干就干！

<!-- more -->

> 前排提醒：经过笔者的踩坑，已经证明无线wifi连接树莓派做OMV的NAS不可行，必须要求连接**网线**！！！本文经过多次修改，如果文中出现WiFi相关信息，请忽略。


在这里先给出我的参考资料，毕竟网上资料鱼龙混杂，官方文档才是最好的！！！

[在树莓派上安装OMV（英文）](https://wiki.omv-extras.org/doku.php?id=omv6:raspberry_pi_install)

[OMV新用户手册（英文）](https://wiki.omv-extras.org/doku.php?id=omv6:new_user_guide)

## 1. 材料准备

硬件设备：

- 树莓派3B+一个
- 电源线一根
- 网线一根
- 32G的TF卡一张
- 离你树莓派近的路由器
- 容量足够的U盘或移动硬盘

所需软件：

- Raspberry Pi Imager，官方推出的软件，用于烧录树莓派系统。[官网下载](https://downloads.raspberrypi.org/imager/imager_latest.exe)
- Windows Terminal或是Putty等终端软件。
- 会简单使用Linux系统的大脑

## 2. 准备系统

现在万事俱备，开始制作咱们的树莓派了

### 2.1 烧录系统

首先，将我们的TF卡插入读卡器中，连接上电脑记住它的盘符，然后打开Raspberry Pi Imager

打开软件，选择操作系统，由于OMV的安装与存储策略，我们**必须选择精简版的64位系统**

![选择操作系统1](https://repo.sxrhhh.top/picgoimage-20230714173738097.png)

![选择操作系统2](https://repo.sxrhhh.top/picgoimage-20230714173902815.png)

最后选择我们插入TF卡的盘符

![选择TF卡](https://repo.sxrhhh.top/picgoimage-20230714174047249.png)

**注意，一旦烧录开始，磁盘上的所有数据都会丢失，请注意备份**

然后不要急着开始烧录，点击右下角的设置

![点击设置](https://repo.sxrhhh.top/picgoimage-20230714174307710.png)

打开ssh服务，并且设置用户名和密码，用户名可以为`pi`，密码需要用心设置，这是你的管理员密码，root账号已被锁定。

**注：请不要将用户名修改为`admin`，这是OMV的默认管理员账号，会引起冲突。**

![设置ssh与密码](https://repo.sxrhhh.top/picgoimage-20230714174350920.png)

~~然后是设置WiFi，这个是无线连接树莓派的必要条件，有网线的可以略过此步骤~~。

**接下来的`配置wifi`请去除勾选，因为即使配置完成，也会在安装OMV后删除有关wifi的一切配置。**

如果错过以上两个步骤，可以通过修改TF卡目录下文件完成WiFi和ssh的设置

![取消配置wifi](https://repo.sxrhhh.top/picgoimage-20230718170333903.png)

语言设置保持上海。

![语言设置](https://repo.sxrhhh.top/picgoimage-20230714175024593.png)

好了，现在可以完成设置，开始烧录了！

**再次提醒，开始烧录后原TF卡内数据将全部丢失！！！**

等待进度条结束，便烧录完成，拔下TF卡插入树莓派通电吧！

### 2.2 配置网络

#### 2.2.1 连接树莓派

在连接到树莓派之前，你需要查看树莓派和路由器的IP地址，才可以连接。

**首先将你的树莓派的网线连接到路由器上**，在本机上，打开Windows Terminal（以下简称cmd），输入`ipconfig`，查看默认网关。

![找到路由器IP](https://repo.sxrhhh.top/picgoimage-20230714175709588.png)

随后，在浏览器中访问路由器，找到树莓派的IP地址

![找到树莓派IP](https://repo.sxrhhh.top/picgoimage-20230714180058425.png)

然后便可在cmd中连接树莓派了

```shell
ssh pi@192.168.0.120
```


#### 2.2.2 更改本机hosts（可选）

为了方便自己的连接，在部署完内网穿透之前，我们可以在windows本机上修改hosts文件

打开`"C:\Windows\System32\drivers\etc\hosts"`

加入

```
192.168.0.120 pi.example.com #建议是你自己的域名，为以后的穿透做准备
```

通过管理员模式保存文件，然后在cmd中ping试试

```shell
ping pi.example.com
```

完成，以后可以用这个主机名连接了。

```shell
ssh pi@pi.example.com
```

 ## 3. 安装OMV

### 3.1 升级系统

在安装OMV之前，我们先接入树莓派，进行一下系统的升级

```bash
sudo apt-get update
sudo apt-get -y upgrade
```

然后重启

```bash
sudo reboot
```

**请注意：在接下来的每次重启后，如果发现ssh无法连接树莓派，请到路由器上重新查看树莓派的IP地址**

### 3.2 自动安装脚本

在树莓派上安装OMV非常简单，只需要运行官方提供的自动安装脚本即可一键安装。

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

如果遇到网络问题（被墙）等问题，可以选择直接下载``install``文件，然后通过ftp软件导入树莓派直接安装。

```bash
sudo bash ./install
```

经过漫长的等待，如果没出现问题，他会提示自动重启，随后你将自动断开连接。

> 根据多种因素，运行此脚本最多可能需要 30 分钟。
>
> ​					——OMV官网

现在，我们的OMV就已经安装完毕，可以进行配置了。

## 4. 配置OMV

### 4.1 基础配置

在浏览器中打开树莓派的IP地址，就可以看到登陆界面，输入你的用户名和密码。

用户名：`admin`	，默认密码：`openmediavault`

#### 4.1.1 打开仪表盘

进入网页，我们点击设置

![打开设置](https://repo.sxrhhh.top/picgoomv6ug_-_dash.jpg)

将里面的所有选项全部勾上，保存。然后你就能看到仪表盘了。

![仪表盘](https://repo.sxrhhh.top/picgoimage-20230718202017641.png)

#### 4.1.2 更改密码

打开下图界面，更改密码

![更改密码](https://repo.sxrhhh.top/picgoimage-20230718202329297.png)

#### 4.1.3 调整登出时间

点击下图设置，调整登出时间，可以保证你不会一离开电脑就登出的烦恼。

![登出时间](https://repo.sxrhhh.top/picgoimage-20230718202525161.png)

随后，保存，点击黄框中的确定，应用配置。

![黄框应用配置](https://repo.sxrhhh.top/picgoimage-20230718202827619.png)

> 这个项目有一点不错，就是保存和应用分离，这样保证你可以放心在多个栏目下更改设置，最后统一应用，既有速度，又有安全性，就是有一点小烦。。。

#### 4.1.4 安装 OMV-Extras

ssh连接树莓派，输入如下命令：

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install |sudo bash
```

同样的，你也可以通过直接下载`install`文件并导入树莓派来安装。

在安装成功后，你将看到OMV-Extras出现在菜单栏。

![菜单栏](https://repo.sxrhhh.top/picgoimage-20230718211158064.png)

### 4.2 配置存储

#### 4.2.1 添加文件系统

对于OMV，它不支持系统和数据共用一个磁盘，因此，将你的U盘或移动硬盘插入树莓派的USB接口。

打开网站后台，选择“磁盘”，然后选择U盘的磁盘，点击上方橡皮擦擦除数据。

<u>如果是系统盘，（如图中的/dev/mmcblk0），则无法擦除数据。</u>

![磁盘界面](https://repo.sxrhhh.top/picgoimage-20230718213702086.png)

然后，点击**“快速”**进行擦除数据。

**注：擦除后U盘内所有数据都将丢失且无法恢复，请注意备份**

随后，依次点击“文件系统”“加号”“EXT4”，选择刚才擦除的U盘，保存，应用。

![建立文件系统](https://repo.sxrhhh.top/picgoimage-20230718214047319.png)

在经历了一段时间的等待后，我们看见了如上图所示的文件系统Online标识，意味着我们文件系统建立成功！

#### 4.2.2 创建网络共享

做NAS，必然需要的是网络共享数据，那么我们现在就来将刚才创建好的文件系统共享吧！

<u>下图勘误：第二栏文件系统并非默认，而是选择刚才创建好的文件系统</u>

![共享文件夹](https://repo.sxrhhh.top/picgoimage-20230718214857678.png)

随后打开 SMB/CIF“Samba网络共享

如下图：

![打开SMB](https://repo.sxrhhh.top/picgoimage-20230718215727688.png)

打开SMB的设置，勾上最上方的启动（enabled），保存，应用。

随后打开下方共享，新建SMB共享，如下图所示。

![新建SMB共享](https://repo.sxrhhh.top/picgoimage-20230718220005434.png)

勾上下图红框中所有选项，注意第一栏为刚才的共享文件夹，第三栏为“允许访客”。其余选项全部保持默认。

![SMB设置](https://repo.sxrhhh.top/picgoimage-20230718220220178.png)

最后保存并应用设置。

最后结果应当如此：

![SMB结果](https://repo.sxrhhh.top/picgoimage-20230718220449259.png)

在最后，我们来到windows本机的“网络”，找到树莓派，双击进入。在此之前你可能需要等待几分钟。

![网络1](https://repo.sxrhhh.top/picgoimage-20230718222004116.png)

![网络2](https://repo.sxrhhh.top/picgoimage-20230718222022061.png)

![网络3](https://repo.sxrhhh.top/picgoimage-20230718222055606.png)

并且，他还可以像正常驱动器一样复制粘贴，新建文件夹和各种东西，非常的赞！！！

你还可以通过映射驱动器路径将它放在更外面的位置方便存储。

![已映射驱动器路径](https://repo.sxrhhh.top/picgoimage-20230718222504237.png)

---

作者：Sxrhhh

个人网站：<https://www.sxrhhh.top>

转载请注明出处.

在个人网站持续更新中……
