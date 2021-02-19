---
title: "利用Jint在C#中运行JS脚本并实现简单计算器"
date: "2019/05/06 17:45:00"
updated: "2019/07/10 18:42:37"
permalink: "using-jint-to-run-js-in-csharp-and-implement-simple-calculator/"
tags:
 - Jint
 - JavaScript
categories:
 - [开发, C#]
---

## 关于Jint

Jint是一个开源的JS脚本引擎，可以让我们在dotnet平台运行js代码，这使我们可以通过这一特性处理很多工作。

关于Jint的更多信息和用例可以参考：[https://github.com/sebastienros/jint](https://github.com/sebastienros/jint)

## Jint的用途

### 数学运算

在日常工作中存在一个需求：用户自己定义表达式，系统根据表达式替换表达式中项目的数值，计算生成另外一个项目的结果。

例如：`[GLO] = [TP] - [ALB]`，已知`[TP]`与`[ALB]`的结果，需要我们求出`[GLO]`的结果。

这个表达式初看其实很简单，但是实际业务中，可能存在多层级的嵌套甚至次幂/开根号/自然对数等的运算，这样如果通过程序来解析表达式进行运算工作量比较大。

过去程序设计是在Web端触发特定事件后通过正则替换表达式中参与运算项目为实际数值，使用`eval`函数执行表达式，获取结果并保存到数据库。但是随着系统不断升级，仅仅依靠前端来更新结果，在某些情况下不够人性化而且不能满足需求。

检索关于算术运算，网上普遍推荐的方案是：
1. 使用数据库拼接SQL语句；
2. 通过C#反射实现类似js的`eval`函数效果；

但是这些方案的缺点都很明显：数据库方案需要保持数据库连接，消耗数据库性能，并且不能使用特殊的函数运算比如对数等求解；C#的反射数值中如果存在整数运算，则会取整，需要手动替换数值增加浮点型数字的标记比较麻烦。

几次碰壁以后，就在想，是不是可以在C#中运行js代码？在网上检索相关资料果然有戏。不过首先尝试的是Microsoft.JScript库，不过这个已经被微软标记为`Obsolete`，并且对于一些语法支持的不完善，所以这里不推荐。

首先我们来实现一个简单的计算器：

```csharp
/// <summary>
/// 获取算术式结果
/// </summary>
/// <param name="input">原始算术式</param>
public static void Calculator(string input = null)
{
    do
    {
        //算式为空提示输入算式
        if (string.IsNullOrEmpty(input))
        {
            Console.WriteLine("请输入一个算术式(退出输入Q)：");
            input = Console.ReadLine();
        }

        //非退出该方法
        if (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase))
        {
            try
            {
                //使用Jint执行算式
                Engine engine = new Engine();
                object result = engine.Execute(input).GetCompletionValue().ToObject();
                Console.WriteLine($"算术式：{input} 输出：{result}");
            }
            catch (Exception exc)
            {
                Console.WriteLine($"算术式：{input} 运算出现异常：{exc.Message}");
            }

            //赋值空让用户循环输入
            input = null;
        }
    } while (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase));
}
```

测试一下效果，已经满足了我们的需求：

![jint1](https://hd2y.oss-cn-beijing.aliyuncs.com/jint1_1562755045285.png)

> 需要注意的是，JavaScript中对于浮点数的支持精度较低，建议运算后设定小数位数进行转换。

通过以上，我们进一步实现上面我们业务中需要的功能，下面是一个简单的实现：

```csharp
/// <summary>
/// 获取表达式结果
/// </summary>
/// <param name="input">原始表达式</param>
public static void GetResultFromExpression(string input = null)
{
    string oldInput;
    Regex regex = new Regex(@"\[([A-Za-z0-9]+)\]");//表达式中项目由字母和数字组成
    do
    {
        //表达式为空提示输入表达式
        if (string.IsNullOrEmpty(input))
        {
            Console.WriteLine("请输入表达式(退出输入Q)：");
            input = Console.ReadLine();
        }

        //用于输出原始表达式
        oldInput = input;

        //用于判断表达式项目是否已经替换
        List<string> items = new List<string>();

        //非退出该方法
        if (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase))
        {
            //匹配表达式中项目并替换
            MatchCollection matches = regex.Matches(input);
            foreach (Match match in matches)
            {
                //已经替换的跳过
                if (items.Contains(match.Value))
                {
                    continue;
                }
                items.Add(match.Value);
                Console.WriteLine($"请为项目：{match.Value} 赋值：");
                string item = Console.ReadLine();
                input = input.Replace(match.Value, item);
            }
            try
            {
                //使用Jint执行表达式
                Engine engine = new Engine();
                object result = engine.Execute(input).GetCompletionValue().ToObject();
                Console.WriteLine($"表达式：{oldInput} 算术式：{input} 输出：{result}");
            }
            catch (Exception exc)
            {
                Console.WriteLine($"表达式：{oldInput} 算术式：{input} 运算出现异常：{exc.Message}");
            }

            //赋值空让用户循环输入
            input = null;
            oldInput = null;
        }
    } while (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase));
}
```

测试效果如下：

![jint2](https://hd2y.oss-cn-beijing.aliyuncs.com/jint2_1562755045287.png)

### 自定义脚本

系统中某些位置，可能需要比较大的自由度去处理业务数据，例如文档的输出，我们能从数据库中读取到结果是数值型，但是实施人员希望我们能处理成文本型，并且设定的条件较多，自由度很大。

当然最直接的方案就是我们将这些集成到业务代码中，友好一点的设置一个参数表，由工程人员维护。但是当这里的业务逻辑复杂或者存在很多的不定因素，这里的维护更新就变得困难。

而这个时候，我们可以通过写一些js脚本，来处理复杂逻辑的问题，例如文档中我们配置了一下内容：

`[ItemName]的结果为[ResultValue]，临床意义为：[Descript]。`

当`[ItemName]`显示的内容是“HIV”时，`[ResultValue]`的值小于1判定`[Descript]`为“阴性”，否则为待复查。若`[ItemName]`不是“HIV”时，`[ResultValue]`的值小于1判定`[Descript]`为“阴性”，否则为阳性。如果`[ResultValue]`中存在非数字部分，则只取数字部分内容。

这里如果用业务代码或者参数来写，可能复杂一些，因为像这样的业务逻辑还有很多，而且可能随时需要调整，但是我们可以将以上的内容稍稍进行调整，我们定义`$#*[JS代码]*#`来包括一段代码，将以上的内容进行调整：

```
[ItemName]的结果为[ResultValue]，临床意义为：$#*
var n = '[ItemName]';
var r = '[ResultValue]'.replace(/[^0-9\+\-\.]/, '');
var d = '';
if (r !== '' && !isNaN(r)) {
    if (n === 'HIV' && r >= 1) {
        d = '待复查';
    }
    else if (r >= 1) {
        d = '阳性';
    } else {
        d = '阴性';
    }
}
d;*#。
```

那么我们写一个测试用例实现以下对以上内容的解析：

```csharp
/// <summary>
/// 执行自定义脚本获取内容
/// </summary>
/// <param name="input">脚本</param>
public static void ExecuteCustomScript(string input = null)
{
    string oldInput;
    Regex regex1 = new Regex(@"\[([A-Za-z0-9]+)\]");//脚本中项目由字母和数字组成
    Regex regex2 = new Regex(@"\$#\*(.+)\*#", RegexOptions.Singleline);//匹配自定义脚本
    do
    {
        //脚本为空提示输入
        if (string.IsNullOrEmpty(input))
        {
            Console.WriteLine("请输入脚本(退出输入Q)：");
            input = Console.ReadLine();
        }

        //用于输出原始脚本
        oldInput = input;

        //用于判断脚本项目是否已经替换
        List<string> items = new List<string>();

        //用于判断脚本内容是否已经替换
        List<string> scripts = new List<string>();

        //非退出该方法
        if (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase))
        {
            //匹配脚本中项目并替换
            {
                MatchCollection matches = regex1.Matches(input);
                foreach (Match match in matches)
                {
                    //已经替换的跳过
                    if (items.Contains(match.Value))
                    {
                        continue;
                    }
                    items.Add(match.Value);
                    Console.WriteLine($"请为项目：{match.Value} 赋值：");
                    string item = Console.ReadLine();
                    input = input.Replace(match.Value, item);
                }
            }

            //匹配内容中自定义脚本并执行替换
            {
                MatchCollection matches = regex2.Matches(input);
                foreach (Match match in matches)
                {
                    //已经替换的跳过
                    if (scripts.Contains(match.Value))
                    {
                        continue;
                    }

                    scripts.Add(match.Value);

                    try
                    {
                        //使用Jint执行内容中自定义脚本
                        Engine engine = new Engine();
                        object result = engine.Execute(match.Groups[1].Value).GetCompletionValue().ToObject();
                        input = input.Replace(match.Value, result.ToString());
                    }
                    catch (Exception exc)
                    {
                        Console.WriteLine($"脚本：{match.Groups[1].Value}\r\n执行出现异常：{exc.Message}");
                    }
                }
            }

            Console.WriteLine($"**********************\r\n原始脚本：{oldInput}\r\n输出内容：{input}");

            //还原公式让用户重新赋值
            input = oldInput;
        }
    } while (!string.Equals(input, "Q", StringComparison.OrdinalIgnoreCase));
}
```

调用：
```csharp
JintTest.ExecuteCustomScript(@"[ItemName]的结果为[ResultValue]，临床意义为：$#*
var n = '[ItemName]';
var r = '[ResultValue]'.replace(/[^0-9\+\-\.]/, '');
var d = '';
if (r !== '' && !isNaN(r)) {
    if (n === 'HIV' && r >= 1) {
        d = '待复查';
    }
    else if (r >= 1) {
        d = '阳性';
    } else {
        d = '阴性';
    }
}
d;*#。");
```

测试效果如下：

![jint3](https://hd2y.oss-cn-beijing.aliyuncs.com/jint3_1562755045286.png)
