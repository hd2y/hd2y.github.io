---
title: "Win10 安装 MySQL 8.0"
date: "2019/07/18 15:36:21"
updated: "2020/02/19 09:39:21"
permalink: "win10-installs-mysql80"
tags:
 - Windows
categories:
 - [开发, 数据库, MySQL]
---

最近在研究 `FreeSql`，需要装 `MySQL` 数据库做一些测试，刚好碰到一些小问题简单记录一下。

## 下载

直接官网下载最新版本，下载 `msi` 文件进行安装，安装过程就是一直下一步了，没什么好说的。

下载地址：[https://dev.mysql.com/downloads/installer/](https://dev.mysql.com/downloads/installer/)

## 本地连接

安装完成可以运行 `MySQL 8.0 Command Line Client` 进行管理，输入安装时设置的 `root` 用户密码登录即可。

登录以后可以简单做一些查询：

```sql
show databases;
use mysql;
select * from user;
```

## Navicat 连接数据库

我是安装在局域网的远程机上，所以出现以下问题，这里做一个记录。

### Error 1130

`MySQL 8.0 Command Line Client` 工具使用 `root` 用户登录数据库，执行以下内容：

```sql
use mysql;
--正常安装后该字段为 host
select 'host' from user where user='root';
--通配符 % 也可以指定具体的 IP 地址
update user set host = '%' where user ='root';
--刷新 MySQL 的系统权限相关表
flush privileges;
--重新查看 user 表是否修改成功
select 'host' from user where user='root';
```

运行 `services.msc` 找到 `MySQL 8.0` 的服务重启。

### Error 2059

`MySQL 8.0` 之前的版本中加密规则为 `mysql_native_password`，而在`MySQL 8.0` 以后的加密规则为 `caching_sha2_password`，最直接的方案就是更新为旧版的加密规则。

因为服务重启，我们重新打开 `MySQL 8.0 Command Line Client` 登录用户 `root`，执行以下命令。

```sql
use mysql;
alter user 'root'@'%' identified with mysql_native_password by '密码';
```

当安装数据库版本为 `8.0.19` 时，使用 DBeaver 连接报错：
```html
The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```

这时我们只需要调整服务器时区为 `Asia/Shanghai` 即可。

![20200219093729](https://hd2y.oss-cn-beijing.aliyuncs.com/20200219093729_1582076319473.png)

以上执行结束，重新使用 Navicat 或 DBeaver 测试连接应该可以正常使用了。
