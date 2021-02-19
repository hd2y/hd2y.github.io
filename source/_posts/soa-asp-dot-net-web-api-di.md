---
title: "SOA —— ASP.NET Web API 依赖注入"
date: "2019/12/02 19:15:37"
updated: "2019/12/25 18:38:55"
permalink: "soa-asp-dot-net-web-api-di"
tags:
 - 依赖注入
 - SOA
 - Autofac
 - WebApi
 - Unity
categories:
 - [开发, C#]
---

依赖注入（Dependency Injection，缩写DI）是将系统中各层对象解耦的一种方式，是实现控制反转（Inversion of Control，缩写IoC）的一种常用方式。

## 搭建框架

首先我们搭建一个用于测试的基本框架，因为最近在了解 `FreeSql`，所以数据访问就使用 `FreeSql`，在这里推荐一下这个项目，叶老板真的很🐂🍺。

下图可以看出项目的简单架构：

![20191130104028](https://hd2y.oss-cn-beijing.aliyuncs.com/20191130104028_1575284957637.png)

### 实体

实体层定义了实体的接口 `IBaseEntity` ：

```csharp
public class IBaseEntity
{
    [Column(IsPrimary = true, IsIdentity = true)]
    public long Id { get; set; }
    public DateTime CreateTime { get; set; } = DateTime.Now;
    public DateTime UpdateTime { get; set; } = DateTime.Now;
}
```

标本类 `Specimen`：

```csharp
public class Specimen : IBaseEntity
{
    public DateTime InspectionDate { get; set; }
    public int Type { get; set; }
    public string SpeicimenNo { get; set; }
    public int State { get; set; }
    public string Barcode { get; set; }
    public DateTime? ResultTime { get; set; }
    public int? ResultUserId { get; set; }
    public DateTime? CheckTime { get; set; }
    public int? CheckUserId { get; set; }
    public DateTime? ReportTime { get; set; }
    public int? ReportUserId { get; set; }
}
```

### 服务接口

服务接口首先定义了服务常用的一些操作 `IBaseService`：

```csharp
public interface IBaseService<T> where T : IBaseEntity
{
    T Insert(T t);
    T Get(int id);
    T Update(T t);
    bool Delete(int id);
}
```

然后不同的业务实体，也有自己的业务逻辑 `ISpecimenService`：

```csharp
public interface ISpecimenService : IBaseService<Specimen>
{
    Specimen CheckSpecimen(int specimenId, int userId);
}
```

### 服务实现

服务接口的实现，首先是 `IBaseService` 的实现 `BaseService`：

```csharp
public class BaseService<T> : IBaseService<T> where T : IBaseEntity
{
    protected virtual IFreeSql FreeSql { get; set; }

    public BaseService(IFreeSql freeSql)
    {
        FreeSql = freeSql;
    }

    public virtual bool Delete(int id)
    {
        return FreeSql.Delete<T>(id).ExecuteAffrows() > 0;
    }

    public virtual T Get(int id)
    {
        return FreeSql.Select<T>(id).First();
    }

    public virtual T Insert(T t)
    {
        int id = (int)FreeSql.Insert<T>().AppendData(t).ExecuteIdentity();
        return FreeSql.Select<T>(id).First();
    }

    public virtual T Update(T t)
    {
        t.UpdateTime = DateTime.Now;
        int rows = FreeSql.Update<T>(t).ExecuteAffrows();
        if (rows == 1)
            return FreeSql.Select<T>(t.Id).First();
        else
            return null;
    }
}
```

`ISpecimenService` 的实现 `SpecimenService`：

```csharp
public class SpecimenService : BaseService<Specimen>, ISpecimenService
{
    public SpecimenService(IFreeSql freeSql) : base(freeSql) { }

    public Specimen CheckSpecimen(int specimenId, int userId)
    {
        return FreeSql.Update<Specimen>().Set(s => new { CheckUserId = userId, CheckTime = DateTime.Now }).ExecuteUpdated().First();
    }
}
```

### Web 站点

首先是需要创建用于初始化 `FreeSql` 的工厂 `FreeSqlFactory`：

```csharp
public class FreeSqlFactory
{
    public static IFreeSql FreeSql { get; private set; }
    static FreeSqlFactory()
    {
        FreeSql = new FreeSql.FreeSqlBuilder()
                  .UseConnectionString(global::FreeSql.DataType.Sqlite, "Data Source=|DataDirectory|\\data.db;Pooling=true;Max Pool Size=10")
                  .UseAutoSyncStructure(true)
                  .Build();
    }
}
```

这里主要是想让该项目简单一些，所以只考虑服务层 `JohnSun.SOA.WebAPI.Service` 的注入，所以直接将数据库访问层依赖细节，使用 `SQLite` 数据库，并使用 `FreeSql` 的 `CodeFirst` 模式。

然后就创建一个 `Web API` 控制器 `SpecimensController`：

```csharp
public class SpecimensController : ApiController
{
    public Specimen Get(int id)
    {
        SpecimenService service = new SpecimenService(FreeSqlFactory.FreeSql);
        return service.Get(id);
    }

    public Specimen Post(DateTime inspectionDate, string speicimenNo, int type, string barcode = null)
    {
        SpecimenService service = new SpecimenService(FreeSqlFactory.FreeSql);
        return service.Insert(new Specimen { InspectionDate = inspectionDate, SpeicimenNo = speicimenNo, Type = type, Barcode = barcode });
    }
}
```

这里只提供两个简单的方法，一个新增方法，一个通过Id查询数据。

### 访问测试

截止到目前，一个简单的测试项目创建完成，我们可以使用 `postman` 对项目进行简单的测试。

首先是调用 `POST` 方法，新增标本 `http://localhost:58683/api/specimens?inspectionDate=2019-11-30&speicimenNo=1001&type=0`：

![20191130101621](https://hd2y.oss-cn-beijing.aliyuncs.com/20191130101621_1575284957660.png)

返回的是一个创建成功后的数据库实体，我们可以通过这个对象的 `id` 再使用 `GET` 方法进行查询 `http://localhost:58683/api/specimens/1`：

![20191130101643](https://hd2y.oss-cn-beijing.aliyuncs.com/20191130101643_1575284957659.png)

## Unity 实现依赖注入

为了方便理解，我们将移除依赖具体实现的步骤罗列出来，方便对控制反转的实现有一个简单的了解。

### 使用 Unity 创建对象

首先我们将对象的创建移交给 `Unity`，首先我们使用 `nuget` 为 `Web` 项目添加 `Unity` 的包引用：

```bash
Install-Package Unity -Version 5.11.1
```

然后我们添加一个统一的依赖注入容器，将需要用到的类型注册到容器中：

```csharp
IUnityContainer container = new UnityContainer();
// 注册单例 因为 ISpecimenService 的构造函数需要提供一个 IFreeSql 的参数
container.RegisterInstance(FreeSqlFactory.FreeSql);
// 注册 SpecimenService 到 ISpecimenService
container.RegisterType<ISpecimenService, SpecimenService>();
```

这样我们创建 `ISpecimenService` 的实例对象就不需要再依赖细节 `SpecimenService`：

```csharp
ISpecimenService service = container.Resolve<ISpecimenService>();
```

### 调整为工厂

我们并不需要每次初始化 `ISpecimenService` 都创建一个依赖注入的容器，实际上这个容器只需要创建一次，所以我们增加一个 `ContainerFactory` 的类型：

```csharp
public class ContainerFactory
{
    public static IUnityContainer Container { get; private set; }
    static ContainerFactory() 
    {
        Container = new UnityContainer();
        Container.RegisterInstance(FreeSqlFactory.FreeSql);
        Container.RegisterType<ISpecimenService, SpecimenService>();
    }
}
```

然后调整我们的控制器 `SpecimensController`：

```csharp
public Specimen Get(int id)
{
    ISpecimenService service = ContainerFactory.Container.Resolve<ISpecimenService>();
    return service.Get(id);
}

public Specimen Post(DateTime inspectionDate, string speicimenNo, int type, string barcode = null)
{
    ISpecimenService service = ContainerFactory.Container.Resolve<ISpecimenService>();
    return service.Insert(new Specimen { InspectionDate = inspectionDate, SpeicimenNo = speicimenNo, Type = type, Barcode = barcode });
}
```

### 调整为使用配置文件

因为 `ContainerFactory` 工厂类注册时仍然存在 `SpecimenService`，所以我们要将改类型从上层中移除，否则我们仍然无法指定 `ISpecimenService`。

Unity 为我们提供了使用配置文件初始化依赖注入容器的方案，所以首先我们移除 `Web` 对于 `JohnSun.SOA.WebAPI.Service` 项目的引用，使服务不再强依赖于该项目，方便后期对产品的升级改造。

但是测试项目仍然需要一个实现，所以我们需要在移除依赖后，手动的将 `JohnSun.SOA.WebAPI.Service/bin` 目录下的文件拷贝到 `JohnSun.SOA.WebAPI.Server/bin` 下。

添加 `Unity.Configuration` 的引用：

```bash
Install-Package Unity.Configuration -Version 5.11.1
```

在 `Web` 项目中添加配置文件 `ConfigFiles/Unity.config`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="unity" type="Microsoft.Practices.Unity.Configuration.UnityConfigurationSection, Unity.Configuration" />
  </configSections>

  <unity>
    <aliases>
      <add alias="ISpecimenService" type="JohnSun.SOA.WebAPI.Interface.ISpecimenService, JohnSun.SOA.WebAPI.Interface" />
      <add alias="SpecimenService" type="JohnSun.SOA.WebAPI.Service.SpecimenService, JohnSun.SOA.WebAPI.Service" />
    </aliases>
    <container>
      <register type="ISpecimenService" mapTo="SpecimenService" />
    </container>
  </unity>
</configuration>
```

调整 `ContainerFactory` 的静态构造函数代码：

```csharp
Container = new UnityContainer();
Container.RegisterInstance(FreeSqlFactory.FreeSql);

string configPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"ConfigFiles\Unity.config");
ExeConfigurationFileMap map = new ExeConfigurationFileMap { ExeConfigFilename = configPath };
Configuration configuration = ConfigurationManager.OpenMappedExeConfiguration(map, ConfigurationUserLevel.None);
UnityConfigurationSection section = (UnityConfigurationSection)configuration.GetSection(UnityConfigurationSection.SectionName);

section.Configure(Container);
```

这时在 `Web` 项目中已经不需要依赖 `JohnSun.SOA.WebAPI.Service` 的任何细节，均可以通过配置文件实现。

> 注意：<br>
> FreeSql 仍然依赖细节，前文已经提到为了测试项目的简单，所以就 `不要在意细节` 了。<br>
> 后面写到这段代码的时候有考虑将 `FreeSqlFactory.FreeSql` 移交给配置文件，但是配置文件 `instance` 节点在 `Unity` 项目中给出的测试用例，无法指定 `value` 为一个程序中的字段或属性，`value` 只能设置一些常量值例如字符串或数字。<br>
> 后面仔细想了一下，如果真的想移除依赖，可能需要增加一个类似 `IFreeSqlInfo` 的对象，将 `IFreeSql` 作为该类型的一个属性，这样我们还可以提供一个多个参数的构造函数，用来指定连接的数据库、数据库类型、连接字符串、是否启用数据库迁移等等。<br>
> 当然，最终也要将 `SpecimenService` 构造函数中的 `IFreeSql` 调整成 `IFreeSqlInfo` 的实现才行。

### 控制器的注入

虽然已经成功添加容器，移除了细节的依赖，但是控制器中我们仍然需要使用 `Resolve` 构造对象，所以下一步我们需要将 `Service` 对象的构造也注入进来。

首先我们需要修改控制器的结构，将 `SpecimensController` 控制器调整为以下代码：

```csharp
public class SpecimensController : ApiController
{
    private readonly ISpecimenService _specimenService;
    public SpecimensController(ISpecimenService specimenService) 
    {
        _specimenService = specimenService;
    }

    public Specimen Get(int id)
    {
        return _specimenService.Get(id);
    }

    public Specimen Post(DateTime inspectionDate, string speicimenNo, int type, string barcode = null)
    {
        return _specimenService.Insert(new Specimen { InspectionDate = inspectionDate, SpeicimenNo = speicimenNo, Type = type, Barcode = barcode });
    }
}
```

这个时候如果直接运行项目肯定是不行的，因为控制器默认无参数的构造函数已经不存在了，会报错。

#### 传统方式注入

根据 MSDN 上的文档，我们可以借助 Web API 定义用于解析依赖项的 `IDependencyResolver` 接口来实现控制器的注入。

```csharp
public interface IDependencyResolver : IDependencyScope, IDisposable
{
    IDependencyScope BeginScope();
}

public interface IDependencyScope : IDisposable
{
    object GetService(Type serviceType);
    IEnumerable<object> GetServices(Type serviceType);
}
```

`IDependencyScope` 接口有两种方法：
+ `GetService` 创建一个类型的实例。
+ `GetServices` 创建指定类型的对象的集合。

`IDependencyResolver` 方法继承 `IDependencyScope` 并添加 `BeginScope` 方法。

当 Web API 创建控制器实例时，它将首先调用 `IDependencyResolver` 的 `GetService` 方法，并传入控制器类型。我们可以借助该特性，来构造任何控制器，如果 `GetService` 返回 `Null`，Web API 将在控制器类上查找无参数的构造函数。

```csharp
public class UnityResolver : IDependencyResolver
{
    protected IUnityContainer container;

    public UnityResolver(IUnityContainer container)
    {
        if (container == null)
        {
            throw new ArgumentNullException("container");
        }
        this.container = container;
    }

    public IDependencyScope BeginScope()
    {
        return new UnityResolver(container.CreateChildContainer());
    }

    public void Dispose()
    {
        Dispose(true);
    }

    protected virtual void Dispose(bool disposing)
    {
        container.Dispose();
    }

    public object GetService(Type serviceType)
    {
        try
        {
            return ContainerFactory.Container.Resolve(serviceType);
        }
        catch (ResolutionFailedException)
        {
            return null;
        }
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        try
        {
            return ContainerFactory.Container.ResolveAll(serviceType);
        }
        catch (ResolutionFailedException)
        {
            return new List<object>();
        }
    }
}
```

最后我们需要将 `HttpConfiguration` 中的 `DependencyResolver` 属性替换为我们实现的 `UnityResolver`。

修改 `WebApiConfig.Register` 方法，增加以下代码：

```csharp
config.DependencyResolver = new UnityResolver(ContainerFactory.Container);
```

#### 使用 Unity.WebAPI

Unity 提供了一个包，方便我们控制器的注入，首先要通过 `nuget` 安装包：

```bash
Install-Package Unity.WebAPI -Version 5.4.0
```

同之前一样，修改 `WebApiConfig.Register` 方法，增加以下代码：

```csharp
config.DependencyResolver = new Unity.WebApi.UnityDependencyResolver(ContainerFactory.Container);
```

`UnityDependencyResolver` 类型是帮我们实现的 `IDependencyResolver` 接口，通过反编译代码可以了解和我们从 `MSDN` 的常规实现基本一致：

```csharp
// Unity.WebApi.UnityDependencyScope
using System;
using System.Collections.Generic;
using System.Web.Http.Controllers;
using System.Web.Http.Dependencies;
using Unity;
using Unity.Resolution;

public class UnityDependencyScope : IDependencyScope, IDisposable
{
	protected IUnityContainer Container
	{
		get;
		private set;
	}

	public UnityDependencyScope(IUnityContainer container)
	{
		Container = container;
	}

	public object GetService(Type serviceType)
	{
		if (typeof(IHttpController).IsAssignableFrom(serviceType))
		{
			return UnityContainerExtensions.Resolve(Container, serviceType, (ResolverOverride[])(object)new ResolverOverride[0]);
		}
		try
		{
			return UnityContainerExtensions.Resolve(Container, serviceType, (ResolverOverride[])(object)new ResolverOverride[0]);
		}
		catch
		{
			return null;
		}
	}

	public IEnumerable<object> GetServices(Type serviceType)
	{
		return UnityContainerExtensions.ResolveAll(Container, serviceType, (ResolverOverride[])(object)new ResolverOverride[0]);
	}

	public void Dispose()
	{
		((IDisposable)Container).Dispose();
	}
}

// Unity.WebApi.UnityDependencyResolver
using System;
using System.Web.Http.Dependencies;
using Unity;
using Unity.WebApi;

public class UnityDependencyResolver : UnityDependencyScope, IDependencyResolver, IDependencyScope, IDisposable
{
	public UnityDependencyResolver(IUnityContainer container)
		: base(container)
	{
	}

	public IDependencyScope BeginScope()
	{
		return (IDependencyScope)(object)new UnityDependencyScope(base.Container.CreateChildContainer());
	}
}
```

## Autofac 实现依赖注入

相较于 `Unity`，其实 `Autofac` 使用更广泛，普遍认为后者的性能表现要优于前者，并且后者拥有中文文档，社区更活跃。

### 代码中为容器注册类型

本质上 Autofac 与前者差异不大，所以这里只是简单的介绍一下使用，更多的知识可以了解官方的文档。

这里添加一个 `AutofacContainerFactory` 的类型：

```csharp
public class AutofacContainerFactory
{
    public static IContainer Container { get; private set; }
    static AutofacContainerFactory()
    {
        // Autofac 在程序中注册
        var builder = new ContainerBuilder();
        builder.RegisterInstance(FreeSqlFactory.FreeSql);
        builder.RegisterType<SpecimenService>().As<ISpecimenService>();
        Container = builder.Build();
    }
}
```

这时无法运行程序，甚至会报错，因为基于 `Unity` 的那套代码，`Service` 的项目依赖已经移除，这里会报错。所以这里知识代码演示。

### 使用配置文件

由于 `Autofac 4.0+` 配置依赖于 `Microsoft.Extensions.Configuration.Xml` 或 `Microsoft.Extensions.Configuration.Json`，并且这两个项目依赖于 `.NET Standard 2.0`，所以需要将项目都升级到 `.NET Framework 4.6.1+`。

另外为了能让项目成功运行起来，并且使用的是 `Autofac` 实现依赖注入，我们还需要引用几个 nuget 程序包：
+ `Autofac.Configuration`：为 `autofac` 提供配置功能。
+ `Autofac.WebApi2`：ASP.NET Web API 控制器的依赖注入。
+ `Microsoft.Extensions.Configuration.Json`：如果使用 json 文件配置 autofac 添加该引用。
+ `Microsoft.Extensions.Configuration.Xml`：如果使用 xml 文件配置 autofac 添加该引用。

调整 `AutofacContainerFactory` 静态构造函数中的代码：

```csharp
public class AutofacContainerFactory
{
    public static IContainer Container { get; private set; }
    static AutofacContainerFactory()
    {
        // Autofac 在程序中注册
        //var builder = new ContainerBuilder();
        //builder.RegisterInstance(FreeSqlFactory.FreeSql);
        //builder.RegisterType<Service.SpecimenService>().As<ISpecimenService>();
        //builder.RegisterApiControllers(AppDomain.CurrentDomain.GetAssemblies());
        //Container = builder.Build();

        // Autofac 使用配置文件注册
        var builder = new ContainerBuilder();
        builder.RegisterInstance(FreeSqlFactory.FreeSql);

        var config = new ConfigurationBuilder();

        // Json 文件
        //string configPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"ConfigFiles\Autofac.json");
        //config.AddJsonFile(configPath);

        // Xml 文件
        string configPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"ConfigFiles\Autofac.config");
        config.AddXmlFile(configPath);

        var module = new ConfigurationModule(config.Build());
        builder.RegisterModule(module);

        builder.RegisterApiControllers(AppDomain.CurrentDomain.GetAssemblies());

        Container = builder.Build();
    }
}
```

添加对应的配置文件到 `ConfigFiles` 文件夹，`Autofac.config` 文件：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<autofac>
  <components name="0">
    <type>JohnSun.SOA.WebAPI.Service.SpecimenService, JohnSun.SOA.WebAPI.Service</type>
    <services name="0" type="JohnSun.SOA.WebAPI.Interface.ISpecimenService, JohnSun.SOA.WebAPI.Interface" />
  </components>
</autofac>
```

`Autofac.json` 文件：

```js
{
  "components": [
    {
      "type": "JohnSun.SOA.WebAPI.Service.SpecimenService, JohnSun.SOA.WebAPI.Service",
      "services": [
        {
          "type": "JohnSun.SOA.WebAPI.Interface.ISpecimenService, JohnSun.SOA.WebAPI.Interface"
        }
      ]
    }
  ]
}
```

最终修改我们的 `WebApiConfig.Register` 方法，将 `config.DependencyResolver` 修改为`AutofacWebApiDependencyResolver`，注意这个类型是在 `Autofac.WebApi2` 中的类型：

```csharp
config.DependencyResolver = new Autofac.Integration.WebApi.AutofacWebApiDependencyResolver(AutofacContainerFactory.Container);
```

这时，如果没有配置错误，就可以正常访问我们的控制器获取数据，需要注意的是：
+ ConfigFiles 文件夹下的配置文件属性“复制到输出目录”应该修改为：“如果较新则复制”。
+ 工厂类 `AutofacContainerFactory` 构建容器时一定要调用 `builder.RegisterApiControllers()` 方法，不同于 Unity 的原因可以参看后文 `AutofacWebApiDependencyResolver` 类型的反编译代码。

### AutofacWebApiDependencyResolver 类型

反编译 `nuget` 包 `Autofac.WebApi2.nupkg` 中的 `Autofac.Integration.WebApi.dll` 文件可以找到该类型：

`Autofac.Integration.WebApi.AutofacWebApiDependencyScope`：

```csharp
using Autofac;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web.Http.Dependencies;

/// <summary>
/// Autofac implementation of the <see cref="T:System.Web.Http.Dependencies.IDependencyScope" /> interface.
/// </summary>
public class AutofacWebApiDependencyScope : IDependencyScope, IDisposable
{
	private bool _disposed;

	private readonly ILifetimeScope _lifetimeScope;

	/// <summary>
	/// Gets the lifetime scope for the current dependency scope.
	/// </summary>
	public ILifetimeScope LifetimeScope => _lifetimeScope;

	/// <summary>
	/// Initializes a new instance of the <see cref="T:Autofac.Integration.WebApi.AutofacWebApiDependencyScope" /> class.
	/// </summary>
	/// <param name="lifetimeScope">The lifetime scope to resolve services from.</param>
	public AutofacWebApiDependencyScope(ILifetimeScope lifetimeScope)
	{
		if (lifetimeScope == null)
		{
			throw new ArgumentNullException("lifetimeScope");
		}
		_lifetimeScope = lifetimeScope;
	}

	/// <summary>
	/// Finalizes an instance of the <see cref="T:Autofac.Integration.WebApi.AutofacWebApiDependencyScope" /> class.
	/// </summary>
	~AutofacWebApiDependencyScope()
	{
		Dispose(disposing: false);
	}

	/// <summary>
	/// Try to get a service of the given type.
	/// </summary>
	/// <param name="serviceType">ControllerType of service to request.</param>
	/// <returns>An instance of the service, or null if the service is not found.</returns>
	public object GetService(Type serviceType)
	{
		return ResolutionExtensions.ResolveOptional((IComponentContext)(object)_lifetimeScope, serviceType);
	}

	/// <summary>
	/// Try to get a list of services of the given type.
	/// </summary>
	/// <param name="serviceType">ControllerType of services to request.</param>
	/// <returns>An enumeration (possibly empty) of the service.</returns>
	public IEnumerable<object> GetServices(Type serviceType)
	{
		if (!ResolutionExtensions.IsRegistered((IComponentContext)(object)_lifetimeScope, serviceType))
		{
			return Enumerable.Empty<object>();
		}
		Type enumerableServiceType = typeof(IEnumerable<>).MakeGenericType(serviceType);
		return (IEnumerable<object>)ResolutionExtensions.Resolve((IComponentContext)(object)_lifetimeScope, enumerableServiceType);
	}

	/// <summary>
	/// Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.
	/// </summary>
	public void Dispose()
	{
		Dispose(disposing: true);
		GC.SuppressFinalize(this);
	}

	/// <summary>
	/// Releases unmanaged and - optionally - managed resources.
	/// </summary>
	/// <param name="disposing">
	/// <see langword="true" /> to release both managed and unmanaged resources;
	/// <see langword="false" /> to release only unmanaged resources.
	/// </param>
	protected virtual void Dispose(bool disposing)
	{
		if (!_disposed)
		{
			if (disposing && _lifetimeScope != null)
			{
				((IDisposable)_lifetimeScope).Dispose();
			}
			_disposed = true;
		}
	}
}
```

`Autofac.Integration.WebApi.AutofacWebApiDependencyResolver`：

```csharp
using Autofac;
using Autofac.Core.Lifetime;
using Autofac.Integration.WebApi;
using System;
using System.Collections.Generic;
using System.Web.Http.Dependencies;

/// <summary>
/// Autofac implementation of the <see cref="T:System.Web.Http.Dependencies.IDependencyResolver" /> interface.
/// </summary>
public class AutofacWebApiDependencyResolver : IDependencyResolver, IDependencyScope, IDisposable
{
	private bool _disposed;

	private readonly ILifetimeScope _container;

	private readonly IDependencyScope _rootDependencyScope;

	private readonly Action<ContainerBuilder> _configurationAction;

	/// <summary>
	/// Gets the root container provided to the dependency resolver.
	/// </summary>
	public ILifetimeScope Container => _container;

	/// <summary>
	/// Initializes a new instance of the <see cref="T:Autofac.Integration.WebApi.AutofacWebApiDependencyResolver" /> class.
	/// </summary>
	/// <param name="container">The container that nested lifetime scopes will be create from.</param>
	/// <param name="configurationAction">A configuration action that will execute during lifetime scope creation.</param>
	public AutofacWebApiDependencyResolver(ILifetimeScope container, Action<ContainerBuilder> configurationAction)
		: this(container)
	{
		if (configurationAction == null)
		{
			throw new ArgumentNullException("configurationAction");
		}
		_configurationAction = configurationAction;
	}

	/// <summary>
	/// Initializes a new instance of the <see cref="T:Autofac.Integration.WebApi.AutofacWebApiDependencyResolver" /> class.
	/// </summary>
	/// <param name="container">The container that nested lifetime scopes will be create from.</param>
	public AutofacWebApiDependencyResolver(ILifetimeScope container)
	{
		if (container == null)
		{
			throw new ArgumentNullException("container");
		}
		_container = container;
		_rootDependencyScope = (IDependencyScope)(object)new AutofacWebApiDependencyScope(container);
	}

	/// <summary>
	/// Finalizes an instance of the <see cref="T:Autofac.Integration.WebApi.AutofacWebApiDependencyResolver" /> class.
	/// </summary>
	~AutofacWebApiDependencyResolver()
	{
		Dispose(disposing: false);
	}

	/// <summary>
	/// Try to get a service of the given type.
	/// </summary>
	/// <param name="serviceType">Type of service to request.</param>
	/// <returns>An instance of the service, or null if the service is not found.</returns>
	public virtual object GetService(Type serviceType)
	{
		return _rootDependencyScope.GetService(serviceType);
	}

	/// <summary>
	/// Try to get a list of services of the given type.
	/// </summary>
	/// <param name="serviceType">ControllerType of services to request.</param>
	/// <returns>An enumeration (possibly empty) of the service.</returns>
	public virtual IEnumerable<object> GetServices(Type serviceType)
	{
		return _rootDependencyScope.GetServices(serviceType);
	}

	/// <summary>
	/// Starts a resolution scope. Objects which are resolved in the given scope will belong to
	/// that scope, and when the scope is disposed, those objects are returned to the container.
	/// </summary>
	/// <returns>
	/// The dependency scope.
	/// </returns>
	public IDependencyScope BeginScope()
	{
		return (IDependencyScope)(object)new AutofacWebApiDependencyScope((_configurationAction == null) ? _container.BeginLifetimeScope(MatchingScopeLifetimeTags.RequestLifetimeScopeTag) : _container.BeginLifetimeScope(MatchingScopeLifetimeTags.RequestLifetimeScopeTag, _configurationAction));
	}

	/// <summary>
	/// Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.
	/// </summary>
	public void Dispose()
	{
		Dispose(disposing: true);
		GC.SuppressFinalize(this);
	}

	/// <summary>
	/// Releases unmanaged and - optionally - managed resources.
	/// </summary>
	/// <param name="disposing">
	/// <see langword="true" /> to release both managed and unmanaged resources;
	/// <see langword="false" /> to release only unmanaged resources.
	/// </param>
	protected virtual void Dispose(bool disposing)
	{
		if (!_disposed)
		{
			if (disposing && _rootDependencyScope != null)
			{
				((IDisposable)_rootDependencyScope).Dispose();
			}
			_disposed = true;
		}
	}
}
```


> 参考：<br>
> + GitHub - FreeSql：[2881099/FreeSql](https://github.com/2881099/FreeSql)<br>
> + GitHub - Unity: [unitycontainer/unity](https://github.com/unitycontainer/unity)<br>
> + GitHub - Autofac: [autofac/Autofac](https://github.com/autofac/Autofac)<br>
> + MSDN - ASP.NET Web API: [ASP.NET Web API](https://docs.microsoft.com/zh-cn/aspnet/web-api/)<br>
> + Autofac 中文文档: [欢迎来到 Autofac 中文文档!](https://autofaccn.readthedocs.io/zh/latest/index.html)

> 源码下载：<br>
> + Gitea：[JohnSun.SOA.WebAPI](https://git.hd2y.net/hd2y/JohnSun.SOA.WebAPI)<br>
> + 百度网盘：[链接](https://pan.baidu.com/s/17Nel1LWA8NFFRGD4iNiS-w)  提取码：60u5
