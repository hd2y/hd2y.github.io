---
title: "C# 基础：float、double 与 decimal"
date: "2020/02/12 11:20:48"
updated: "2020/02/12 11:20:48"
permalink: "csharp-foundation-float-double-decimal"
tags:
 - decimal
categories:
 - [开发, C#]
---

## 前言

我想看到这个标题的时候，很多小伙伴都有点诧异，这么基础的内容，不就是“浮点数值类型”吗，为什么要写这么基础的入门知识？

当然，我想大家也都知道，想要高精度需要使用 `decimal`，如果存储的数值比较大要用 `double`，甚至可能都可以背下来三者的范围：

| C# 类型/关键字 | 大致范围 | 精度 | 大小 | `.NET` 类型 | 类型后缀 |
|:---------------|:---------|:-----|:-----|:------------|:---------|
| `float` | `±1.5 x 10⁻⁴⁵` 至 `±3.4 x 10³⁸` | 大约 `6-9` 位数字 | 4 个字节 | `System.Single` | `f` 或 `F` |
| `double` | `±5.0 × 10⁻³²⁴` 到 `±1.7 × 10³⁰⁸` | 大约 `15-17` 位数字 | 8 个字节 | `System.Double` | `d` 或 `D`，不带后缀的小数默认为 `double` |
| `decimal` | `±1.0 x 10⁻²⁸` 至 `±7.9228 x 10²⁸ ` | `28-29` 位 | 16 个字节 | `System.Decimal` | `m` 或 `M` |

## 二进制浮点型

因为 `decimal` 又被称之为“货币类型”，所以很多小伙伴在写代码时，只要不是用于财会系统，存储的内容和财富金钱没有关系，通通都使用 `double` 类型。

> 毕竟 `double` 是默认小数类型，写数字的时候还要带个后缀真的是太麻烦了，默认的就好。难道 `double` 类型提供的十几位的小数不够我们用吗？

那么我们执行以下代码：

```csharp
double num1 = 10000000000000000d + 1d;
Console.WriteLine($"double: 10000000000000000 + 1 = {num1}");
Console.WriteLine($"double: 10000000000000000 == 10000000000000001 is {10000000000000000d == 10000000000000001d}");
double num2 = 0.1d * 0.1d;
Console.WriteLine($"double: 0.1 × 0.1 == 0.01 is {num2 == 0.01}");

// 程序运行输出：
// double: 10000000000000000 + 1 = 10000000000000000
// double: 10000000000000000 == 10000000000000001 is True
// double: 0.1 × 0.1 == 0.01 is False
```

以上两个结果都不符合我们的预期，但是很多人在写程序的时候忽略了，或者没有预想到会出现这种错误。

主要原因有：
+ 错误的认为数据类型范围内的数字，都可以正确的被存储；
+ 将“精度”理解为该数据类型可以存储的小数位数；
+ 不知道或没有意识到 `float` 与 `double` 属于二进制浮点型，其用于表示十进制数字是近似值，不宜进行运算；

## 十进制浮点型

`decimal` 类型在其范围和精度内的十进制数完全准确。相反，用二进制数表示十进制数，则可能造成舍入错误。

`decimal` 被表示成 `±N×10ᵏ`。其中 N 是 96 位的正整数，而 `-28≤k≤0`。

而浮点数是 `±N×2ᵏ` 的任意数字。其中 N 是用固定数量位数（`float` 是 24，`double` 是 53）表示的正整数，k 是 `-149～+104`(float) 或者 `-1075～+970`(double)。

所以，我们将之前不符合预期的浮点数运算，修改为使用 `decimal` 类型：

```csharp
decimal num1 = 10000000000000000m + 1m;
Console.WriteLine($"decimal: 10000000000000000 + 1 = {num1}");
Console.WriteLine($"decimal: 10000000000000000 == 10000000000000001 is {10000000000000000m == 10000000000000001m}");
decimal num2 = 0.1m * 0.1m;
Console.WriteLine($"decimal: 0.1 × 0.1 == 0.01 is {num2 == 0.01m}");

// 程序运行输出：
// decimal: 10000000000000000 + 1 = 10000000000000001
// decimal: 10000000000000000 == 10000000000000001 is False
// decimal: 0.1 × 0.1 == 0.01 is True
```

将不会再出现“预料之外的运算错误”。

当时我们仍然需要注意几点：
+ `decimal` 类型是所有数据类型中速度最慢的（一般情况下，在我们业务中这个损耗可以忽略不计）；
+ 其数据范围内的数字仍然和其精度相关，不在精度范围内的数字仍然不能正确存储，并且要注意精度不是小数位数；

**注意：** `decimal` 在很多文章中不被认为是 `浮点数` 或 `浮点类型`，而被称作 `十进制数` 或 `十进制类型`，这里为了方便比较，所以统称 `float`、`double` 与 `decimal` 为浮点型。

> 参考：
> + MSDN：[浮点数值类型（C# 引用）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types)
> + MSDN：[类型 —— Decimal 类型](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/language-specification/types#the-decimal-type)

