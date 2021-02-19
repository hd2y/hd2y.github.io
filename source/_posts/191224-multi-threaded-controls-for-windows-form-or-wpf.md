---
title: "多线程操作 Windows Form 或 WPF 的控件"
date: "2019/12/24 19:28:46"
updated: "2019/12/25 18:36:55"
permalink: "multi-threaded-controls-for-windows-form-or-wpf/"
tags:
 - UI
 - 多线程 
 - WinForm
 - WPF
categories:
 - [开发, C#]
---

## 前言

写代码就是这样，不是天天写的代码，很长一段时间不用，下次再用大概率是已经忘记了，不知道怎么处理。

虽然可以从搜索引擎中找到答案，但是肯定没有自己整理的看着舒服，所以后面会把 C/S 开发时遇到的一些小坑整理以下。

## 多线程操作 UI 组件

今天写一个 HTTP 监听的小工具，其中需要输出一个日志窗，其中会由于日志类型的不同，调整 `RichTextBox` 的 `SelectionColor` 和 `SelectionBackColor` 以更新日志窗体的前景色或背景色。

当然可能涉及多线程中输出日志，所以更新组件内容自然也用了 `Invoke` 与 `BeginInvoke` 方法。

因为写日志时会将之前设置的前景色以及背景色取出来，等输出完成后将原本的前景色或背景色设置回去，所以调用方法以后就在想，这个过程需不需要加锁。

因为万一在输出 `Error` 日志时，将颜色设置成红色了，那另外一个线程需要写 `Info` 类型的日志，那前景色与背景色岂不是就乱掉了。

然后就简单写了个段，测试了一下：

```csharp
for (int i = 0; i < 20; i++)
{
    int num = i;
    
    Task.Factory.StartNew(() =>
    {
        richTextBox1.BeginInvoke(new Action(() => 
        {
            richTextBox1.SelectionColor = Color.Red;
            richTextBox1.AppendText($"{DateTime.Now:HH:mm:ss.fff} {Thread.CurrentThread.ManagedThreadId:00000} 任务一开始执行 {num}\r\n");
            Thread.Sleep(10);
            richTextBox1.SelectionColor = Color.Red;
            richTextBox1.AppendText($"{DateTime.Now:HH:mm:ss.fff} {Thread.CurrentThread.ManagedThreadId:00000} 任务一执行结束 {num}\r\n");
        }));
    });
    Task.Factory.StartNew(() =>
    {
        richTextBox1.BeginInvoke(new Action(() =>
        {
            richTextBox1.AppendText($"{DateTime.Now:HH:mm:ss.fff} {Thread.CurrentThread.ManagedThreadId:00000} Task two begains {num}\r\n");
            Thread.Sleep(1);
            richTextBox1.AppendText($"{DateTime.Now:HH:mm:ss.fff} {Thread.CurrentThread.ManagedThreadId:00000} Task two is over {num}\r\n");
        }));
    });
}
```

执行效果：

![20191224201545](https://hd2y.oss-cn-beijing.aliyuncs.com/20191224201545_1577190005074.png)

可以看到输出的线程 ID 全部是主线程的 ID：`00001`，所以这时候才想起来，无论使用同步方法 `Invoke`，还是异步方法 `BeginInvoke`，都仅仅知识针对 UI 主线程外的其他线程，实际上调用以后的委托只有一个 UI 线程来负责执行。

否则怎么可能避免 `线程间操作无效: 从不是创建控件的线程访问它。`，所以就是杞人忧天了。

### Windows Form

`WinForm` 的控件基类型 `Control` 提供了 `Invoke` 方法与 `BeginInvoke`，多线程中如果需要操作 UI 组件（赋值操作），可以使用这两个方法。

区别是 `Invoke` 是同步方法，当前线程会等待 UI 主线程将该委托执行完成，而 `BeginInvoke` 是异步的则不会等待 UI 主线程的操作。

传递的委托我常常使用以下几种写法，都没有问题：

```csharp
Task.Factory.StartNew(() => 
{
    // 多线程中可以从控件中取值
    Color color = richTextBox1.SelectionColor;
    // 但是不可以在多线程中为控件赋值
    // richTextBox1.SelectionColor = color;

    // BeginInvoke 相对于当前线程异步执行，不会等待 UI 主线程的更新
    this.BeginInvoke((Action)delegate 
    {
        richTextBox1.SelectionColor = color;
    });

    // Invoke 相对于当前线程同步执行，会等待 UI 主线程将委托执行完成
    this.Invoke(new EventHandler(delegate 
    {
        richTextBox1.SelectionColor = color;
    }));

    // 传递一个委托的方式多种多样，这时我常用的几种写法
    richTextBox1.BeginInvoke(new Action(() =>
    {
        richTextBox1.SelectionColor = color;
    }));
});
```

> 注意：我们使用任何组件来执行 `Invoke` 或 `BeginInvoke` 都是一样的，例如上面这个例子，无论是使用 `this` 指代的当前窗体，还是这个窗体的富文本框 `richTextBox1`，最终目的和效果都是在 UI 主线程中执行代码。

### WPF

同 `WinForm` 一样，WPF 中主线程维护的 UI 子线程也不能直接更新，但是不同的是 WPF 是通过 `Dispatcher` 处理，由 Dispatcher 来管理线程工作项队列。

```csharp
Task.Factory.StartNew(() =>
{
    // 多线程中可以从控件中取值
    Brush brush = richTextBox1.Background;
    // 但是不可以在多线程中为控件赋值
    // richTextBox1.Background = brush;

    // BeginInvoke 相对于当前线程异步执行，不会等待 UI 主线程的更新
    this.Dispatcher.BeginInvoke((Action)delegate
    {
        richTextBox1.Background = brush;
    });

    // Invoke 相对于当前线程同步执行，会等待 UI 主线程将委托执行完成
    this.Dispatcher.Invoke(new EventHandler(delegate
    {
        richTextBox1.Background = brush;
    }));

    // 传递一个委托的方式多种多样，这时我常用的几种写法
    richTextBox1.Dispatcher.BeginInvoke(new Action(() =>
    {
        richTextBox1.Background = brush;
    }));
});
```

其继承关系可以参考我从网上找到的一幅图：

![DispatcherObject继承关系](https://hd2y.oss-cn-beijing.aliyuncs.com/DispatcherObject%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB_1577189987990.png)

> 注：因为 `Windows XP` 支持 `.NET Framewrok` 的最后一个版本是 `.NET Framewrok 4.0`，所以没有特别说明，我习惯上创建的 `Windows Form` 或 `WPF` 等客户端程序选择的框架都是 `.NET Framewrok 4.0`。

> 参考：
> - MSDN：[Control Class](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.control?view=netframework-4.8)
> - MSDN：[Dispatcher Class](https://docs.microsoft.com/en-us/dotnet/api/system.windows.threading.dispatcher?view=netframework-4.8)
