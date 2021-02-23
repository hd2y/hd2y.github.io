---
title: "ASP.NET Core - 000 开发环境配置"
date: "2020/11/22 18:26:03"
updated: "2020/11/22 18:27:41"
permalink: "aspnet-core-000-development-environment-configuration/"
tags:
 - VSCode
 - VS
categories:
 - [开发, C#, "ASP.NET Core"]
---

## IDE 推荐

主流的 C# 开发工具如下：
+ Rider：收费、跨平台，习惯使用 JetBrains 家的产品首选。
+ Visual Studio：社区版免费、不支持跨平台，Windows 下首选。
+ Visual Studio Code：免费、跨平台、流行，配置、调试以及使用不及以上两个傻瓜化，有学习成本。

## 我的开发环境

这里我的开发环境是：`Windows` + `Visual Studio Code` + `WSL 2`。

当然，我平时不这么用，因为 `Visual Studio 2019` 还是香的，但是接下来内容主要抱着学习的目的，这样的环境可以帮助我了解 `.NET Core` 的一些细节。

### `Terminal` 与 `WSL 2`

该环境需要 `Windows 10` 版本不低于 `2004`，升级可以前往官网下载 [升级工具](https://www.microsoft.com/zh-cn/software-download/windows10)。

安装 WSL 2 需要启用 `适用于 Linux 的 Windows 子系统` 选项，也可以使用 PowerShell 运行命令：

``` bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

接下来前往应用商店安装 `Ubuntu 20.04 LTS` 以及 `Windows Terminal`。

详细的 `Windows Terminal` 以及 `WSL 2` 安装美化参考：[Win10 Terminal + WSL 2 安装配置指南，精致开发体验 - 精致码农 - 博客园 (cnblogs.com)](https://www.cnblogs.com/willick/p/13924325.html)

### `WSL 2` 安装 `.NET Core`

首先修改镜像源，修改 `sources.list` 文件，我的路径是：`C:\Users\hd2y\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu20.04onWindows_79rhkp1fndgsc\LocalState\rootfs\etc\apt`

修改为阿里云的源：

```bash
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

修改以后执行命令：

```bash
sudo apt-get update
sudo apt-get upgrade
```

然后安装 `.NET Core SDK`，分别需要执行以下命令：

```bash
# 将 Microsoft 包签名密钥添加到受信任密钥列表，并添加包存储库
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# 安装 .NET SDK
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-5.0

# 安装 .NET Core 3.1 SDK
sudo apt-get install -y dotnet-sdk-3.1
```

安装完成后可以运行 `dotnet --list-sdks` 命令查看已安装的 SDK ，可以运行 `dotnet --help` 命令查看帮助信息。

以上步骤安装完成以后，`dotnet` 命令使用的是 `.NET 5.0` 的 SDK，可以通过以下命令切换到 `.NET Core 3.1`：

```bash
dotnet new globaljson --sdk-version 3.1.404 --force
```

以上命令会在文件夹创建一个 `global.json` 文件：

```json
{
  "sdk": {
    "version": "3.1.404"
  }
}
```

详细的 `WSL 2` 中配置 `.NET Core` 开发环境可以参考系列文章：[WSL 2 准备 .NET Core 开发环境 - luzemin - 博客园 (cnblogs.com)](https://www.cnblogs.com/talentzemin/p/12575606.html)

### `VS Code` 配置

需要安装的几个插件：
+ C#
+ Visual Studio IntelliCode
+ .NET Core Test Explorer
+ vscode-solution-explorer
+ Remote - WSL

![vscode-keyboard-shortcuts-for-windows](https://hd2y.oss-cn-beijing.aliyuncs.com/vscode-keyboard-shortcuts-for-windows-acb5985293e84e8e8394611748d2d75f.png)

具体使用 `Visual Studio Code` 开发 `.NET Core` 可以参考：[使用 Visual Studio Code 开发 .NET Core 看这篇就够了 - 依乐祝 - 博客园 (cnblogs.com)](https://www.cnblogs.com/yilezhu/p/9926078.html)

## 测试

创建一个控制台项目，输出经典的 `Hello world!`。

```bash
mkdir code
cd code
mkdir 001_HelloWorld
cd 001_HelloWorld
dotnet new console --name HelloWorld
```

在 VS Code 中使用 `Ctrl + Shift + P` 搜索 `Remote-WSL: Open Folder in WSL`，打开我们新建项目所在的文件夹。

修改 Main 函数内容为：
```csharp
Console.WriteLine(Environment.OSVersion);
Console.WriteLine("Hello World!");
```

此时在终端输入 `dotnet run`，将打印：
```bash
Unix 4.4.0.19041
Hello World!
```

![demo](https://hd2y.oss-cn-beijing.aliyuncs.com/demo-41682b793120418fae6532bee902ae87.png)


