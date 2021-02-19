---
title: "Win10 中 TreeView 控件显示问题"
date: "2019/08/12 18:55:16"
updated: "2020/02/11 13:45:27"
permalink: "display-problem-of-treeview-control-in-win10/"
tags:
 - TreeView
categories:
 - [操作系统, 工具]
---

## 前言

最近重装了系统，Win10 中 TreeView 控件自然又是不能显示的。

![20190812153344](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812153344_1565607827452.png)

之前在 CSDN 中整理过解决方案，但是早已经放弃了在 CSDN 写文章，所以在个人博客再重新整理一下这个问题的处理方案。

## 解决方案

### 下载与安装

百度网盘：[http://pan.baidu.com/s/1cANFPG](http://pan.baidu.com/s/1cANFPG) 密码：zblv

解压文件到：`C:\inetpub\wwwroot`

![20190812154140](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812154140_1565607827452.png)

> 注意：压缩文件时没有注意，有两个层级，文件夹名称都是 `webctrl_client`，要进第二层再解压缩。

### 效果

解压到对应目录后直接刷新，树形控件即可正常显示，但是需要注意的是，该控件仅提供显示，并不是 VS 控件。

![20190812154217](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812154217_1565607827453.png)
