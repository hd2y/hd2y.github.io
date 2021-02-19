---
title: "创建支持多个dotnet版本的类库项目"
date: "2019/04/03 20:55:00"
updated: "2019/07/10 14:38:26"
permalink: "create-a-class-library-project-that-supports-multiple-dotnet-versions"
tags:
 - 项目
 - .NET Standard
categories:
 - [开发, C#]
---

## 如何创建支持支持多个 `dotnet` 版本的类库项目

开发中，经常会遇到需要所开发的类库同时支持 `net40` 、 `net451` 、 `netstandart2.0` 等版本。

随意打开一些常用的开源项目比如“Dapper”就会发现，项目并不会针对不同的 `dotnet` 版本，创建不同分支，而是一套代码支持了多个不同的 `dotnet` 版本。

而针对不同版本的语言特性，只需要使用 `#if` 预处理指令进行处理即可，可以节约了我们的开发成本，方便我们统一的管理项目源码。

以下部分的内容是基于 Visual Studio 2017 进行的实践，建议安装最新的VS版本。

### 项目文件 `*.csproj`

#### 创建项目

**注意：**创建的项目类型必须是 `.NET Standard` 。

![](/upload/2019/3/2019021914395620190403205200764.png)

#### 编辑 `*.csproj` 文件

在项目上右键，右键菜单中便有 `编辑 *.csproj` 的选项：

![](/upload/2019/3/2019021914453220190403205158717.png)

如果是新建的 `类库(.NET Standard)` 项目，`*.csproj` 文件内容大概是：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

</Project>
```

正常一个扩展后的内容是：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <PackageId>Test</PackageId>
    <PackageTags>test1;test2</PackageTags>
    <Authors>John Sun</Authors>
    <TargetFrameworks>net40;net451;netstandard2.0</TargetFrameworks>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Jint" Version="2.11.58" />
    <PackageReference Include="Newtonsoft.Json" Version="12.0.1" />
    <PackageReference Include="NLog" Version="4.5.11" />
    <PackageReference Include="NLog.Config" Version="4.5.11" />
    <PackageReference Include="NPOI" Version="2.4.1" />
  </ItemGroup>
  <ItemGroup Condition="'$(TargetFramework)' == 'net40' OR '$(TargetFramework)' == 'net451'">
    <Reference Include="System" />
    <Reference Include="System.Data" />
    <Reference Include="System.Web" />
    <Reference Include="System.Xml" />
    <Reference Include="System.Xml.Linq" />
    <Reference Include="Microsoft.CSharp" />
    <PackageReference Include="dapper_net40" Version="1.0.0" />
  </ItemGroup>
  <ItemGroup Condition="'$(TargetFramework)' == 'netstandard2.0'">
    <PackageReference Include="Microsoft.CSharp" Version="4.5.0" />
    <PackageReference Include="dapper_net40" Version="1.0.0" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\src\Test\TestCore.csproj" />
  </ItemGroup>
</Project>
```

#### 需要关注的配置节点

+ `PropertyGroup`: 配置项目的基本信息
  + `TargetFrameworks`: 配置项目对框架元包的引用，注意默认一个引用时是 `TargetFramework`，需要修改为复数
  + `PackageId`: 指定生成包的名称
  + `PackageTags`: 标记
  + `PackageVersion`: 指定生成的包所具有的版本
  + `Authors`: 以分号分隔的包作者列表
  + `OutputType`: 可以使用 `<OutputType>Exe</OutputType>` 指定项目为控制台项目
  + ……
+ `ItemGroup`: 向项目添加依赖项
  + `PackageReference`: nuget程序包
  + `Reference`: 程序集引用
  + `ProjectReference`: 项目引用

### `C#` 预处理器指令

我们在开发中可能会遇到部分代码在不同的 `dotnet` 版本上有不同的实现，或者部分程序类型/方法等只能在指定的 `dotnet` 版本上使用，这个时候我们就可以使用 `C#` 的预处理指令 `#if` 。

#### 针对不同的目标框架使用最新的 API:

```csharp
public class MyClass
{
    static void Main()
    {
#if NET40
        WebClient _client = new WebClient();
#else
        HttpClient _client = new HttpClient();
#endif
    }
    //...
}
```

#### 在目标框架下屏蔽一个方法

```csharp
public class MyClass
{
#if !NET20
    public async void OpenAsync()
    {
        //...
    }
#endif
    //...
}
```

## 参考

+ [.NET Core 的 csproj 格式的新增内容](https://docs.microsoft.com/zh-cn/dotnet/core/tools/csproj)
+ [使用 .NET Core SDK 1.0 管理依赖项](https://docs.microsoft.com/zh-cn/dotnet/core/tools/dependencies)
+ [.NET Standard](https://docs.microsoft.com/zh-cn/dotnet/standard/net-standard)
+ [C# 预处理器指令](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/preprocessor-directives/)
