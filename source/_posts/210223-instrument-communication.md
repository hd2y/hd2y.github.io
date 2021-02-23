---
title: "检验仪器常见通信方案"
date: "2021/02/23 09:44:37"
updated: "2021/02/23 09:44:37"
permalink: "instrument-communication/"
tags:
 - LIS
 - 串口
categories:
 - [开发, 业务]
---

## 一、串口

### 串口通信

串口通信（Serial Communication）的概念非常简单，串口按位（bit）发送和接收字节的通信方式。

串口通信最重要的参数是波特率、数据位、停止位和奇偶校验，对于两个进行通信的端口，这些参数必须匹配。

RS-232 标准是电气通信中最常见的串行连接标准，所以工作中串口通信一般都是RS-232协议通信，而在系统中或一些调试工具中通信端口也会称之为COM口（Cluster communication port）即串行通信端口。

RS-232 是美国电子工业联盟（EIA）制定的串行数据通信的接口标准，原始编号全称是 EIA-RS-232（简称 232，RS232）。它被广泛用于计算机串行接口外设连接。

### RS-232 设置：

串口号：所使用的串行接口。常见如 COM1、COM2。

波特率：是指从一设备发到另一设备的波特率，即每秒钟多少符號。典型的波特率是 300, 1200, 2400, 9600, 19200, 115200 等。一般通信两端设备都要设为相同的波特率，但有些设备也可设置为自动检测波特率。

校验位：是用来验证数据的正确性。常见如 NONE 、ODD 、EVEN。

数据位：数据位紧跟在起始位之后，是通信中的真正有效信息。数据位的位数由通信双方共同约定，一般可以是 6 位、7 位或 8 位，比如标准的 ASCII 码是 0~127（7 位），扩展的 ASCII 码是 0~255（8 位）。

停止位：停止位在最后，用以标志一个字符传送的结束，对应于逻辑 1（高电平）状态。停止位可以是 1 位、1.5 位或 2 位。

### 硬件连接

实际连接中，我们可以通过设备后方的串口端口完成连接。

但是需要注意的是，建议由仪器工程师完成连接工作，LIS 开发人员再进行调试测试。

如果没有仪器工程师指导，自行连接，需要将连接的仪器和电脑关闭断电后再操作，否则可能因为连接时的电流，烧坏硬件设备。

![instrument-communication-01](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-01-6e3bf374a85e4f6ea6c1b771d5a1bf13.png)

如上图为电脑上的串口端口，以及连接需要使用的线材。如果没有串口端口，可以考虑使用 USB 转串口的转接线，但是这类设备可能会因为驱动或兼容问题，不够稳定。

### 软件驱动

在某些情况下，硬件连接将不适用。例如开发人员调试、仪器软件与 LIS 系统部署在同一台电脑，这时如果考虑购置双串口的电脑自然是不合理的。

建议可以考虑使用软件驱动，可以使用 VSPD 工具虚拟串口：

![instrument-communication-02](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-02-e8782f4a7d28458aadabddf952c2a703.png)

### 发送/接收数据

发送或接收数据这里建议使用一款名为【串口调试助手 SComAssistant V2.2】的工具。

![instrument-communication-03](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-03-63889c1d5f4c402dacd3ca761a18c9f3.png)

该工具可以实现一些较为简单的数据收、发操作，基本能够满足需求，如果不满足需要还可以考虑 AccessPort 等工具。

![instrument-communication-04](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-04-d3311182020a440eb2974f088161aa81.png)

推荐 AccessPort 工具的另外一个原因是，其还可以作为一个监控工具使用，即类似“抓包工具”来使用。

以上串口工具已经打包，可以直接下载: https://pan.baidu.com/s/1CqcharAtWJLifPI0dSRSQA  密码: 5q2g

## 二、网口

### 网口通信

网口通信即通过以太网连接到仪器，进行数据的传输的通信方式。

因为绝大多数使用 TCP/IP 协议进行通信，所以一般指基于 TCP/IP 协议进行数据传输的通信方式。

而 Socket 通信与 TCP/IP 协议都是基础知识，这里不再展开叙述。

### 硬件连接

网线连接网口这种很基础的知识，这里应该不用再赘述了。

![instrument-communication-05](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-05-af7134a500384d22b56a276fd710b77f.png)

这里需要注意的是一个双网口电脑问题。

理论上仪器软件所在的电脑可以接入路由器，与 LIS 所在电脑在同一个局域网内，双方就可以通信。

但是部分仪器厂商会因为网络安全的问题，要求仪器软件不接入局域网，直接与 LIS 所在电脑直接连接。

这样仪器将会占用电脑的网口，所以只有采购双网口的电脑供 LIS 使用。

当然也可以考虑再采购一个外置网卡，但是考虑驱动与兼容性问题，建议采购时优先考虑双网口的电脑。

### 发送/接收数据

网口的测试工具，这里推荐【TCP/UDP Socket 调试工具（Socket Tool）】：

![instrument-communication-06](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-06-6ebf6691f3674ee8b60168ebebbfe5da.png)

另外还有 sokit 工具，体验也不错：

![instrument-communication-07](https://hd2y.oss-cn-beijing.aliyuncs.com/instrument-communication-07-e8112bb987354fbc80bf6620da40f712.png)

如果需要抓包，这里推荐 wireshark，使用教程建议参考博客园文章：https://www.cnblogs.com/mq0036/p/11187138.html

以上网口工具已经打包，可以直接下载：https://pan.baidu.com/s/1SkbeDkbE0KN46Qp9ShNXuw  密码: cfhm

### 三、其他

一般主流仪器只会提供串口、网口两种通信方式。

但是如果是国产仪器，或者国内代理商对仪器软件进行了汉化，再或者实验室使用了第三方的 LIS 而且想保留，那么就可能需要考虑其他的对接方案。

例如希森美康血球仪国内代理商会安装 Laborman 与仪器连接，用户在没有使用 LIS 可以直接在该软件上出具报告。而如果需要与仪器连接就需要读取仪器输出的 cfd 后缀文件。

有些国内一些镜检仪器，软件上没有针对性的开发通信模块，而是提供数据库访问权限，让 LIS 直接读取他们软件的数据库。

而针对这些其他的连接方式，选择一个最简单、最稳定的选项即可。
