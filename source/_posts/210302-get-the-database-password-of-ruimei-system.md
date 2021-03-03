---
title: "获取瑞美系统数据库密码"
date: "2021/03/02 09:05:00"
updated: "2021/03/02 09:05:00"
permalink: "get-the-database-password-of-ruimei-system/"
tags:
 - LIS
 - 瑞美
categories:
 - [开发, 业务]
---

## 前言

瑞美单机版默认打包了其所有已经开发的仪器通信接口，并且包含一个配置文件，记录了软件安装后初始化的一些参数。

包括：labitem、labitemref、labiteminter、labitemval、labitemparam、labitemexp。

当前是打算写一个小工具，将瑞美内已经维护的这些参数，批量读取到我们的云检测数据库中，作为以后新增仪器的基础参数，方便未来维护仪器以及检验项目信息。

但是目前参数文件内的部分内容含义不明，需要通过瑞美的数据库表结构，完成这些内容的匹配。

## 安装

瑞美的软件可以在其官网下载：http://www.ruimei.com.cn/p/softdownload/index.html

这里建议下载《瑞美单机版 5.0》，解压可以得到安装文件：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-01.png)

安装时会提示选择安装的仪器型号，下拉列表实际提供的是 instr 文件夹内的子文件夹列表。

确认安装后，会拷贝对应文件夹内的通信接口程序，并读取其内的参数文件，向数据库添加默认的参数信息。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-02.png)

## 使用

安装以后即可启动瑞美程序，简单测试一下程序是否安装正常。

默认安装后没有登录密码，无需输入直接“登录”即可进入主界面。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-03.png)

启动时通信接口程序会同时启动，如果需要测试通信接口，密码为 `13311667706`。

然后就需要找到瑞美的数据库，打开接口根目录：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-04.png)

可以很清楚的看到了，`database` 内存放的就是其数据库文件，`dbdriver` 为其驱动文件，`E411` 则是主程序、通信接口文件所在的目录。

## 反编译

> 以下反编译操作仅供学习。

网上检索了一些资料，并没有找到关于瑞美单机版数据库如何连接，只知道其使用的数据库是 `sybase`。

另外，查看了下 `ODBC`，发现安装后多出了一个名为 `lis2017` 的数据源。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-05.png)

而该数据库文件默认是带有密码，无法使用 `Windows 身份认证` 等认证方式进行登录。

而找到密码最简单的自然是通过反编译，查看程序里如何获取数据库身份认证信息。

这里推荐使用 `PBKiller` 进行反编译，反编译的程序可以选择瑞美的通信解码程序，也就是之前提到的 `instr` 内的程序。

`PBKiller` 下载地址：http://www.greenxf.com/soft/118706.html

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-06.png)

`PBKiller` 查找内容使用体验不好，建议导出后使用 `Notepad++` 等文本工具检索。

因为猜测其用户名没有修改，默认为 `DBA`，在 `Notepad++` 中打开导出的所有文件，输入该关键字直接搜索“所有打开的文件”，可以很容易的找到其设置数据库连接信息的内容。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-07.png)

可以看到，其如果使用的数据库为 `lis2002` 时，密码为 `thisisapig`，否则密码为 `thisisaflyingpig`。

注：另外，开发接口时如果不知道接收到的内容如何解析，可以参考瑞美接口内的反编译内容，经研究发现 `wf_addresult` 为其保存解析结果的方法，可以查找该方法锁定解析的位置。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-08.png)

## 连接数据库

首先需要启动数据库，可以运行瑞美根目录 `dbdriver` 文件夹下的 `dbeng9.exe` 程序，托管数据库文件。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-09.png)

如果程序成功启动，可以看到其显示托管的端口，上图显示端口为 `2638`。

数据库建议使用 `sybase central` 进行连接，下载地址：http://www.121down.com/soft/softview-93247.html

该程序为绿色版，需要运行批处理脚本注册，如果无法正常启动，可以尝试运行 `卸载Sybase Central插件.bat` 脚本后，重新注册。

启动 `sybase central` 后，选择 `工具` → `连接` → `Sybase IQ`，标识标签下，用户输入 `DBA`，口令输入 `thisisaflyingpig`，数据库标签下，我们可以查找可供连接的服务器：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-10.png)

正常情况下,这时所有准备工作就完成了,右键连接的数据库,选择 `Open Interactive SQL` 可以打开 SQL 脚本编辑框:

![](https://hd2y.oss-cn-beijing.aliyuncs.com/get-the-database-password-of-ruimei-system-11.png)

> 注意：  
> 1. 以上过程仅供学习。
> 2. 部分单工、双工接口如果不想开发上位机接口，可以通过读写瑞美数据库数据实现。
> 3. 瑞美单机版数据库使用版本比较低，印象中测试需要使用 `.NET Framework 2.0` 提供的驱动才能连接成功。
> 4. 目前网络版使用比较多，使用的一般是 `SQL Server` 数据库，如果需要对接，建议直接向瑞美工程师申请，开放视图进行读取。
