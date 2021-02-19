---
title: "C#中的反射"
date: "2019/04/01 14:03:00"
updated: "2019/07/11 10:21:28"
permalink: "reflector-in-csharp/"
tags:
 - 反射
categories:
 - [开发, C#]
---

## dll-IL-metadata-反射

`C#` 高级语言(人类语言) -> 编译器(编译) -> `DLL/EXE` (`metadata`(元数据)+`IL`(中间语言)) -> `CLR/JIT` -> 机器码(`CPU` 执行)

```mermaid
graph LR
A(C#高级语言) --> B(编译器)
B --> C(DLL / EXE)
C --> D(CLR / JIT)
D --> E(机器码)
```

## 反射

### 反射

反射是 `.net framework` 提供一个访问 `metadata` 的帮助类库，可以获取信息并使用。

### 使用

创建类型用于测试：

```csharp
public class Animal
{
    public string Name { get; set; }
    public void Action()
    {
        Console.WriteLine("eat");
    }
}
public class GenericClass<T1, T2>
{
    public void Show(T1 t)
    {
        Console.WriteLine($"type is {typeof(T1).Name} value equals {t}");
    }
}
public class Human : Animal
{
    public Human(string name, string country)
    {
        this.Country = country;
        this.Name = name;
    }
    public string Country { get; set; }
    public static void Hobby()
    {
        Console.WriteLine("sleep");
    }
    public void SayHi()
    {
        Console.WriteLine($"My name is {this.Name}, I'm come from {this.Country}!");
    }
}
```

测试：

```csharp
//引用命名空间
using System.Reflection;
//加载程序集
Assembly assembly1 = Assembly.Load("TestClassLibrary");//文件名
Assembly assembly2 = Assembly.LoadFile(System.IO.Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "TestClassLibrary.dll"));//文件路径
Assembly assembly3 = Assembly.LoadFrom(System.IO.Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "TestClassLibrary.dll"));//文件名或文件路径
//获取属性 模块 类型
assembly1.GetCustomAttributes();
assembly1.GetModules();
assembly1.GetTypes();
//获取指定类型
Type type1 = assembly1.GetType("TestClassLibrary.Human");//普通类
Type type2 = assembly1.GetType("TestClassLibrary.GenericClass`2");//泛型类
//泛型类或泛型方法使用前需要指定泛型类型
type2 = type2.MakeGenericType(typeof(string), typeof(string));//Method使用MakeGenericMethod方法
//实例化
object oSingleton1 = Activator.CreateInstance(type1, new object[] { "Kangkang", "China" });//有参数构造函数实例化
object oSingleton2 = Activator.CreateInstance(type2);//调用实例化方法入参需要传入实例化对象 静态方法可忽略
type1.GetMethod("Action").Invoke(oSingleton1, null);//实例化方法
type1.GetMethod("Hobby").Invoke(oSingleton1, null);//静态方法
type1.GetMethod("Hobby").Invoke(null, null);//静态方法
type1.GetMethod("SayHi").Invoke(oSingleton1, null);//实例化方法
type2.GetMethod("Show").Invoke(oSingleton2, new object[] { "test" });//泛型类有参数的方法
```

## 字段和属性值

```csharp
Assembly assembly = Assembly.Load("TestClassLibrary");
Type type = assembly.GetType("TestClassLibrary.Human");
object oSingleton = Activator.CreateInstance(type, new object[] { "Kangkang", "China" });
var fields = type.GetFields();//获取字段
foreach (var item in fields)
{
    Console.WriteLine($"name:{item.Name} value:{item.GetValue(oSingleton)}");
}
var properties = type.GetProperties();//获取属性
foreach (var item in properties)
{
    Console.WriteLine($"name:{item.Name} type:{item.GetValue(oSingleton)}");//GetValue() 获取属性或字段值
}
type.GetProperty("Name").SetValue(oSingleton, "Xiaoming");//SetValue() 设置属性或字段值
type.GetMethod("SayHi").Invoke(oSingleton, null);
```

## 反射的优点和局限

### 缺点

- 代码量较多书写麻烦
- 性能消耗较大
- 避开编译器检查

### 优点

- 动态可拓展
- 工厂模式
- 分析类库文件
- 访问不能访问的变量和属性破解第三方的代码
