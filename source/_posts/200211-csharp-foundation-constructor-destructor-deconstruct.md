---
title: "C# 基础：构造函数、析构函数、解构函数"
date: "2020/02/11 12:40:18"
updated: "2020/02/11 12:55:58"
permalink: "csharp-foundation-constructor-destructor-deconstruct/"
tags:
 - Note
 - 弃元
 - 析构函数
 - 解构函数
 - 元组
 - 构造函数
categories:
 - [开发, C#]
---

因为总感觉对于 C# 的知识掌握的不够透彻，所以最近在 Kindle 上购买了《C# 7.0 本质论》，复习一些基础知识，有了一些新发现。

这里开一个小专题，记录一些书上提到的，在日常编程中可能会比较少用到，但是很有意思的一些内容。

当然构造函数、析构函数，这些都是很熟悉的内容了，但是解构函数还是第一次听说，讲解这个用法的时候，顺便把这两个的提一下，算是凑凑字数吧。`_(:з)∠)_`

## 构造函数（Constructor）

> 构造函数定义了在类实例化过程中发生的事情。

类默认有一个无参数的构造函数，如果我们将该无参数构造函数重写为私有构造函数，该对象将无法直接实例化。

```csharp
public class TestClass1 { }

public class TestClass2
{
    private TestClass2() { }
}

class Program
{
    static void Main(string[] args)
    {
        // 调用默认的无参数构造函数
        TestClass1 class1 = new TestClass1();
        // 重写了唯一的无参数构造函数，并私有化，下面实例化代码将报错
        // TestClass2 class2 = new TestClass2();
    }
}
```

### 静态构造函数

> 静态构造函数用于初始化任何静态数据，或执行仅需执行一次的特定操作。将在创建第一个实例或引用任何静态成员之前自动调用静态构造函数。

静态构造函数可以说是实现“单例模式”最偷懒的一种做法，另外如果我们需要在类型构造之前做一些事情的话，可以使用静态构造函数，甚至可以重新加载程序集，具体可以参看[
C# 打包引用程序集与动态链接库](https://www.hd2y.net/archives/csharp-package-reference-assemblies-and-dynamic-link-libraries)这篇文章。

这里通过以下一段代码展示其调用顺序：

```csharp
public class TestClass
{
    static TestClass()
    {
        Console.WriteLine("调用了 TestClass 的静态构造函数");
    }
    public TestClass()
    {
        Console.WriteLine("调用了 TestClass 的实例构造函数");
    }
    public static void TestMethod1()
    {
        Console.WriteLine("调用了 TestClass 的静态方法 TestMethod1");
    }
    public void TestMethod2()
    {
        Console.WriteLine("调用了 TestClass 的实例方法 TestMethod2");
    }
}

class Program
{
    static void Main(string[] args)
    {
        TestClass.TestMethod1();
        TestClass @class = new TestClass();
        @class.TestMethod2();
        Console.ReadKey();
    }
}

// 程序运行输出：
// 调用了 TestClass 的静态构造函数
// 调用了 TestClass 的静态方法 TestMethod1
// 调用了 TestClass 的实例构造函数
// 调用了 TestClass 的实例方法 TestMethod2
```

可以看到静态构造函数最先被调用，实际上如果我们代码中出现了 `TestClass` 这一类型，无论是要调用其静态方法、静态字段、静态属性，还是想要实例化该对象，都将会先触发其静态函数的执行。

### 构造函数链

构造函数可以像普通的静态与实例方法一样重载，重载时我们可以通过关键字 `this` 调用当前对象的另一个构造函数，如果想要调用父类的构造函数，可以使用 `base` 关键字。

```csharp
public class Phone
{
    public Phone(string phoneNumber)
    {
        PhoneNumber = phoneNumber;
        Console.WriteLine($"调用 Phone 的构造函数：PhoneNumber={PhoneNumber}");
    }
    public string PhoneNumber { get; set; }
    public void Call(string phoneNumber)
    {
        Console.WriteLine($"{PhoneNumber} 向 {phoneNumber} 呼出了电话！");
    }
}

public class ApplePhone : Phone
{
    public ApplePhone(string phoneNumber, string systemVersion, int batteryCapacity) : base(phoneNumber)
    {
        SystemVersion = systemVersion;
        BatteryCapacity = batteryCapacity;
        Console.WriteLine($"调用 ApplePhone 的构造函数：PhoneNumber={PhoneNumber}, SystemVersion={SystemVersion}, BatteryCapacity={BatteryCapacity}");
    }
    public ApplePhone(string phoneNumber) : this(phoneNumber, "未知", -1)
    {
        Console.WriteLine($"调用 ApplePhone 的构造函数：PhoneNumber={PhoneNumber}");
    }
    public string SystemVersion { get; set; }
    public int BatteryCapacity { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        TestClass.TestMethod1();
        Phone phone = new ApplePhone("13888888888");
        phone.Call("119");
        Console.ReadKey();
    }
}

// 运行程序输出：
// 调用 Phone 的构造函数：PhoneNumber=13888888888
// 调用 ApplePhone 的构造函数：PhoneNumber=13888888888, SystemVersion=未知, BatteryCapacity=-1
// 调用 ApplePhone 的构造函数：PhoneNumber=13888888888
// 13888888888 向 119 呼出了电话！
```

可以看到，使用构造函数创建实例时，其会先执行链接的构造函数。

## 析构函数（Destructor）

虽然我们在很多文章中，了解到用于资源释放的这个特殊的函数为“析构函数”，但是实际上在 MSDN 以及 C# 的中文工具书中，其更多被称之为“终结器”。

> 终结器（也称为 析构函数）用于在垃圾回收器收集类实例时执行任何必要的最终清理操作。

实际上我们是无法控制何时调用析构函数，但是因其调用是在垃圾回收器决定，所以我们可以通过强制的垃圾回收来实现该析构函数的调用。

```csharp
public class Phone
{
    public Phone(string phoneNumber)
    {
        PhoneNumber = phoneNumber;
        Console.WriteLine($"调用 Phone 的构造函数：PhoneNumber={PhoneNumber}");
    }
    public string PhoneNumber { get; set; }
    public void Call(string phoneNumber)
    {
        Console.WriteLine($"{PhoneNumber} 向 {phoneNumber} 呼出了电话！");
    }
    ~Phone()
    {
        Console.WriteLine($"调用 Phone 的析构函数");
    }
}

public class ApplePhone : Phone
{
    public ApplePhone(string phoneNumber, string systemVersion, int batteryCapacity) : base(phoneNumber)
    {
        SystemVersion = systemVersion;
        BatteryCapacity = batteryCapacity;
        Console.WriteLine($"调用 ApplePhone 的构造函数：PhoneNumber={PhoneNumber}, SystemVersion={SystemVersion}, BatteryCapacity={BatteryCapacity}");
    }
    public ApplePhone(string phoneNumber) : this(phoneNumber, "未知", -1)
    {
        Console.WriteLine($"调用 ApplePhone 的构造函数：PhoneNumber={PhoneNumber}");
    }
    public string SystemVersion { get; set; }
    public int BatteryCapacity { get; set; }
    ~ApplePhone()
    {
        Console.WriteLine($"调用 ApplePhone 的析构函数");
    }
}

class Program
{
    static void Main(string[] args)
    {
        TakeCall();
        GC.Collect();
        Console.ReadKey();
    }

    public static void TakeCall()
    {
        Phone phone = new ApplePhone("13888888888");
        phone.Call("119");
    }
}

// 运行程序输出：
// 调用 Phone 的构造函数：PhoneNumber=13888888888
// 调用 ApplePhone 的构造函数：PhoneNumber=13888888888, SystemVersion=未知, BatteryCapacity=-1
// 调用 ApplePhone 的构造函数：PhoneNumber=13888888888
// 13888888888 向 119 呼出了电话！
// 调用 ApplePhone 的析构函数
// 调用 Phone 的析构函数
```

可以注意到，垃圾回收执行后，析构函数的执行顺序为先执行派生类的析构函数。

另外需要注意的是，如果我们直接在 `TakeCall` 方法中，调用 `GC.Collect()` 将不会触发该析构函数，因为垃圾回收只会回收那些无法再访问的代码块的对象。

因为这个特点，建议在使用比较稀缺的资源时，通过实现 `IDisposable` 接口来完成资源的释放，以保证这些资源能够及时的释放，提高程序的性能。

## 解构函数（Deconstruct）

如果熟悉元组的内容，并了解过“析构元组”的知识，应该对以下内容不会陌生：

```csharp
class Program
{
    static void Main(string[] args)
    {
        var (year, _, _, _, _, _) = GetDateTimeInfo("2020年2月11日");
        Console.WriteLine($"{year}年是{(DateTime.IsLeapYear(year) ? "闰年" : "平年")}");
        Console.ReadKey();
    }

    public static (int year, int month, int day, int hour, int minute, int second) GetDateTimeInfo(string datetime)
    {
        int year = 0, month = 1, day = 1, hour = 0, minute = 0, second = 0;
        var info = Regex.Split(datetime, "[^0-9]");
        if (info.Length > 0 && !string.IsNullOrEmpty(info[0])) year = int.Parse(info[0]);
        if (info.Length > 1 && !string.IsNullOrEmpty(info[1])) month = int.Parse(info[1]);
        if (info.Length > 2 && !string.IsNullOrEmpty(info[2])) day = int.Parse(info[2]);
        if (info.Length > 3 && !string.IsNullOrEmpty(info[3])) hour = int.Parse(info[3]);
        if (info.Length > 4 && !string.IsNullOrEmpty(info[4])) minute = int.Parse(info[4]);
        if (info.Length > 5 && !string.IsNullOrEmpty(info[5])) second = int.Parse(info[5]);

        return (year, month, day, hour, minute, second);
    }
}

// 运行程序输出：
// 2020年是闰年
```

> C# 提供内置的元组析构支持，可在单个操作中解包一个元组中的所有项。用于析构元组的常规语法与用于定义元组的语法相似：将要向其分配元素的变量放在赋值语句左侧的括号中。

> 对于非元组类型的解构，C# 不提供内置支持。但是，用户作为类、结构或接口的创建者，可通过实现一个或多个 Deconstruct 方法来析构该类型的实例。该方法返回 void，且要析构的每个值由方法签名中的 out 参数指示。

将以上代码中的 `GetDateTimeInfo` 方法，改造成 `DateTimeInfo` 结构体：

```csharp
class Program
{
    static void Main(string[] args)
    {
        // var (year, _, _, _, _, _) = GetDateTimeInfo("2020年2月11日");
        var (year, _, _) = new DateTimeInfo("2020年2月11日");
        Console.WriteLine($"{year}年是{(DateTime.IsLeapYear(year) ? "闰年" : "平年")}");
        Console.ReadKey();
    }
}

public struct DateTimeInfo
{
    public DateTimeInfo(string datetime)
    {
        Year = 0;
        Month = 1;
        Day = 1;
        Hour = 0;
        Minute = 0;
        Second = 0;
        var info = Regex.Split(datetime, "[^0-9]");
        if (info.Length > 0 && !string.IsNullOrEmpty(info[0])) Year = int.Parse(info[0]);
        if (info.Length > 1 && !string.IsNullOrEmpty(info[1])) Month = int.Parse(info[1]);
        if (info.Length > 2 && !string.IsNullOrEmpty(info[2])) Day = int.Parse(info[2]);
        if (info.Length > 3 && !string.IsNullOrEmpty(info[3])) Hour = int.Parse(info[3]);
        if (info.Length > 4 && !string.IsNullOrEmpty(info[4])) Minute = int.Parse(info[4]);
        if (info.Length > 5 && !string.IsNullOrEmpty(info[5])) Second = int.Parse(info[5]);
    }
    public int Year { get; }
    public int Month { get; }
    public int Day { get; }
    public int Hour { get; }
    public int Minute { get; }
    public int Second { get; }
    public void Deconstruct(out int year, out int month, out int day)
    {
        (year, month, day, _, _, _) = this;
    }
    public void Deconstruct(out int year, out int month, out int day, out int hour, out int minute, out int second)
    {
        year = Year;
        month = Month;
        day = Day;
        hour = Hour;
        minute = Minute;
        second = Second;
    }
}

// 运行程序输出：
// 2020年是闰年
```

以上涉及到“弃元”的内容，MSDN 上的描述为：析构元组时，通常只需要关注某些元素的值。 从 C# 7.0 开始，便可利用 C# 对弃元的支持，弃元是一种仅能写入的变量，且其值将被忽略 。 在赋值中，通过下划线字符 (_) 指定弃元。 可弃元任意数量的值，且均由单个弃元 `_` 表示。

弃元操作，不仅仅在以上析构元组与解构自定义对象时用到，当我们调用的方法存在 out 参数，但是该参数不需要时，我们也可以使用。

例如，我们需要判断一个字符串是否为 `decimal` 数字，可以调用 C# 自带的 TryParse 方法，这时我们不需要转换成功的结果，就可以舍弃：

```csharp
string strNum = "2e+8";
bool isNum = decimal.TryParse("2e+8", System.Globalization.NumberStyles.Any, null, out _);
Console.WriteLine($"{strNum} 判断是否为数字的结果：{isNum}");

// 运行程序输出：
// 2e+8 判断是否为数字的结果：True
```

但需要注意的是，这时我们需要携带 `out` 关键字，该关键字不能舍弃。


> 参考：
> + MSDN：[构造函数（C# 编程指南）](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/constructors)
> + MSDN：[终结器（C# 编程指南）](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/destructors)
> + MSDN：[析构元组和其他类型](https://docs.microsoft.com/zh-cn/dotnet/csharp/deconstruct)
> + MSDN：[C# 7.0 中的新增功能 - 弃元](https://docs.microsoft.com/zh-cn/dotnet/csharp/whats-new/csharp-7#discards)

