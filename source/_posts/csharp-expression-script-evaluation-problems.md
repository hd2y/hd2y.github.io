---
title: "C# 中表达式计算问题"
date: "2020/12/07 00:24:40"
updated: "2020/12/07 00:24:40"
permalink: "csharp-expression-script-evaluation-problems"
tags:
 - 正则
 - JavaScript
 - Math
 - Jint
categories:
 - [开发, C#]
---

问题：计算公式为 `[A/G] = [ALB] / ([TP] - [ALB])`，已知 `[TP]` 与 `[ALB]` 的值，求得 `[A/G]`。

这是一个很简单的四则运算问题，下面将演示如何在 C# 代码中如何简单、高效的进行求解。

## 一、解析算式

首先，需要考虑的是如何将已知的内容，替换到表达式中，其实想必大家和我一样，想到了正则的替换：

```csharp
/// <summary>
/// 解析表达式内容
/// </summary>
/// <param name="text">文本</param>
/// <param name="values">需要替换的值</param>
/// <returns>解析后的内容</returns>
public static string Parse(string text, Dictionary<string, string> values)
{
    var regex = new Regex(@"\[.+?\]");
    return regex.Replace(text, match =>
    {
        return values[match.Value];
    });
}
```

然后可以进行简单的测试：

```csharp
[Fact]
public void ParseSuccess()
{
    string text = "[ALB] / ([TP] - [ALB])";
    var values = new Dictionary<string, string>
    {
        {"[ALB]", "2.3"},
        {"[TP]", "5.4"},
    };
    var result = Expression.Parse(text, values);
    Assert.Equal("2.3 / (5.4 - 2.3)", result);
}
```

## 二、进行运算

内容替换完成后，需要解析以上字符串，获取运算结果。网上会有一些讲解，如何用算法解析，再进行运算。

但复杂的算法不是我们追求的，我们要的是在尽可能简单的前提下完成运算工作。

以下介绍的是一些易用，且代码非常简单的解决方案。

### 1. 使用 DataTable 的 Computer 方法

DataTable 大家应该非常熟悉，而 Computer 如果有使用过 DataTable 的聚合函数，应该也了解这个方法。

这里也是利用了 Computer 方法支持四则运算的特点：

```csharp
private static readonly DataTable _table = new DataTable();

public static string UseDataTableComputer(string exp)
{
    return _table.Compute(exp, "").ToString();
}
```

### 2. 使用数据库计算

数据库查询支持四则运算，这个是常识，所以同样的我们可以使用数据库查询获取四则运算结果。

如果采用这个方法，建议用 SQLite 的内存模式，不用担心数据库断连，导致查询出现问题。

```csharp
private static SQLiteConnection _conn;
private static SQLiteConnection _connection
{
    get
    {
        if (_conn == null)
        {
            _conn = new SQLiteConnection("Data Source=:memory:;Version=3;");
            _conn.Open();
        }
        return _conn;
    }
}

public static string UseDatabase(string exp)
{
    using (SQLiteCommand command = new SQLiteCommand($"select {exp}", _connection))
    {
        return command.ExecuteScalar().ToString();
    }
}
```

> 以上使用 NuGet 引用引用了 `System.Data.SQLite.Core`。

### 3. 使用 JavaScript 解析

说到 JavaScript 运行代码段，肯定想到了 `eval` 这个方法，而在 C# 中其实有很多开源的 JavaScript 解释器，可以运行 JavaScript 代码。

下面就是使用 Jint 实现的一个解决方案：

```csharp
private static readonly JsValue _jsValue = new Engine().Execute(@"function calc(exp) { return eval(exp); }").GetValue("calc");

public static string UseJavaScript(string exp)
{
    return _jsValue.Invoke(exp).ToObject().ToString();
}
```

> 以上使用 NuGet 引用引用了 `Jint`。

### 4. 使用 `Math.NET` 计算

如果有用 JavaScript 处理过一些复杂的数学问题，例如代数求解，应该知道 `algebra.js` 与 `math.js` 这两个库。

C# 同样有一个工具包，可以处理复杂的数学问题，就是 [Math.NET Team (github.com)](https://github.com/mathnet)。

而四则运算这样简单的问题自然不在话下，甚至感觉有点大材小用：

```csharp
public static string UseMathNet(string exp)
{
    var expression = SymbolicExpression.Parse(exp);
    return expression.Evaluate(null).RealValue.ToString();
}
```

> 以上使用 NuGet 引用引用了 `MathNet.Symbolics`。

### 5. 使用 Liquid 计算

`liquid` 是一种开源的模板语言，类似 `Razor`，但是相对来说更简单、高效。

其支持四则运算，所以如果模板内容就是一个四则运算，输出的内容就是我们需要的结果：

```csharp
public static string UseLiquid(string exp)
{
    return Template.Parse($"{{{{{exp}}}}}").Render();
}
```

> 以上使用 NuGet 引用引用了 `Scriban`。

**注意：** 以上使用的 `Jint`、`Math.NET`、`Scriban` 均在 GitHub 开源，如何使用也可以到 GitHub 了解，其实这里仅仅使用了它们最简单的一个功能，强烈推荐都了解一下，它们的功能在实际项目中还可以有更广泛的应用。

## 三、测试

xUnit 测试文件：

```csharp
using System;
using System.Collections.Generic;
using Xunit;

public class CalculationShould
{
    [Theory]
    [MemberData(nameof(TestData.List), MemberType = typeof(TestData))]
    public void UseDataTableComputerPass(string exp, double result)
    {
        var value = Convert.ToDouble(Calculation.UseDataTableComputer(exp));
        value = Math.Round(value, 2, MidpointRounding.AwayFromZero);
        Assert.Equal(result, value);
    }

    [Theory]
    [MemberData(nameof(TestData.List), MemberType = typeof(TestData))]
    public void UseDatabasePass(string exp, double result)
    {
        var value = Convert.ToDouble(Calculation.UseDatabase(exp));
        value = Math.Round(value, 2, MidpointRounding.AwayFromZero);
        Assert.Equal(result, value);
    }

    [Theory]
    [MemberData(nameof(TestData.List), MemberType = typeof(TestData))]
    public void UseJavaScriptPass(string exp, double result)
    {
        var value = Convert.ToDouble(Calculation.UseJavaScript(exp));
        value = Math.Round(value, 2, MidpointRounding.AwayFromZero);
        Assert.Equal(result, value);
    }

    [Theory]
    [MemberData(nameof(TestData.List), MemberType = typeof(TestData))]
    public void UseLiquidPass(string exp, double result)
    {
        var value = Convert.ToDouble(Calculation.UseLiquid(exp));
        value = Math.Round(value, 2, MidpointRounding.AwayFromZero);
        Assert.Equal(result, value);
    }

    [Theory]
    [MemberData(nameof(TestData.List), MemberType = typeof(TestData))]
    public void UseMathNetPass(string exp, double result)
    {
        var value = Convert.ToDouble(Calculation.UseMathNet(exp));
        value = Math.Round(value, 2, MidpointRounding.AwayFromZero);
        Assert.Equal(result, value);
    }

    public class TestData
    {
        private static readonly List<object[]> Data = new List<object[]>
        {
            new object[] {"1 + 2", 3.0},
            new object[] {"10.0 / (5 - 2)", 3.33},
            new object[] {"3.14", 3.14},
            new object[] {"1 + 2 * 3 / 4.0", 2.5}
        };

        public static IEnumerable<object[]> List => Data;
    }
}
```

测试结果：

![calc-test](https://www.hd2y.net/upload/2020/12/calc-test-2a7b61d2a7ae47bbab8a1d6dcaa7e1bc.png)

## 四、优缺点

谈优缺点之前，首先看一下性能，测试代码如下：

```csharp
class Program
{
    static void Main(string[] args)
    {
        BenchmarkRunner.Run<CalculationBenchmark>(new DebugInProcessConfig());

        Console.ReadKey();
    }
}

[RPlotExporter]
public class CalculationBenchmark
{
    private List<string> _exps = new List<string> { "1 + 2", "10.0 / (5 - 2)", "3.14", "1 + 2 * 3 / 4.0" };

    [Benchmark]
    public List<string> UseDatabase()
    {
        return _exps.Select(exp => Calculation.UseDatabase(exp)).ToList();
    }

    [Benchmark]
    public List<string> UseDataTableComputer()
    {
        return _exps.Select(exp => Calculation.UseDataTableComputer(exp)).ToList();
    }

    [Benchmark]
    public List<string> UseJavaScript()
    {
        return _exps.Select(exp => Calculation.UseJavaScript(exp)).ToList();
    }

    [Benchmark]
    public List<string> UseLiquid()
    {
        return _exps.Select(exp => Calculation.UseLiquid(exp)).ToList();
    }

    [Benchmark]
    public List<string> UseMathNet()
    {
        return _exps.Select(exp => Calculation.UseMathNet(exp)).ToList();
    }
}
```

测试结果如下：

|               Method |       Mean |     Error |    StdDev |
|--------------------- |-----------:|----------:|----------:|
|          UseDatabase |  20.287 us | 0.3213 us | 0.2848 us |
| UseDataTableComputer |   4.210 us | 0.0643 us | 0.1093 us |
|        UseJavaScript |  20.248 us | 0.1872 us | 0.1659 us |
|            UseLiquid | 110.420 us | 1.7733 us | 1.8974 us |
|           UseMathNet |  12.116 us | 0.1938 us | 0.1813 us |

可以看到在性能测试中，`DataTable` 的 `Computer` 方法表现最好，但是，更推荐用 `UseJavaScript` 与 `UseMathNet`。

原因如下：

1. `UseDataTableComputer` 只支持四则运算，如果遇到指数、对数等情况则无法进行运算。
2. `UseDatabase`  需要依赖数据库，另外如果遇到整数运算，例如 `10 / 3`，默认是整除，暂时没有找到好的解决方案处理这个问题，这也是为什么上面的例子中每个算式都有浮点数。
3. `UseLiquid` 应用中感觉和 `UseJavaScript` 很相似，所以在 `UseJavaScript` 表现更好的前提下，更推荐用后者。
4. `UseMathNet` 有更好的性能，并且支持在算式中使用一些常量，例如 `e`、`π`，运算符也支持指数符号 `^`，但是只能处理与数学相关的问题（不支持位运算）。
5. `UseJavaScript` 更灵活而且更方便测试，有更好的扩展性，但是复杂的运算需要使用函数来处理，不如 `UseMathNet`  易读。

> 备注：目前我是用 JavaScript，因为公式不是由客户维护，所以不需要开发编辑器，所有计算公式纯粹靠手撸。  
> 目前碰到过最复杂的一个计算公式是：`100 * Math.sqrt(-12 + 2.38 * Math.LN2([T4]) + 0.0626 * Math.LN2([CA125Ⅱ]))/(1 + Math.sqrt(-12 + 2.38 * Math.LN2([T4]) + 0.0626 * Math.LN2([CA125Ⅱ])))`。  
> 当然应用中还涉及一些其他的问题，比如 `[GLB] = [TP] / [ALB]`，`[A/G] = [ALB] / [GLB]`，需要用到递归进行处理；计算项因为浮点数精度的问题，所有结果都要指定小数位数等。这个后面有时间再慢慢补。

