---
title: "使用 Sublime Text 替代 Windows 平台的 Notepad++"
date: "2019/10/24 09:15:43"
updated: "2020/02/11 13:16:54"
permalink: "replacing-the-npp-of-the-windows-platform-with-sublime-text/"
tags:
 - IDE
categories:
 - [操作系统, MacOS]
---

在 Windows 平台进行 LIS 接口开发时，经常需要使用 npp(Notepad++) 来进行通讯文本的查阅与解析。

好处主要一下几点：
+ 可以查看 ASCII 字符；
+ 可以使用正则对数据进行调整；
+ 可以查看到当前编辑位置以及选中文本长度；

因为 npp 是 Windows 系统的软件，无法跨平台使用，所以现在使用 macOS 就需要找到一个替代品。

开始是尝试过 ue 等一些网友推荐的软件，发现不行，绝望的以为找不到替代品。但是在 CoolQ 开发者聊天群聊天的时候，无意中听到 Coxxs 说 Sublime 其实也不错，于是下载下来试了一下，可以说是真香了。

以下整理以下安装时遇到的一些问题。

## 下载

[https://pan.baidu.com/s/1_4twEeQGKXXddnsIrF1SrA](https://pan.baidu.com/s/1_4twEeQGKXXddnsIrF1SrA) 提取码：k744

## 关闭更新

菜单栏 -> Preferences -> Settings -> User Settings 配置添加以下内容：

```json
"update_check": false
```

注意保持json内容的结果完整，可能需要补逗号。

## 使用GBK

中文乱码的问题需要安装包，包括 `ConvertToUTF8` 以及 `Codecs33`。

安装的时候如果失败，需要配置一下下载连接方式，打开 菜单栏 -> Preferences -> Package Settings -> Package Control -> User Settings 将以下内容追加到配置文件：

```json
    "downloader_precedence":
    {
        "linux":
        [
            "curl",
            "urllib",
            "wget"
        ],
        "osx":
        [
            "curl",
            "wget"
        ],
        "windows":
        [
            "wininet"
        ]
    }
```

注意保持 json 内容的结果完整，可能需要补逗号。

以上配置完成以后，都需要重启 Sublime Text 才能生效，现在还有宏以及同时编辑多行的问题还需要研究一下，总的来说替代 npp 应该是没问题了，最关键是跨平台很方便。
