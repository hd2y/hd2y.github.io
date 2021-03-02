---
title: "使用 Fiddler 调试 WebService 服务"
date: "2021/03/02 10:17:00"
updated: "2021/03/02 10:17:00"
permalink: "debugging-web-service-services-using-fiddler/"
tags:
 - Fiddler
 - WebService
categories:
 - [开发, 工具]
---

## 前言

近期一个老客户的古董级数据上传工具出现了问题，反映一个通过 `WebService` 进行数据通信的工具无法正常使用。

因为通信接口是旧版的，源码不太好找，并且该通信接口 `WebService` 服务地址配置后无法生效，不能进行本地调试。

为了尽快的锁定问题，考虑用 `HTTP` 转发的方式来排查问题产生原因，以下是问题处理的流程，供以后处理类似问题时参考。

## 锁定问题原因

首先，我们需要出现问题的节点，通过日志文件可以看到错误提示中存在两个问题：

```html
由于目标计算机积极拒绝，无法连接。 127.0.0.1:3349
System.Web.Services.Protocols.SoapException: 服务器无法处理请求。 ---> System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 YunHuLIS.MacWS.MacInfo.SetItemSample(String instrument, String itemCode, String result, String seq, String barcode, DateTime datetime)
   --- 内部异常堆栈跟踪的结尾 ---
System.Web.Services.Protocols.SoapException: 服务器无法处理请求。 ---> System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 YunHuLIS.MacWS.MacInfo.SetItemSample(String instrument, String itemCode, String result, String seq, String barcode, DateTime datetime)
   --- 内部异常堆栈跟踪的结尾 ---
 
由于目标计算机积极拒绝，无法连接。 127.0.0.1:3349
System.Web.Services.Protocols.SoapException: 服务器无法处理请求。 ---> System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 YunHuLIS.MacWS.MacInfo.SetItemSample(String instrument, String itemCode, String result, String seq, String barcode, DateTime datetime)
   --- 内部异常堆栈跟踪的结尾 ---
 
由于目标计算机积极拒绝，无法连接。 127.0.0.1:3349
System.Web.Services.Protocols.SoapException: 服务器无法处理请求。 ---> System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 YunHuLIS.MacWS.MacInfo.SetItemSample(String instrument, String itemCode, String result, String seq, String barcode, DateTime datetime)
   --- 内部异常堆栈跟踪的结尾 ---
System.Web.Services.Protocols.SoapException: 服务器无法处理请求。 ---> System.NullReferenceException: 未将对象引用设置到对象的实例。
   在 YunHuLIS.MacWS.MacInfo.SetItemSample(String instrument, String itemCode, String result, String seq, String barcode, DateTime datetime)
   --- 内部异常堆栈跟踪的结尾 ---
```

首先是使用 `Socket` 与仪器无法连接，其次 `WebService` 也有异常抛出。

经检查确认仪器的传输是否正常，网口是否能正常监听，确认抛出异常的部分日志，可能是当时仪器软件没有启动导致。

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-01.png)

使用 `Socket Tool` 工具监听，可以看到接口可以正常连接：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-02.png)

至于 `WebService` 的问题，只能确认在调用 `SetItemSample` 方法时出现了问题。

## 调试

首先使用接口能够正常登录，可以确认现场访问 `WebService` 是正常的，不存在网络问题。

之前出现过入参出现 `0x00` 的字符，服务方法访问异常，经检查通信接口调用入参是正确的，方法不存在问题，那么可以断定就是 `SetItemSample` 方法内的业务有问题。

前文提到，目前接口内的服务是编译在源码中的，无法通过配置修改，正常的思路是找到源码修改成本地的服务地址或者反编译修改内部的服务地址，方便本地调试。

当然这里不推荐这样做，如果想以最快的速度锁定服务的问题，我们还可以将通信接口的请求转发到本地。

这时候就要祭出 `Fiddler` 了：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-03.png)

从下图可以看到，很多向 `WebService` 的请求 `500`（服务器内部错误），错误内容是内部抛出了：`System.NullReferenceException`

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-04.png)

这时，我们可以选择 `AutoResponder` 选项卡，设置转发的规则：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-05.png)

这时，我们的调试环境可以将抛出捕获的异常：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-06.png)

这时可以看到，`SoapHeader` 携带的用户信息，获取失败，所以抛出了异常。

经检查，问题发生在通信接口中，没有对用户账户的输入进行限制，`SoapHeader` 中的用户 账号信息中携带了空格。

而在服务中，登录时对空格进行了处理，但是后续调用其他服务方法时，服务却没有处理账号中的空格，导致了该问题的产生。

同样的，我们还可以通过 `Fiddler` 编辑我们的请求，来确认是否是因为 `SoapHeader` 的问题：

![](https://hd2y.oss-cn-beijing.aliyuncs.com/debugging-web-service-services-using-fiddler-07.png)

经验证，删除了登录账号中的空格，接口就可以正常使用了。

当然后续就是视情况，对出现问题的接口进行调整。祖传代码，维护时就会发现真的是什么妖魔鬼怪的代码都有啊。
