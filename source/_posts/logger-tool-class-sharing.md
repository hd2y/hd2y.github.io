---
title: "Logger 工具类分享"
date: "2019/12/25 20:15:22"
updated: "2019/12/25 20:16:17"
permalink: "logger-tool-class-sharing"
tags:
 - 日志
categories:
 - [开发, C#]
---

## 前言

稍大一些的系统设计，日志模块我们一般都会选用 `log4net` 或 `NLog`，开源并且功能齐全，配置功能很好用。

但是偶尔我们也会开发一些小工具，只是简单的输出一些日志，如果再选择这些开源项目的方案，不免显得有点“沉重”。

后面就针对这简单的使用场景，写了一个“简陋”的工具类 `Logger.cs` 满足一些简单场景的使用。

## 源代码

不说废话，直接看代码：

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Text;

namespace JohnSun.Utility
{
    /// <summary>
    /// 日志输出方式
    /// </summary>
    [Flags]
    public enum LogType
    {
        None = 0,
        File = 1,
        Console = 2,
        All = 3
    }

    /// <summary>
    /// 日志 严重性
    /// </summary>
    [Flags]
    public enum LogLevel
    {
        [Description("None")]
        None = 0,
        [Description("跟踪")]
        Trace = 1,
        [Description("调试")]
        Debug = 2,
        [Description("消息")]
        Info = 4,
        [Description("警告")]
        Warn = 8,
        [Description("错误")]
        Error = 16,
        [Description("致命")]
        Fatal = 32,
        [Description("All")]
        All = 63
    }

    /// <summary>
    /// 日志
    /// </summary>
    public class Logger
    {
        private static readonly Dictionary<LogLevel, string> _logLevelDescription = new Dictionary<LogLevel, string>();

        /// <summary>
        /// 日志等级的描述信息
        /// </summary>
        public static Dictionary<LogLevel, string> LogLevelDescription
        {
            get
            {
                if (_logLevelDescription.Count == 0)
                {
                    Array values = Enum.GetValues(typeof(LogLevel));
                    foreach (object value in values)
                    {
                        LogLevel level = (LogLevel)value;
                        FieldInfo info = typeof(LogLevel).GetField(level.ToString());
                        object[] attrs = info.GetCustomAttributes(false);
                        if (attrs.ToList().Find(attr => attr is DescriptionAttribute) is DescriptionAttribute attribute)
                        {
                            _logLevelDescription[level] = attribute.Description;
                        }
                        else
                        {
                            _logLevelDescription[level] = level.ToString();
                        }
                    }
                }
                return _logLevelDescription;
            }
        }

        /// <summary>
        /// 设置日志记录等级
        /// </summary>
        public static LogLevel LogLevel { get; set; } = LogLevel.All;

        /// <summary>
        /// 日志输出方式
        /// </summary>
        public static LogType LogType { get; set; } = LogType.File;

        private static void LogToConsole(string message, ConsoleColor textColor)
        {
            Console.OutputEncoding = Encoding.UTF8;
            Console.ForegroundColor = textColor;
            Console.WriteLine(message);
            Console.ResetColor();

            Console.OutputEncoding = Encoding.Default;
        }

        private static void LogToConsole(string message, ConsoleColor textColor, ConsoleColor backColor)
        {
            Console.OutputEncoding = Encoding.UTF8;
            Console.ForegroundColor = textColor;
            Console.BackgroundColor = backColor;
            Console.Write(message);
            Console.ResetColor();
            Console.WriteLine();

            Console.OutputEncoding = Encoding.Default;
        }

        static void LogToConsole(LogLevel v, string message)
        {
            switch (v)
            {
                case LogLevel.Trace:
                    LogToConsole(message, ConsoleColor.Black, ConsoleColor.DarkYellow);
                    break;
                case LogLevel.Debug:
                    LogToConsole(message, ConsoleColor.Cyan);
                    break;
                case LogLevel.Info:
                    LogToConsole(message, ConsoleColor.Yellow);
                    break;
                case LogLevel.Warn:
                    LogToConsole(message, ConsoleColor.Red);
                    break;
                case LogLevel.Error:
                    LogToConsole(message, ConsoleColor.Gray, ConsoleColor.Red);
                    break;
                case LogLevel.Fatal:
                    LogToConsole(message, ConsoleColor.Yellow, ConsoleColor.Red);
                    break;
            }
        }

        private static readonly object _sync = new object();

        static void LogToFile(LogLevel v, string message)
        {
            string directory = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Logs");
            string path = Path.Combine(directory, DateTime.Now.ToString("yyyy-MM-dd") + ".txt");
            lock (_sync)
            {
                if (!Directory.Exists(directory))
                {
                    Directory.CreateDirectory(directory);
                }
                File.AppendAllText(path, message, Encoding.UTF8);
            }
        }

        static void Log(LogLevel v, string message, Exception exception = null)
        {
            if ((v & LogLevel) == 0)
                return;
            if ((LogType & LogType.Console) != 0)
                LogToConsole(v, $"{DateTime.Now.ToString("HH:mm:ss")} {LogLevelDescription[v]}: {message}{(exception == null ? "" : $"\r\n异常信息：{exception.Message} {exception.InnerException?.Message}\r\n堆栈跟踪：{exception.StackTrace}")}");
            if ((LogType & LogType.File) != 0)
                LogToFile(v, $"{DateTime.Now.ToString("HH:mm:ss")} {LogLevelDescription[v]}: {message}{(exception == null ? "" : $"\r\n异常信息：{exception.Message} {exception.InnerException?.Message}\r\n堆栈跟踪：{exception.StackTrace}")}\r\n");
            LogEvent?.Invoke(v, message, exception);
        }

        /// <summary>
        /// 跟踪
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Trace(string message, Exception exception = null) => Log(LogLevel.Trace, message, exception);

        /// <summary>
        /// 调试
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Debug(string message, Exception exception = null) => Log(LogLevel.Debug, message, exception);

        /// <summary>
        /// 消息
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Info(string message, Exception exception = null) => Log(LogLevel.Info, message, exception);

        /// <summary>
        /// 警告
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Warn(string message, Exception exception = null) => Log(LogLevel.Warn, message, exception);

        /// <summary>
        /// 错误
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Error(string message, Exception exception = null) => Log(LogLevel.Error, message, exception);

        /// <summary>
        /// 致命
        /// </summary>
        /// <param name="message"></param>
        /// <param name="exception"></param>
        public static void Fatal(string message, Exception exception = null) => Log(LogLevel.Fatal, message, exception);

        /// <summary>
        /// 日志记录
        /// </summary>
        /// <param name="v">日志等级</param>
        /// <param name="message">消息</param>
        public delegate void LogAction(LogLevel v, string message, Exception exception = null);

        /// <summary>
        /// 日志记录事件
        /// </summary>
        public static event LogAction LogEvent;
    }
}
```

默认提供了两种日志的输出方式，可以通过标识枚举 `LogType` 控制，可以输出到文件或输出到控制台。

默认提供了六种日志严重程度，可以通过标识枚举 `LogLevel` 控制输出哪些类型的日志。

如果这些不能满足需求，例如有将日志输出到 `Windows Form`、`WPF`、数据库等的需求，可以通过注册 `LogEvent` 事件实现。

## 常规用法

默认提供了写文件与输出到控制台的用法，可以参考以下例子：

```csharp
// 设置日志输出方式
Logger.LogType = LogType.Console | LogType.File; // 等用于 LogType.All

// 设置输出日志的类型
Logger.LogLevel = LogLevel.All; // 这里设置所有日志类型都输出

// 常见输出
Logger.Trace($"这是一条{Logger.LogLevelDescription[LogLevel.Trace]}信息！");
Logger.Debug($"这是一条{Logger.LogLevelDescription[LogLevel.Debug]}信息！");
Logger.Info($"这是一条{Logger.LogLevelDescription[LogLevel.Info]}信息！");
Logger.Warn($"这是一条{Logger.LogLevelDescription[LogLevel.Warn]}信息！");
Logger.Error($"这是一条{Logger.LogLevelDescription[LogLevel.Error]}信息！");
Logger.Fatal($"这是一条{Logger.LogLevelDescription[LogLevel.Fatal]}信息！");

// 带异常信息的输出
Exception exception = new Exception("这是一条用于测试的异常信息");
Logger.Trace($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Trace]}信息！", exception);
Logger.Debug($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Debug]}信息！", exception);
Logger.Info($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Info]}信息！", exception);
Logger.Warn($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Warn]}信息！", exception);
Logger.Error($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Error]}信息！", exception);
Logger.Fatal($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Fatal]}信息！", exception);

Console.ReadKey();
```

输出到文件的效果，会根据日期创建文件夹：

![20191225192058](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225192058_1577275012804.png)

输出到控制台的效果，不同日志类型设置了不同的颜色用于区分：

![20191225192128](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225192128_1577275012795.png)

## 注册事件

偶尔会开发一些窗体应用程序，需要将日志输出到窗体控件，这里以输出到富文本框 `RichTextBox` 为例。

`WinForm` 用法：

创建一个 `RichTextBox` 控件用于输出：

```csharp
// 设置日志输出方式 这里不启用写文件和输出到控制台
Logger.LogType = LogType.None;

// 设置输出日志的类型 因为不用默认输出方式 这里可以不用设置
Logger.LogLevel = LogLevel.All;

// 注册日志输出事件 输出到富文本框
Logger.LogEvent += (level, msg, exc) =>
{
    this.Invoke(new Action(() => 
    {
        string text = $"{DateTime.Now.ToString("HH:mm:ss")} {Logger.LogLevelDescription[level]}: {msg}{(string.IsNullOrEmpty(exc?.Message) ? "" : $"\r\n异常信息：{exc.Message}")}{(string.IsNullOrEmpty(exc?.StackTrace) ? "" : $"\r\n堆栈跟踪：{exc.StackTrace}")}\r\n";
        switch (level)
        {
            case LogLevel.Trace:
                richTextBox1.SelectionColor = Color.DarkGray;
                break;
            case LogLevel.Debug:
                richTextBox1.SelectionColor = Color.SlateGray;
                break;
            case LogLevel.Info:
                richTextBox1.SelectionColor = Color.Green;
                break;
            case LogLevel.Warn:
                richTextBox1.SelectionColor = Color.OrangeRed;
                break;
            case LogLevel.Error:
                richTextBox1.SelectionColor = Color.Red;
                break;
            case LogLevel.Fatal:
                richTextBox1.SelectionColor = Color.Red;
                richTextBox1.SelectionBackColor = Color.Yellow;
                break;
            default:
                return;
        }
        richTextBox1.AppendText(text);
    }));
};

// 常见输出
Logger.Trace($"这是一条{Logger.LogLevelDescription[LogLevel.Trace]}信息！");
Logger.Debug($"这是一条{Logger.LogLevelDescription[LogLevel.Debug]}信息！");
Logger.Info($"这是一条{Logger.LogLevelDescription[LogLevel.Info]}信息！");
Logger.Warn($"这是一条{Logger.LogLevelDescription[LogLevel.Warn]}信息！");
Logger.Error($"这是一条{Logger.LogLevelDescription[LogLevel.Error]}信息！");
Logger.Fatal($"这是一条{Logger.LogLevelDescription[LogLevel.Fatal]}信息！");

// 带异常信息的输出
Exception exception = new Exception("这是一条用于测试的异常信息");
Logger.Trace($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Trace]}信息！", exception);
Logger.Debug($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Debug]}信息！", exception);
Logger.Info($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Info]}信息！", exception);
Logger.Warn($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Warn]}信息！", exception);
Logger.Error($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Error]}信息！", exception);
Logger.Fatal($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Fatal]}信息！", exception);
```

日志打印效果：

![20191225194312](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225194312_1577275012794.png)

`WPF` 用法：

```csharp
// 设置日志输出方式 这里不启用写文件和输出到控制台
Logger.LogType = LogType.None;

// 设置输出日志的类型 因为不用默认输出方式 这里可以不用设置
Logger.LogLevel = LogLevel.All;

// 注册日志输出事件 输出到富文本框
Logger.LogEvent += new Logger.LogAction((level, msg, exc) =>
{
    this.Dispatcher.Invoke((Action)delegate
    {
        Paragraph paragraph = new Paragraph();
        Run run = new Run() { Text = $"{DateTime.Now.ToString("HH:mm:ss")} {Logger.LogLevelDescription[level]}: {msg}{(string.IsNullOrEmpty(exc?.Message) ? "" : $"\r\n异常信息：{exc.Message}")}{(string.IsNullOrEmpty(exc?.StackTrace) ? "" : $"\r\n堆栈跟踪：{exc.StackTrace}")}" };

        switch (level)
        {
            case LogLevel.Trace:
                run.Foreground = Brushes.DarkGray;
                break;
            case LogLevel.Debug:
                run.Foreground = Brushes.SlateGray;
                break;
            case LogLevel.Info:
                run.Foreground = Brushes.Green;
                break;
            case LogLevel.Warn:
                run.Foreground = Brushes.OrangeRed;
                break;
            case LogLevel.Error:
                run.Foreground = Brushes.Red;
                break;
            case LogLevel.Fatal:
                run.Foreground = Brushes.Red;
                run.Background = Brushes.Yellow;
                break;
        }

        paragraph.Inlines.Add(run);

        // 这个新日志将会输出到富文本框的顶部
        // if (this.richTextBox1.Document.Blocks.FirstBlock == null)
        // {
        //     this.richTextBox1.Document.Blocks.Add(paragraph);
        // }
        // else
        // {
        //     this.richTextBox1.Document.Blocks.InsertBefore(this.richTextBox1.Document.Blocks.FirstBlock, paragraph);
        // }
        // this.richTextBox1.ScrollToHome();

        this.richTextBox1.Document.Blocks.Add(paragraph);
        this.richTextBox1.ScrollToEnd();
    });
});

// 常见输出
Logger.Trace($"这是一条{Logger.LogLevelDescription[LogLevel.Trace]}信息！");
Logger.Debug($"这是一条{Logger.LogLevelDescription[LogLevel.Debug]}信息！");
Logger.Info($"这是一条{Logger.LogLevelDescription[LogLevel.Info]}信息！");
Logger.Warn($"这是一条{Logger.LogLevelDescription[LogLevel.Warn]}信息！");
Logger.Error($"这是一条{Logger.LogLevelDescription[LogLevel.Error]}信息！");
Logger.Fatal($"这是一条{Logger.LogLevelDescription[LogLevel.Fatal]}信息！");

// 带异常信息的输出
Exception exception = new Exception("这是一条用于测试的异常信息");
Logger.Trace($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Trace]}信息！", exception);
Logger.Debug($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Debug]}信息！", exception);
Logger.Info($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Info]}信息！", exception);
Logger.Warn($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Warn]}信息！", exception);
Logger.Error($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Error]}信息！", exception);
Logger.Fatal($"这是一条带有异常信息的{Logger.LogLevelDescription[LogLevel.Fatal]}信息！", exception);
```

日志打印效果：

![20191225195340](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225195340_1577275015861.png)

如果想在事件中过滤日志，可以这样配置：

```csharp
// 设置日志输出方式 这里不启用写文件和输出到控制台
Logger.LogType = LogType.None;

// 使用该属性控制窗体中日志输出 输出 Trace Debug 级别以外的日志
Logger.LogLevel = LogLevel.All ^ (LogLevel.Trace | LogLevel.Debug);

// 注册日志输出事件 输出到富文本框
Logger.LogEvent += new Logger.LogAction((level, msg, exc) =>
{
    // 判断是否在允许输出的日志类型内
    if ((Logger.LogLevel & level) != level)
        return;

    this.Dispatcher.Invoke((Action)delegate
    {
        // 同 WPF 输出 代码省略
    });
});

// 调用日志输出 同 WPF 输出 代码省略
```

日志打印效果，可以看到被排除的跟踪与调试日志已经不再显示：

![20191225200543](https://hd2y.oss-cn-beijing.aliyuncs.com/20191225200543_1577275706738.png)

写入到数据库、发邮件等同样可以通过注册事件实现，因为很简单这里不再举例。
