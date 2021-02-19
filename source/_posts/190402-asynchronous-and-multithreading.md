---
title: "异步多线程"
date: "2019/04/02 22:18:00"
updated: "2019/07/11 11:10:47"
permalink: "asynchronous-and-multithreading/"
tags:
 - 多线程
categories:
 - [开发, C#]
---

## 基本概念

### 进程

首先打开任务管理器，可以查看电脑当前运行的进程。  

从任务管理器里面可以看到当前所有正在运行的进程。

**那么究竟什么是进程呢？**

进程( `Process` )是 `Windows` 系统中的一个基本概念，它包含着一个运行程序所需要的资源。一个正在运行的应用程序在操作系统中被视为一个进程，进程可以包括一个或多个线程。  

线程是操作系统分配处理器时间的基本单元，在进程中可以有多个线程同时执行代码。进程之间是相对独立的，一个进程无法访问另一个进程的数据(除非利用分布式计算方式)，一个进程运行的失败也不会影响其他进程的运行， `Windows` 系统就是利用进程把工作划分为多个独立的区域的。  

进程可以理解为一个程序的基本边界。是应用程序的一个运行例程，是应用程序的一次动态执行过程。

### 线程

在任务管理器， 性能 -> `CPU` 中可以查看当前电脑当前运行的线程数量。  

线程( `Thread` )是进程中的基本执行单元，是操作系统分配 `CPU` 时间的基本单位，一个进程可以包含若干个线程，在进程入口执行的第一个线程被视为这个进程的主线程。  

在 `.NET` 应用程序中，都是以 `Main()` 方法作为入口的，当调用此方法时系统就会自动创建一个主线程。线程主要是由 `CPU` 寄存器、调用栈和线程本地存储器( `Thread Local Storage` ， `TLS` )组成的。  

`CPU` 寄存器主要记录当前所执行线程的状态，调用栈主要用于维护线程所调用到的内存与数据， `TLS` 主要用于存放线程的状态信息。

## 多线程

### 多线程的优缺点

多线程的优点：

- 可以同时完成多个任务；
- 可以使程序的响应速度更快；
- 可以让占用大量处理时间的任务或当前没有进行处理的任务定期将处理时间让给别的任务；
- 可以随时停止任务；
- 可以设置每个任务的优先级以优化程序性能。

那么可能有人会问：为什么可以多线程执行呢？

总结起来有下面两方面的原因：
- CPU运行速度太快，硬件处理速度跟不上，所以操作系统进行分时间片管理。这样，从宏观角度来说是多线程并发的，因为CPU速度太快，察觉不到，看起来是同一时刻执行了不同的操作。但是从微观角度来讲，同一时刻只能有一个线程在处理。
- 目前电脑都是多核多CPU的，一个CPU在同一时刻只能运行一个线程，但是多个CPU在同一时刻就可以运行多个线程。

然而，多线程虽然有很多优点，但是也必须认识到多线程可能存在影响系统性能的不利方面，才能正确使用线程。

不利方面主要有如下几点：

- 线程也是程序，所以线程需要占用内存，线程越多，占用内存也越多。
- 多线程需要协调和管理，所以需要占用CPU时间以便跟踪线程。
- 线程之间对共享资源的访问会相互影响，必须解决争用共享资源的问题。
- 线程太多会导致控制太复杂，最终可能造成很多程序缺陷。

#### 多线程

当启动一个可执行程序时，将创建一个主线程。在默认的情况下， `C#` 程序具有一个线程，此线程执行程序中以 `Main` 方法开始和结束的代码， `Main()` 方法直接或间接执行的每一个命令都有默认线程(主线程)执行，当 `Main()` 方法返回时此线程也将终止。  

一个进程可以创建一个或多个线程以执行与该进程关联的部分程序代码。在 `C#` 中，线程是使用 `Thread` 类处理的，该类在 `System.Threading` 命名空间中。  

使用 `Thread` 类创建线程时，只需要提供线程入口，线程入口告诉程序让这个线程做什么。通过实例化一个 `Thread` 类的对象就可以创建一个线程。  

创建新的 `Thread` 对象时，将创建新的托管线程。 `Thread` 类接收一个 `ThreadStart` 委托或 `ParameterizedThreadStart` 委托的构造函数，该委托包装了调用 `Start` 方法时由新线程调用的方法，示例代码如下：

```csharp
Thread thread=new Thread(new ThreadStart(method));//创建线程
thread.Start();                                   //启动线程
```

上面代码实例化了一个 `Thread` 对象，并指明将要调用的方法 `method()` ，然后启动线程。 `ThreadStart` 委托中作为参数的方法不需要参数，并且没有返回值。 `ParameterizedThreadStart` 委托一个对象作为参数，利用这个参数可以很方便地向线程传递参数，示例代码如下：

```csharp
Thread thread=new Thread(new ParameterizedThreadStart(method));//创建线程
thread.Start(3);                                               //启动线程
```

创建多线程的步骤：

1. 编写线程所要执行的方法
2. 实例化 `Thread` 类，并传入一个指向线程所要执行方法的委托。(这时线程已经产生，但还没有运行)
3. 调用 `Thread` 实例的 `Start` 方法，标记该线程可以被 `CPU` 执行了，但具体执行时间由 `CPU` 决定

### `System.Threading.Thread` 类

Thread类是是控制线程的基础类，位于System.Threading命名空间下，具有4个重载的构造函数：

- `Thread(ParameterizedThreadStart)` 初始化 `Thread` 类的新实例，指定允许对象在线程启动时传递给线程的委托。要执行的方法是有参的。
- `Thread(ParameterizedThreadStart, Int32)` 初始化 `Thread` 类的新实例，指定允许对象在线程启动时传递给线程的委托，并指定线程的最大堆栈大小。
- `Thread(ThreadStart)` 初始化 `Thread` 类的新实例。要执行的方法是无参的。
- `Thread(ThreadStart, Int32)` 初始化 `Thread` 类的新实例，指定线程的最大堆栈大小。
- `ThreadStart` 是一个无参的、返回值为 `void` 的委托。
- `ParameterizedThreadStart` 是一个有参的、返回值为 `void` 的委托。

**注意：** `ParameterizedThreadStart` 委托的参数类型必须是 `Object` 的。如果使用的是不带参数的委托，不能使用带参数的 `Start` 方法运行线程，否则系统会抛出异常。但使用带参数的委托，可以使用 `thread.Start()` 来运行线程，这时所传递的参数值为 `null` 。

### 线程的常用属性

- `CurrentContext` 获取线程正在其中执行的当前上下文。
- `CurrentThread` 获取当前正在运行的线程。
- `ExecutionContext` 获取一个 `ExecutionContext` 对象，该对象包含有关当前线程的各种上下文的信息。
- `IsAlive` 获取一个值，该值指示当前线程的执行状态。
- `IsBackground` 获取或设置一个值，该值指示某个线程是否为后台线程。
- `IsThreadPoolThread` 获取一个值，该值指示线程是否属于托管线程池。
- `ManagedThreadId` 获取当前托管线程的唯一标识符。
- `Name` 获取或设置线程的名称。
- `Priority` 获取或设置一个值，该值指示线程的调度优先级。
- `ThreadState` 获取一个值，该值包含当前线程的状态。

#### 线程的标识符

`ManagedThreadId` 是确认线程的唯一标识符，程序在大部分情况下都是通过 `Thread.ManagedThreadId` 来辨别线程的。而Name是一个可变值，在默认时候， `Name` 为一个空值 `Null` ，开发人员可以通过程序设置线程的名称，但这只是一个辅助功能。

#### 线程的优先级别

当线程之间争夺 `CPU` 时间时， `CPU` 按照线程的优先级给予服务。高优先级的线程可以完全阻止低优先级的线程执行。

`.NET` 为线程设置了 `Priority` 属性来定义线程执行的优先级别，里面包含5个选项，其中 `Normal` 是默认值。除非系统有特殊要求，否则不应该随便设置线程的优先级别。

- `Lowest` 可以将 `Thread` 安排在具有任何其他优先级的线程之后。
- `BelowNormal` 可以将 `Thread` 安排在具有 `Normal` 优先级的线程之后，在具有 `Lowest` 优先级的线程之前。
- `Normal` 默认选择。可以将 `Thread` 安排在具有 `AboveNormal` 优先级的线程之后，在具有 `BelowNormal` 优先级的线程之前。
- `AboveNormal` 可以将 `Thread` 安排在具有 `Highest` 优先级的线程之后，在具有 `Normal` 优先级的线程之前。
- `Highest`     可以将 `Thread` 安排在具有任何其他优先级的线程之前。

#### 线程的状态

通过 `ThreadState` 可以检测线程是处于 `Unstarted` 、 `Sleeping` 、 `Running` 等等状态，它比 `IsAlive` 属性能提供更多的特定信息。  

前面说过，一个应用程序域中可能包括多个上下文，而通过 `CurrentContext` 可以获取线程当前的上下文。

`CurrentThread` 是最常用的一个属性，它是用于获取当前运行的线程。

#### `System.Threading.Thread` 的方法

`Thread` 中包括了多个方法来控制线程的创建、挂起、停止、销毁，以后来的例子中会经常使用。

- `Abort()` 终止本线程。
- `GetDomain()` 返回当前线程正在其中运行的当前域。
- `GetDomainId()` 返回当前线程正在其中运行的当前域Id。
- `Interrupt()` 中断处于 `WaitSleepJoin` 线程状态的线程。
- `Join()` 已重载。 阻塞调用线程，直到某个线程终止时为止。
- `Resume()` 继续运行已挂起的线程。
- `Start()` 执行本线程。
- `Suspend()` 挂起当前线程，如果当前线程已属于挂起状态则此不起作用
- `Sleep()` 把正在运行的线程挂起一段时间。

### 前台线程和后台线程

- 前台线程：只有所有的前台线程都结束，应用程序才能结束。默认情况下创建的线程都是前台线程。
- 后台线程：只要所有的前台线程结束，后台线程自动结束。通过 `Thread.IsBackground` 设置后台线程。必须在调用 `Start` 方法之前设置线程的类型，否则一旦线程运行，将无法改变其类型。通过 `BeginXXX` 方法运行的线程都是后台线程。后台线程一般用于处理不重要的事情，应用程序结束时，后台线程是否执行完成对整个应用程序没有影响。如果要执行的事情很重要，需要将线程设置为前台线程。

### 线程同步

所谓同步：是指在某一时刻只有一个线程可以访问变量。  

如果不能确保对变量的访问是同步的，就会产生错误。  

`C#` 为同步访问变量提供了一个非常简单的方式，即使用 `C#` 语言的关键字 `Lock` ，它可以把一段代码定义为互斥段，互斥段在一个时刻内只允许一个线程进入执行，而其他线程必须等待。在 `C#` 中，关键字 `Lock` 定义如下：

```csharp
Lock(expression)
{
    statement_block
}
```

`expression` 代表你希望跟踪的对象：如果你想保护一个类的实例，一般地，你可以使用 `this` ；如果你想保护一个静态变量(如互斥代码段在一个静态方法内部)，一般使用类名就可以了。而 `statement_block` 就算互斥段的代码，这段代码在一个时刻内只可能被一个线程执行。

### 跨线程访问

创建窗体应用程序，增加一个测试按钮和一个文本框，点击“测试”，创建一个线程，从0循环到10000给文本框赋值，代码如下：

```csharp
private void btn_Test_Click(object sender, EventArgs e)
{
    //创建一个线程去执行这个方法:创建的线程默认是前台线程
    Thread thread = new Thread(new ThreadStart(Test));
    //Start方法标记这个线程就绪了，可以随时被执行，具体什么时候执行这个线程，由CPU决定
    //将线程设置为后台线程
    thread.IsBackground = true;
    thread.Start();
}
private void Test()
{
    for (int i = 0; i < 10000; i++)
    {               
        this.textBox1.Text = i.ToString();
    }
}
```

>运行结果会报错：线程间操作无效：不是创建“textBox1”的线程访问它。  

>产生错误的原因：textBox1是由主线程创建的，thread线程是另外创建的一个线程，在.NET上执行的是托管代码，C#强制要求这些代码必须是线程安全的，即不允许跨线程访问Windows窗体的控件。

解决方案：

1. 在窗体的加载事件中，将 `C#` 内置控件( `Control` )类的 `CheckForIllegalCrossThreadCalls` 属性设置为 `false` ，屏蔽掉 `C#` 编译器对跨线程调用的检查。

```csharp
private void Form1_Load(object sender, EventArgs e)
{
       //取消跨线程的访问
       Control.CheckForIllegalCrossThreadCalls = false;
}
```

> 使用上述的方法虽然可以保证程序正常运行并实现应用的功能，但是在实际的软件开发中，做如此设置是不安全的(不符合 `.NET` 的安全规范)，在产品软件的开发中，此类情况是不允许的。如果要在遵守.NET安全标准的前提下，实现从一个线程成功地访问另一个线程创建的空间，要使用 `C#` 的方法回调机制。

2. 使用回调函数

回调实现的一般过程： `C#` 的方法回调机制，也是建立在委托基础上的，下面给出它的典型实现过程。

- 定义、声明回调。

```csharp
//定义回调
private delegate void DoSomeCallBack(Type para);
//声明回调
DoSomeCallBack doSomaCallBack;
```

可以看出，这里定义声明的“回调”( `doSomaCallBack` )其实就是一个委托。

- 初始化回调方法。

```csharp
doSomeCallBack = new DoSomeCallBack(DoSomeMethod);
```

> 所谓“初始化回调方法”实际上就是实例化刚刚定义了的委托，这里作为参数的 `DoSomeMethod` 称为“回调方法”，它封装了对另一个线程中目标对象（窗体控件或其他类）的操作代码。

- 触发对象动作

```csharp
Opt obj.Invoke(doSomeCallBack, arg);
```

其中 `Opt obj` 为目标操作对象，在此假设它是某控件，故调用其 `Invoke` 方法。 `Invoke` 方法签名为：

```csharp
object Control.Invoke(Delegate method, params object[] args);
```

它的第一个参数为委托类型，可见“触发对象动作”的本质，就是把委托 `doSomeCallBack` 作为参数传递给控件的 `Invoke` 方法，这与委托的使用方式是一模一样的。
最终作用于对象 `Opt obj` 的代码是置于回调方法体 `DoSomeMethod()` 中的，如下所示：

```csharp
private void DoSomeMethod(type para)
{
    //方法体
    Opt obj.someMethod(para);
}
```

如果不用回调，而是直接在程序中使用 `Opt obj.someMethod(para);` ，则当对象 `Opt obj` 不在本线程（跨线程访问）时就会发生上面所示的错误。

从以上回调实现的一般过程可知： `C#` 的回调机制，实质上是委托的一种应用。在 `C#` 网络编程中，回调的应用是非常普遍的，有了方法回调，就可以在 `.NET` 上写出线程安全的代码了。

使用方法回调，实现给文本框赋值：

```csharp
namespace MultiThreadDemo
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }
        //定义回调
        private delegate void setTextValueCallBack(int value);
        //声明回调
        private setTextValueCallBack setCallBack;
        private void btn_Test_Click(object sender, EventArgs e)
        {
            //实例化回调
            setCallBack = new setTextValueCallBack(SetValue);
            //创建一个线程去执行这个方法:创建的线程默认是前台线程
            Thread thread = new Thread(new ThreadStart(Test));
            //Start方法标记这个线程就绪了，可以随时被执行，具体什么时候执行这个线程，由CPU决定
            //将线程设置为后台线程
            thread.IsBackground = true;
            thread.Start();
        }
        private void Test()
        {
            for (int i = 0; i < 10000; i++)
            {               
                //使用回调
                textBox1.Invoke(setCallBack, i);
            }
        }
        /// <summary>
        /// 定义回调使用的方法
        /// </summary>
        /// <param name="value"></param>
        private void SetValue(int value)
        {
            this.textBox1.Text = value.ToString();
        }
    }
}
```

### 终止线程

若想终止正在运行的线程，可以使用 `Abort()` 方法。

## 同步和异步

同步和异步是对方法执行顺序的描述。

同步：等待上一行完成计算之后，才会进入下一行。

> 例如：请同事吃饭，同事说很忙，然后就等着同事忙完，然后一起去吃饭。

异步：不会等待方法的完成，会直接进入下一行，是非阻塞的。

> 例如：请同事吃饭，同事说很忙，那同事先忙，自己去吃饭，同事忙完了他自己去吃饭。

同步方法和异步方法的区别：

1. - 同步方法由于主线程忙于计算，所以会卡住界面。
   - 异步方法由于主线程执行完了，其他计算任务交给子线程去执行，所以不会卡住界面，用户体验性好。
2. - 同步方法由于只有一个线程在计算，所以执行速度慢。
   - 异步方法由多个线程并发运算，所以执行速度快，但并不是线性增长的（资源可能不够）。多线程也不是越多越好，只有多个独立的任务同时运行，才能加快速度。
3. - 同步方法是有序的。
   - 异步多线程是无序的：启动无序，执行时间不确定，所以结束也是无序的。一定不要通过等待几毫秒的形式来控制线程启动/执行时间/结束。

### 回调

- 异步多线程是无序的，可以通过使用回调解决异步多线程是无序的问题。
- 获取委托异步调用的返回值：使用 `EndInvoke()` 可以获取委托异步调用的返回值。注意调用该方法会等待异步执行完成，所以在主线程中执行，会强制等待委托执行完成，而在回调中执行因为委托其实已经执行完成，所以此时 `EndInvoke()` 单纯为了获取委托执行的结果。

