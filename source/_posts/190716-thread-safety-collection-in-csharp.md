---
title: "C# 中的线程安全集合"
date: "2019/07/16 08:55:53"
updated: "2020/02/11 14:05:52"
permalink: "thread-safety-collection-in-csharp/"
tags:
 - 多线程 
categories:
 - [开发, C#]
---

前几天很久以前做的一个接口被反馈经常会报错崩溃，查看系统日志为某个字段为空的错误。

排查程序代码发现该类型是 `List` 集合，虽然程序启动时会给该变量初始化，但是因为会在多线程中访问这个集合，所以在多线程中访问 `List` 集合操作集合进行增删操作时，可能会由于 `List` 在“扩容”变成了一个空对象，而在此时访问就会出现上述问题。

解决方案当然也是很简单，因为 `List` 集合不是线程安全的集合，可以在操作集合的位置加锁。但是由于集合访问的位置较多，所以干脆调整代码使用 `C#` 自带的线程安全集合 `ConcurrentBag`。

## C#中的线程安全集合

`System.Collections.Concurrent` 命名空间提供多个线程安全集合类。当有多个线程并发访问集合时，应使用这些类代替 `System.Collections` 和 `System.Collections.Generic` 命名空间中的对应类型。 但是，不保证通过扩展方法或通过显式接口实现访问集合对象是线程安全的，可能需要由调用方进行同步。

| 类 | 说明 |
|:---|:-----|
| BlockingCollection<T> |为实现 IProducerConsumerCollection<T> 的线程安全集合提供阻塞和限制功能。|
| ConcurrentBag<T> | 表示对象的线程安全的无序集合。|
| ConcurrentDictionary<TKey,TValue> | 表示可由多个线程同时访问的键/值对的线程安全集合。|
| ConcurrentQueue<T> | 表示线程安全的先进先出 (FIFO) 集合。|
| ConcurrentStack<T> | 表示线程安全的后进先出 (LIFO) 集合。|
| OrderablePartitioner<TSource> | 表示将一个可排序数据源拆分成多个分区的特定方式。|
| Partitioner | 提供针对数组、列表和可枚举项的常见分区策略。|
| Partitioner<TSource> | 表示将一个数据源拆分成多个分区的特定方式。|

| 接口 | 说明 |
|:-----|:-----|
| IProducerConsumerCollection<T> | 定义供制造者/使用者用来操作线程安全集合的方法。 此接口提供一个统一的表示（为生产者/消费者集合），从而更高级别抽象如 BlockingCollection<T> 可以使用集合作为基础的存储机制。|

> 参考：
> + MSDN：[System.Collections.Concurrent Namespace](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.concurrent)
