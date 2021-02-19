---
title: "23种设计模式与常用设计模式"
date: "2019/04/07 11:09:36"
updated: "2019/07/10 17:02:31"
permalink: "23-design-patterns-and-common-design-patterns"
tags:
 - 设计模式
categories:
 - [开发, 基础]
---

## 设计模式

### 设计模式

设计模式(`Design Pattern`)是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。

目的是为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。

### 设计模式三大类型

设计模式分为三种类型，共23类。

- 创建型模式：创建型模式用来处理对象的创建过程，单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式。
- 结构型模式：结构型模式用来处理类或者对象的组合，适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。
- 行为型模式：行为型模式用来对类或对象怎样交互和怎样分配职责进行描述，模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式、状态模式、策略模式、职责链模式、访问者模式。

## 单例模式

单例模式(`Singleton Pattern`)保证系统中，应用该模式的类一个类只有一个实例。即一个类只有一个对象实例。

### 实现方式

#### 饿汉式

第一时间创建实例

```csharp
/// <summary>
/// 静态构造函数
/// *该方法由CLR保证，程序第一次使用这个类型前被调用，且只调用一次
/// </summary>
public class Singleton2
{
    static Singleton2()
    {
        Thread.Sleep(1000);
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton2)}\t构造成功\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
    }
	//测试
    public static void Show()
    {
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton2)}\t单例模式测试开始\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
        Singleton2 singleton = null;
        for (int i = 0; i < 10; i++)
        {
            Task.Factory.StartNew(() =>
            {
                singleton = new Singleton2();
                Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton2)}\t获取单例\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
            });
        }
    }
}
/// <summary>
/// 静态字段
/// *该方法由CLR保证，程序第一次使用这个类型前被初始化，且只初始化一次
/// </summary>
public class Singleton3
{
    private Singleton3()
    {
        Thread.Sleep(1000);
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton3)}\t构造成功\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
    }
    private static Singleton3 _singleton = new Singleton3();
    public static Singleton3 CreateInstance()
    {
        return _singleton;
    }
	//测试
    public static void Show()
    {
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton3)}\t单例模式测试开始\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
        Singleton3 singleton = null;
        for (int i = 0; i < 10; i++)
        {
            Task.Factory.StartNew(() =>
            {
                singleton = CreateInstance();
                Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton3)}\t获取单例\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
            });
        }
    }
}
```

#### 懒汉式

需要才创建实例

```csharp
/// <summary>
/// 双if判断加锁
/// *内层if判断保证单例模式
/// *外层if判断避免加锁阻碍多线程运行
/// *加锁避免多线程对象被多次创建
/// </summary>
public class Singleton1
{
    private Singleton1()
    {
        Thread.Sleep(1000);
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton1)}\t构造成功\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
    }
    private static volatile Singleton1 _singleton = null;
    private static object Singleton_Lock = new object();
    public static Singleton1 CreateInstance()
    {
        if (_singleton == null)//不为空才去等待锁
        {
            lock (Singleton_Lock)
            {
                Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton1)}\t等待锁之后释放\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
                Thread.Sleep(500);
                if (_singleton == null)//保证不被重复创建
                {
                    _singleton = new Singleton1();
                }
            }
        }
        return _singleton;
    }

    //测试
    public static void Show()
    {
        Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton1)}\t单例模式测试开始\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
        Singleton1 singleton = null;
        List<Task> listTask = new List<Task>();
        for (int i = 0; i < 10; i++)
        {
            listTask.Add(Task.Factory.StartNew(() =>
            {
                singleton = CreateInstance();
                Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton1)}\t获取单例\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
            }));
        }
        Task.WaitAll(listTask.ToArray());
        for (int i = 0; i < 10; i++)
        {
            Task.Factory.StartNew(() =>
            {
                singleton = CreateInstance();
                Console.WriteLine($"{DateTime.Now.ToString("HH:mm:ss")}: {nameof(Singleton1)}\t获取单例\t线程ID为{Thread.CurrentThread.ManagedThreadId}。");
            });
        }
    }
}
```

## 三种工厂模式

简单工厂、工厂方法、抽象工厂都属于设计模式中的创建型模式。其主要功能都是帮助我们把对象的实例化部分抽象取了出来，优化了系统的架构，并且增强了系统的扩展性。

### 简单工厂

简单工厂(`Simple Factory`)：简单工厂模式的工厂类一般是使用静态方法，通过接收的参数不同来返回不同的对象实例。不修改代码的话，是无法扩展的。  

- 优点：客户端可以免除直接创建产品对象的责任，而仅仅是“消费”产品。简单工厂模式通过这种做法实现了对责任的分割。
- 缺点：由于工厂类集中了所有实例的创建逻辑，违反了高内聚责任分配原则，将全部创建逻辑集中到了一个工厂类中；它所能创建的类只能是事先考虑到的，如果需要添加新的类，则就需要改变工厂类了。

配置文件

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="BallType" value="Test.exe,Test.Factory.Basketball"/>
  </appSettings>
</configuration>
```

实体与枚举

```csharp
public class Sportsman
{
    public Sportsman(string name, bool gender, int age, string country)
    {
        Name = name;
        Gender = gender;
        Age = age;
        Country = country;
    }
    public string Name { get; set; }
    public bool Gender { get; set; }
    public int Age { get; set; }
    public string Country { get; set; }
    public void PlayGame(IBall ball)
    {
        Console.WriteLine($"You're watching {ball.PlayGame()} The name of the sportsman is {Name}. {(Gender ? "He" : "She")} is a {Age} years old {(Gender ? "boy" : "girl")} from {Country}.");
    }
}
public interface IBall
{
    string PlayGame();
}
public enum BallType
{
    Basketball,
    Football,
    Baseball,
    Volleyball
}
    
```

简单工厂

```csharp
public class SimpleFactory
{
    public static void Show()
    {
        Sportsman sportsman = new Sportsman("Kangkang", true, 17, "China");
        {
            Basketball basketball = new Basketball();//1.左右都是细节
            sportsman.PlayGame(basketball);
        }
        {
            IBall ball = new Football();//2.左边是抽象 右边是细节
            sportsman.PlayGame(ball);
        }
        {
            //遵循依赖倒置 但是违背了单一职责
            Console.WriteLine("********简单工厂-转移细节********");
            IBall ball = CreateBallGame(BallType.Volleyball);//3.没有细节 细节被转移
            sportsman.PlayGame(ball);
        }
        {
            //结合配置项+反射 IOC雏形
            Console.WriteLine("********简单工厂-配置项********");
            IBall ball = CreateBallFromConfiguration();
            sportsman.PlayGame(ball);
        }
    }

    /// <summary>
    /// **集中了矛盾**
    /// 细节没有消失只是转移
    /// 转移了矛盾并没有消除矛盾
    /// </summary>
    /// <param name="type"></param>
    /// <returns></returns>
    public static IBall CreateBallGame(BallType type)
    {
        IBall ball = null;
        switch (type)
        {
            case BallType.Basketball:
                ball = new Basketball();
                break;
            case BallType.Football:
                ball = new Football();
                break;
            case BallType.Baseball:
                ball = new Baseball();
                break;
            case BallType.Volleyball:
                ball = new Volleyball();
                break;
            default:
                throw new Exception("unknown ball type.");
        }
        return ball;
    }

    /// <summary>
    /// IOC的雏形
    /// </summary>
    /// <returns></returns>
    public static IBall CreateBallFromConfiguration()
    {
        string[] aTemp = ConfigurationManager.AppSettings.Get("BallType").Split(',');
        Assembly assembly = Assembly.LoadFrom(aTemp[0]);
        Type type = assembly.GetType(aTemp[1]);
        IBall ball = Activator.CreateInstance(type) as IBall;
        return ball;
    }
}

public class Baseball : IBall
{
    public string PlayGame()
    {
        return "a baseball tv show.";
    }
}
public class Football : IBall
{
    public string PlayGame()
    {
        return "a football tv show.";
    }
}
public class Baseball : IBall
{
    public string PlayGame()
    {
        return "a baseball tv show.";
    }
}
public class Volleyball : IBall
{
    public string PlayGame()
    {
        return "a volleyball tv show.";
    }
}
```

简单工厂实现

```csharp
public class SimpleFactory
{
    public static void Show()
    {
        Sportsman sportsman = new Sportsman("Kangkang", true, 17, "China");
        {
            Basketball basketball = new Basketball();//1.左右都是细节
            sportsman.PlayGame(basketball);
        }
        {
            IBall ball = new Football();//2.左边是抽象 右边是细节
            sportsman.PlayGame(ball);
        }
        {
            //遵循依赖倒置 但是违背了单一职责
            Console.WriteLine("********简单工厂-转移细节********");
            IBall ball = CreateBallGame(BallType.Volleyball);//3.没有细节 细节被转移
            sportsman.PlayGame(ball);
        }
        {
            //结合配置项+反射 IOC雏形
            Console.WriteLine("********简单工厂-配置项********");
            IBall ball = CreateBallFromConfiguration();
            sportsman.PlayGame(ball);
        }
    }

    /// <summary>
    /// **集中了矛盾**
    /// 细节没有消失只是转移
    /// 转移了矛盾并没有消除矛盾
    /// </summary>
    /// <param name="type"></param>
    /// <returns></returns>
    public static IBall CreateBallGame(BallType type)
    {
        IBall ball = null;
        switch (type)
        {
            case BallType.Basketball:
                ball = new Basketball();
                break;
            case BallType.Football:
                ball = new Football();
                break;
            case BallType.Baseball:
                ball = new Baseball();
                break;
            case BallType.Volleyball:
                ball = new Volleyball();
                break;
            default:
                throw new Exception("unknown ball type.");
        }
        return ball;
    }

    /// <summary>
    /// IOC的雏形
    /// </summary>
    /// <returns></returns>
    public static IBall CreateBallFromConfiguration()
    {
        string[] aTemp = ConfigurationManager.AppSettings.Get("BallType").Split(',');
        Assembly assembly = Assembly.LoadFrom(aTemp[0]);
        Type type = assembly.GetType(aTemp[1]);
        IBall ball = Activator.CreateInstance(type) as IBall;
        return ball;
    }
}
```

### 工厂方法

工厂方法(`Factory Method`)：工厂方法是针对每一种产品提供一个工厂类。通过不同的工厂实例来创建不同的产品实例。在同一等级结构中，支持增加任意产品。

- 优点：允许系统在不修改具体工厂角色的情况下引进新产品。
- 缺点：由于每加一个产品，就需要加一个产品工厂的类，增加了额外的开发量。

提供工厂类

```csharp
public interface IFactory
{
    IBall CreateBall();
}
public class BaseballFactory : IFactory
{
    public IBall CreateBall()
    {
        return new Baseball();
    }
}
public class FootballFactory : IFactory
{
    public IBall CreateBall()
    {
        return new Football();
    }
}
public class BaseballFactory : IFactory
{
    public IBall CreateBall()
    {
        return new Baseball();
    }
}
public class VolleyballFactory : IFactory
{
    public IBall CreateBall()
    {
        return new Volleyball();
    }
}
```

工厂方法实现

```csharp
public class FactoryMethod
{
    public static void Show()
    {
        Sportsman sportsman = new Sportsman("Kangkang", true, 17, "China");
        {
            //每个业务对应一个工厂 将业务细节的依赖转移到中间层（工厂）
            //优点：允许系统在不修改具体工厂角色的情况下引进新产品
            //缺点：由于每加一个产品，就需要加一个产品工厂的类，增加了额外的开发量
            Console.WriteLine("********工厂方法********");
            IFactory factory = new BasketballFactory();
            IBall ball = factory.CreateBall();
            sportsman.PlayGame(ball);
        }
    }
}
```

### 抽象工厂

抽象工厂(`Abstract Factory`)：抽象工厂是应对产品族概念的。应对产品族概念而生，增加新的产品线很容易，但是无法增加新的产品。比如，每个汽车公司可能要同时生产轿车、货车、客车，那么每一个工厂都要有创建轿车、货车和客车的方法。

- 优点：向客户端提供一个接口，使得客户端在不必指定产品具体类型的情况下，创建多个产品族中的产品对象。
- 缺点：增加新的产品等级结构很复杂，需要修改抽象工厂和所有的具体工厂类，对“开闭原则”的支持呈现倾斜性。

例如游戏加载需要英雄、资源、技能等

```csharp
public abstract class AbstractFactory
{
    public abstract IHero CreateHero();
    public abstract ISkill CreateSkill();
    public abstract IResource CreateResource();
    public abstract IWeapon CreateWeapon();
}

public interface IHero
{
    string ShowHero();
}

public interface ISkill
{
    string ShowSkill();
}

public interface IResource
{
    string ShowResource();
}

public interface IWeapon
{
    string ShowWeapon();
}
```

## 装饰器模式

装饰器模式(Decorator Pattern)：允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。动态地给一个对象添加一些额外的职责。

就增加功能来说，装饰器模式相比生成子类更为灵活。一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

**类与类之间的关系：** 纵向有继承和实现，横向有依赖、关联、组合、聚合。**结构型设计模式：组合优于继承**。

- 优点：装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。
- 缺点：多层装饰比较复杂。

学生学习免费或付费课程享有不同的权利，除享受免费课程外，付费学生还有课前资料预习、课后作业指导、资料复习。

```csharp
public abstract class AbstractStudent
{
    public int Id { get; set; }
    public string Name { get; set; }
    public abstract void Study();

    public static void Show()
    {
        AbstractStudent student = new VIPStudent() { Id = 999, Name = "Kangkang" };
        student.Study();
        Console.WriteLine("************************");
        StudentPreviewDecorator previewStudent = new StudentPreviewDecorator(student);
        previewStudent.Study();
        Console.WriteLine("************************");
        AbstractStudent abstractStudent = new StudentPreviewDecorator(student);
        abstractStudent.Study();
        Console.WriteLine("************************");
        student = new StudentPayDecorator(student);
        student = new StudentPreviewDecorator(student);
        student = new StudentHomeworkDecorator(student);
        student = new StudentReviewDecorator(student);
        student.Study();
    }
}

public class PoolStudent : AbstractStudent
{
    public override void Study()
    {
        Console.WriteLine("Take free courses.");
    }
}

public class VIPStudent : AbstractStudent
{
    public override void Study()
    {
        Console.WriteLine("Take free and paid courses.");
    }
}
```

继承也可以实现，但是不够灵活

```csharp
public class BaseStudentInherit : VIPStudent
{
    public override void Study()
    {
        Console.WriteLine("Pay.");
        Console.WriteLine("Preview.");
        base.Study();
    }
}
```

付费、预习、作业、复习等装饰器

```csharp
public class StudentPayDecorator : BaseStudentDecorator
{
    public StudentPayDecorator(AbstractStudent student) : base(student)
    { }
    public override void Study()
    {
        Console.WriteLine("Pay.");
        base.Study();
    }
}
public class StudentPreviewDecorator : BaseStudentDecorator
{
    public StudentPreviewDecorator(AbstractStudent student) : base(student)
    {
    }
    public override void Study()
    {
        Console.WriteLine("Preview.");
        base.Study();
    }
}
public class StudentHomeworkDecorator : BaseStudentDecorator
{
    public StudentHomeworkDecorator(AbstractStudent student) : base(student)
    { }

    public override void Study()
    {
        base.Study();
        Console.WriteLine("Do homework.");
    }
}
public class StudentReviewDecorator:BaseStudentDecorator
{
    public StudentReviewDecorator(AbstractStudent student) : base(student)
    { }

    public override void Study()
    {
        base.Study();
        Console.WriteLine("Review.");
    }
}
```

## 代理模式

代理模式(`Proxy Pattern`)：一个类代表另一个类的功能，为其他对象提供一种代理以控制对这个对象的访问。主要解决在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。

在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

优点：
- 职责清晰。 
- 高扩展性。
- 智能化。

缺点：
- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。
- 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

代理模式：只能传达原有逻辑，不能新增业务逻辑。

```csharp
public interface ISubject
{
    bool GetSomething();
    bool DoSomething();
}
public class RealSubject : ISubject
{
    public bool DoSomething()
    {
        Console.WriteLine("Passenger by train.");
        return true;
    }

    public bool GetSomething()
    {
        Console.WriteLine("Passengers buy tickets.");
        return true;
    }
}
public class ProxySubject : ISubject
{
    private ISubject _subject = new RealSubject();
    public bool DoSomething()
    {
        Console.WriteLine("Before do something.");
        bool result = _subject.DoSomething();
        Console.WriteLine("After do something.");
        return result;
    }

    public bool GetSomething()
    {
        Console.WriteLine("Before get something.");
        bool result = _subject.GetSomething();
        Console.WriteLine("After get something.");
        return result;
    }

    public static void Show()
    {
        ISubject realSubject = new RealSubject();
        realSubject.GetSomething();
        realSubject.DoSomething();
        Console.WriteLine("************************");
        ISubject proxySubject = new ProxySubject();
        proxySubject.GetSomething();
        proxySubject.DoSomething();
    }
}
```

## 观察者模式

观察者模式(`Observer Pattern`)：当对象间存在一对多关系时，则使用观察者模式。比如，当一个对象被修改时，则会自动通知它的依赖对象。

观察者模式属于行为型模式。定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

主要解决一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

优点：
- 观察者和被观察者是抽象耦合的。
- 建立一套触发机制。

缺点：
- 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
- 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。 
- 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

一只神奇的猫

```csharp
/// <summary>
/// 一直神奇的猫，猫叫之后触发：
/// Baby cry
/// Brother turn
/// Dog woof
/// Father roar
/// Mother whisper
/// Mouse run
/// Neighbor awake
/// Stealer hide
/// </summary>
public class Cat
{
    public void Meow()
    {
        Console.WriteLine($"{GetType().Name} meow...");
        new Brother().Turn();
        new Dog().Woof();
        new Father().Roar();
        new Mother().Whisper();
        new Mouse().Run();
        new Neighbor().Awake();
        new Stealer().Hide();
    }

    public event Action MeowHandler;
    public void MeowEvent()
    {
        Console.WriteLine($"{GetType().Name} meow...");
        MeowHandler?.Invoke();
    }

    private List<IObserver> _listObserver = new List<IObserver>();
    public void Add(IObserver observer)
    {
        _listObserver.Add(observer);
    }
    public void Remove(IObserver observer)
    {
        _listObserver.Remove(observer);
    }
    public void MeowObserver()
    {
        Console.WriteLine($"{GetType().Name} meow...");
        foreach (var item in _listObserver)
        {
            item.Action();
        }
    }

    public static void Show()
    {
        Console.WriteLine("************普通方法************");
        Cat cat = new Cat();
        cat.Meow();
        Console.WriteLine("************ 事  件 ************");
        cat.MeowHandler += () => new Brother().Turn();
        cat.MeowHandler += () => new Dog().Woof();
        cat.MeowHandler += () => new Father().Roar();
        cat.MeowHandler += () => new Mother().Whisper();
        cat.MeowHandler += () => new Mouse().Run();
        cat.MeowHandler += () => new Neighbor().Awake();
        cat.MeowHandler += () => new Stealer().Hide();
        cat.MeowEvent();
        Console.WriteLine("************观 察 者************");
        cat.Add(new Brother());
        cat.Add(new Dog());
        cat.Add(new Father());
        cat.Add(new Mother());
        cat.Add(new Mouse());
        cat.Add(new Neighbor());
        cat.Add(new Stealer());
        cat.MeowObserver();
    }
}
public interface IObserver
{
    void Action();
}
public class Baby : IObserver
{
    public void Action()
    {
        Cry();
    }

    public void Cry()
    {
        Console.WriteLine($"{GetType().Name} cry...");
    }
}
public class Brother : IObserver
{
    public void Action()
    {
        Turn();
    }

    public void Turn()
    {
        Console.WriteLine($"{GetType().Name} turn...");
    }
}
public class Dog : IObserver
{
    public void Action()
    {
        Woof();
    }

    public void Woof()
    {
        Console.WriteLine($"{GetType().Name} Woof...");
    }
}
public class Father : IObserver
{
    public void Action()
    {
        Roar();
    }

    public void Roar()
    {
        Console.WriteLine($"{GetType().Name} roar...");
    }
}
public class Mother : IObserver
{
    public void Action()
    {
        Whisper();
    }

    public void Whisper()
    {
        Console.WriteLine($"{GetType().Name} whisper...");
    }
}
public class Mouse : IObserver
{
    public void Action()
    {
        Run();
    }

    public void Run()
    {
        Console.WriteLine($"{GetType().Name} Run...");
    }
}
public class Neighbor : IObserver
{
    public void Action()
    {
        Awake();
    }

    public void Awake()
    {
        Console.WriteLine($"{GetType().Name} awake...");
    }
}
public class Stealer : IObserver
{
    public void Action()
    {
        Hide();
    }

    public void Hide()
    {
        Console.WriteLine($"{GetType().Name} hide...");
    }
}
```
