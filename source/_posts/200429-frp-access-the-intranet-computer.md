---
title: "使用 frp 实现内网穿透访问内网电脑"
date: "2020/04/29 22:55:35"
updated: "2020/05/22 09:21:22"
permalink: "frp-access-the-intranet-computer/"
tags:
 - frp
 - mstsc
 - 远程
categories:
 - [开发, 工具]
---

远程办公的工具，一般会选择 TeamViewer 或者向日葵，但是这些都是收费工具。

免费的 TeamViewer 经常会被禁用，又不想用破解版，收费价格比较贵。向日葵虽然收费可以接收，但是使用过都知道，使用体验比 TeamViewer 差了太多。

我个人更倾向于使用 `mstsc`，但是访问公司内网电脑，就需要想办法穿透到内网来使用。

## 准备工作

后续的操作需要一台有公网 IP 的电脑或服务器，否则就可以跳过这篇文章了。

我的服务器系统为 `CentOS 7.6`，其他的可以参考，除了下载的程序包不同外，其他大同小异。

首先是下载安装包，下载地址：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)。

我下载的时候，最新发布版本是 `v0.33.0`，建议使用最新版。

因为我使用的服务器是 `CentOS`，所以要下载一个 `Linux` 的程序包。客户端是 `Windows 10`，需要下载一个 `Windows` 的程序包。

而 `Linux` 选项又那么多，如果不能确定可以先通过以下命令打印出服务器类型：

```bash
# 查看服务器类型
uname -m
x86_64
```

如果输出是 `i386` 或 `i686` 需要下载 `386`，如果和我一样是 `x86_64` 需要下载 `amd64`，如果是 `arm` 需要选择 `arm64` 或 `arm`。

## 服务端配置

可以通过以下命令，下载并解压： 

```bash
# 下载 frp
wget https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz frp.tar.gz
# 解压到 frp 文件夹
tar -zxvf frp.tar.gz --strip-components 1 -C ./frp
```

解压完成后可以先查看解压的内容：

```bash
# 进入解压目录
cd frp
# 列出文件夹内文件信息
ls
frpc_full.ini  frps  frps_full.ini  frps.ini  LICENSE  systemd
```

很容易理解，`*.ini` 为配置文件，`frps` 为 **s** 结尾就是服务端，`frpc` 为 **c** 结尾就是客户端。

我们部署的是服务端，所以只需要关注 `frps.ini` 与 `frps`。

首先修改 `frps.ini` 配置文件：

```bash
# 修改 frp 服务配置文件
vim /root/frp/frps.ini
```

内容如下：

```ini
[common]
bind_port = 7000
dashboard_port = 7500
token = 12345678
dashboard_user = admin
dashboard_pwd = 12345678
```

配置节点的内容说明如下：
+ `bind_port`：服务端监听的端口，客户端配置的时候需要用到。
+ `token`：就是一个平平无奇的 `token`，同样是配置客户端时要用到的。
+ `dashboard_port`：状态面板的端口，服务启动以后可以通过该端口查看状态信息。
+ `dashboard_user`：状态面板的登录用户名。
+ `dashboard_pwd`：状态面板的登录密码。

配置完成保存后，就可以通过该配置文件启动 `frp` 服务端了。

```bash
# 启动 frp 服务
./frps -c frps.ini
```

成功启动后，我们可以登录状态面板查看。

![frps bashboard](https://www.hd2y.net/upload/2020/04/frps%20bashboard-a59be7c170714d5f826b7a97416c1779.png)

此时虽然启动成功，但是如果我们 `Ctrl + C` 退出，程序也就跟着退出了，我们需要让该服务端后台运行，并且能够开机启动，就需要执行以下操作。

```bash
# 创建后台启动模版
vim /etc/systemd/system/frp.service
```

该文件内容如下：

```ini
[Unit]
Description=frps
After=network.target

[Service]
ExecStart=/root/frp/frps -c /root/frp/frps.ini 

[Install]
WantedBy=multi-user.target
```

配置完成就可以启动服务了。

```bash
# 启动测试
systemctl start frp.service
# 查看启动状态
systemctl status frp.service
# 开机自启
systemctl enable frp.service
```

## 客户端配置

客户端的话就需要调戏以下 `frpc.exe` 与 `frpc.ini` 了。

`frpc.ini` 文件修改如下：

```ini
[common]
server_addr = 111.231.237.167
server_port = 7000
token = 12345678
[rdp]
type = tcp
local_ip = 127.0.0.1           
local_port = 3389
remote_port = 7001  
[smb]
type = tcp
local_ip = 127.0.0.1
local_port = 445
remote_port = 7002
```

配置节点的内容说明如下：
+ `server_addr`：这个很好理解了，就是我们服务器的 IP 地址。
+ `server_port`：前文 `frps` 中配置的服务器监听的端口。
+ `token`：前文 `frps` 中配置的通行证。
+ `local_ip`：本地 IP，无需赘述。
+ `local_port`：需要转发到的本地端口，因为 `Windows` 的远程端口默认是 `3389`，所以这里使用的是 `3389`。
+ `remote_port`：服务端转发用的端口，例如我这里配置的是 `7001`，那么我从其他电脑远程到这台电脑时，就可以使用 `111.231.237.167:7001` 来完成连接。

配置完成后需要启动客户端，才可以从其他电脑访问。（这不是废话吗？）

```bash
cd C:\frp
./frpc -c frpc.ini
```

这时有个命令窗不能关闭，太麻烦了，如果想隐藏，建议创建一个批处理文件，并放到启动文件夹下。

`frpc.bat` 文件内容如下：

```bash
@echo off
cd /d %~dp0
if "%1" == "h" goto begin
mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin
REM
cd C:\frp
frpc -c frpc.ini
exit
```

可以通过运行 `%programdata%\Microsoft\Windows\Start Menu\Programs\Startup` 打开启动文件夹，把文件放进去就能开机启动。

执行该批处理文件，把软件运行起来，接下来就是测试效果了。

> 注意：`cd /d %~dp0` 这一行不添加，以管理员身份运行脚本将报错 `当前页面脚本发生错误：系统找不到指定的文件`。

> 注意：客户端节点 `[rdp]` 与 `[smb]` 为代理任务名，在多台电脑配置时，除需要修改 `remote_port` 外，该名称也需要调整，否则执行 `frpc -c frpc.ini` 将提示名称被占用。

## 测试效果

当然，正常情况下 Windows 的 `mstsc` 肯定是没问题了，接下来要玩点花样。

这里首先使用我的小黑果来完成测试，使用的软件是 `Microsoft Remote Desktop`，测试效果如下。

![remote test](https://www.hd2y.net/upload/2020/04/remote%20test-39e2bb06096e45daaa45c2bf9214c5a0.jpg)

而且移动设备也可以连接，例如本文部分内容是小米手机安装了 `RD Client` 后，连接了罗技的蓝牙鼠标与键盘编辑的，输入和鼠标使用都很正常，就是有点费眼睛。

![image.png](https://www.hd2y.net/upload/2020/04/image-ea7543dd30784070a5ce4deeff972f84.png)

另外我还尝试了 iPad 连接使用，毕竟 iPad OS 现在吹的神乎其神。

但是实际体验鼠标体验很差，而键盘连接后在远程工具中既不能调用远程电脑的输入法，也不能使用 iPad 的输入法输入，还是别想拿 iPad 充当生产力工具了，还是老老实实爱奇艺吧。

> 参考：
> + GitHub：[fatedier / frp](https://github.com/fatedier/frp)
> + StackOverflow：[How to detect 386, amd64, arm, or arm64 OS architecture via shell/bash](https://stackoverflow.com/questions/48678152/how-to-detect-386-amd64-arm-or-arm64-os-architecture-via-shell-bash)
