---
title: "C#中的文件操作"
date: "2019/04/02 18:01:00"
updated: "2019/07/11 10:41:45"
permalink: "file-operations-in-csharp/"
categories:
 - [开发, C#]
---

## 文件夹/文件 操作

### 文件夹/文件检查

主要是 `Directory` / `File` / `DirectoryInfo` / `FileInfo` 几个类来操作检查。

**注：Path是路径字符串的拼接、剪切、检查操作类，并不会执行IO操作。**

### 文件夹/文件新增

```csharp
Directory.CreateDirectory(path);//新增
File.Create(path);//新增或覆盖
File.AppendText(path);//新增或追加
```

> 文件夹新增时，dotnet会自动逐层创建，例如“C:\test1\test2”，“test1”文件夹不存在会先创建“test1”文件夹。

### 文件夹/文件复制

```csharp
//文件夹无法复制
File.Copy(path1, path2);
```

### 文件夹/文件移动

```csharp
Directory.Move(path1, path2);
File.Move(path1, path2);
```

### 文件夹/文件删除

```csharp
Directory.Delete(path, true);//第二个参数是指定是否删除目录内文件夹与文件 若不指定默认false 当为false文件夹内存在文件或文件夹会报错
File.Delete(path);
```
 
## 文件写入/读取

### 常见写入方法：

```csharp
//新增或覆盖
using (FileStream fileStream = File.Create(path))
{
    string sText = "My name is Li Lei.";
    byte[] bytes = Encoding.Default.GetBytes(sText);
    fileStream.Write(bytes, 0, bytes.Length);
}
//新增或覆盖
using (FileStream fileStream = File.Create(path))
{
    using (StreamWriter streamWriter = new StreamWriter(fileStream))
    {
        string sText = "My name is Han Meimei.";
        streamWriter.Write(sText);
    }
}
//新增或追加
using (StreamWriter streamWriter = File.AppendText(path))
{
    string sText = "Nice to meet you.";
    byte[] bytes = Encoding.Default.GetBytes(sText);
    streamWriter.BaseStream.Write(bytes, 0, bytes.Length);
}
//新增或追加
using (StreamWriter streamWriter = File.AppendText(path))
{
    string sText = "Nice to meet you too.";
    streamWriter.Write(sText);
}
```

### 常见读取方法：

```csharp
//一次读取
byte[] bytes = File.ReadAllBytes(path);
string sText = File.ReadAllText(path);
string[] aLineText = File.ReadAllLines(path);
IEnumerable<string> listLineText = File.ReadLines(path);
//大文件读取
using (FileStream fileStream = File.OpenRead(path))
{
    int length = 5;
    int result = 0;
    do
    {
        byte[] bytes = new byte[length];
        result = fileStream.Read(bytes, 0, 5);
        Console.WriteLine(Encoding.Default.GetString(bytes));
    } while (length == result);
}
```

### 磁盘操作

```csharp
DriveInfo[] aInfo = DriveInfo.GetDrives();
foreach (var info in aInfo)
{
    Console.WriteLine("----------------------------------------------------");
    Console.WriteLine("驱动器的名称：" + info.Name);
    Console.WriteLine("驱动器根目录：" + info.RootDirectory);
    Console.WriteLine("指示驱动器上可用空闲空间总量：" + info.AvailableFreeSpace);
    Console.WriteLine("文件系统名称：" + info.DriveFormat);
    Console.WriteLine("驱动器类型：" + info.DriveType);
    Console.WriteLine("获取一个指示驱动器是否已经准备好：" + info.IsReady);
    Console.WriteLine("获取驱动器上可用空闲空间总量：" + info.TotalFreeSpace);
    Console.WriteLine("获取驱动器空间总大小：" + info.TotalSize);
    Console.WriteLine("获取或设置驱动器的卷标：" + info.VolumeLabel);
}
```

## 后记

这部分原本感觉没必要提交上来，因为这部分内容太太基础。

但是实际工作中，经常会遗忘一部分方法怎么使用，所以提交上来供以后查阅。

另外由于经常翻看前辈代码，IO有部分需要特别注意一下：
- 多线程操作文件读写，一定要加锁，尽量避免使用`try`处理文件被占用引发的异常；
- 文件夹创建碰到多层级无需逐层创建，dotnet底层会自动帮我们逐层创建；
- 记录日志文件，内容使用流操作写文件，直接使用 `File.AppendText()` 方法即可，而且不用判断文件是否存在，dotnet会在不存在时自动帮我们创建；

先写这些，有些细节以后想到了再补。
