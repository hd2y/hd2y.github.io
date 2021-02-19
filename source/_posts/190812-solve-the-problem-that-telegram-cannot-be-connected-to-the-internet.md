---
title: "解决 Telegram 无法联网的问题"
date: "2019/08/12 19:13:01"
updated: "2020/02/11 13:44:00"
permalink: "solve-the-problem-that-telegram-cannot-be-connected-to-the-internet/"
tags:
 - 代理
 - Telegram
categories:
 - [操作系统, 软件]
---

## 前言

最近重新安装了 Telegram，但是发现无法登录，代理工具已经正常开启，Google 也可以访问，但是 Telegram 的网络检测总是不通过，简单研究了一下终于发现了怎么解决网络连接的问题。

## 解决方案

首先我们需要打开代理设置的界面，可以点击左下角旋转的网络连接的图标：

![20190812115937](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812115937_1565608467455.png)

选择添加代理（Add Proxy）：

![20190812120000](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812120000_1565608467456.png)

弹出的界面，设置我们代理工具使用的协议和端口，设置后保存：

![20190812120036](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812120036_1565608467455.png)

保存以后如上图左下角，可以看到连接已经正常。

> 附：汉化可以搜搜 `@zh_CN`，然后选择搜索结果的第一个安装语言包，如下图：
> ![20190812192040](https://hd2y.oss-cn-beijing.aliyuncs.com/20190812192040_1565609023103.png)

> 注意：MacOS 设置后可能需要重启软件才能生效，这是一个历史遗留问题，在 GitHub 上有讨论。

