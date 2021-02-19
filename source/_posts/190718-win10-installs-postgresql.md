---
title: "Win10 安装 PostgreSQL"
date: "2019/07/18 17:12:21"
updated: "2020/02/11 13:56:46"
permalink: "win10-installs-postgresql/"
categories:
 - [开发, 数据库, PostgreSQL]
---

## 下载安装

直接在官网下载需要版本的安装包：[PostgreSQL Database Download](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

安装过程还是无聊的下一步，这里不再赘述。

## 启动

安装成功可以从开始菜单找到 `pgAdmin`，运行管理我们安装的数据库。

打开以后需要我们输入密码，输入我们安装时设置的数据库密码即可。

![postgresql1](https://hd2y.oss-cn-beijing.aliyuncs.com/postgresql1_1563442901788.png)

如果不习惯英文，可以在：`Configure pgAdmin` -> `Miscellaneous` -> `User language` 中选择中文。

选择数据库服务器，查看属性可以查看到我们当前数据库的基本信息：

![postgresql2](https://hd2y.oss-cn-beijing.aliyuncs.com/postgresql2_1563442901770.png)

## 远程连接

### 防火墙设置

Win10 默认会有防火墙限制端口访问，所以首先我们需要添加入站规则，允许其他电脑连接数据库服务器端口，例如我们安装的这台默认是 `5432`。

首先我们找到防火墙的高级设置：运行 `control` 打开控制面板 -> Windows Defender 防火墙 -> 高级设置

添加新的入站规则：入站规则 -> 新建规则 -> 选择“端口” -> 选择“TCP” -> 选择“特定端口”并输入`5432` -> 选择“允许连接” -> 配置文件默认全选 -> 名称建议使用“PostgreSQL”

![postgresql3](https://hd2y.oss-cn-beijing.aliyuncs.com/postgresql3_1563442901769.png)

> 如果 `ping` 命令无法使用，可以设置入站规则启用“文件和打印机共享(回显请求 - ICMPv4-In)”
> ![postgresql5](https://hd2y.oss-cn-beijing.aliyuncs.com/postgresql5_1563443190959.png)

配置完成以后，可以使用 `telnet` 测试是否可以连接这个端口，需要先到程序中启用 `telnet` 功能，服务器和客户端电脑都要启用。

![postgresql4](https://hd2y.oss-cn-beijing.aliyuncs.com/postgresql4_1563442901769.png)

客户端电脑命令行测试：

```html
telnet 192.168.101.145 5432
```

### 允许远程 IP

配置了端口访问，PostgreSQL 仍然没有结束，默认情况下其只允许使用 `127.0.0.1` 这个IP来访问服务器，我们可以通过配置 `pg_hba.conf` 来解除客户端限制。

首先我们到安装目录找到这个文件，例如：`C:\Program Files\PostgreSQL\11\data\pg_hba.conf`

找到 IP 设置这一段：

```html
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

添加我们需要连接数据库服务器的客户端的 IP：

```html
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             192.168.100.143/32            md5
```

保存以后无需重启服务器，正常情况下使用 Navicat 或 DBeaver 进行测试即可正常连接。
