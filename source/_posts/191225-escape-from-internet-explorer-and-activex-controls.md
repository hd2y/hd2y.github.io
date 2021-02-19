---
title: "逃离 IE 浏览器与 ActiveX 控件"
date: "2019/12/25 18:25:06"
updated: "2019/12/25 18:36:02"
permalink: "escape-from-internet-explorer-and-activex-controls/"
tags:
 - ActiveX
 - IE
 - 身份证读卡
 - Chrome
categories:
 - [开发, C#]
---

## 前言

因为公司产品是 `B/S` 架构，所以很多与硬件交互资源的功能，不可避免的要使用 `ActiveX` 控件。

但是因为安全性问题，现代浏览器例如 Chrome、Firefox、Edge 等都已经放弃了对 ActiveX 的支持，所以这个方案已经不再是合适的选择。

当然现阶段还是有很多解决方案，例如不使用新特性与新功能，坚定不移的使用支持 ActiveX 的浏览器版本；使用开源的一些浏览器组件例如 MiniBlink，开发自己的“浏览器”等等。

但是对于开发来说，面对前端涌现的很多新技术，因为浏览器版本太低，无法支持一些新特性而无法使用畏手畏脚，是一种很痛苦的开发体验。

## 解决打印问题

因为最初产品设计的问题，没有使用水晶报表、FastReport 等报表控件，所以打印现在还依托于 HTML 打印。

用过的其实都知道，浏览器本身以及市面上一些 ActiveX 打印控件，其实对打印功能支持都有限，一些复杂的功能无法实现，而且经常要被一些内容溢出、所见非所得等问题折磨。

而产品升级到用 Chrome 就更麻烦了，本身自带的打印功能，产品发布以后基本没有使用价值，和 `C/S` 架构的打印体验更是差了十万八千里。

因为之前发过一篇将打印迁移到使用 LODOP 打印控件，所以这里不再赘述：[Web下打印体验最好的打印控件LODOP](https://www.hd2y.net/archives/lodop-the-best-print-control-for-printing-experience-under-web)

现在已经将所有的打印例如条码、报表、报告单迁移到 LODOP 上。而且在不支持 ActiveX 的浏览器版本，可以使用其提供的 CLodop 控件，其会使用 HTTP 与 WebSocket 进行通信，控制打印，目前上线的项目实际体验比过去使用 ActiveX 也要好很多。

## 解决读卡问题

如果仅仅是磁条卡、二维码等，刷卡和扫码设备会直接将文本输出，可以使用输入框接收。

但如果是芯片卡，例如 IC 卡、身份证、社保卡、银行卡等，读卡器厂商会提供动态链接库给第三方开发使用。

一般如果是 `C/S` 架构，自然不会存在读卡问题，但是 `B/S` 架构，网上提供的资料一般都是让封装成 ocx 使用。

这里建议是参考 CLodop 的解决方案，开发一个小工具，使用 HTTP 监听处理读卡请求：

![20191225160112](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225160112_1577269066727.png)

监听工具这里依然是 C# 的例子，可以托管到 WinForm、WPF、控制台应用程序，但是需要注意 Win7 以上 Windows 版本必须 `以管理员身份运行`。建议是托管于 Windows 服务。

首先我们基于 `HttpListener` 封装一个简单的 `HttpServer`：

```csharp
/// <summary>
/// 使用HttpListener实现的HttpServer
/// </summary>
public class HttpServer
{
    /// <summary>
    /// 监听
    /// </summary>
    public HttpListener Listener { get; }

    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="prefixes">定义url</param>
    /// <param name="auth">指定身份验证 默认匿名</param>
    public HttpServer(IEnumerable<string> prefixes, AuthenticationSchemes auth = AuthenticationSchemes.Anonymous)
    {
        if (!HttpListener.IsSupported)
        {
            Console.WriteLine("Windows XP SP2 or Server 2003 is required to use the HttpListener class.");
        }
        else if (prefixes == null || prefixes.Count() == 0)
        {
            Console.WriteLine("初始化服务监听失败，服务监听链接不能为空！");
        }
        else
        {
            try
            {
                Listener = new HttpListener
                {
                    AuthenticationSchemes = auth
                };
                foreach (string prefix in prefixes)
                {
                    Listener.Prefixes.Add(prefix);
                }
            }
            catch (Exception exc)
            {
                Console.WriteLine($"开启监听服务出现未经处理的异常：{exc.Message}");
            }
        }
    }

    /// <summary>
    /// Http监听程序响应事件委托
    /// </summary>
    /// <param name="ctx"></param>
    public delegate void ResponseEventArges(HttpListenerContext ctx);

    /// <summary>
    /// Http监听程序响应事件
    /// </summary>
    public event ResponseEventArges ResponseEvent;

    /// <summary>
    /// 请求的回调
    /// </summary>
    private AsyncCallback _asyncCallback;

    /// <summary>
    /// 开启监听服务
    /// </summary>
    public void Start()
    {
        if (Listener == null)
        {
            Console.WriteLine("监听服务初始化失败，无法开启监听。");
        }
        else if (!Listener.IsListening)
        {
            try
            {
                Listener.Start();
                _asyncCallback = new AsyncCallback(GetContextAsyncCallback);
                Listener.BeginGetContext(_asyncCallback, null);
                Console.WriteLine($"开启监听：{string.Join("、", Listener.Prefixes)}。");
            }
            catch (Exception exc)
            {
                Console.WriteLine($"开启监听服务出现未经处理的异常：{exc.Message}");
            }
        }
        else
        {
            Console.WriteLine($"监听服务正在运行，无需重复开启。");
        }
    }

    /// <summary>
    /// 关闭服务监听
    /// </summary>
    public void Stop()
    {
        try
        {
            Listener?.Stop();
            Console.WriteLine($"已关闭监听服务。");
        }
        catch (Exception exc)
        {
            Console.WriteLine($"关闭监听服务出现未经处理的异常：{exc.Message}");
        }
    }

    /// <summary>
    /// 异步检查传入的请求
    /// </summary>
    /// <param name="result">结果</param>
    public void GetContextAsyncCallback(IAsyncResult result)
    {
        if (result.IsCompleted)
        {
            HttpListenerContext ctx = Listener.EndGetContext(result);
            if (ResponseEvent != null)
            {
                ResponseEvent.Invoke(ctx);
            }
            else
            {
                dynamic data = new ExpandoObject();
                data.success = false;
                data.msg = $"未注册http请求处理事件。";
                ctx.Response.StatusCode = 200;
                // 允许跨域访问
                ctx.Response.AppendHeader("Access-Control-Allow-Origin", "*");
                ctx.Response.ContentType = "application/json";
                ctx.Response.ContentEncoding = Encoding.UTF8;
                JavaScriptSerializer serializer = new JavaScriptSerializer();
                byte[] binaryData = Encoding.UTF8.GetBytes(serializer.Serialize(data));
                ctx.Response.OutputStream.Write(binaryData, 0, binaryData.Length);
            }
            ctx.Response.Close();
        }
        Listener.BeginGetContext(_asyncCallback, null);
    }

    /// <summary>
    /// 获取请求上下文中传递的参数，注意参数名已经默认转换为小写
    /// </summary>
    /// <param name="ctx">HttpListener请求上下文</param>
    /// <returns>参数数据</returns>
    public static Dictionary<string, string> GetData(HttpListenerContext ctx)
    {
        Dictionary<string, string> data = new Dictionary<string, string>();
        if (ctx == null)
        {
            throw new ArgumentNullException(nameof(ctx));
        }
        if (ctx.Request.HttpMethod == "GET")
        {
            for (int i = 0; i < ctx.Request.QueryString.AllKeys.Length; i++)
            {
                data[ctx.Request.QueryString.AllKeys[i].ToLower()] = ctx.Request.QueryString[ctx.Request.QueryString.AllKeys[i]];
            }
        }
        else
        {
            string rawData;
            using (StreamReader reader = new StreamReader(ctx.Request.InputStream, ctx.Request.ContentEncoding))
            {
                rawData = reader.ReadToEnd();
            }
            if (!string.IsNullOrEmpty(rawData))
            {
                string[] parameters = rawData.Split('&');
                for (int i = 0; i < parameters.Length; i++)
                {
                    string[] kvPair = parameters[i].Split('=');
                    if (kvPair.Length > 1)
                    {
                        string key = HttpUtility.UrlDecode(kvPair[0], ctx.Request.ContentEncoding);
                        string value = HttpUtility.UrlDecode(kvPair[1], ctx.Request.ContentEncoding);
                        data[key.ToLower()] = value;
                    }
                }
            }
        }
        return data;
    }
}
```

然后我们可以借助这个封装的类型，简单的托管一个 HTTP 服务：

```csharp
class Program
{
    static void Main(string[] args)
    {
        string[] prefexes = new string[] { "http://*/read_card/" };
        HttpServer server = new HttpServer(prefexes);
        server.ResponseEvent += (ctx) =>
        {
            JavaScriptSerializer serializer = new JavaScriptSerializer();
            bool success = false;
            string msg = string.Empty;
            string data = null;
            Dictionary<string, string> parameters = HttpServer.GetData(ctx);
            try
            {
                if (parameters.Count == 0)
                {
                    msg = "入参不能为空！";
                }
                else if (!parameters.ContainsKey("token") || parameters["token"] != "sMO9sIU5OiNyIWDtKXSneWuW7TO7Kuev")
                {
                    msg = "token 无效";
                }
                else if (!parameters.ContainsKey("key"))
                {
                    msg = "无法获取必要的[key]信息！";
                }
                else
                {
                    if (parameters["key"] == "USER_NAME")
                    {
                        success = true;
                        data = Environment.UserName;
                    }
                    else if (parameters["key"] == "USER_DOMAIN_NAME")
                    {
                        success = true;
                        data = Environment.UserDomainName;
                    }
                    else
                    {
                        msg = "未知的请求！";
                    }
                }

                Console.WriteLine($"响应数据：{msg}");
            }
            catch (Exception exc)
            {
                msg = $"错误：{exc.Message}";
            }
            finally
            {
                var result = new { success, msg, data };
                ctx.Response.StatusCode = 200;
                ctx.Response.AppendHeader("Access-Control-Allow-Origin", "*");
                ctx.Response.ContentType = "application/json";
                ctx.Response.ContentEncoding = Encoding.UTF8;
                byte[] binaryData = Encoding.UTF8.GetBytes(serializer.Serialize(result));
                ctx.Response.OutputStream.Write(binaryData, 0, binaryData.Length);
            }
        };
        server.Start();
        Console.WriteLine("监听程序启动，回车键终止程序运行！");
        Console.ReadLine();
    }
}
```

如果我们调试程序，Visual Studio 又没有使用管理员身份证运行，很容易出现以下问题：

![20191225175729](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225175729_1577269066727.png)

如果直接双击打开应该也会有类似问题，需要右键该程序以管理员身份运行才可以，后面会另外写一篇文章，说明如何处理这个问题。

正常启动程序，并使用 Postman 访问效果如下：

![20191225180916](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225180916_1577269066740.png)

这里给出的例子没有给出具体使用动态链接库读卡的例子，这样更方便测试，一般读卡器厂商会提供动态链接库各种编程语言的例子，抄过来就能使用。

同样的，更多需要使用 ActiveX 的功能，例如 CA 签名、U盾 等也可以使用这个方案实现。

> 参考：
> - Wiki：[ActiveX](https://zh.wikipedia.org/wiki/ActiveX)
> - MSDN：[HttpListener Class](https://docs.microsoft.com/en-us/dotnet/api/system.net.httplistener?view=netframework-4.8)
> - Lodop 官网：[http://www.lodop.net/](http://www.lodop.net/)
