---
title: "Win10 安装 Oracle"
date: "2019/07/19 16:04:06"
updated: "2020/02/11 13:49:16"
permalink: "win10-installs-oracle/"
categories:
 - [开发, 数据库, Oracle]
---

## 下载安装

首先下载地址：[https://www.oracle.com/technetwork/cn/database/enterprise-edition/downloads/index.html](https://www.oracle.com/technetwork/cn/database/enterprise-edition/downloads/index.html)

我试了一下是可以用迅雷下载，速度还可以。

安装没有什么需要特别注意的，基本上一直下一步安装就可以了。

安装完服务可能会弹出一些数据库初始化的信息框，根据提示操作运行即可。

## TNS 配置

首先我们可以测试一下是否可以正常使用，安装过程中有一步是设置密码，可以用这个密码登录 `SQL Plus`。

![oracle1](https://hd2y.oss-cn-beijing.aliyuncs.com/oracle1_1563523509015.png)

虽然可以正常访问，但是还是建议重新配置一下服务器上的监听程序。

打开随 Oracle 一起安装的 `Net Configuration Assitant`。

先配置监听程序：监听程序配置 -> 重新配置 -> `Listener` -> 使用标准端口号 `1521`

配置以后继续配置 Net 服务名：本地 Net 服务名配置 -> 服务名随意取一个 -> 主机名填客户端 IP -> 使用标准端口号 `1521` -> 完成

tns重新配置以后我们要找一下 tnsnames 的文件，在安装目录下，可以参考我安装的路径：`C:\app\czh\product\12.2.0\dbhome_1\network\admin`

将 `Host` 配置为 `localhost` 的修改为服务器的 IP 地址。

## 远程连接

首先应该安装 Oracle 的客户端，之前下载的页面对应版本查看全部有具体的下载：[https://www.oracle.com/technetwork/cn/database/enterprise-edition/downloads/oracle12c-windows-3633015-zhs.html](https://www.oracle.com/technetwork/cn/database/enterprise-edition/downloads/oracle12c-windows-3633015-zhs.html)

> 注意：网上搜索客户端会出现“即时客户端”或精简版客户端，如果需要进行开发，不建议安装。

安装以后找到 tnsnames 文件，将服务端配置的信息直接拷贝到客户端文件中，例如：`C:\app\client\Administrator\product\12.1.0\client_1\network\admin\tnsnames.ora`。

可以使用 `tnsping` 功能检查是否可以正常连接：

![oracle2](https://hd2y.oss-cn-beijing.aliyuncs.com/oracle2_1563523508898.png)

当然如果有问题，有可能是防火墙的问题，入站规则设置一下一般就可以。 

Oracle 数据库的管理“墙裂”建议使用 `PL/SQL Developer`，而且建议安装 `12.0` 以后的版本。

如果只是简单查一下数据，当然 Navicat 与 DBeaver 也是不错的选择：

![oracle3](https://hd2y.oss-cn-beijing.aliyuncs.com/oracle3_1563523508867.png)

> 注意：安装完成以后有一次测试连接出现了 `ora-12514`，检查 tnsping 命令正常，并且客户端 SERVICE_NAME 和 HOST 配置也没有问题纠结了好久，最后使用重启大法，重启了服务器自己好了，很是郁闷。
