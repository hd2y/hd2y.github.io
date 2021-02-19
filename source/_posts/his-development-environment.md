---
title: "环境部署"
date: "2020/05/26 09:07:33"
updated: "2020/05/26 09:07:33"
permalink: "his-development-environment"
tags:
 - HIS
 - LIS
 - HongYang
categories:
 - [开发, C#]
---

## 配置数据库

首先需要安装数据库，安装的版本必须是 `Oracle 11.2.0.4.0`。

安装完成后，需要创建两个用户：

```sql
create user userhita170524 identified by hishita;
grant connect,resource,dba to userhita170524;
create user useremr_hita170524 identified by hita;
grant connect,resource,dba to useremr_hita170524;
```

用户创建完成后，使用 `imp` 导入数据前还需要创建表空间。

```sql
create tablespace USERS datafile 'C:\app\oradata\USERS.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace MAIN datafile 'C:\app\oradata\MAIN.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZICHAN datafile 'C:\app\oradata\TSP_ZICHAN.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_WUZI datafile 'C:\app\oradata\TSP_WUZI.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2012 datafile 'C:\app\oradata\TSP_MENZHEN2012.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGCANSHU datafile 'C:\app\oradata\TSP_XITONGCANSHU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HYBASEFLAT datafile 'C:\app\oradata\TSP_HYBASEFLAT.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HYBASEFLAT_IDX datafile 'C:\app\oradata\TSP_HYBASEFLAT_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGCANSHU_IDX datafile 'C:\app\oradata\TSP_XITONGCANSHU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR datafile 'C:\app\oradata\TSP_EMR.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGSHUJU datafile 'C:\app\oradata\TSP_XITONGSHUJU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGSHUJU_IDX datafile 'C:\app\oradata\TSP_XITONGSHUJU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN datafile 'C:\app\oradata\TSP_MENZHEN.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU datafile 'C:\app\oradata\TSP_ZONGHESHUJU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU_IDX datafile 'C:\app\oradata\TSP_ZONGHESHUJU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_CPMIS datafile 'C:\app\oradata\TSP_CPMIS.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_CPMIS_IDX datafile 'C:\app\oradata\TSP_CPMIS_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN datafile 'C:\app\oradata\TSP_ZHUYUAN.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_GRADING datafile 'C:\app\oradata\TSP_GRADING.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_GRADING_IDX datafile 'C:\app\oradata\TSP_GRADING_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_BASEFLAT datafile 'C:\app\oradata\TSP_BASEFLAT.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_BASEFLAT_IDX datafile 'C:\app\oradata\TSP_BASEFLAT_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN_IDX datafile 'C:\app\oradata\TSP_MENZHEN_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace SYSTEM datafile 'C:\app\oradata\SYSTEM.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HULIZHILIANG datafile 'C:\app\oradata\TSP_HULIZHILIANG.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HYFYSJ datafile 'C:\app\oradata\TSP_HYFYSJ.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HYFYSJ_IDX datafile 'C:\app\oradata\TSP_HYFYSJ_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_LISSHUJU datafile 'C:\app\oradata\TSP_LISSHUJU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_LISCANSHU datafile 'C:\app\oradata\TSP_LISCANSHU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_LISSHUJU_IDX datafile 'C:\app\oradata\TSP_LISSHUJU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2017_IDX datafile 'C:\app\oradata\TSP_MENZHEN2017_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2018_IDX datafile 'C:\app\oradata\TSP_MENZHEN2018_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_LISCANSHU_IDX datafile 'C:\app\oradata\TSP_LISCANSHU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2015 datafile 'C:\app\oradata\TSP_MENZHEN2015.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2015_IDX datafile 'C:\app\oradata\TSP_MENZHEN2015_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2016 datafile 'C:\app\oradata\TSP_MENZHEN2016.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2016_IDX datafile 'C:\app\oradata\TSP_MENZHEN2016_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2017 datafile 'C:\app\oradata\TSP_MENZHEN2017.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2018 datafile 'C:\app\oradata\TSP_MENZHEN2018.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2015 datafile 'C:\app\oradata\TSP_ZHUYUAN2015.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2015_IDX datafile 'C:\app\oradata\TSP_ZHUYUAN2015_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2016 datafile 'C:\app\oradata\TSP_ZHUYUAN2016.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2016_IDX datafile 'C:\app\oradata\TSP_ZHUYUAN2016_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2017 datafile 'C:\app\oradata\TSP_ZHUYUAN2017.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2017_IDX datafile 'C:\app\oradata\TSP_ZHUYUAN2017_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2018 datafile 'C:\app\oradata\TSP_ZHUYUAN2018.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2020_IDX datafile 'C:\app\oradata\TSP_MENZHEN2020_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_MENZHEN2020 datafile 'C:\app\oradata\TSP_MENZHEN2020.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_OA datafile 'C:\app\oradata\TSP_OA.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_OA_IDX datafile 'C:\app\oradata\TSP_OA_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_OAFILEIMG datafile 'C:\app\oradata\TSP_OAFILEIMG.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_PAIDUI datafile 'C:\app\oradata\TSP_PAIDUI.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_PAIDUI_IDX datafile 'C:\app\oradata\TSP_PAIDUI_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_RANKAPPRAISAL datafile 'C:\app\oradata\TSP_RANKAPPRAISAL.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace CRDS2 datafile 'C:\app\oradata\CRDS2.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_TIJIAN datafile 'C:\app\oradata\TSP_TIJIAN.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_TIJIAN_IDX datafile 'C:\app\oradata\TSP_TIJIAN_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_WUZI_IDX datafile 'C:\app\oradata\TSP_WUZI_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_WXSHUYE datafile 'C:\app\oradata\TSP_WXSHUYE.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_WXSHUYE_IDX datafile 'C:\app\oradata\TSP_WXSHUYE_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XUEYE datafile 'C:\app\oradata\TSP_XUEYE.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XUEYE_IDX datafile 'C:\app\oradata\TSP_XUEYE_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HISWEB datafile 'C:\app\oradata\TSP_HISWEB.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2015 datafile 'C:\app\oradata\TSP_ZONGHESHUJU2015.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2015_IDX datafile 'C:\app\oradata\TSP_ZONGHESHUJU2015_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2016 datafile 'C:\app\oradata\TSP_ZONGHESHUJU2016.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2017 datafile 'C:\app\oradata\TSP_ZONGHESHUJU2017.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2018 datafile 'C:\app\oradata\TSP_ZONGHESHUJU2018.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_HISWEB_IDX datafile 'C:\app\oradata\TSP_HISWEB_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHECANSHU datafile 'C:\app\oradata\TSP_ZONGHECANSHU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHECANSHU_IDX datafile 'C:\app\oradata\TSP_ZONGHECANSHU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2016_IDX datafile 'C:\app\oradata\TSP_ZONGHESHUJU2016_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2017_IDX datafile 'C:\app\oradata\TSP_ZONGHESHUJU2017_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU2018_IDX datafile 'C:\app\oradata\TSP_ZONGHESHUJU2018_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN_IDX datafile 'C:\app\oradata\TSP_ZHUYUAN_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZHUYUAN2018_IDX datafile 'C:\app\oradata\TSP_ZHUYUAN2018_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR datafile 'C:\app\oradata\TSP_EMR.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR_IDX datafile 'C:\app\oradata\TSP_EMR_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace USERS datafile 'C:\app\oradata\USERS.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2017 datafile 'C:\app\oradata\TSP_EMR2017.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2018 datafile 'C:\app\oradata\TSP_EMR2018.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2019 datafile 'C:\app\oradata\TSP_EMR2019.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2020 datafile 'C:\app\oradata\TSP_EMR2020.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGCANSHU datafile 'C:\app\oradata\TSP_XITONGCANSHU.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_XITONGCANSHU_IDX datafile 'C:\app\oradata\TSP_XITONGCANSHU_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2017_IDX datafile 'C:\app\oradata\TSP_EMR2017_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2018_IDX datafile 'C:\app\oradata\TSP_EMR2018_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2019_IDX datafile 'C:\app\oradata\TSP_EMR2019_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_EMR2020_IDX datafile 'C:\app\oradata\TSP_EMR2020_IDX.dbf' size 50M autoextend on next 10M maxsize unlimited;
create tablespace TSP_ZONGHESHUJU datafile 'C:\app\oradata\TSP_ZONGHESHUJU.dbf' size 50M autoextend on next 10M maxsize unlimited;
```

表空间创建完成后，便可以恢复数据了：

```sql
C:\app\hd2y\product\11.2.0\dbhome_1\BIN\imp userhita170524/hishita@127.0.0.1/ORCL file=F:\Oracle\userhita170524.dmp log=F:\Oracle\userhita170524.log buffer=819200 full=y
C:\app\hd2y\product\11.2.0\dbhome_1\BIN\imp useremr_hita170524/hita@127.0.0.1/ORCL file=F:\Oracle\useremr_hita170524.dmp log=F:\Oracle\useremr_hita170524.log buffer=819200 full=y
```

至此，数据已经恢复，可以安装 `Web` 环境。

## 安装 Web 环境

将 `Web` 解压，建议解压到 `E:\HongYang` 下。

解压后，在 `IIS` 默认站点右键“添加应用程序”，别名输入 `his_hita`，应用程序池选择 `.NET v4.5 Classic`，物理路径选择刚刚解压文件的 web 路径下。

首先还需要到服务中，启动 `ASP.NET State Service` 服务，建议设置自动启动。

这时还不要着急访问，还需要到 `Web` 下，修改 `DBConfig.xml` 文件，调整数据库配置文件，并将对应的数据库配置文件的内容适当调整，修改 `tns`，如果用户名密码修改也应适当调整。

如果没有问题，可以使用 [http://127.0.0.1/his_hita/W_LIS/login.html](http://127.0.0.1/his_hita/W_LIS/login.html) 访问 `LIS` 登录界面。

如果出现问题，可以到 [http://127.0.0.1/his_hita/W_LIS/Common/PaddleCard.ashx?type=-1&id=0001](http://127.0.0.1/his_hita/W_LIS/Common/PaddleCard.ashx?type=-1&id=0001) 测试，查看是否是数据库连接问题。

如果出现 `System.Data.OracleClient 需要 Oracle 客户端软件 version 8.1.7 或更高版本` 可以尝试以下操作：

1. 管理工具 → 计算机管理 → 本地用户和组 → 组 → `Administrators` 属性，添加 `NETWORK SERVICE`；
2. `Oracle` 安装路径 → 属性 → 安全，添加 `NETWORK SERVICE` 允许完全控制，确定一下 `ORACLE` 下 `BIN` 目录有没有这个权限。
3. 重启IIS。

这时应该就可以解决数据库无法连接的问题。

## 登录使用

`LIS` 需要在 `Chrome` 内核浏览器下访问，登录界面为：[http://127.0.0.1/his_hita/W_LIS/login.html](http://127.0.0.1/his_hita/W_LIS/login.html)

`HIS` 需要在 `IE` 下访问：[http://127.0.0.1/his_hita/](http://127.0.0.1/his_hita/)

常用工号：

+ `2999`：医生
+ `3999`：护士
+ `4999`：收费
+ `5999`：检验
+ `8888`：信息科

密码默认为 `0000`，如果错误可以将数据库内密码更新为 `0000` 即可访问。

需要注意的是，HIS 使用 IE 还需要设置添加兼容站点、设置安全站点、允许 ActiveX 控件控制、禁用弹窗拦截、禁用缓存等设置。

如果树形控件无法显示，需要按照文章 [Win10 中 TreeView 控件显示问题](https://www.hd2y.net/archives/display-problem-of-treeview-control-in-win10) 进行设置。
