---
title: "部署企业知识库 Confluence"
date: "2021/03/29 15:11:00"
updated: "2021/03/29 15:11:00"
permalink: "deploy-the-enterprise-knowledge-base-confluence/"
tags:
 - Confluence
 - Wiki
categories:
 - [开发, 工具]
---

## 一、Docker 部署

Confluence Docker Hub：https://hub.docker.com/r/cptactionhank/atlassian-confluence

```bash
# 创建容器
# 映射端口如果被占用可以调整例如 18090:8090
docker run -d --name confluence -p 8090:8090 --user root:root cptactionhank/atlassian-confluence:latest
```

## 二、安装过程

以下过程是在 Windows 下操作，将 Confluence 部署到 CentOS，需要拷贝文件或编辑文件，建议使用 `finalshell` 工具连接创建 SSH 连接。

### 1. 数据库

准备一个数据库，MySQL、PostgreSQL 等数据库均可，这里以 MySQL 为例，安装过程略。

首先需要调整 InnoDB 日志文件大小，Confluence 建议日志文件大小再 256M 以上。

在 Linux 下设置过程如下：

```bash
# 暂停 MySQL 服务
service mysqld stop

# 删除旧的日志文件
rm -f /var/lib/mysql/ib_logfile*

# 编辑 my.cnf 调整 innodb_log_file_size=256M
vim /etc/my.conf

# 重新启动 MySQL
service mysqld start
```

然后就是创建数据库以及授权：

```sql
-- 创建数据库
CREATE DATABASE `confluence` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- 创建用户
CREATE USER 'confluence'@'%' IDENTIFIED BY 'password';

-- 授权
GRANT ALL ON confluence.* TO 'confluence'@'%';

-- 登录 root 用户更改事务隔离级别
-- MySQL 8.x
SET GLOBAL transaction_isolation='READ-COMMITTED';
-- MySQL 5.x
SET GLOBAL tx_isolation='READ-COMMITTED';
```

### 2. 代理配置

首先调整 Confluence 的 Tomcat 配置，默认使用 `HTTPS` 代理访问。

```bash
# 复制配置文件
docker cp confluence:/opt/atlassian/confluence/conf/server.xml /opt/

# 编辑配置文件 Connector 节点，修改为示例 HTTPS 代理部分的内容、
#
#    <Connector port="8090" connectionTimeout="20000" redirectPort="8443"
#               maxThreads="48" minSpareThreads="10"
#               enableLookups="false" acceptCount="10" debug="0" URIEncoding="UTF-8"
#               protocol="org.apache.coyote.http11.Http11NioProtocol"
#               scheme="https" secure="true" proxyName="<subdomain>.<domain>.com" proxyPort="443"/>
#
# 注意需要修改 proxyName 属性为自己的域名

# 将配置文件覆盖回去
docker cp /opt/server.xml confluence:/opt/atlassian/confluence/conf/server.xml
```

这里使用 Caddy 进行代理：

```bash
# 安装 Caddy 软件包
yum install caddy -y

# 使用 vim 编辑 Caddyfile
vim /etc/caddy/conf.d/Caddyfile.conf

# 修改配置文件，代理指定接口并且 HTTP 访问自动跳转到 HTTPS
#
# https://<subdomain>.<domain>.com {
#  gzip
#  tls <user>@<emaildomain>.com
#  proxy / localhost:18090 {
#   transparent
#  }
# }
# 
# http://<subdomain>.<domain>.com {
#  redir https://<subdomain>.<domain>.com{url}
# }
#
# 注意需要调整域名信息以及邮箱，邮箱用于自动是申请域名证书

# 开启自启 Caddy 服务
systemctl enable caddy

# 启动 Caddy
service caddy start

# 停止运行 Caddy
service caddy stop

# 重启 Caddy
service caddy restart

# 查看 Caddy 运行状态
service caddy status
```

### 3. 安装及破解

访问 `https://<subdomain>.<domain>.com` 执行以下安装流程。

安装页面可以调整语言，建议修改为中文。

1. `设置 Confluence` 选择 `产品安装`。
2. `选择功能` 根据需求选择。
3. `授权码` 过程见下方破解流程。
   + 下载授权工具：https://pan.xunlei.com/s/VMSNMp0MczvkJfcam4CAxLXlA1 提取码：`zrv6`。
   + 复制 jar 包：`docker cp confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar /opt/`。
   + 打开授权工具：`java -jar confluence_keygen.jar`。
   + 将授权页面的 `Server ID` 拷贝到授权工具中，并补充必填信息后点击 `.gen!` 按钮生成 Key。
   + 将从容器中拷贝下来的 `atlassian-extras-decoder-v2-3.4.1.jar` 更名为 `atlassian-extras-2.4.jar`，然后点击 `.patch!` 进行破解。
   + 当显示 `Jar successfully patched.` 表示破解成功，将破解后的文件名改回 `atlassian-extras-decoder-v2-3.4.1.jar`。
   + 将文件拷贝回容器中覆盖原有文件：`docker cp /opt/atlassian-extras-decoder-v2-3.4.1.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar`。
   + 刷新页面，将授权工具生成的 Key 填入并继续。
4. `配置数据库` 选择 `我自己的数据库`。
5. `设置你的数据库` 填写自己的数据库信息及密码，填写以后等待安装完成即可。

**注意：** 使用授权工具需要安装 Java。

## 三、常用目录

+ 产品开发
  + 001-团队大纲
  + 002-团队成员
  + 003-新人指南
  + 004-技术架构
  + 005-会议纪要
  + 006-团队规范
  + 007-踩坑历程
  + 008-业务梳理
  + 009-部门团建
  + 010-团队分享
  + 011-业务小组
  + 012-团队总结

> 参考：
> + [教你如何通过Docker安装配置知识管理系统：confluence](https://blog.csdn.net/qq_36154886/article/details/113246166)