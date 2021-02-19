---
title: "使用 CCProxy 实现代理上网"
date: "2019/07/18 17:04:16"
updated: "2020/02/11 13:58:00"
permalink: "using-ccproxy-to-realize-agent-internet-access"
tags:
 - 代理
 - CCProxy
categories:
 - [操作系统, 软件]
---

因为公司电脑如果想要正常上网需要申请，而我有一台远程机需要联网而又懒得申请，便找了个代理软件来联网。

首先下载代理软件 `CCProxy`：[Proxy Server Software for Windows](https://www.youngzsoft.net/ccproxy/)

## 安装与配置

同样是傻瓜式的下一步，安装后启动即可，需要注意的是软件要安装在有外网的电脑上。

默认如果是英文界面，可以进行设置：Option -> Advance -> Language -> OK，建议设置以后重启一下。

注意设置界面“请选择本地局域网 IP 地址”可能识别是错误的，如果是错误的需要取消自动检测，选择正确的局域网 IP。

![ccproxy1](https://hd2y.oss-cn-beijing.aliyuncs.com/ccproxy1_1563440884263.png)

设置保存以后建议重新启动代理软件。

## 远程机配置

打开 IE 浏览器，设置代理：Internet 选项 -> 连接 -> 局域网设置

![ccproxy2](https://hd2y.oss-cn-beijing.aliyuncs.com/ccproxy2_1563440884261.png)

然后选择高级进行如下设置：

![ccproxy3](https://hd2y.oss-cn-beijing.aliyuncs.com/ccproxy3_1563440884260.png)

需要注意的是端口要与我们设置的端口保持一致。

设置以后“没有意外的话”就可以正常上网了：

![ccproxy4](https://hd2y.oss-cn-beijing.aliyuncs.com/ccproxy4_1563440884265.png)
