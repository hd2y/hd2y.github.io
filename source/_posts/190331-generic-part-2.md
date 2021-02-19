---
title: "泛型（Part 2）"
date: "2019/03/31 22:10:00"
updated: "2019/07/11 11:08:56"
permalink: "generic-part-2/"
tags:
 - 泛型
categories:
 - [开发, C#]
---

## 泛型约束

### 基类约束

可以访问基类的属性和方法，限制以后参数类型只能是该类型或其子类。

```csharp
public static void GenericMethod<T>(T t) where T :GenericClass{}
```

### 接口约束

可以访问接口的属性和方法，限制以后参数类型必须有实现该接口。

```csharp
public static void GenericMethod<T>(T t) where T : IGeneric{}
```

### 引用类型约束

限制参数类型只能是引用类型。

```csharp
public static void GenericMethod<T>(T t) where T : class{}
```

### 值类型约束

限制参数类型只能是值类型。

```csharp
public static void GenericMethod<T>(T t) where T : struct{}
```

### 无参数构造函数约束

限制参数类型可以被无参数构造函数实例化。

```csharp
public static void GenericMethod<T>(T t) where T : new(){}
```

> 注：以上可以根据需求多重约束，叠加使用，其是“且”的关系。

```csharp
public static void GenericMethod<T>(T t) where T : GenericClass,IGeneric{}
```

## 协变与逆变

### 使用场景

逆变（contravariant）与协变（covariant）是C#4新增的概念，在此之前泛型的参数是不能变化的，无论是“逆”还是“顺”(协)。

而在泛型参数上添加了in关键字作为泛型修饰符的话，那么那个泛型参数就只能用作方法的输入参数，或者只写属性的参数，不能作为方法返回值等，总之就是只能是“入”，不能出。out关键字反之。

- 协变(covariant)：out 修饰返回值。例如 `Func<out T>`
- 逆变(contravariant)：in 修饰传入参数。例如 `Action<in T>`

### 应用

```csharp
public class Bird{}

public class Sparrow:Bird{}
{
    Bird bird1 = new Bird();
    Bird bird2 = new Sparrow();
    Sparrow sparrow1 = new Sparrow();
    //Sparrow sparrow2 = new Bird(); //麻雀是鸟 但是鸟未必是麻雀
}

public interface ICustomerOut<out T>
{
    T Get();//协变 只能作为返回值 不能作为传入参数
}

public interface ICustomerIn<in T>
{
    void Set(T t);//逆变 只能作为传入参数 不能作为返回值
}

public class CustomerOut<T> : ICustomerOut<T>
{
    public T Get()
    {
        return default(T);
    }
}

public class CustomerIn<T> : ICustomerIn<T>
{
    public void Set(T t)
    {

    }
}
```

```csharp
{
    List<Bird> listBird1 = new List<Bird>();
    //List<Bird> listBird2 = new List<Sparrow>();    //不是父子关系，没有继承关系
    List<Bird> listBird3 = new List<Sparrow>().Select(s => s as Bird).ToList();//一群麻雀一定是一群鸟
}

{//协变 只能作为返回值 不能作为传入参数
    IEnumerable<Bird> listBird1 = new List<Bird>();
    IEnumerable<Bird> listBird2 = new List<Sparrow>(); //协变 一群麻雀一定是一群鸟 类型转换由编译器执行
    //IEnumerable<Sparrow> listBird3 = new List<Bird>();
    
    ICustomerOut<Bird> customer4 = new CustomerOut<Bird>();
    ICustomerOut<Bird> customer5 = new CustomerOut<Sparrow>();
    //ICustomerOut<Sparrow> customer6 = new CustomerOut<Bird>();//接口应用
    
    Action<int> action = (a) => { };//委托应用
}

{//逆变 只能作为传入参数 不能作为返回值
    ICustomerIn<Bird> customer1 = new CustomerIn<Bird>();
    //ICustomerIn<Bird> customer2 = new CustomerIn<Sparrow>();
    ICustomerIn<Sparrow> customer3 = new CustomerIn<Bird>();//接口应用
    
    Func<int> func = () => { return default(int); }; //委托应用
}
```

### 总结

逆变(in) 英语单词字面理解就是入参，而中文的字面意思就是逆向的即从子向父转换，可以参考系统提供的委托 `Action<T>`。

`Action<T>` 委托这样使用可以理解为一个已经限制了入参为派生类的委托，那么这个委托将不能赋值给入参为基类型的委托。也就是说如果已经限制了一个委托的入参类型，那么这个委托的入参只能是该类型或该类型的派生类型，否则会影响委托内方法、属性等的调用。所以反之我们可以将一个入参为基类型的委托赋值给一个入参是派生类型的委托。

```csharp
// public delegate void Action<in T>(T obj);
Action<Animal> actAnimal = (animal) =>
{
    animal.Eat();
    //animal.Meow();
};
Action<Cat> actCat = (cat) =>
{
    cat.Eat();
    cat.Meow();
};
// actAnimal = actCat; //报错：猫咪会喵喵叫，但是不是所有动物都会喵喵叫。
actCat = actAnimal;
```

协变(out) 英语单词字面理解就是出参，而中文的字面意思就是正向(协有顺的意思)的即从父向子转换，可以参考系统提供的委托 `Func<T>`。

`Func<T>` 委托这样使用可以理解为一个已经限制返回值为基类型的委托，那么这个委托将不能赋值给返回值为派生类的委托。也就是说已经限制了一个委托的返回值类型，那么这个委托的返回值只能是该类型或该类型的基类型，因为我们并不能确定该返回值的类型是否是该派生类型。所以反之我们可以将返回值为派生类的委托赋值给一个返回值为基类型的委托。

```csharp
// public delegate TResult Func<out TResult>();
Func<Animal> funcAnimal = () =>
{
    Animal animal = null;
    if (new Random().Next() % 2 == 0)
    {
        animal = new Animal();
    }
    else
    {
        animal = new Cat();
    }
    return animal;
};
Func<Cat> funcCat = () =>
{
    Cat cat = null;
    if (new Random().Next() % 2 == 0)
    {
        //cat = new Animal();
    }
    else
    {
        cat = new Cat();
    }
    return cat;
};
funcAnimal = funcCat;
// funcCat = funcAnimal;//报错：猫咪是动物，但是动物未必是喵咪。
```

> 如果感觉协变和逆变的内容难以理解，可以参看博客园的文章[逆变与协变详解](http://www.cnblogs.com/lemontea/archive/2013/02/17/2915065.html)，实际我们只需要了解基本概念以及学会如何使用即可，对于内部的实现原理不必深究。

## 泛型缓存

因为每一个泛型类型，都会生成不同副本，适合不同类型，需要缓存一份数据的场景。而无论是线程安全性还是检索速度，泛型缓存都要比字典类型表现更佳。

```csharp
public class GenericCache<T>
{
    static GenericCache()
    {
        GetCache();
    }
	
    public GenericCache()//实例化 相当于手动更新缓存
    {
	    _cache = null;
    }
	
	private static List<T> _cache = null;
    
	public static List<T> GetCache()
    {
        if (_cache == null)
        {
            //为缓存赋值 若需要实时更新 可以自己设置策略检查更新是否过期
            //_cache = DbHelper.GetAll<T>();
        }
        return _cache;
    }
}
```
