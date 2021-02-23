---
title: "Windows XP 提示激活无法启动"
date: "2020/03/24 11:30:08"
updated: "2020/11/03 20:51:21"
permalink: "windows-xp-prompt-activation-can-not-start/"
tags:
 - Windows XP
 - 虚拟机
categories:
 - [操作系统, MacOS]
---

[MacOS 安装 Windows XP SP3 测试环境](https://www.hd2y.net/archives/macos-installation-windows-xp-sp3-test-environment) 一文我在 MacOS 中使用 Parallels Desktop 安装了 Windows XP 用于测试。

但是最近使用时出现了一些问题，这里记录一下。

## 提示激活死循环

当成功启动虚拟机后，Windows XP 系统提示：“在您可以登录前，此副本的 Windows 必须被 Microsoft 激活。您现在想激活它吗？”

选择“否”，系统关机。（没有成功关机，卡在壁纸上无响应。）

选择“是”，提示：“Windows已激活，单击‘确定’以退出。”

但是我们点击“确定”后，却闪回“在您可以登录前，此副本的 Windows 必须被 Microsoft 激活。您现在想激活它吗？”

至此，系统启动进入死循环。

### 解决方案

解决该问题，首先我们需要关机，进入安全模式。

首先我们右键 Dock 里的 Windows 图标，停止运行 Windows XP 虚拟机：

![stop running](https://hd2y.oss-cn-beijing.aliyuncs.com/stop%20running-6e2d6923d5884ec7be27de6357e69331.jpg)

然后选择配置，进入“硬件” -> “启动顺序”，将“选择启动时的引导设备”勾选起来。

![open boot](https://hd2y.oss-cn-beijing.aliyuncs.com/open%20boot-f360cde3b4634f24ac7366f77235acf9.jpg)

重新启动我们的 Windows XP 虚拟机，在启动时按“F8”，选择“安全模式”进入。

在安全模式下，打开 cmd，运行一下命令：

```bash
rundll32.exe syssetup,SetupOobeBnk
```

运行后重启，即可正常进入系统。

> 以上命令只是延长试用期，建议直接安装 VOL 版，使用密钥激活。

```bash
ed2k://|file|zh-hans_windows_xp_professional_with_service_pack_3_x86_cd_vl_x14-74070.iso|630237184|EC51916C9D9B8B931195EE0D6EE9B40E|/

key MRX3F-47B9T-2487J-KWKMF-RPWBY
```

## 安装 IE8

系统内置的为 IE6，可以下载 IE8 的安装包进行安装：[下载地址](http://download.microsoft.com/download/1/6/1/16174D37-73C1-4F76-A305-902E9D32BAC9/IE8-WindowsXP-x86-CHS.exe)

> 参考：
> + 百度知道：[等求助，XP提示已激活，确是个死循环进不了系统](https://zhidao.baidu.com/question/139652708321204485.html)
> + MOS：[苹果电脑虚拟机装windows7双系统 进入安全模式](https://mos86.com/1087.html)
> + 轻松E站：[Internet Explorer 8 官方离线安装包下载地址](https://www.51-n.com/t-4492-1-1.html)
