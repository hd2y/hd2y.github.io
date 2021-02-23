---
title: "ASP.NET Core - 001 源码编译与调试"
date: "2020/11/29 20:34:56"
updated: "2020/11/29 20:34:56"
permalink: "aspnet-core-001-source-code-compilation-and-debugging/"
tags:
 - GitHub
 - 调试
categories:
 - [开发, C#, "ASP.NET Core"]
---

## 一、源码编译

需要编译的源码的代码仓库有两个，一个是 [aspnetcore](https://github.com/dotnet/aspnetcore)，另外一个是 [extensions](https://github.com/dotnet/extensions)。

如果有代理，编译的过程比较简单，参考微软的文档进行编译即可。
+ `ASP.NET Core`：[Build ASP.NET Core from Source —— GitHub](https://github.com/dotnet/aspnetcore/blob/master/docs/BuildFromSource.md)
+ `.NET Extensions`：[Build .NET Extensions from Source —— GitHub](https://github.com/dotnet/extensions/blob/master/docs/BuildFromSource.md)

如果没有代理，那么整个编译流程可能非常慢，甚至无法正常进行，可以B站这个视频执行编译：[编译并调试 Asp.Net Core 源码 —— B站视频](https://www.bilibili.com/video/BV1964y1F7hQ)

如果使用代理软件，建议使用 Global（全局）代理，同时命令行与 Powershell 需要设置代理：
```bash
# CMD
set http_proxy=http://127.0.0.1:7890 & set https_proxy=http://127.0.0.1:7890

# Powershell
$Env:http_proxy="http://127.0.0.1:7890";$Env:https_proxy="http://127.0.0.1:7890"
```

编译时可能出现的错误：`C4819` 或其他的错误。实际代码是没有问题的，是编码的锅，解决方案：系统编码设置使用 `UTF-8`：
> 系统设置 → 时间和语言 → 语言 → 相关设置 → 管理语言设置 → 管理 → 更改系统区域设置 → 勾选：使用 Unicode UTF-8 提供全球语言支持

设置以后重启系统即可，网上反馈设置后部分软件可能出现乱码，目前我没发现类似问题。但是，我第一次设置后，系统所有图标丢失，部分软件无法正常使用，但是取消选项又重新勾选了一次就正常了，如果像我一样出现问题，可以多设置几次试试。

> 补：发现了一个小问题，登录穿越火线，频道名、房间名、游戏玩家昵称的中文不显示了。/(ㄒoㄒ)/~~

## 二、调试

后续需要学习 `ASP.NET Core`，并深入了解其实现原理，所以肯定希望能够直接调试源码。

这里提供两个调试方案，第一个方案是借助调试工具 dnSpy 来进行调试，第二则是直接使用 Visual Studio 进行调试。

### 1. dnSpy 反编译调试

首先需要下载 dnSpy：[dnSpy/dnSpy: .NET debugger and assembly editor (github.com)](https://github.com/dnSpy/dnSpy)，解压后可以直接打开。

然后我们可以随意创建一个 `ASP.NET Core` 的示例项目，打开 Visual Studio Code，创建一个 MVC 项目并编译。

```bash
dotnet new mvc -n 002_Debug
dotnet build
```

找到生成的 `002_Debug.dll` 拖到 dnSpy 中，找到程序入口设置断点并运行，即可进入调试：

![debug-with-dnspy](https://hd2y.oss-cn-beijing.aliyuncs.com/debug-with-dnspy-8dddc8ba484d41b487ddd11bd554bcd3.png)

### 2. 源服务器支持

首先如果是使用 Visual Studio Code，设置 `launch.json` 文件，增加如下内容：
```json
//关闭 “仅我的代码” 项
"justMyCode": false,
"symbolOptions": {
  //查找并下载symbol文件
  "searchMicrosoftSymbolServer": true
}
```

![debug-with-vsc](https://hd2y.oss-cn-beijing.aliyuncs.com/debug-with-vsc-7f07d5a6789c4356a83847d62bc689a0.png)

如果时使用 Visual Studio：
> 选项 → 调试 → 常规 → 取消 “仅我的代码” 的勾选 → 勾选 “启用源链接支持”  
> 选项 → 调试 → 符号 → 符号文件(.pdb)的位置 → 勾选 “Microsoft 符号服务器”

配置完成后，调试时将会下载符号文件，可以正常调试 `ASP.NET Core` 的源代码。

![debug-with-vs](https://hd2y.oss-cn-beijing.aliyuncs.com/debug-with-vs-eb893b497ff2435b97be84efcca271ce.png)

### 3. 使用源码调试

首先是 `ASP.NET Core`，前文我们已经进行了编译，我们可以在 `\artifacts\installers\Debug` 找到 `aspnetcore-runtime-3.1.9-dev-win-x64.msi` 进行安装。

然后是 `.NET Extensions`，我们可以在项目中创建一个 `NuGet.config`，然后配置本地程序包源：
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="local1" value="C:\Users\hd2y\code\extensions\artifacts\packages\Debug\Shipping" />
    <add key="local2" value="C:\Users\hd2y\code\aspnetcore\artifacts\packages\Debug\Shipping" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  </packageSources>
</configuration>
```

这时，运行程序进入调试，就可以通过单步调试进入 `ASP.NET Core` 的源码。

如果以上操作无法正常调试，可以 F12 查看对应的方法的程序集版本。

例如我的程序集版本是 `3.1.10`，而我编译的 `ASP.NET Core` 的版本是 `3.1.9`，调试就无法正常进入。

这里有两种解决方案，第一种是查看 `3.1.9` 的版本对应的 `.NET SDK` 的版本进行安装，然后指定引用程序集的版本：
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <RootNamespace>_002_Debug</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="3.1.9" />
  </ItemGroup>
</Project>
```

> 不指定版本的话，在我测试的时候不知道为什么转到定义程序集版本是 `3.1.8`，而不是文档中提到的 `3.1.9`。  
> 直接指定会使用 `NuGet.config` 配置的本地程序包源，如果设置后发现仍旧不行，可能 NuGet 有缓存，需要在 选项 → NuGet 包管理器 → 常规 → 清除所有 NuGet 缓存。

第二种方案是根据当前 `.NET SDK` 对应的 `ASP.NET Core` 版本，重新进行编译，具体版本对应关系可以查看 `.NET Core SDK` 的下载页：[Download .NET Core 3.1 (Linux, macOS, and Windows) (microsoft.com)](https://dotnet.microsoft.com/download/dotnet-core/3.1)
