---
layout: post
title: Ubuntu 踩坑记录
date: 2018-01-13 19:52:26
categories: Linux
tags: Ubuntu Linux
author: renshuo
mathjax: true
---

* content
{:toc}
本文记录安装使用 `Ubuntu`  过程中遇到的问题和解决方法

目前使用的是 `16.04 LTS` 版本

<!--more-->

## 安装

双系统，128G SSD + 1T机械硬盘，UEFI + GPT分区，Windows10装在了固态硬盘里，Ubuntu装在了机械硬盘里，`Ubuntu`一共分了`200G` 左右

### 安装过程

1. 关闭 `Windows10` 电源设置里的快速启动，关闭 `BIOS` 里的 `secure boot`
2. 使用[UltraiSO](https://cn.ultraiso.net/)软碟通制作U盘启动盘
3. 从UEFI的U盘启动，开始安装，选择 `Try Ubuntu without installing` 或者 `Install Ubuntu` 都可以进入安装程序
4. 安装的步骤都差不多，主要讲一下分区，我一共给了`Ubuntu 200G`，以下大小仅供参考

| 分区挂载点  | 说明                       | 大小         |
| ------ | ------------------------ | ---------- |
| `swap` | 交换分区                     | 和电脑的内存大小相同 |
| `efi`  | 用来安装`Ubuntu`的启动引导器       | 建议`512M`   |
| /      | 系统根目录，类似`Windows`下面的`C`盘 | `100G`     |
| /home  | 各种文件                     | 剩余部分       |

安装完成后就可以使用了，如果重启之后直接进入的是`Windows`系统，请查看`BIOS`里的启动顺序，将 `Ubuntu` 放在第一个就好了

`Windows` 下可以使用 [`EasyUEFI`](https://www.easyuefi.com/) 这款软件管理 `EFI/UEFI`

> 参考文章：
>
> [UEFI + GPT双硬盘安装win10和Ubuntu16.04双系统](http://blog.csdn.net/mtllyb/article/details/78586540)
>
> [Uefi+GPT下双硬盘安装win10、Ubuntu双系统教程](http://tieba.baidu.com/p/4485636906?fr=ala0&pstaala=3&tpl=5&fid=834104)

## 遇到过的问题

### 显卡驱动

可以在 `设置 -> 软件和更新 -> 附加驱动`里选择安装最新的显卡驱动

千万不要使用离线安装包，手动更新驱动，更新系统软件的时候会很容易出问题

如果手动安装显卡驱动，更新系统软件之后，开机的登录页面出现循环登录的问题，使用 `Ctrl + Alt + F1` 进入纯命令窗口，打印根目录的 `.xsession_error` 文件，如果出现了以下内容：

```
openConnection: connect: 没有那个文件或目录
cannot connect to brltty at :0
upstart: gnome-session (Unity) main 进程 (3692)以状态 1 结束
upstart: unity-settings-daemon main 进程 (3682)已经被TERM 信号杀死
upstart: logrotate main 进程 (3531)已经被TERM 信号杀死
upstart: bamfdaemon main 进程 (3611)已经被TERM 信号杀死
upstart: hud main 进程 (3680)已经被TERM 信号杀死
upstart: 从告知的D-Bus总线断开
upstart: unity-panel-service main 进程 (3702)已经被TERM 信号杀死
```

说明是显卡驱动的问题，请按照以下方式解决

```
sudo apt-get remove --purge nvidia-*
sudo apt-get autoremove #特别重要
sudo apt-get install -f #特别重要
sudo reboot
```

重启之后，再次进入命令控制台，安装显卡驱动

```
sudo apt-get install nvidia-<驱动版本号>
```

驱动版本好需要去官网查，比如我的是 `nvidia-384`

> 参考文章
>
> [解决：Ubuntu16.04循环登录](https://www.jianshu.com/p/d45434f28ca0)

### 更新系统软件

在系统的设置里已经可以选择系统源了，所以不需要自己手动修改系统文件，推荐的清华源和中科大源

### 双系统时间不一致

重启切换回 `Windows10` 后，时间总是不对

首先保证`Ubuntu`下时间正确，命令行执行

```
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
```

然后将时间更新到硬件上

```
sudo hwclock --localtime --systohc
```

> 参考文章
>
> [windows10和ubuntu16.04双系统下时间不对的问题](http://blog.51cto.com/tyjhz/1888709)

