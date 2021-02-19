---
title: "串口通讯调试与开发"
date: "2019/05/21 17:44:00"
updated: "2019/07/10 18:17:53"
permalink: "the-debugging-and-development-of-serial-communication/"
tags:
 - 串口
categories:
 - [开发, C#]
---

工作中，LIS系统经常需要开发接口与仪器或流水线对接，串口通讯应该是最常见的几种通讯方式之一了。

至于串口是什么、串口通讯又是什么，这些基本概念就不多做介绍了。

这里就针对于我在串口开发中碰到过的一些问题，做一个简单的分享。

## 串口模拟

无论是开发还是测试，肯定都离不开串口的模拟测试，我们不大可能直接在电脑上安装两个串口，所以模拟串口就是最优解了，从网上可以很轻松的搜索到模拟串口的软件。

这里我建议使用`Virtual Serial Port Driver`，大家可以搜索`VSPD`下载：

![serialport0](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport0_1562753721959.png)

安装什么的就不过多解释了，安装完成后我们可以打开并添加模拟串口或者删除模拟的串口，界面如下：

![serialport1](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport1_1562753721959.png)

如果对应的串口被使用，也可以从这里看出其串口的状态：

![serialport2](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport2_1562753721959.png)

当然我们可以在我们的电脑中查看现有的串口，设备管理器中可以查看：

![serialport3](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport3_1562753721967.png)

## 串口调试工具

串口调试工具就是用于串口数据接收/发送的工具软件，比较简单易用的是“串口调试助手V2.2”。

![serialport4](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport4_1562753721978.png)

但是使用中有几点需要注意：
1. 偶尔会发生崩溃，这是串口调试工具的问题；
2. 只能使用COM1-COM4这几个串口，有局限性；
3. 与某些需要检测串口监听状态的仪器连接，虽然已经打开串口但是可能仪器仍然提示连接失败；

如果碰到以上问题，那就只能只能自己找解决方案，另外如果具备一定开发经验后，建议根据自身需求来定制自己的串口调试工具，可以避免以上问题。

## 串口监视精灵

除了串口调试助手以外，另外比较推荐的工具就是串口监视精灵了，顾名思义其就是类似于网口通讯中的抓包工具。

当初从网上找到这个工具的原因是之前LIS系统替换第三方LIS系统，有一个仪器的双工接口开发一直存在问题，所以想看一下第三方LIS怎么实现的双工通讯消息的交互。

当然如果开发的串口通讯工具中没有输出日志，也可以使用该工具进行监听。

一般我们网上能够找到的分为无DLL版和安装版，这里建议使用安装版。

![serialport5](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport5_1562753721992.png)

上图为无Dll版，需要选择监听串口所在的进程，中文会乱码，能监听到串口的关闭但是打开的事件则没有输出。

![serialport6](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport6_1562753722038.png)

上图是安装版，使用安装版需要注意如果系统是x64，第一次使用需要点击软件界面上的“启用64位系统驱动签名”，另外重启。如果监听不到数据，尝试先关闭通讯软件，打开监听后再打开通讯软件。

安装版能够监听到的数据就更完整一些，且每次数据的收发都统一显示为一条记录，这就比无Dll版本的更加友好，但是同样存在的问题是因为是ASC编码所以中文仍然是乱码，需要自己使用HEX编码进行转换。

另外安装版也附带有一个串口调试工具可以使用：

![serialport7](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport7_1562753722078.png)

## 串口通讯开发

### 获取当前电脑的串口

引入`System.IO.Ports`命名空间，然后可以使用`SerialPort.GetPortNames()`获取当前电脑的所有串行端口名称。

```csharp
using System;
using System.IO.Ports;

namespace SerialPortTestConsole
{
    class Program
    {
        static void Main(string[] args)
        {
            string[] ports = SerialPort.GetPortNames();
            foreach (string port in ports)
            {
                Console.WriteLine(port);
            }
            Console.ReadKey();
        }
    }
}
```

### 串口数据收发

引入`System.IO.Ports`命名空间，然后实例化`SerialPort`对象，使用`Open`方法打开串口，`DataReceived`事件进行数据接收，`Write`方法可以进行数据发送。

```csharp
using System;
using System.IO.Ports;
using System.Text;

namespace SerialPortTestConsole
{
    class Program
    {
        static void Main(string[] args)
        {
            //注意：以下为串口初始化常用的属性，赋值为默认值，如果没有赋值属性默认也是这些值
            SerialPort port = new SerialPort()
            {
                PortName = "COM1",//端口号
                BaudRate = 9600,//波特率
                DataBits = 8,//数据位
                Parity = Parity.None,//校验位
                StopBits = StopBits.One,//停止位
                Encoding = Encoding.ASCII,//文本转换编码
                WriteBufferSize = 2048,//输出缓存
                ReadBufferSize = 4096,//输入缓存
                WriteTimeout = -1,//写超时
                ReadTimeout = -1,//读超时
                RtsEnable = false,//RTS信号：建议启用
                DtrEnable = false,//DTR信号：建议启用
            };

            //数据接收事件 输出到控制台
            port.DataReceived += new SerialDataReceivedEventHandler((sender, e) =>
            {
                while (port.BytesToRead > 0)
                {
                    int length = port.BytesToRead;
                    byte[] buffer = new byte[length];
                    port.Read(buffer, 0, length);
                    Console.WriteLine(port.Encoding.GetString(buffer));
                }
            });

            //打开串口
            if (!port.IsOpen)
            {
                try
                {
                    port.Open();
                    Console.WriteLine($"串口打开成功：{port.PortName}");
                }
                catch (Exception exception)
                {
                    Console.WriteLine($"打开串口失败：{exception.Message}");
                }
            }


            if (port.IsOpen)
            {
                //输入串口发送内容
                string input = "";
                do
                {
                    Console.WriteLine("请输入要发送的内容：");
                    input = Console.ReadLine();
                    if (!string.IsNullOrEmpty(input))
                    {
                        byte[] buffer = port.Encoding.GetBytes(input);
                        port.Write(buffer, 0, buffer.Length);
                    }
                } while (!string.IsNullOrEmpty(input));

                //关闭串口释放资源
                port.Close();
            }
            port.Dispose();
        }
    }
}
```

### 一个简单的调试工具

通过以上例子基本上对串口数据手法有个简单的了解，这里我们设计一个工具，来实现串口调试工具的基本功能。

`SerialPort`的属性/方法/事件基本没有什么好说的了，这里主要需要了解HEX数据的转换：

```csharp
/// <summary>
/// 字符串拓展方法
/// </summary>
public static partial class StringExtension
{
    /// <summary>
    /// 将指定字符串转换为十六进制Hex字符串形式。
    /// </summary>
    /// <param name="input">要转换的原始字符串。</param>
    /// <param name="e">编码</param>
    /// <returns>转换后的内容</returns>
    public static string ToHex(this string input, Encoding e = null)
    {
        e = e ?? Encoding.UTF8;
        byte[] byteArray = e.GetBytes(input);
        return BitConverter.ToString(byteArray).Replace('-', ' ');
    }

    /// <summary>
    /// 将十六进制Hex字符串转换为原始字符串。
    /// </summary>
    /// <param name="input">十六进制字符串内容</param>
    /// <param name="e">编码</param>
    /// <returns>原始字符串内容</returns>
    public static string ReHex(this string input, Encoding e = null)
    {
        e = e ?? Encoding.UTF8;
        input = Regex.Replace(input, "[^0-9A-F]", "");
        if (input.Length <= 0) return "";
        byte[] vBytes = new byte[input.Length / 2];
        for (int i = 0; i < vBytes.Length; i++)
            vBytes[i] = byte.Parse(input.Substring(i * 2, 2), NumberStyles.HexNumber);
        return e.GetString(vBytes);
    }

    /// <summary>
    /// 将十六进制Hex字符串转换为二进制数据。
    /// </summary>
    /// <param name="input">十六进制字符串内容</param>
    /// <returns>原始字符串内容</returns>
    public static byte[] ReHexToBuffer(this string input)
    {
        input = Regex.Replace(input, "[^0-9A-F]", "");
        if (string.IsNullOrEmpty(input)) return new byte[0];
        byte[] vBytes = new byte[input.Length / 2];
        for (int i = 0; i < vBytes.Length; i++)
            vBytes[i] = byte.Parse(input.Substring(i * 2, 2), NumberStyles.HexNumber);
        return vBytes;
    }
}

/// <summary>
/// 二进制数据拓展方法
/// </summary>
public static partial class BufferExtension
{
    /// <summary>
    /// 将指定二进制数据转换为十六进制Hex字符串形式。
    /// </summary>
    /// <param name="input">要转换的二进制数据。</param>
    /// <returns>转换后的内容</returns>
    public static string ToHex(this byte[] input)
    {
        return BitConverter.ToString(input).Replace('-', ' ');
    }
}
```

然后就是简单实现后的效果：

![serialport8](https://hd2y.oss-cn-beijing.aliyuncs.com/serialport8_1562753722104.png)

源代码下载：[https://github.com/hd2y/SerialPortTest](https://github.com/hd2y/SerialPortTest)
