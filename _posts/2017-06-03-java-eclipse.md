---
layout: post
title:  Eclipse的安装配置
date: 2017-06-03 10:06:38
categories: 工具
tags: Java 工具
author: renshuo
mathjax: true
---

* content
{:toc}

我叕要开始学习Java了，由于强迫症的原因，看见eclipse又出新版本了，准备重新弄一个玩，但是每次配置eclipse的步骤总是记不住，所以写一个eclipse的安装配置手册。

<!--more-->

## 下载

[eclipse的下载地址](https://www.eclipse.org/downloads/eclipse-packages/)，又两种方式，一种是比较传统的压缩包，一种是常规的安装文件。

## 安装

使用安装包进行安装只要按照步骤来就行了，压缩包直接解压到一个目录，然后将运行文件添加到开始菜单或者桌面。使用安装包安装的时候可能是网不好，遇到了一些问题，所以还是使用压缩包安装吧。直接解压。

## 打开

第一次打开会选择workspace，然后会进入一个欢迎页面，这些都不重要。。。

## 外观配置

窗口主题设置：Preferences -> Appearance -> Theme -> Dark，或者使用插件[**Darkest Dark Theme for Eclipse**](https://www.genuitec.com/tech/darkest-dark/)，最新的eclipse Oxygen版本的黑色版适配的比较好，基本不用使用插件。

字体大小配置：Appearance -> Colors and Fonts -> Basic -> Text Font -> Edit

代码高亮主题设置：直接导入epf文件

注释模板导入：Preferences -> Java -> Code Style -> Code Templates -> Import

## 插件

* [Waka Time](https://wakatime.com/help/plugins/eclipse)

## eclipse导出导入配置文件

也可以将常用的配置导出为**.epf**文件，然后再导入。

* 导出：在已配好的工作空间中，选择File->Export->General->Preferece，导出全部，设置导出的路径和文件名
* 导入：在新建的工作空间中，File->Import->General->Preference，选择刚才的配置文件，选中import all