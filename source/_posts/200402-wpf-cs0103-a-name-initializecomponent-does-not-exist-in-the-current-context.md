---
title: "WPF：CS0103-当前上下文中不存在名称 “InitializeComponent”"
date: "2020/04/02 16:53:32"
updated: "2020/04/02 16:53:32"
permalink: "wpf-cs0103-a-name-initializecomponent-does-not-exist-in-the-current-context/"
categories:
 - [开发, C#, WPF]
---

最近开发的项目需要借助 `WPF (.NET Core)`，创建基于 `net40`、`net45`、`net472`、`netcoreapp3.1` 等多个框架的 WPF 程序，方便为不同的系统部署。

## 问题

项目的创建都很顺利，当基于 `WPF (.NET Framework)` 的代码向新项目迁移时，发现无法生成，错误列表出现如下错误提示：

```html
严重性	代码	说明	项目	文件	行	禁止显示状态
错误	CS0103	当前上下文中不存在名称“InitializeComponent”	WpfApp1	C:\Users\hd2y\source\repos\WpfApp1\WpfApp1\MainWindow.xaml.cs	25	活动
错误	MC1000	未知的生成错误“Could not find assembly 'mscorlib, Version=2.0.5.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e'. Either explicitly load this assembly using a method such as LoadFromAssemblyPath() or use a MetadataAssemblyResolver that returns a valid assembly.”	WpfApp1	C:\Program Files\dotnet\sdk\3.1.300-preview-015048\Sdks\Microsoft.NET.Sdk.WindowsDesktop\targets\Microsoft.WinFX.targets	225	
```

## 排查

开始排查了项目加载问题、引用程序集、以及部分代码的实现，最终发现在添加 Autofac 的引用时会出现这个问题。

在必应上检索发现了 Autofac 官网的这篇文章：[Why are “old versions” of the framework (e.g., System.Core 2.0.5.0) referenced?](https://autofaccn.readthedocs.io/en/latest/faq/pcl.html)

但是我的开发环境 `.NET Framework` 应该不存在问题，至于使用 `Autofac 4.x` 及以上版本，因为需要支持 `.NET Framework 4.0` 也是不可能的。

## 解决

因为暂时没有想到其他解决方案的话，只能换依赖注入工具了。

虽然用习惯了 Autofac，换成 Unity 感觉各种不方便，但是最新版仍然支持 `.NET Framework 4.0` 不得不说：“真香！”

> 注：Unity 的配置包 `Unity.Configuration` 目前不支持 `.NET Framework 4.5` 以下版本，因为项目对这方面要求不高，所以自定义了。如果依赖配置文件来完成依赖注入，那在 `.NET Framework 4.0` 下，只能另外想办法了。
