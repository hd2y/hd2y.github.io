---
title: "匿名类型、扩展方法、var、dynamic"
date: "2019/04/02 17:33:00"
updated: "2019/07/11 10:42:58"
permalink: "anonymous-type-extension-method-var-dynamic/"
categories:
 - [开发, C#]
---

## 匿名类型

`C#`在`3.0`版本以后，允许使用`new`关键字直接创造对象，方便我们在临时使用一特定类型时，无需单独的创建一个类：

```csharp
object user = new
{
    Id = 1,
    Name = "Kangkang",
    Age = 12
};
//Console.WriteLine(user.Id);//无法获取 因为是object类型 该问题可以使用var关键字解决
```

## var 关键字

在方法范围内声明的变量可以具有隐式“类型” `var`。

隐式类型本地变量为强类型，就像用户已经自行声明该类型，但编译器决定类型一样(语法糖)。

### 声明现有类型

编译器会自行推断左侧变量的类型，如下代码编译器会识别为`int`类型：

```csharp
var i = 1;
```

### 声明匿名类型

创建的匿名类型我们并不知道编译器为我们生成的类型名称，但是我们可以使用`var`关键字让编译器自己根据编译的结果来推断：

```csharp
var user = new
{
    Id = 1,
    Name = "Kangkang",
    Age = 12
};
Console.WriteLine(user.Id);
```

### 特点

- 必须在定义时初始化，也就是必须是 `var s = "abcd"` 形式
- 一但初始化完成，就不能再给变量赋与初始化值类型不同的值
- var要求是局部变量
- 使用var定义变量和object不同，它在效率上和使用强类型方式定义变量完全一样

```csharp
{
    Console.WriteLine("**************** 匿名类型与var ****************");
    var user = new
    {
        Id = 1,
        Name = "Kangkang",
        Age = 12
    };
    //user.Name = "Xiaoming";//匿名类型属性只读
    Type type = user.GetType();
    var properties = type.GetProperties();
    foreach (var item in properties)
    {
        //只读属性反射也无法修改属性值
        //if (item.Name == "Name")
        //{
        //    item.SetValue(user, "Xiaoming", null);
        //}
        Console.WriteLine($"Property name is {item.Name}, value is {item.GetValue(user, null)}.");
    }
}
```

## 扩展方法

### 定义

静态类里的静态方法，第一个参数类型前加this关键字  

```csharp
public static class ExtensionClass
{
    public static Human CheckHuman(this Human human)
    {
        human = human ?? new Human("Kangkang","China");
        human.Name = string.IsNullOrEmpty(human.Name) ? "EmptyName" : human.Name;
        human.Country = string.IsNullOrEmpty(human.Country) ? "China" : human.Country;
        return human;
    }
}
```

如上，在不修改类型封装的前提下，给类型额外的扩展一个方法（密封类也可以），扩展的方法为实例方法。

### 使用

既然为实例方法，我们便可以像实例方法一样调用：

```csharp
{
    Console.WriteLine("**************** 扩展方法的使用 ****************");
    Human human = null;
    human.CheckHuman().SayHi();
}
```

**注意：**如果与实例方法相同，优先实例方法；不能滥用扩展方法，尤其是基类型。

## dynamic

### 简介

`dynamic` 类型是一种静态类型，但类型为 `dynamic` 的对象会跳过静态类型检查。 大多数情况下，该对象就像具有类型 `object` 一样。 在编译时，将假定类型化为 `dynamic` 的元素支持任何操作。 因此，您不必考虑对象是从 `COM` `API`、从动态语言（例如 `IronPython`）、从 `HTML` 文档对象模型 (DOM)、从反射还是从程序中的其他位置获取自己的值。 但是，如果代码无效，则在运行时会捕获到错误。

### 与 `var` 对比

一旦被编译，编译期会自动匹配var 变量的实际类型，并用实际类型来替换该变量的申明，这看上去就好像我们在编码的时候是用实际类型进行申明的。
而dynamic被编译后，实际是一个object类型，只不过编译器会对dynamic类型进行特殊处理，让它在编译期间不进行任何的类型检查，而是将类型检查放到了运行期。  

以 `var` 声明的变量，支持“智能感知”，因为 `Visual Studio` 能推断出 `var` 类型的实际类型，而以 `dynamic` 声明的变量却不支持“智能感知”，因为编译器对其运行期的类型一无所知。对 `dynamic` 变量使用“智能感知”，会提示“此操作将在运行时解析”。

### 简单使用

```csharp
dynamic para = new System.Dynamic.ExpandoObject();
para.name = "Kangkang";
para.country = "China";
para.age = 12;

// 可以直接调用
Console.WriteLine(para.name);

// 可以重新赋值其他类型
para.name = 007;
Console.WriteLine(para.name);

// 可以理解其为一个特殊的字典 我们可以使用字典的方式遍历
foreach (var item in para as IDictionary<string, object>)
{
    Console.WriteLine($"{item.Key} {item.Value}");
}

// 调用不存在的“属性”编译时不会报错 但是运行时会报错
// Microsoft.CSharp.RuntimeBinder.RuntimeBinderException:“'System.Dynamic.ExpandoObject' does not contain a definition for 'hobby'”
Console.WriteLine(para.hobby);
```
