---
layout: post
title: Ubuntu 配置和美化
date: 2018-01-13 20:52:45
categories: Linux
tags: Ubuntu Linux
author: renshuo
mathjax: true
typora-copy-images-to: pictures\Ubuntu
---

* content
{:toc}
主要是记录在使用 `Ubuntu` 的过程中，发现的一些比较有意思的软件和系统的美化教程

目前使用的是 `16.04 LTS` 版本

<!--more-->

## 软件篇

### 系统工具软件

#### [shadowsocks-qt5](https://github.com/shadowsocks/shadowsocks-qt5)

科学上网工具，官方的安装指南在 [`Wiki`](https://github.com/shadowsocks/shadowsocks-qt5/wiki) 中，连接结点后，系统的`设置 -> 网络 -> 网络代理` 要设置成 `手动 -> sock主机：127.0.0.1，端口：1080`

#### [Unity Tweak Tool](https://launchpad.net/unity-tweak-tool)

各种方便的设置

#### [Indicator Sysmonitor](https://launchpad.net/indicator-sysmonitor)

一个可以在标题状态栏显示使用率和网速的软件，添加网速之后长度会随数值变化，可以按照下面方法修改：

```
sudo gedit /usr/lib/indicator-sysmonitor/sensors.py 
```

然后修改 `B_UNITS` ：

``` python
B_UNITS = ['KB', 'MB', 'GB', 'TB']
```

还要修改下面 `def bytes_to_human(bytes_)` 返回的字符串格式：

``` python
return '{: 4d}{:2s}'.format(int(bytes_), B_UNITS[unit]) # 4前面加了空格，2后面加了s
```

再将系统的字体改成等宽字体，就完全没问题了，等过段时间想想不修改字体怎么解决这个问题

### 应用软件

#### [GoldenDict](http://goldendict.org/)

`Linux`下最好用的词典软件，没有之一。`Windows`下也能用这个，我现在用的是欧路词典，离线的字典可以去 [掌上百科](http://www.pdawiki.com/)找

## 美化篇

### GRUB2主题更换 

1. 在`/boot/grub`里创建`GRUB2`主题目录`themes` 

```
sudo mkdir -p /boot/grub/themes
```

2. 将下载的主题文件夹放入上一步的 `themes` 目录中
3. 修改 `/etc/default/grub`

```
sudo gedit /etc/default/grub
```

添加 `GRUB_THEME=”/boot/grub/themes/<主题文件夹名>/theme.txt” ` 并保存

4. 更新`GRUB`

```
sudo update-grub
```

**关机**，再开机就能看到效果了

5. 修改启动顺序也可以在 `/etc/default/grub` 这个文件中更改

> 参考文章
>
> [Ubuntu16.04定制启动界面【添加grub2主题】](http://www.ahutcloud.cn/616#content)
>
> [GRUB2主题包手动安装设置及其简易修改](https://www.jianshu.com/p/b956db975af5)
>
> [GRUB2主题包手动安装设置及其简易修改](http://tieba.baidu.com/p/4196513782)

### 主题图标字体

其实所有的东西基本上都是 `添加源 -> 更新源 -> 下载安装` 三个步骤，之后安装之后的配置使用，可是用上面提到的 `Unity Tweak Tool` 进行设置，下载的时候**注意要符合系统版本**

![效果图](/pictures/Ubuntu/Ubuntu-theme.png)

#### 主题

[Themes Collection by NoobsLab](https://launchpad.net/~noobslab/+archive/ubuntu/themes) 添加方法 `sudo add-apt-repository ppa:noobslab/themes`

可以都下载下来，喜欢哪个用哪个

我目前使用的是 `arc-flatabulous-theme` 黑色版本的 `flatabulous-theme`

 与 `Mac` 类似的底部图标栏 `cairo-dock` 我没有使用

#### 图标

[PPA for the Numix Project Ltd](https://launchpad.net/~numix/+archive/ubuntu/ppa) 添加方法 `sudo add-apt-repository ppa:numix/ppa`

[Collection of icons PPA](https://launchpad.net/~noobslab/+archive/ubuntu/icons) 添加方法 `sudo add-apt-repository ppa:noobslab/icons`

[Paper Themes (Daily Builds)](https://launchpad.net/~snwh/+archive/ubuntu/pulp)

目前图标使用的是 `paper`

#### 鼠标

使用的是`mac os`的指针，这里也安装了主题和图标

```
sudo add-apt-repository ppa:noobslab/macbuntu
sudo apt-get update
sudo apt-get install macbuntu-os-icons-lts-v7
sudo apt-get install macbuntu-os-ithemes-lts-v7
```

#### 字体

**fonts-wqy-microhei** *文泉驿微米黑*   `sudo apt-get install fonts-wqy-microhei`

### 终端

[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

> 参考文章
>
> [不美翻怎么开发!Ubuntu 16.04 LTS深度美化!!!](https://www.jianshu.com/p/4bd2d9b1af41)