---
title: "WPF 系列教程：从 WPF (.NET Core) 开始"
date: "2020/04/04 19:32:43"
updated: "2020/04/04 19:34:23"
permalink: "wpf-series-of-tutorials-from-wpf-dotnet-core-commencement/"
categories:
 - [开发, C#, WPF]
---

如果是一名 `.NETer`，应该有用 Windows Form 与 WPF 创建过一些简单项目，如果没用过至少也应该听说过。

开始我也用 Windows Form 与 WPF 写过很多小工具。

当然后期用 Windows Form 布局感觉太过繁琐耗费精力，现在已经完全投入了 WPF 的怀抱。

因为在这过程中碰到过很多问题，为此查看过很多资料，这里就简单整理一下，以后再碰到这些问题，方便 Copy。

## 初始化代码仓库

因为我是用 git 托管代码，所以创建项目的第一步，就是先创建一个 git 仓库。

可以选择 github、gitlab，如果担心以后有被美国“制裁”的风险，也可以选择国内的 gitee。当然也可以自己使用 gitlab、gitea 或者 gogs 搭建自己的 git 服务。

虽然选择很多，但是创建一个代码仓的过程都没什么区别，下图是如何在 github 上创建一个仓库。

![new repository](https://hd2y.oss-cn-beijing.aliyuncs.com/new%20repository-7f874e646b2e4875a24b8a75c6559812.png)

创建成功后，待仓库初始化完成会自动跳转到如下界面：

![repository homepage](https://hd2y.oss-cn-beijing.aliyuncs.com/repository%20homepage-b7820b2b2fa84db0a2c9e30e82098321.png)

这时我们可以选择用 `Open in Visual Studio`，使用 Visual Studio 直接克隆下载并打开这个仓库。

![git clone](https://hd2y.oss-cn-beijing.aliyuncs.com/git%20clone-6ddd26b0ea2b40eba314fa4eacd30d08.png)

建议也可以安装 git，使用 clone 命令进行下载。下载完成后，可以打开 Visual Studio，选择打开本地文件夹来查看初始化的文件。

```bash
git --version
cd C:\Users\hd2y\source\repos
git clone https://github.com/hd2y/WpfExample.git
```

## 创建项目

为了方便管理解决方案内的项目，一般我都会新建一个空白的解决方案。有时，使用搜索模板功能是找不到“空白解决方案”的，我们可以在 “所有语言” `->` “所有平台” `->` “其他” 中找到。

![create solution](https://hd2y.oss-cn-beijing.aliyuncs.com/create%20solution-11b71c122be1473a80d3552d9b14c663.png)

然后填写需要创建的解决方案名称，并选择所在文件夹，完成创建。

![set new solution](https://hd2y.oss-cn-beijing.aliyuncs.com/set%20new%20solution-c91bfc4a88744d87a55204e98c9f3da4.png)

创建完成后，在资源管理器中内容如下图。

![view in explorer](https://hd2y.oss-cn-beijing.aliyuncs.com/view%20in%20explorer-f09dd05aeb764bc6931ea5105f4917b6.png)

但是这时我们的解决方案打开后空无一物，这时我们需要创建两个文件夹，一个是 `Solution Items`，用于管理 `.gitignore`、`LICENSE` 与仓库的自述文件，另外一个是 `src`，用于存放代码文件。

如果需要添加单元测试，还需要增加一个文件夹 `test` 存放测试代码，而如果有一些静态文件，建议创建 `docs` 与 `row` 文件夹进行存储。

注意，除 `Solution Items` 文件夹外，其他文件夹还要在文件资源管理器创建文件夹，与解决方案中的文件夹一一对应，因为解决方案中的文件夹实际是虚拟文件夹。

创建完成后，我们需要从文件资源管理器中将仓库初始化生成的文件，拷贝到解决方案的 `Solution Items` 文件夹，请注意这个过程中，只是解决方案中可以查看到这几个文件，这些文件的路径并不会发生变化。

完成以上以后，我们就可以真正的创建项目了，在 `src` 文件夹上点击右键并添加“新建项目”，选择 “C#” `->` “Windows” `->` “桌面” 中的 `WPF App (.NET Core)`：

![create wpf project](https://hd2y.oss-cn-beijing.aliyuncs.com/create%20wpf%20project-fca9da6f50db46aabf289baead0df449.png)

注意需要选择到 `src` 目录下，以上都完成后，我们文件资源管理器以及解决方案中的结构如下：

![view the solution and explorer](https://hd2y.oss-cn-beijing.aliyuncs.com/view%20the%20solution%20and%20explorer-c71c938bb5134375a32003663038ed8c.png)

## 调整项目依赖框架

`.NET Standard` 类库项目以及 `.NET Core` 应用项目，在双击项目名后，都可以打开一个 xml 文件。

我们可以调整 xml 文件的配置，默认的 xml 文件有一个 `TargetFramework` 节点，如果是依赖框架是多个，我们要改成复数形式，再修改里面依赖框架的值，多个用 `;` 分隔。

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFrameworks>net40;net472;netcoreapp3.1</TargetFrameworks>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

</Project>
```

修改完成，可能会提示重新加载项目，确认后我们的依赖项会变成下图：

![project dependencies](https://hd2y.oss-cn-beijing.aliyuncs.com/project%20dependencies-604809d5c4d049579588781666a68a64.png)

这时我们再生成项目，`\bin\Debug` 文件夹就会生成多个文件夹，与我们选择的依赖框架一一对应。

## 提交修改

代码写好了，自然要提交到代码仓库，此时团队资源管理器中会显示当前的 Git 仓库。

选择“项目”下的“更改”，输入一些本次提交的描述信息，点击“全部提交”。

注意，此时还没有正式的将本次提交推送到远程仓库，还需要点击“同步”下的“推送”才行。

如果使用命令行提交，可以参考以下：

```bash
git add .
git commit -m '创建项目'
git push -u origin master
```

当然以上都是默认已经配置登录，如果没有登录的话，则需要登录 github 账号。
