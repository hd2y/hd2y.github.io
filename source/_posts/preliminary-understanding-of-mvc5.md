---
title: "初步了解 MVC5"
date: "2019/05/31 16:13:00"
updated: "2020/02/11 20:43:46"
permalink: "preliminary-understanding-of-mvc5"
tags:
 - MVC
categories:
 - [开发, C#]
---

`ASP.NET MVC` 是一个适用于 WEB 应用程序的经典模型 `Model-View-Controller` 模式。相对于 `Web Forms` 一个单一的整体，`ASP.NET MVC` 是由连接在一起的各种代码层所组成。

## `Global.asax` 文件

### `Global.asax` 文件概述

`Global.asax` 这个文件包含全局应用程序事件的事件处理程序。它响应应用程序级别和会话级别事件的代码。

运行时， `Global.asax` 将被编译成一个动态生成的 `.NET Framework` 类，该类是从 `HttpApplication` 基类派生的。

因此在 `Global.asax` 中的代码可以访问 `HttpApplication` 类中所有的 `public` 或者 `protected` 的成员。

`Global.asax` 不被用户直接请求，但 `Global.asax` 中的代码会被自动执行来响应特定的应用程序事件。

`Global.asax` 是可选的，而且在一个 web 项目中是唯一的，它应该处于网站的根目录。

### 一个请求的完整处理过程

以下过程由 `Internet Information Service（inetinfo.exe）—— IIS` 执行：

+ 客户端发出请求
+ 验证请求
+ 给请求授权
+ 确定请求的缓存 
+ 获取缓存状态
+ 在请求的处理程序执行前
+ http 处理程序执行请求 （`asp.net` 页面由 `aspnet_wp.exe` 执行）
+ 在请求的处理程序执行后
+ 释放请求状态
+ 更新请求缓存
+ 请求结束

### `Global.asax` 中的事件

`Global.asax` 中的所有事件可以分成两种，一种是满足特定事件时才会被触发，一种是每次请求都会被按照顺序执行的事件。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using System.Web.Optimization;
using System.Web.Routing;

namespace JohnSun.MVC5.Web
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 在Web应用程序的生命周期里就执行一次
            // 在应用程序第一次启动和应用程序域创建事被调用
            // 适合处理应用程序范围的初始化代码
            AreaRegistration.RegisterAllAreas();// 注册 Area
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters); // 注册 Filter
            RouteConfig.RegisterRoutes(RouteTable.Routes);// 注册 Route
            BundleConfig.RegisterBundles(BundleTable.Bundles);// 注册 Bundle
        }

        void Application_End(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 在应用程序关闭时运行的代码，在最后一个HttpApplication销毁之后执行
            // 比如 IIS 重启，文件更新，进程回收导致应用程序转换到另一个应用程序域
        }

        void Session_Start(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 会话开始时执行
        }

        void Session_End(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 会话结束或过期时执行
            // 不管在代码中显式的清空 Session 或者 Session 超时自动过期，此方法都将被调用
        }

        void Application_Init(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 在每一个 HttpApplication 实例初始化的时候执行
        }

        void Application_Disposed(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 在应用程序被关闭一段时间之后，在 .net 垃圾回收器准备回收它占用的内存的时候被调用。
            // 在每一个 HttpApplication 实例被销毁之前执行
        }

        void Application_Error(object sender, EventArgs e)
        {
            // 不是每次请求都调用
            // 所有没有处理的错误都会导致这个方法的执行
        }


        /*********************************************************************/
        // 每次请求都会按照顺序执行以下事件
        /*********************************************************************/

        void Application_BeginRequest(object sender, EventArgs e)
        {
            // 每次请求时第一个出发的事件，这个方法第一个执行
        }

        void Application_AuthenticateRequest(object sender, EventArgs e)
        {
            // 在执行验证前发生，这是创建验证逻辑的起点
        }

        void Application_AuthorizeRequest(object sender, EventArgs e)
        {
            // 当安全模块已经验证了当前用户的授权时执行
        }

        void Application_ResolveRequestCache(object sender, EventArgs e)
        {
            // 当 ASP.NET 完成授权事件以使缓存模块从缓存中为请求提供服务时发生，从而跳过处理程序（页面或者是 WebService）的执行。
            // 这样做可以改善网站的性能，这个事件还可以用来判断正文是不是从 Cache 中得到的。
        }

        //------------------------------------------------------------------------
        // 在这个时候，请求将被转交给合适程序。例如：web 窗体将被编译并完成实例化
        //------------------------------------------------------------------------

        void Application_AcquireRequestState(object sender, EventArgs e)
        {
            // 读取了 Session 所需的特定信息并且在把这些信息填充到 Session 之前执行
        }

        void Application_PreRequestHandlerExecute(object sender, EventArgs e)
        {
            // 在合适的处理程序执行请求前调用
            // 这个时候，Session 就可以用了
        }

        //-------------------------------------------------
        //在这个时候，页面代码将会被执行，页面呈现为 HTML
        //-------------------------------------------------

        void Application_PostRequestHandlerExecute(object sender, EventArgs e)
        {
            // 当处理程序完成对请求的处理后被调用。
        }

        void Application_ReleaseRequestState(object sender, EventArgs e)
        {
            // 释放请求状态
        }

        void Application_UpdateRequestCache(object sender, EventArgs e)
        {
            // 为了后续的请求，更新响应缓存时被调用
        }

        void Application_EndRequest(object sender, EventArgs e)
        {
            // EndRequest 是在响应 Request 时最后一个触发的事件
            // 但在对象被释放或者从新建立以前，适合在这个时候清理代码
        }

        void Application_PreSendRequestHeaders(object sender, EventArgs e)
        {
            // 向客户端发送 Http 标头之前被调用
        }

        void Application_PreSendRequestContent(object sender, EventArgs e)
        {
            // 向客户端发送 Http 正文之前被调用
        }
    }
}
```

## 路由 `Routing`

`ASP.NET MVC` 不再是要依赖于物理页面，我们可以使用自己的语法自定义 URL，通过这些语法来指定资源和操作。语法通过 URL 模式集合表达，也称为路由。

路由是代表 URL 绝对路径的模式匹配字符串。所以路由可以是一个常量字符串，也可能包含一些占位符。

新建一个 `ASP.NET MVC` 项目，在 `Global.asax` 文件我们可以看到路由在这里注册，让程序在启动的时候得到处理。

```csharp
RouteConfig.RegisterRoutes(RouteTable.Routes);//注册 Route
```

转到定义可以看到注册的规则：

```csharp
public class RouteConfig
{
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
        );
    }
}
```

注意：因为匹配路由是按照添加顺序去解析，所以基本的路由规则是从特殊到一般排列，否则可能导致解析异常或 404 Not Found。  

具体配置参考文章：[史上最全的 ASP.NET MVC 路由配置](http://www.cnblogs.com/zeusro/p/RouteConfig.html) 与 [Attribute Routing in ASP.NET MVC 5](https://blogs.msdn.microsoft.com/webdev/2013/10/17/attribute-routing-in-asp-net-mvc-5/)。

## 控制器 `Controller`

![mvc1](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc1_1562741921709.png)

`ASP.NET MVC` 会调用不同的控制器类（和其内部不同的操作方法）这取决于传入 URL。所使用的 `ASP.NET MVC` 的默认 URL 路由逻辑使用这样的格式来判定哪些代码以便调用：`/[Controller]/[ActionName]/[Parameters]`。 
 
项目建立以后有一个默认的控制器 `\Controllers\HomeController.cs`

```csharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }

    public ActionResult About()
    {
        ViewBag.Message = "Your application description page.";

        return View();
    }

    public ActionResult Contact()
    {
        ViewBag.Message = "Your contact page.";

        return View();
    }
}
```

在 `Controllers` 文件夹右键 -> 添加 可以看到创建控制器的选项。  

控制器每一个方法为一个 `Action`，每个 `Action` 对应的是项目内 `\Views\[Controller]\[ActionName].cshml`，在方法内右键，菜单有“添加视图”与“转到视图”选项。  

### `ActionResult` 返回值

**（一）视图类型**

返回视图：

```csharp
public ActionResult Index()
{
    return View();
}
```

返回分部视图：

```csharp
public ActionResult ViewTest()
{
    return PartialView();
}
```

**注意：** 以上均需要对 `Action` 添加视图文件，若无 `Action` 方法的视图文件也可以直接通过视图名称或视图路径指定视图文件，具体可以查看 `View()` 方法重载。

**（二）文本类型**

返回 `JavaScript` 脚本：

```csharp
public ActionResult ContentJS()
{
    return Content("alert('test js.');", "text/javascript");
}
```

返回 `CSS` 样式：

```csharp
public ActionResult ContentCSS()
{
    HttpCookie cookie = Request.Cookies["theme"] ?? new HttpCookie("theme", "default");
    switch (cookie.Value)
    {
        case "Theme1": return Content("body{font-family: SimHei; font-size:1.2em}", "text/css");
        case "Theme2": return Content("body{font-family: KaiTi; font-size:1.2em}", "text/css");
        default: return Content("body{font-family: SimSong; font-size:1.2em}", "text/css");
    }
}
```

**（三）`JSON` 类型**

```csharp
public ActionResult JsonTest()
{
    return Json(new { name = "Kangkang", country = "China", email = "kangkang@163.com" }, JsonRequestBehavior.AllowGet);
}
```

**（四）图片多媒体类型**

```csharp
public ActionResult ImageTest(string id)
{
    string path = Server.MapPath($"/images/{id}.gif");
    return File(path, "image/gif");
}
```

**（五）`Javascript` 脚本类型**

```csharp
public ActionResult JavaScriptTest()
{
    return JavaScript("alert('test js.');");
}
```

**（六）文件类型（下载）**

```csharp
public ActionResult FileTest(string id)
{
    string fileName = "可爱.gif";// 客户端保存的文件名
    string filePath = Server.MapPath($"/images/1.gif");// 路径
    return File(new FileStream(filePath, FileMode.Open), "image/gif", fileName);
}
```

**注意：**
+ `FileContentResult`：是针对文件内容创建的 `FileResult`，它只是调用当前 `HttpResponse` 的 `OutputStream` 属性的 `Write` 方法直接将表示文件内容的字节数组写入响应输出流。
+ `FilePathResult`：是一个根据物理文件路径创建 `FileResult`。
+ `FileStreamResult`：允许我们通过一个用于读取文件内容的流来创建 `FileResult`。

**可以参考：** [了解 ASP.NET MVC 几种 ActionResult 的本质：FileResult](http://www.cnblogs.com/artech/archive/2012/08/14/action-result-02.html)

**（七）返回 `null` 或者 `void`**

```csharp
public ActionResult Empty()
{
    return null;
}
```

**（八）返回未经授权浏览状态**

```csharp
public ActionResult HttpUnauthorizedResult()
{
    return new HttpUnauthorizedResult();
}
```

**注意：** 响应给客户端错误代码 `401`（未经授权浏览状态），如果程序启用了 `Forms` 验证，并且客户端没有任何身份票据，则会跳转到指定的登录页。

**（九）页面跳转**

```csharp
public ActionResult Redirect()
{
    // 直接返回指定的url地址  
    return Redirect("http://www.google.com");
}
```

+ `RedirectToRouteResult`：直接使用 `Action Name` 进行跳转，也可以加上 `ControllerName` 以及参数。
+ `RedirectToActionResult`：指定路由进行跳转。

```csharp
public ActionResult RedirectActionResult()
{
    return RedirectToAction("Index", "Home", new { id = 1, name = "Kangkang" });
}

public ActionResult RedirectResult()
{
    return RedirectToRoute("Default", new { controller = "Home", action = "Index" });
}
```

**总结：** 这些返回类型的共同点，那便是对 `Action` 有一定的要求：
+ 必须是一个 `public` 方法
+ 必须是实例方法
+ 不能被重载
+ 必须返回 `ActionResult` 类型

## 视图 View

![mvc2](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc2_1562741921709.png)

### Views 文件夹下常用文件种类

文件类型    | 扩展名     | 概述
------------|------------|------
`HTML`        | `.html` 或 `htm` | 静态 `html` 文件；<br>在开发中通用性最高的页面；<br>属于 `ASP.NET MVC` 常用页面；
`Razor` 文件   | `.cshtml`    | 动态 `MVC Razor` 文件；<br>`ASP.NET MVC` 中，属于常用文件；<br>采用 Razor 语法格式；<br>基本取代 `.aspx` 文件；
`WebForm` 文件 | `.aspx`      | 动态 `ASPX` 文件；<br>属于 `WebForm` 架构文件；<br>在 `ASP.NET MVC` 中，也可以用该页面布局页面但不推荐；<br>在 `ASP.NET MVC` 中基本被 cshtml 文件取代
`ASP` 文件     | `.asp`       | 传统 `ASP` 文件，目前已过时；<br>发展：`asp`=>`aspx`=>`cshtml`；<br>对应架构：`ASP`=>`ASP.NET WebForm`=>`ASP.NET MVC`

分析：
+ `ASP.NET MVC` 页面基本被放在 Views 文件夹下；
+ 利用 `ASP.NET MVC` 模板生成框架，Views 文件夹下的默认页面为 `.cshtml` 页面；
+ `ASP.NET MVC` 默认页面为 Razor 格式的页面，因此默认页面为 `.cshtml` 页面；
+ `ASP.NET MVC` 中，支持 WebForm 页面，即 `.aspx` 页面；
+ `ASP.NET MVC` 中，支持静态 html 页面；

### 默认 Views 文件夹包含内容

文件夹名称          | 概述
--------------------|------
*`Account` 文件夹*     | *包含用于注册并登录用户账户的页面*
`Home` 文件夹          | 存储首页和 `About` 页面信息
`Share` 文件夹         | 存储控制器间分享的视图，如布局页面和模板页等
`_ViewStart.cshtml` | 程序最开始执行的页面

分析：
+ 这里没添加 `Account` 控制器；
+ 默认约定：在 `Controllers` 新增一个控制器，就会默认地在 `Views` 文件夹下新增一个视图文件夹，用来存放该控制器添加的视图，如上图中增加 `Home` 控制器，在 `Views` 下就自动新增加 `Home` 文件，用来存放是 `Home` 控制器视图； 

### 视图种类

**（一）起始视图：`_ViewStart.cshtml`**

```html
@*@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}*@

<div>
    <h2>这里是 _ViewStart.cshtml 视图</h2>
    <p>原则上程序运行时，这个视图首先被运行，其他视图在这个视图之后被运行。</p>
</div>
```

浏览 `http://127.0.0.1/Home/Index`

![mvc8](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc8_1562741921787.png)

**（二）布局视图：`_Layout.cshtml`**

首先将 `_ViewStart.cshtml` 文件还原修改其指向的 `_LayoutDemo.cshtml` 文件

```html
@{
    Layout = "~/Views/Shared/_LayoutDemo.cshtml";
}
```

添加 `_LayoutDemo.cshtml` 文件

```html
@{
    ViewBag.Title = "LayoutDemo";
}
<!DOCTYPE html>

<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>@ViewBag.Title</title>
</head>
<body>
    <h2>---如下内容来源为视图---</h2>
    <div>
        @RenderBody()
    </div>
    <h2>---如下内容为Footer区---</h2>
    <div>
        <footer>Copyright by John Sun.</footer>
    </div>
</body>
</html>
```

重新浏览 `http://127.0.0.1/Home/Index`

![mvc7](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc7_1562741921772.png)

+ `_Layout.cshtml` 基本结构就是 HTML 基本结构(其实 `.aspx` 和 `.cshtml` 结构，均是 html 结构)；
+ 在 `_LayoutDemo.cshtml` 文件一个后台代码：`@RenderBody()`。`@RenderBody()` 表示视图体，此外还有 `@RenderSection()` 表示部分视图和节点；

**（三）弱类型视图**

Controller 向 View 传递少量数据，一般情况，我们可以归为两大类别：弱类别传递（ViewBag，ViewData，TempData）和强类别传递（强类型视图）。然而，在实际操作中，当涉及大量数据时，弱类别就显得不是那么方便，此时，一般采用强类型视图。强类型视图一般由三部分构成，即控制器层，视图层和模型层，三者之间调用关系可表示为：

![mvc3](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc3_1562741921726.png)

**（1）ViewData 和 TempData**

ViewData 只在当前 Action 中有效，生命周期和 View 相同；

TempData 的数据至多只能经过一次 Controller 传递，并且每个元素至多只能被访问一次，访问以后，自动被删除。

TempData 一般用于临时的缓存内容或抛出错误页面时传递错误信息，可以将 TempData 在使用之前存储到相应的 ViewData 中以备循环使用。

TempData 保存在 Session 中，Controller 每次执行请求的时候，会从 Session 中先获取 TempData，而后清除 Session，获取完 TempData 数据，虽然保存在内部字典对象中，但是其集合中的每个条目访问一次后就从字典表中删除。具体代码层面，TempData 获取过程是通过 SessionStateTempDataProvider.LoadTempData 方法从 ControllerContext 的 Session 中读取数据，而后清除 Session，故 TempData 只能跨 Controller 传递一次。

*HomeController.cs*

```csharp
public ActionResult Index()
{
    ViewBag.Message = "Welcome to ASP.NET MVC!";
    ViewData["myName"] = "我的名字";
    TempData["myAgeOne"] = "26岁";
    TempData["myAgeTwo"] = "27岁";
    return View();
}
```

*Index.cshtml 文件*

```csharp
姓名：@ViewData["myName"]
<br />
年龄1：@TempData["myAgeOne"]
```

*About.cshtml 文件*

```csharp
姓名：@ViewData["myName"]
<br />
年龄1：@TempData["myAgeOne"]
<br />
年龄2：@TempData["myAgeTwo"]
```

**（2）ViewBag 和 ViewData**

```csharp
ViewBag.Name = ViewData["Name"];
```

ViewData                            | ViewBag
------------------------------------|------------------------------------
Key/Value 集合                       | dynamic 类型对象
`ASP.NET MVC 1` 就有                | `ASP.NET MVC 3` 才有
基于 `ASP.NET 3.5`                   | 基于 `ASP.NET 4.0` 与 `.NET Framework`
ViewData 较快                        | ViewBag 较慢
视图中查询数据需要转换合适的类型    | 视图中查询数据不需要类型转换
有一些类型转换代码                  | 可读性好

**（四）强类型视图**

增加实体类 *UserInfo.cs*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace JohnSun.MVC5.Web.Models
{
    public class UserInfo
    {
        public int UserId { get; set; }
        public string UserName { get; set; }
        public string Country { get; set; }
        public string Hobby { get; set; }
        public string Email { get; set; }
    }
}
```

*HomeController.cs* 增加 Action 方法

```csharp
public ActionResult ModelTest()
{
    return View(new List<UserInfo>
    {
        new UserInfo(){ UserId = 1, UserName = "Kangkang", Country = "China", Hobby = "Basketball", Email = "kangkang@163.com" },
        new UserInfo(){ UserId = 2, UserName = "Mary", Country = "America", Hobby = "Football", Email = "mary@gmail.com" },
        new UserInfo(){ UserId = 3, UserName = "Jane", Country = "Canada", Hobby = "Compute game", Email = "jane@yahoo.com" },
    });
}
```

增加视图 *ModelTest.cshtml*

```html
@model IEnumerable<JohnSun.MVC5.Web.Models.UserInfo>
@{
    ViewBag.Title = "ModelTest";
}

<h2>ModelTest</h2>
@foreach (var info in Model)
{
    <p>@($"{info.UserId}  {info.UserName}  {info.Country}")</p>
}
```

运行效果：

![mvc4](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc4_1562741921734.png)

**（五）分布页**

我们在 `/Views/Shared` 文件夹下创建一个分布页 `_PartialPageDemo.cshtml`，并向该页面中添加一段代码：

```html
<h1 style="color:red;">我是分布页</h1>
```

在 *HomeController.cs* 文件中增加返回视图的方法

```csharp
public ActionResult TestPartialPage()
{
    return PartialView();
}
```

在 /Views/Home 增加对应视图文件 *TestPartialPage.cshtml*

```html
<h1 style="color:red;">我是供控制器调用的分布页</h1>
```

在 *Index.cshtml* 中调用

```html
@{
    ViewBag.Title = "Home Page";
}

@Html.Partial("~/Views/Shared/_PartialPageDemo.cshtml")
@Html.Partial("~/Views/Home/TestPartialPage.cshtml")
@Html.Action("TestPartialPage")
```

显示效果如下：

![mvc5](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc5_1562741921744.png)

调用分布页的几种方式：
+ @Html.Partial() 提供分布页路径
+ @Html.Action() 提供控制器返回视图的方法
+ 通过 Ajax 方式调用

## 区域 Area

`ASP.NET MVC` 有预定义的目录规则，框架根据这些目录规则去加载各种类。

在 MVC 单项目中，随着业务越来越复杂多样，我们会希望按照功能对代码按文件夹分门别类。

如果在默认的目录结构下业务混合，这样不方便管理和维护；如果另开新项目，又比较散乱。那么 MVC 有没有这样一种机制来相对独立这些模块呢？答案是肯定的，这就是 MVC 的 Area 区域技术，用来实现在一个 MVC 项目中组织和维护多个相对独立的模块。

在 VS 中右键单击项目，在弹出的菜单中选择“添加 (A)”->“Area...”，在弹出的对话框中输入区域名称（遵守 C# 标示符命名规则）即可（比如输入 System），VS 将自动在根目录创建 Areas 文件夹，此文件夹下每个独立的 Area 一个文件夹，System 文件夹内也是一样的 Models、Controllers、Views 结构。

![mvc6](https://hd2y.oss-cn-beijing.aliyuncs.com/mvc6_1562741921755.png)

唯一不同的是多了一个 `SystemAreaRegistration.cs`（区域注册类），用于向 MVC 框架注册路由等信息，`Global.asax.cs` 中会自动调用该类的 RegisterArea 方法。新建 Area 后 VS 自动创建相关目录结构，按需修改 SystemAreaRegistration 路由即可。

```csharp
using System.Web.Mvc;

namespace JohnSun.MVC5.Web.Areas.System
{
    public class SystemAreaRegistration : AreaRegistration 
    {
        public override string AreaName 
        {
            get 
            {
                return "System";
            }
        }

        public override void RegisterArea(AreaRegistrationContext context) 
        {
            context.MapRoute(
                "System_default",
                "System/{controller}/{action}/{id}",
                new { action = "Index", id = UrlParameter.Optional }
            );
        }
    }
}
```

**注意：** 当 Area 中控制器与 MVC 中控制器同名，比如 HomeController，此时访问首页会异常提示：找到多个与名为“Home”的控制器匹配的类型。如果为此请求(`{controller}/{action}/{id}`)提供服务的路由没有指定命名空间以搜索与此请求相匹配的控制器，则会发生这种情况。如果是这样，请通过调用带有 `namespaces` 参数的 `MapRoute` 方法的重载来注册此路由。

此时修改 *RouteConfig.cs* 文件中注册路由的方法，添加 HomeController 的命名空间指向即可。

```csharp
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional },
    namespaces: new string[] { "JohnSun.MVC5.Web.Controllers" }
);
```

> 快速上手入门视频：[微软的船新框架 ASP.NET Core - 10 分钟讲完 MVC 基础 by Anduin](https://www.bilibili.com/video/av29975319)
