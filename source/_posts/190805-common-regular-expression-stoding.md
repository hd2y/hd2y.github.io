---
title: "常用正则表达式整理"
date: "2019/08/05 15:05:39"
updated: "2020/02/11 13:49:43"
permalink: "common-regular-expression-stoding/"
tags:
 - 正则
categories:
 - [开发]
---

## ASTM 协议数据处理

正则内容：`(\x04)|(\x05)|(\x02[0-7]{1})|((\x03|\x17)[0-9A-F]{2}\r\n)`

目的：处理 ASTM 协议数据内容，消除校验符等。

![astm](./190805-common-regular-expression-stoding-01.gif)
