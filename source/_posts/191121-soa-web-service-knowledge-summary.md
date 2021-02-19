---
title: "SOA —— WebService 知识点总结"
date: "2019/11/21 17:28:29"
updated: "2019/12/25 18:43:24"
permalink: "soa-web-service-knowledge-summary/"
tags:
 - WebService
 - SOA
categories:
 - [开发, C#]
---

虽然 `WebService` 已经很 `Low` 了，但是胜在简单。所以很多小公司或者公司内部仍然会使用这个做一些接口。

这里总结一下 `WebService` 的一些使用技巧，以及经验总结。

## 创建服务端

`WebService` 宿主是 `IIS`，所以我们需要先创建一个 `ASP.Net Web` 的空项目，当然如果选择 `MVC` 或 `WebForm` 也没有影响。

![20191120110944](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120110944_1574327411385.png)

创建以后我们就可以添加对应的服务文件，如下图：

![20191120111531](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120111531_1574327411395.png)

会生成一个 `*.asmx` 文件与 一个 `*.asmx.cs` 文件，结构与 `WebForm` 的窗体页面或一般处理程序等一致。

如果我们没有将后台代码 `cs` 文件另外单独存放的需求，那么就不需要调整，直接修改展开对 `cs` 文件进行修改即可。

这里我们添加一些方法用于测试：

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Services;

namespace JohnSun.SOA.WebService.Server
{
    /// <summary>
    /// MyWebService 的摘要说明
    /// </summary>
    [WebService(Namespace = "http://tempuri.org/")]
    [WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
    [System.ComponentModel.ToolboxItem(false)]
    // 若要允许使用 ASP.NET AJAX 从脚本中调用此 Web 服务，请取消注释以下行。 
    // [System.Web.Script.Services.ScriptService]
    public class MyWebService : System.Web.Services.WebService
    {
        private static List<UserInfo> _users = new List<UserInfo>()
        {
            new UserInfo(){ Id = 1, Name = "Kangkang", Country = "China" },
            new UserInfo(){ Id = 2, Name = "John", Country = "America" },
            new UserInfo(){ Id = 3, Name = "Jane", Country = "France" },
            new UserInfo(){ Id = 4, Name = "Han Meimei", Country = "China" },
        };
        [WebMethod]
        public string HelloWorld()
        {
            return "Hello World";
        }

        [WebMethod]
        public decimal Sum(decimal x, decimal y)
        {
            return x + y;
        }

        [WebMethod]
        public UserInfo GetUserInfo(int id)
        {
            return _users.Find(u => u.Id == id);
        }

        [WebMethod]
        public List<UserInfo> GetUsers(string country)
        {
            return _users.FindAll(u => u.Country == country);
        }

        [WebMethod]
        public UserInfo[] GetAllUsers()
        {
            return _users.ToArray();
        }
    }

    public class UserInfo
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Country { get; set; }
    }
}
```

需要注意的是：
+ 如果我们需要调用一个方法，则需要将方法标记为 `WebMethod` 特性，才能调用。
+ 这里不遵循重载，所以方法名不能重复，即便入参不同也不行。
+ 入参和返回值都可以是数组或集合，但是调用服务时，可以配置，这里再调用时会说明。

完成后，我们就可以右键该文件，在浏览器打开访问该服务。

![20191120114307](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120114307_1574327414494.png)

另外，我们也可以在浏览器里直接调用服务。

![20191120114405](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120114405_1574327414490.png)

## 连接服务端

服务端托管完成以后，我们就可以使用客户端完成对服务端的调用了，客户端的项目没有什么限制，但是建议是使用 `.NET Framework 4.0` 或以上版本，否则添加服务的界面可能会与截图演示的有些许区别。

这里为了方便演示，直接添加一个 `WinForm` 的项目，然后可以在引用中，选择添加服务引用：

![20191120115420](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120115420_1574327414496.png)

在弹出的添加页面，录入我们的服务地址，点击发现，添加我们需要的服务，另外我们需要调整这个服务的命名控件，需要注意的是不要与其他命名控件或类型重名，否则会比较麻烦：

![20191120115726](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120115726_1574327417649.png)

服务创建后，在窗体中简单写一些调用服务的代码进行测试：

```csharp
private void button1_Click(object sender, EventArgs e)
{
    MyWebServiceSoapClient client = null;
    try
    {
        client = new MyWebServiceSoapClient();
        client.Open();
        string h = client.HelloWorld();
        Log(LogLevel.Info, $"调用 HelloWorld 方法成功：{h}");
        decimal x = 1m;
        decimal y = 2m;
        decimal d = client.Sum(x, y);
        Log(LogLevel.Info, $"调用 Sum 方法成功：Sum({x}, {y}) = {d}");
        int id = 1;
        UserInfo info = client.GetUserInfo(id);
        JavaScriptSerializer serializer = new JavaScriptSerializer();
        Log(LogLevel.Info, $"调用 GetUserInfo 方法成功：GetUserInfo({id}) = {serializer.Serialize(info)}");
        string country = "China";
        UserInfo[] users = client.GetUsers(country);
        Log(LogLevel.Info, $"调用 GetUsers 方法成功：GetUsers({country}) 获取到用户数量： {users.Length}");
        UserInfo[] allUsers = client.GetAllUsers();
        Log(LogLevel.Info, $"调用 GetAllUsers 方法成功，获取到用户数量： {allUsers.Length}");
        client.Close();
    }
    catch (Exception exc)
    {
        if (client != null)
            client.Abort();
        Log(LogLevel.Error, "测试服务失败：" + exc.Message);
    }
}
```

测试一下调用：

![20191120162309](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120162309_1574327417642.png)

这里主要需要注意两点：

1. `MyWebServiceSoapClient` 并未继承 `IDisposable` 接口，所以不能使用 `using` 来关闭连接，使用上文的写法即可，否则资源释放存在问题。
2. 无论返回值是集合还是数组，我们只能使用一种类型，服务创建以后我们也可以指定，如上虽然我们 `GetUsers` 在服务中定义返回集合，但是调用时返回的仍然是数组。

如果我们想让服务默认返回的数据类型调整为集合，可以在引用的服务上右键，选择“配置服务引用”，将集合类型调整为 `System.Collections.Generic.List`，同样的字典类型也可以调整。

![20191120194502](https://hd2y.oss-cn-beijing.aliyuncs.com/20191120194502_1574327417655.png)

## 添加身份认证

以上服务方法都没有进行身份认证，如果一些私有的方法，就会有安全问题，我们也可以在服务中配置简单身份认证。

比较常用的是使用 `SoapHeader` 为服务方法，添加 `SOAP 标头`，这样我们在调用服务时就需要传入一个标头信息，我们可以使用这个信息来传递用于用户验证的信息。

首先需要调整 `WebService` 服务端的代码，定义一个继承自 `SoapHeader` 的类型 `AuthenticationHeader`：

```csharp
public class AuthenticationHeader : SoapHeader
{
    public string UserName { get; set; }
    public string Password { get; set; }
    public string Token { get; set; }
    public void Validate()
    {
        if (UserName == "admin" && Password == "admin")
        {
            MD5 md5 = MD5.Create();
            Token = Convert.ToBase64String(md5.ComputeHash(Encoding.UTF8.GetBytes(Password)));
        }
        else
        {
            throw new SoapException("Failed to verify user login information.", SoapException.ServerFaultCode);
        }
    }
}
```

修改服务后台类，增加一个用于接收 `SoapHeader` 的字段，为需要附加标头的方法标记 `SoapHeader` 特性，并指定 `SOAP 标头` 的数据赋值给服务后台类的哪个字段：

```csharp
public AuthenticationHeader authenticationHeader;

[WebMethod]
[SoapHeader("authenticationHeader")]
public string Validate()
{
    if (authenticationHeader != null)
    {
        authenticationHeader.Validate();
        return authenticationHeader.Token;
    }
    else
        return null;
}

[WebMethod]
[SoapHeader("authenticationHeader", Direction = SoapHeaderDirection.InOut)]
public UserInfo[] GetAllUsers()
{
    if (authenticationHeader != null)
        authenticationHeader.Validate();
    else
        return null;
    return _users.ToArray();
}
```

修改完成以后需要重新编译这个服务，并且在客户端更新服务引用，更新完成以后 `GetAllUsers` 方法应该会报错，因为需要我们传递一个 `AuthenticationHeader`，可以将客户端调用服务方法的代码略作调整：

```csharp
AuthenticationHeader header = new AuthenticationHeader() { UserName = "admin", Password = "admin" };
UserInfo[] allUsers = null;
try
{
    allUsers = client.GetAllUsers(ref header);
    Log(LogLevel.Info, $"调用 GetAllUsers 方法成功，获取到用户数量： {allUsers?.Length}，Token：{header.Token}");
}
catch (Exception exc)
{
    Log(LogLevel.Error, $"调用 GetAllUsers 方法失败：{exc.Message}");
}

try
{
    header = new AuthenticationHeader() { UserName = "test", Password = "test" };
    string token = client.Validate(header);
    if (token != null)
    {
        Log(LogLevel.Info, $"调用 Validate 方法成功：{token}");
    }
    else
    {
        Log(LogLevel.Error, $"调用 Validate 方法失败，请验证用户名和密码。");
    }
}
catch (Exception exc)
{
    Log(LogLevel.Error, $"调用 Validate 方法失败：{exc.Message}");
}
```

修改以后可以执行测试，运行客户端查看一下效果：

![20191121151110](https://hd2y.oss-cn-beijing.aliyuncs.com/20191121151110_1574327420894.png)

当然除以上方法外，我们还可以提供一个登录的服务方法，如果验证成功返回一个 `token`，然后私有的方法增加一个 `token` 的入参，每次调用方法前进行验证，因为比较简单这里不再过多演示。

## 动态调整服务

目前来说服务是固定指向了一个我们生成服务时的地址，当然如果我们想要修改服务地址也是很简单的，只需要打开 `app.config` 文件，修改默认生成的配置信息：

![20191121155612](https://hd2y.oss-cn-beijing.aliyuncs.com/20191121155612_1574327420895.png)

这样我们要严格的依赖这个配置文件，而且如果我们想要配置多个服务地址，在一个地址无法连接时自动切换到其他服务，好像是不可实现的，所以第一步就是移除掉对配置文件的依赖，我们删除 `app.config` 文件，重新生成项目并运行：

```html
17:19:31 Error: 测试服务失败：在 ServiceModel 客户端配置部分中，找不到引用协定“MyServiceTest.MyWebServiceSoap”的默认终结点元素。这可能是因为未找到应用程序的配置文件，或者是因为客户端元素中找不到与此协定匹配的终结点元素。
```

这个其实很简单，因为没有了配置文件读取不到服务的链接地址，我们可以在初始化客户端的时候，指定服务端连接：

```csharp
client = new MyWebServiceSoapClient(new BasicHttpBinding(), new EndpointAddress("http://localhost:15178/MyWebService.asmx"));
```

因为已经不再依赖配置文件，这样我们初始化客户端时就可以更自由，我们可以创建一个工厂来初始化服务：

```csharp
using JohnSun.SOA.WebService.Client.MyServiceTest;
using System;
using System.Collections.Generic;
using System.Linq;
using System.ServiceModel;
using System.Text;

namespace JohnSun.SOA.WebService.Client
{
    public class SoapClientFactory
    {
        public static bool TryGetSoapClient(out MyWebServiceSoapClient soapClient, params string[] urls)
        {
            soapClient = null;
            if (urls == null || urls.Length == 0)
                return false;
            foreach (string url in urls)
            {
                try
                {
                    soapClient = new MyWebServiceSoapClient(new BasicHttpBinding(), new EndpointAddress(url));
                    soapClient.HelloWorld();
                    break;
                }
                catch
                {
                    if (soapClient != null)
                        soapClient.Abort();
                    soapClient = null;
                }
            }
            return soapClient != null;
        }
    }
}
```

然后初始化服务可以调整为以下代码：

```csharp
if (!SoapClientFactory.TryGetSoapClient(out client
    , "http://localhost:80/MyWebService.asmx"
    , "http://localhost:1008/MyWebService.asmx"
    , "http://localhost:15178/MyWebService.asmx"))
{
    Log(LogLevel.Error, "初始化服务失败！");
    return;
}
else
{
    Log(LogLevel.Info, $"初始化服务成功：{client.Endpoint.ListenUri}");
}
```

测试效果：

![20191121162727](https://hd2y.oss-cn-beijing.aliyuncs.com/20191121162727_1574327420895.png)

我们已经解决了必须通过 `app.config` 来配置服务地址的问题，但是，如果我们只有 `wsdl` 文件，无法连接服务添加服务应该怎么处理呢？

首先我们下载服务的 `wsdl` 文件用于演示，在服务后面添加 `wsdl` 参数即可下载：`http://localhost:15178/MyWebService.asmx?wsdl`

然后和通过链接添加服务一样，不过我们输入的是下载下来的 `wsdl` 文件的文件路径：

![20191121163348](https://hd2y.oss-cn-beijing.aliyuncs.com/20191121163348_1574327424355.png)

其实如果我们做出来的服务要提供给第三方调用，也是通过这种方式，将 `wsdl` 文件下载下来，发送给第三方即可。

> 参考：<br>
> + MSDN - `SoapHeader` 类：[https://docs.microsoft.com/zh-cn/dotnet/api/system.web.services.protocols.soapheader?view=netframework-4.8](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.services.protocols.soapheader?view=netframework-4.8)<br>
> + MSDN - `SoapException` 类：[https://docs.microsoft.com/zh-cn/dotnet/api/system.web.services.protocols.soapexception?view=netframework-4.8](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.services.protocols.soapexception?view=netframework-4.8)<br>
> + MSDN - `ClientBase<TChannel>` 类：[https://docs.microsoft.com/zh-cn/dotnet/api/system.servicemodel.clientbase-1?view=netframework-4.8](https://docs.microsoft.com/zh-cn/dotnet/api/system.servicemodel.clientbase-1?view=netframework-4.8)<br>
> + 维基百科 - `WSDL`：[https://zh.wikipedia.org/wiki/WSDL](https://zh.wikipedia.org/wiki/WSDL)

> 源码下载：<br>
> + Git：[https://git.hd2y.net/hd2y/JohnSun.SOA.WebService](https://git.hd2y.net/hd2y/JohnSun.SOA.WebService)<br>
> + 百度网盘：[https://pan.baidu.com/s/1-KePcEeDDEbrclmSo3B6DQ](https://pan.baidu.com/s/1-KePcEeDDEbrclmSo3B6DQ) 提取码：`kbvy`
