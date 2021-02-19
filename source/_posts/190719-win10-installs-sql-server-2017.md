---
title: "Win10 安装 SQL Server 2017"
date: "2019/07/19 11:40:32"
updated: "2020/02/11 13:53:37"
permalink: "win10-installs-sql-server-2017/"
categories:
 - [开发, 数据库, PostgreSQL]
---

## 安装

下载自然是在：[https://msdn.itellyou.cn/](https://msdn.itellyou.cn/)

安装和旧版并没有太大的区别，因为我们只是用来测试，序列号位置可以直接选择 `Developer`，另外需要安装 `Java` 的运行环境：[Java SE Runtime Environment 8 Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)。

需要注意的是如果我们勾选了机器学习安装过程会比较久，而且如果电脑没有网络需要下载机器学习包才能继续安装。

具体可以参考：[实例的 SQL Server 数据库内分析的累积更新 CAB 下载](https://docs.microsoft.com/zh-cn/sql/advanced-analytics/install/sql-ml-cab-downloads?view=sql-server-2017)

因为下载比较慢，我也上传了一份到了百度网盘：[https://pan.baidu.com/s/1WkdpZK1qC8Ft4PaYEBGByQ](https://pan.baidu.com/s/1WkdpZK1qC8Ft4PaYEBGByQ) 提取码：`xybu`

里面 `1033` 结尾是英文版，`2052` 对应的是中文版，可以针对自己安装的 SqlServer 版本选择下载，如果不对的话无法执行下一步操作。

安装的时候登录选项尽量选择混合登录，启用 `sa` 用户。

如果已经安装了旧版本，例如我还安装了 2012，建议实例名添加安装的服务器版本方便区分，例如我设置的是 `MSSQLSERVER2017`。

另外安装完成我们还需要安装：[下载 SQL Server Management Studio (SSMS)](https://docs.microsoft.com/zh-cn/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017)

## 远程连接

打开 `SQL Server 配置管理器`，如果安装了其他版本的数据库并且曾经启用过远程连接，在 `SQL Server 网络配置` 中禁用 `TCP/IP` 与 `Named Pipes`，并启用 `MSSQLSERVER2017` 中的 `TCP/IP` 与 `Named Pipes`。

双击 `MSSQLSERVER2017` 中的 `TCP/IP`，找到 `IP地址` 中的 `IPAll`，`TCP 端口` 修改为 `1433`。

![sqlserver1](https://hd2y.oss-cn-beijing.aliyuncs.com/sqlserver1_1563507669459.png)

调整网络配置以后，还需要到 `SQL Server` 服务中重启 `SQL Server (MSSQLSERVER)` 与 `SQL Server (MSSQLSERVER2017)` 的服务。

正常情况下，我们在命令行中运行 `netstat -ano` 可以找到 `1433` 端口，并且该端口对应的 `PID` 与我们安装的 `SQL Server 2017` 的一致。

![sqlserver2](https://hd2y.oss-cn-beijing.aliyuncs.com/sqlserver2_1563507669460.png)

这时正常我们就能使用 SSMS、Navicat 或 DBeaver 连接 SQL Server 2017 了：

![sqlserver3](https://hd2y.oss-cn-beijing.aliyuncs.com/sqlserver3_1563507669460.png)

如果仍然不能访问可能需要设置一下入站规则，设置以后可以使用 `telnet` 测试端口是否可以连接。

![sqlserver4](https://hd2y.oss-cn-beijing.aliyuncs.com/sqlserver4_1563507669459.png)
