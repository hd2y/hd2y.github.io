---
title: "Attribute特性简介"
date: "2019/04/06 13:24:00"
updated: "2019/07/11 10:27:38"
permalink: "attribute/"
tags:
 - Attribute
categories:
 - [开发, C#]
---

## Attribute 特性

### 特性简介

特性（`Attribute`）是用于在运行时传递程序中各种元素（比如类、方法、结构、枚举、组件等）的行为信息的声明性标签。您可以通过使用特性向程序添加声明性信息。一个声明性标签是通过放置在它所应用的元素前面的方括号（`[ ]`）来描述的。  

 特性（`Attribute`）用于添加元数据，如编译器指令和注释、描述、方法、类等其他信息。`.Net` 框架提供了两种类型的特性：预定义特性和自定义特性。  

**与注释的区别：** 注释仅在 `IDE` 环境中使用，特性可以影响编译器或程序运行，可以在不破坏类型封装的前提下，为对象增加额外的信息，执行额外的行为。

### 特性语法

```csharp
[attribute(positional_parameters, name_parameter = value, ...)]  
element
```

特性（`Attribute`）的名称和值是在方括号内规定的，放置在它所应用的元素之前。`positional_parameters` 规定必需的信息，`name_parameter` 规定可选的信息。

## 预定义特性

### `AttributeUsage`

预定义特性 `AttributeUsage` 描述了如何使用一个自定义特性类。它规定了特性可应用到的项目的类型。

```csharp
[AttributeUsage(validon, AllowMultiple=allowmultiple, Inherited=inherited)]
```

+ 参数 `validon` 规定特性可被放置的语言元素。它是枚举器 AttributeTargets 的值的组合。默认值是 `AttributeTargets.All`。  
+ 参数 `allowmultiple`（可选的）为该特性的 `AllowMultiple` 属性（`property`）提供一个布尔值。如果为 `true`，则该特性是多用的。默认值是 `false`（单用的）。  
+ 参数 `inherited`（可选的）为该特性的 `Inherited` 属性（`property`）提供一个布尔值。如果为 `true`，则该特性可被派生类继承。默认值是 `false`（不被继承）。

例如：

```csharp
[AttributeUsage(AttributeTargets.Class | ttributeTargets.Constructor | AttributeTargets.Field | AttributeTargets.Method | AttributeTargets.Property, AllowMultiple = true)]
```

### `Conditional`

这个预定义特性标记了一个条件方法，其执行依赖于它顶的预处理标识符。它会引起方法调用的条件编译，取决于指定的值，比如 `Debug` 或 `Trace`。例如，当调试代码时显示变量的值。

```csharp
[Conditional(conditionalSymbol)]
```

例如：

```csharp
[Conditional("DEBUG")]
```

**注意：** `Conditional` 特性的方式与 `#if...#endif`，`Conditional` 特性的方法是否生效是取决于调用方，而用 `#if` 方式是否生效是取决于方法定义所在的程序集。

```csharp
[Conditional("Conditional")]
private void TestOne()
{
    //需要设置项目 属性 -> 生成 -> 条件编译符号 为：Conditional
    Console.WriteLine("this is test function one.");
}
private void TestTwo()
{
    //需要设置项目 属性 -> 生成 -> 条件编译符号 为：Conditional
    #if Conditional
    Console.WriteLine("this is test function two.");
    #endif
}
```

**注意：**`Conditional` 特性可以多个，同时使用多个多个编译符号，多个之间是“或”的关系。

### Obsolete

这个预定义特性标记了不应被使用的程序实体。它可以让您通知编译器丢弃某个特定的目标元素。例如，当一个新方法被用在一个类中，但是您仍然想要保持类中的旧方法，您可以通过显示一个应该使用新方法，而不是旧方法的消息，来把它标记为 `obsolete`（过时的）。  

规定该特性的语法如下：

`[Obsolete(message)]` 或 `[Obsolete(message, iserror)]`  

+ 参数 `message`，是一个字符串，描述项目为什么过时的原因以及该替代使用什么。  
+ 参数 `iserror`，是一个布尔值。如果该值为 `true`，编译器应把该项目的使用当作一个错误。默认值是 `false`（编译器生成一个警告）。

## 自定义特性

`.Net` 框架允许创建自定义特性，用于存储声明性的信息，且可在运行时被检索。该信息根据设计标准和应用程序需要，可与任何目标元素相关。  

创建并使用自定义特性包含四个步骤：

+ 声明自定义特性；构建自定义特性；
+ 在目标程序元素上应用自定义特性；
+ 通过反射访问特性；
+ 最后一个步骤包含编写一个简单的程序来读取元数据以便查找各种符号。元数据是用于描述其他数据的数据和信息。该程序应使用反射来在运行时访问特性。

### 声明自定义特性  

一个新的自定义特性应派生自 System.Attribute 类。例如：

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Constructor | AttributeTargets.Field | AttributeTargets.Method | AttributeTargets.Property, AllowMultiple = true)]
public class DeveloperInfoAttribute : Attribute { }
```

### 构建自定义特性

每个特性必须至少有一个构造函数。必需的定位（`positional`）参数应通过构造函数传递。例如：

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Constructor | AttributeTargets.Field | AttributeTargets.Method | AttributeTargets.Property, AllowMultiple = true)]
public class DeveloperInfoAttribute : Attribute
{
    public DeveloperInfoAttribute(string name, string date)
    {
        DeveloperName = name;
        DeveloperDate = date;
    }
    public string DeveloperName { get; set; }
    public string DeveloperDate { get; set; }
    public string Describe { get; set; }
}
```

### 应用自定义特性

通过把特性放置在紧接着它的目标之前，来应用该特性。例如：

```csharp
[DeveloperInfo("John", "2018-06-23", Describe = "this is a test class.")]
public class Business
{
    [DeveloperInfo("John", "2018-06-23", Describe = "this is a test property.")]
    public int TestProperty { get; set; }
    [DeveloperInfo("John", "2018-06-23", Describe = "this is a test method.")]
    public void TestMethod() { }
}
```

### 通过反射访问特性

```csharp
{
    object[] attr = typeof(Business).GetCustomAttributes(false);
    for (int i = 0; i < attr.Length; i++)
    {
        if (attr[i] is DeveloperInfoAttribute)
        {
            DeveloperInfoAttribute developer = attr[i] as DeveloperInfoAttribute;
            Console.WriteLine($"Name:{developer.DeveloperName} Date:{developer.DeveloperDate} Describe:{developer.Describe}");
        }
    }
}
{
    MethodInfo[] method = typeof(Business).GetMethods();
    for (int i = 0; i < method.Length; i++)
    {
        object[] attr = method[i].GetCustomAttributes(false);
        for (int j = 0; j < attr.Length; j++)
        {
            if (attr[j] is DeveloperInfoAttribute)
            {
                DeveloperInfoAttribute developer = attr[j] as DeveloperInfoAttribute;
                Console.WriteLine($"Name:{developer.DeveloperName} Date:{developer.DeveloperDate} Describe:{developer.Describe}");
            }
        }
    }
}
{
    PropertyInfo[] property = typeof(Business).GetProperties();
    for (int i = 0; i < property.Length; i++)
    {
        object[] attr = property[i].GetCustomAttributes(false);
        for (int j = 0; j < attr.Length; j++)
        {
            if (attr[j] is DeveloperInfoAttribute)
            {
                DeveloperInfoAttribute developer = attr[j] as DeveloperInfoAttribute;
                Console.WriteLine($"Name:{developer.DeveloperName} Date:{developer.DeveloperDate} Describe:{developer.Describe}");
            }
        }
    }
}
```

### 注意事项

 + 自定义的 `Attribute` 必须直接或者间接继承 `System.Attribute`。
 + 这里有一个约定：所有自定义的特性名称都应该有个 `Attribute` 后缀。因为当你的 `Attribute` 施加到一个程序的元素上的时候，编译器先查找你的 `Attribute` 的定义，如果没有找到，那么它就会查找 特性名称 + `Attribute` 的定义。如果都没有找到，那么编译器就报错。
 + `Attribute` 可以关联的元素包括：程序集(`assembly`)、模块(`module`)、类型(`type`)、属性(`property`)、事件(`event`)、字段(`field`)、方法(`method`)、参数(`param`)、返回值(`return`)。
 + `AttributeTargets` 目标包括：
   - `All`：可以对任何应用程序元素应用属性；
   - `Assembly`：可以对程序集应用属性；
   - `Class`：可以对类应用属性；
   - `Constructor`：可以对构造函数应用属性；
   - `Delegate`：可以对委托应用属性；
   - `Enum`：可以对枚举应用属性；
   - `Event`：可以对事件应用属性；
   - `Field`：可以对字段应用属性；
   - `GenericParameter`：可以对泛型参数应用属性；
   - `Interface`：可以对接口应用属性；
   - `Method`：可以对方法应用属性；
   - `Module`：`Module` 指的是可移植的可执行文件（`.dll` 或 `.exe`），而非 `Visual Basic` 标准模块；
   - `Parameter`：可以对参数应用属性；
   - `Property`：可以对属性 (`Property`) 应用属性 (`Attribute`)；
   - `ReturnValue`：可以对返回值应用属性；
   - `Struct`：可以对结构应用属性，即值类型。
 + `AttributeUsageAttribute`中的3个属性（`Property`）说明：
   - `ValidOn`：该定位参数指定可在其上放置所指示的属性 (`Attribute`) 的程序元素。`AttributeTargets` 枚举数中列出了可在其上放置属性 (`Attribute`) 的所有可能元素的集合。可通过按位“或”运算组合多个 `AttributeTargets` 值，以获取所需的有效程序元素组合。
   - `AllowMultiple`：该命名参数指定能否为给定的程序元素多次指定所指示的属性。
   - `Inherited`：该命名参数指定所指示的属性能否由派生类和重写成员继承。
 + `Attribute` 检测方法：
   - `IsDefined`：如果至少有一个指定的 `Attribute` 派生类的实例与目标关联，就返回 `true`。这个方法效率很高，因为他不构造（反序列化）`Attribute` 类的任何实例。
   - `GetCustomAttributes`：返回一个数组，其中每个元素都是应用于目标的指定 `Attribute` 类的一个实例。
   - `GetCustomAttributesData`：获取 `CustomAttributesData` 特性信息。
