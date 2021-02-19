---
title: "泛型 （Part 1）"
date: "2019/03/31 18:12:00"
updated: "2019/07/11 10:56:31"
permalink: "generic-part-1"
tags:
 - 泛型
categories:
 - [开发, C#]
---

## 什么是泛型

起始版本为 *.net framework 2.0 CLR升级* 支持的，非语法糖。**为解决针对不同参数类型，有相同的操作行为。**

- 延迟声明：声明方法的时候并没有指定参数的类型，而是等调用的时候指定。
- 延迟思想：推迟一切可以推迟的。（提高程序的灵活性和扩展性）

打印泛型集合和字典：

```csharp
Console.WriteLine(typeof(List<>));
Console.WriteLine(typeof(Dictionary<,>));
```

输出内容

```  
System.Collections.Generic.List\`1[T]
System.Collections.Generic.Dictionary\`2[TKey,TValue]
```

编译的时候，类型参数被编译为占位符，程序运行时，JIT即时编译为是类型，所以其性能近似普通方法。

## 泛型方法

- 定义：`MethodName<T>(T t){}`  其中T是类型参数，名称不做限制。
- 使用：`MethodName<int>(123)` 或 `MethodName(123)`  当类型参数可以由编译器推断时，可以不指定参数类型。

### 普通方法

优点是直观、性能较好；缺点为相同逻辑不能重用，拓展性差。

```csharp
public static void ShowInt(int i)
{
    Console.WriteLine($"type:{i.GetType()}\tvalue:{i}");
}
public static void ShowString(string s)
{
    Console.WriteLine($"type:{s.GetType()}\tvalue:{s}");
}
public static void ShowDateTime(DateTime dt)
{
    Console.WriteLine($"type:{dt.GetType()}\tvalue:{dt}");
}
```

### object 方法

优点是基类型可以用子类代替，解决相同逻辑复用问题；缺点是存在拆装箱，影响性能。

```csharp
public static void ShowObject(Object o)
{
    Console.WriteLine($"type:{o.GetType()}\tvalue:{o}");
}
```

### 泛型方法

可以复用，同时不存在性能问题。

```csharp
public static void Show<T>(T t)
{
    Console.WriteLine($"type:{t.GetType()}\tvalue:{t}");
}
```

### 调用

```csharp
//普通方法
CommonMethod.ShowInt(123);
CommonMethod.ShowString("123");
CommonMethod.ShowDateTime(DateTime.Now);
//object方法
CommonMethod.ShowObject(123);
CommonMethod.ShowObject("123");
CommonMethod.ShowObject(DateTime.Now);
//泛型方法
CommonMethod.Show(123);
CommonMethod.Show("123");
CommonMethod.Show(DateTime.Now);
```

输出内容  

```
type:System.Int32   value:123
type:System.String  value:123
type:System.DateTime    value:2018/5/1 20:14:33
type:System.Int32	value:123
type:System.String	value:123
type:System.DateTime	value:2018/5/1 20:14:33
type:System.Int32	value:123
type:System.String	value:123
type:System.DateTime	value:2018/5/1 20:14:33
```

> 注：object是一切类型的父类；通过继承，子类拥有父类的一切属性和行为；任何子类出现的地方，都可以用父类来代替；值类型转换为引用类型是装箱，引用类型转换为值类型是拆箱。

## 其他使用场景

除泛型方法以外，还可以使用泛型类、泛型接口、泛型委托。

### 1. 泛型类

```csharp
public class GenericClass<T1, T2>
{
    public void Show<T1>(T1 t1){ }
	public T2 Get()
	{
	    return default(T2);
	}
}
```

### 2. 泛型接口

```csharp
public class IGeneric<T>
{
    T IMethod(T t);
}
```

### 3. 泛型委托

```csharp
public delegate T GetHandler<T>();
```

> 注：普通类不能直接继承自泛型类也不能直接实现泛型接口，指定参数类型后才可以。泛型类可以继承泛型类。

```csharp
public class GenericClass<T1, T2>{}
public class Child: GernericClass<int, string>, IStudy<DateTime>{}
public class ChildGeneric<T1, T2>: GenericClass<T1, T2>{}
```
