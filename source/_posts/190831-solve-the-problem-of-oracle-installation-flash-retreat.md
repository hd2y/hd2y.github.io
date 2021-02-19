---
title: "解决 Oracle 安装闪退问题"
date: "2019/08/31 19:46:37"
updated: "2020/02/11 13:39:01"
permalink: "solve-the-problem-of-oracle-installation-flash-retreat/"
tags:
 - 闪退
categories:
 - [开发, 数据库, Oracle]
---

Oracle 19c 安装注意事项：
+ 解压路径不能带有空格与特殊字符，否则安装程序闪退；
+ 解压的内容在安装后不能删除，因为一些基础工具例如 dmp、sqlplus、tns 配置文件等安装后会在该解压目录下。

---

最近在研究 [WTM 快速开发框架](https://github.com/dotnetcore/WTM)，但是目前还不支持 Oracle 数据库，所以想看下怎么兼容 Oracle 数据库。

因为不喜欢在自己的电脑上装数据库服务端，所以开始用的是公司的 Oracle 数据库进行测试，但是发现 Oracle 提供的 `Oracle.ManagedDataAccess.Core`总是报错，大概内容是语句不正确的结束。

注入日志发现生成的 SQL 中带有 `fetch first` 的内容，之前还在纠结Oracle 数据库不支持类似 MySQL 中 `limit` 的写法，毕竟 SQL Server 都开始支持了，现在反而被这个难住了。

查了下资料，发现 W3Cschool 上关于该关键字的语句标注了 `以下查询语句仅能在Oracle 12c以上版本执行`，所以公司的 `11g` 自然不能用了。

并且查了下资料发现网上 Oracle 发表的关于 `ODP.NET Core` 的文档写道：
```html
Oracle released a beta version of ODP.NET Core in February 2018. Oracle plans to release a production version of ODP.NET Core during the third quarter of 2018 at the same time as Oracle Data Access Components (ODAC) 18c.
```

好吧，既然这样直接在自己电脑上安装一个最新版 19c 好了，卸载掉我的 12c 客户端以后开始第一次安装。

一切都很顺利，但是，安装好以后我发现在 C 盘找不到 `tnsnames.ora` 文件，开始没在意，一边删除着 E 盘的安装文件，一边打开 `Oracle Net Configuration Assitant`，此时“文件不存在”和E盘安装文件的“删除错误，文件被占用”提示同时弹出，我悲伤的发现不知道为什么 Oracle 被安装到了我的移动硬盘E盘上，而且这些文件已经被我删除了大半。

无奈只能从网上找手动卸载 Oracle 的方式卸载后准备重装，主要就是停止服务、删除注册表、清除安装文件、删除配置文件和环境变量信息。

一切就绪后我重新把文件解压到C盘，然后准备开始安装，安装前特意把移动硬盘拔了下来，避免再出现“事故”。但是点击 `setup.exe` 却闪退了，无法进入安装操作。

开始我以为是卸载不完全的问题，正常也都会这么想吧，毕竟潜意识里一直认为 Oracle 不好卸载，花了半个小时，重新理了一遍注册表以及清除了 Windows 的 Temp 文件夹，重启电脑后安装，仍然闪退。

正在一筹莫展的时候，看着我解压放置的文件夹名，突然来了灵感，我解压的文件目录名称为 `Oracle Unzip`，我试着把文件夹名字中的空格删除，打开安装文件，居然运行了。

此时内心只有 woc 了，不知道能说什么，重新安装的时间顺便吐槽记录一下，然后完事儿继续干活了。😓

![20190831194414](https://hd2y.oss-cn-beijing.aliyuncs.com/20190831194414_1567254095315.png)
