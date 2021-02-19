---
title: "线程与线程池"
date: "2019/04/03 20:21:00"
updated: "2019/07/11 11:27:37"
permalink: "thread-and-thread-pool"
tags:
 - 多线程
categories:
 - [开发, C#]
---

## Thread

- `Thread` 是前台线程，启动后需要执行完成后才会退出。但是可以通过 `IsBackground` 将其设置为后台线程，程序退出该线程也会立即退出。
- 如果希望等待线程执行完成后再继续执行，可以使用 `Join()` 方法。
- `Thread` 没有回调，也没有返回值。

## ThreadPool

### CLR线程池：

管理线程开销最好的方式：

- 尽量少的创建线程并且能将线程反复利用(线程池初始化时没有线程，有程序请求线程则创建线程)；
- 最好不要销毁而是挂起线程达到避免性能损失(线程池创建的线程完成任务后以挂起状态回到线程池中，等待下次请求)；
- 通过一个技术达到让应用程序一个个执行工作，类似于一个队列(多个应用程序请求线程池，线程池会将各个应用程序排队处理)；
- 如果某一线程长时间挂起而不工作的话，需要彻底销毁并且释放资源(线程池自动监控长时间不工作的线程，自动销毁)；
- 如果线程不够用的话能够创建线程，并且用户可以自己定制最大线程创建的数量(当队列过长，线程池里的线程不够用时，线程池不会坐视不理)；

实现：

- `C#2.0` 时支持，实现为享元模式、单例模式。
- CLR线程池并不会在CLR初始化时立即建立线程，而是在应用程序要创建线程来运行任务时，线程池才初始化一个线程。
- 线程池初始化时是没有线程的，线程池里的。线程的初始化与其他线程一样，但是在完成任务以后，该线程不会自行销毁，而是以挂起的状态返回到线程池。直到应用程序再次向线程池发出请求时，线程池里挂起的线程就会再度激活执行任务。这样既节省了建立线程所造成的性能损耗，也可以让多个任务反复重用同一线程，从而在应用程序生存期内节约大量开销。

**注意：**通过CLR线程池所建立的线程总是默认为后台线程，优先级数为ThreadPriority.Normal。

### 工作者线程与I/O线程

`CLR` 线程池分为工作者线程( `workerThreads` )与 `I/O` 线程( `completionPortThreads` )两种:
- 工作者线程：主要用作管理 `CLR` 内部对象的运作，通常用于计算密集的任务。
- `I/O` ( `Input`/ `Output` )线程主要用于与外部系统交互信息，如输入输出， `CPU` 仅需在任务开始的时候，将任务的参数传递给设备，然后启动硬件设备即可。等任务完成的时候， `CPU` 收到一个通知，一般来说是一个硬件的中断信号，此时 `CPU` 继续后继的处理工作。在处理过程中， `CPU` 是不必完全参与处理过程的，如果正在运行的线程不交出 `CPU` 的控制权，那么线程也只能处于等待状态，即使操作系统将当前的 `CPU` 调度给其他线程，此时线程所占用的空间还是被占用，而并没有 `CPU` 处理这个线程，可能出现线程资源浪费的问题。如果这是一个网络服务程序，每一个网络连接都使用一个线程管理，可能出现大量线程都在等待网络通信，随着网络连接的不断增加，处于等待状态的线程将会很消耗尽所有的内存资源。可以考虑使用线程池解决这个问题。

线程池的最大值一般默认为1000、2000。当大于此数目的请求时，将保持排队状态，直到线程池里有线程可用。

使用 `CLR` 线程池的工作者线程一般有两种方式：

- 通过 `ThreadPool.QueueUserWorkItem()` 方法；
- 通过委托；

**注意：**不论是通过ThreadPool.QueueUserWorkItem()还是委托，调用的都是线程池里的线程。

### `ThreadPool` 类常用方法

通过以下两个方法可以读取和设置CLR线程池中工作者线程与I/O线程的最大线程数：

- `ThreadPool.GetMax(out in workerThreads,out int completionPortThreads)`
- `ThreadPool.SetMax(int workerThreads,int completionPortThreads)`

若想测试线程池中有多少线程正在投入使用，可以通过 `ThreadPool.GetAvailableThreads(out in workThreads,out int conoletionPortThreads)` 方法。

| 方法 | 说明 |
| :--- | :--- |
| `GetAvailableThreads` | 剩余空闲线程数 |
| `GetMaxThreads` | 最多可用线程数，所有大于此数目的请求将保持排队状态，直到线程池线程变为可用 |
| `GetMinThreads` | 检索线程池在新请求预测中维护的空闲线程数。|
| `QueueUserWorkItem` | 启动线程池里得一个线程(队列的方式，如线程池暂时没空闲线程，则进入队列排队) |
| `SetMaxThreads` | 设置线程池中的最大线程数 |
| `SetMinThreads` | 设置线程池最少需要保留的线程数 |

### 各种调用线程池线程的方法

#### 通过 `QueueUserWorkItem` 启动工作者线程

`ThreadPool` 线程池中有两个重载的静态方法可以直接启动工作者

```csharp
ThreadPool.QueueUserWorkItem(waitCallback);
ThreadPool.QueueUserWorkItem(waitCallback,Object);
```

通过 `ThreadPool.QueueUserWork` 启动工作者线程非常方便，但是 `WaitCallback` 委托指向的必须是一个带有 `object` 参数的无返回值方法。所以这个方法启动的工作者线程仅仅适合于带单个参数和无返回值的情况。如果要传递多个参数和要有返回值，那就只有通过委托。

#### `BeginInvoke` 与 `EndInvoke` 委托异步调用线程
1. 建立一个委托对象，通过 `IAsyncResult BeginInvoke(string name,AsyncCallback callback,object state)` 异步调用委托方法， `BeginInvoke` 方法除最后的两个参数外，其他参数都是与方法参数相对应的;
2. 利用 `EndInvoke` ( `IAsyncResult` --上一步 `BeginInvoke` 返回的对象)方法就可以结束异步操作，获取委托的运行结果;

**缺点：**不知道异步操作什么时候执行完，什么时候开始调用EndInvoke，因为一旦EndInvoke主线程就会处于阻塞等待状态。

#### `IAsyncResult` 轮询  

克服上面提到的缺点，可以好好利用 `IAsyncResult` 提高主线程的工作性能：

  - `IsCompleted` 属性：获取异步操作是否已完成；
  - `WaitOne` ：判断单个异步线程是否完成；
  - `WaitAny` ：判断是否异步线程是否有指定数量个已完成；
  - `WaitAll` ：判断是否所有的异步线程已完成；

```csharp
Func<string, int, string> func = (name, age) =>
{
    Thread.Sleep(20000);
    return $"My name is {name}, I'm {age} years old.";
};
IAsyncResult asyncResult = func.BeginInvoke("Jane", 12, null, null);
//IAsyncResult.IsCompleted属性
while (!asyncResult.IsCompleted)
{
    Thread.Sleep(500);
    Console.WriteLine("*************WAITING*************");
}
////AsyncWaitHandle.WaitOne()方法
//while (!asyncResult.AsyncWaitHandle.WaitOne(200))
//{
//    Console.WriteLine("*************WAITING*************");
//}
////WaitHandle.WaitAny()方法
//WaitHandle[] waitHandleList1 = new WaitHandle[] { asyncResult.AsyncWaitHandle };
//while (WaitHandle.WaitAny(waitHandleList1, 200) > 0)
//{
//    Console.WriteLine("*************WAITING*************");
//}
////WaitHandle.WaitAll()方法
//WaitHandle[] waitHandleList2 = new WaitHandle[] { asyncResult.AsyncWaitHandle };
//while (WaitHandle.WaitAll(waitHandleList2, 200))
//{
//    Console.WriteLine("*************WAITING*************");
//}
Console.WriteLine(func.EndInvoke(asyncResult));
```

#### `IAsyncResult` 回调函数

使用轮询方式来检测异步方法的状态非常麻烦，而且影响了主线程，效率不高。我们可以使用IAsyncResult对象，当异步线程完成了就直接调用实现定义好的处理函数。

```csharp
Func<string, int, string> func = (name, age) =>
{
    Thread.Sleep(2000);
    return $"My name is {name}, I'm {age} years old.";
};
IAsyncResult asyncResult = null;
AsyncCallback callback = (t) => 
{
    Console.WriteLine(func.EndInvoke(asyncResult));
};
asyncResult = func.BeginInvoke("Jane", 12, callback, null);
```

**注意：**

- 回调函数依然是在辅助线程中执行的，这样就不会影响主线程的运行。
- 线程池的线程默认是后台线程。但是如果主线程比辅助线程优先完成，那么程序已经卸载，回调函数未必会执行。如果不希望丢失回调函数中的操作，要么把异步线程设为前台线程，要么确保主线程将比辅助线程迟完成。

### `ManualResetEvent`

常用方法： `Set()` 、 `ReSet()` 、 `WaitOne()`

- `Set()` : 用于向 `ManualResetEvent` 发送信号，使其取消阻塞状态（唤醒进程）或者开始阻塞进程，这基于  `ManualResetEvent` 的初始状态。
- `ReSet()` : 将 `ManualResetEvent` 的状态重置至初始状态（即使用 `Set()` 方法之前的状态）。
- `WaitOne()` : 使 `ManualResetEvent` 进入阻塞状态，开始等待唤醒信号。如果有信号，则不会阻塞，直接通过。
- 信号 : 当 `new ManualResetEvent(bool arg)` 时， `arg` 参数就是信号状态，假如为 `false` ，则表示当前无信号，如果为 `true` ，则有信号。
