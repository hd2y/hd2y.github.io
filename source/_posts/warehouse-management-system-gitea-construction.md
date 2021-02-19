---
title: "代码仓管理系统 Gitea 搭建"
date: "2019/11/12 10:08:15"
updated: "2019/12/25 18:44:08"
permalink: "warehouse-management-system-gitea-construction"
tags:
 - Git
 - Gitea
categories:
 - [开发, 工具]
---

开始用二进制文件搭建，结果一直启动不起来，后来换了一个版本可以启动起来，但是设置自启动总是提示找不到 `[Unit]`，但是配置文件是配置了的。

很无奈，太菜了只能用 Docker 来安装了，还简单一些。

## 升级 Git

```bash
# 安装依赖
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum install  gcc perl-ExtUtils-MakeMaker

# 卸载 git
git --version
yum remove git

# 安装 git
cd /usr/local/src/
wget https://www.kernel.org/pub/software/scm/git/git-2.24.0.tar.xz
tar -vxf git-2.24.0.tar.xz
cd git-2.24.0
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
source /etc/profile
git --version
```

## 安装 Docker

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum install docker-ce

# 开机自启
sudo systemctl enable docker

# 启动服务
sudo systemctl start docker
```

## 安装 Gitea

```bash
docker pull gitea/gitea:latest
sudo mkdir -p /var/lib/gitea
docker run -d --name=gitea -p 10022:22 -p 10080:3000 -v /var/lib/gitea:/data gitea/gitea:latest
```

到这里就可以使用端口 `10080` 访问了，可以登录进网站执行一些初始化的操作，例如设置数据库信息，网站根目录等。

## 配置 Gitea

安装时我们将 docker 的数据持久化到 `/var/lib/gitea` 文件夹下，所以可以到该文件夹下找 `app.ini` 文件。

我这里使用的是 mysql 数据库，性能会比 SQLite3 好很多，如果需要的话自行安装。

```bash
vim /var/lib/gitea/gitea/conf/app.ini
```

```ini
APP_NAME = Gitea: Git with a cup of tea
RUN_MODE = prod
RUN_USER = git

[repository]
ROOT = /data/git/repositories

[repository.local]
LOCAL_COPY_PATH = /data/gitea/tmp/local-repo

[repository.upload]
TEMP_PATH = /data/gitea/uploads

[server]
APP_DATA_PATH    = /data/gitea
SSH_DOMAIN       = git.hd2y.net
HTTP_PORT        = 3000
ROOT_URL         = https://git.hd2y.net/
DISABLE_SSH      = true
SSH_PORT         = 10022
SSH_LISTEN_PORT  = 10022
LFS_START_SERVER = true
LFS_CONTENT_PATH = /data/git/lfs
DOMAIN           = git.hd2y.net
LFS_JWT_SECRET   = ****
OFFLINE_MODE     = false

[database]
PATH     = /data/gitea/gitea.db
DB_TYPE  = mysql
HOST     = 127.0.0.1:3306
NAME     = gitea
USER     = root
PASSWD   = ******
SSL_MODE = disable
CHARSET  = utf8
```

## 配置 Caddy

Caddy 类似 Nginx 的反向代理软件，但是配置会简单很多，并且可以自动帮我们申请 SSL 证书。

```bash
# 安装 Caddy 软件包
yum install caddy -y

# 使用 vim 编辑 Caddyfile
vim /etc/caddy/conf.d/Caddyfile.conf
```

对该配置文件进行修改：

```json
https://git.hd2y.net {
 gzip
 tls hd2y@outlook.com
 proxy / localhost:10080 {
  transparent
 }
}

http://git.hd2y.net {
 redir https://git.hd2y.net{url}
}
```

修改完成之后启动 Caddy 服务即可。

```bash
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

## 参考文档

> Gitea 官方文档：[https://docs.gitea.io/zh-cn/](https://docs.gitea.io/zh-cn/)
> 
> Caddy 官方文档：[https://github.com/caddyserver/caddy/wiki/v2:-Documentation](https://github.com/caddyserver/caddy/wiki/v2:-Documentation)
