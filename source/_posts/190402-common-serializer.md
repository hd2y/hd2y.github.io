---
title: "常见序列化器"
date: "2019/04/02 21:54:00"
updated: "2019/07/11 09:55:27"
permalink: "common-serializer/"
tags:
 - 序列化
 - Json
 - Xml
categories:
 - [开发, C#]
---

## 常见序列化器

### 二进制序列化器

命名空间 `System.Runtime.Serialization.Formatters.Binary;`

```csharp
//序列化
using (FileStream fileStream = new FileStream(path, FileMode.Create, FileAccess.ReadWrite))
{
    BinaryFormatter binaryFormatter = new BinaryFormatter();//创建二进制序列化器
    binaryFormatter.Serialize(fileStream, ListPersons);
}
//反序列化
using (FileStream fileStream = new FileStream(path, FileMode.Open, FileAccess.ReadWrite))
{
    BinaryFormatter binaryFormatter = new BinaryFormatter();//创建二进制序列化器
    fileStream.Position = 0;//重置流位置
    List<Human> list = (List<Human>)binaryFormatter.Deserialize(fileStream);
    foreach (var item in list)
    {
        item.Action();
    }
}
```

### `Soap` 序列化器

命名空间 `System.Runtime.Serialization.Formatters.Soap;`

```csharp
//序列化
using (FileStream fileStream = new FileStream(path, FileMode.Create, FileAccess.ReadWrite))
{
    SoapFormatter soapFormatter = new SoapFormatter();//创建Soap序列化器
    soapFormatter.Serialize(fileStream, ListPersons.ToArray());//不支持泛型
}
//反序列化
using (FileStream fileStream = new FileStream(path, FileMode.Open, FileAccess.ReadWrite))
{
    SoapFormatter soapFormatter = new SoapFormatter();//创建Soap序列化器
    fileStream.Position = 0;//重置流位置
    List<Human> list = ((Human[])soapFormatter.Deserialize(fileStream)).ToList();//不支持泛型
    foreach (var item in list)
    {
        item.Action();
    }
}
```

### XML序列化器

命名空间： `System.Xml.Serialization;`  

**XML：标记语言**

```csharp
//序列化
using (FileStream fileStream = new FileStream(path, FileMode.Create, FileAccess.ReadWrite))
{
    XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Human>));//创建Xml序列化器 需要指定对象的类型
    xmlSerializer.Serialize(fileStream, ListPersons);
}
//反序列化
using (FileStream fileStream = new FileStream(path, FileMode.Open, FileAccess.ReadWrite))
{
    XmlSerializer xmlSerializer = new XmlSerializer(typeof(List<Human>));//创建Xml序列化器 需要指定对象的类型
    fileStream.Position = 0;//重置流位置
    List<Human> list = (List<Human>)xmlSerializer.Deserialize(fileStream);
    foreach (var item in list)
    {
        item.Action();
    }
}
```

### `Json` 序列化器

命名空间：`System.Web.Script.Serialization;` 或引用 `System.Web.Extentsion;` 或者 `Newtonsoft.Json;`

```csharp
//JavaScriptSerializer序列化
{//内置
    JavaScriptSerializer javaScriptSerializer = new JavaScriptSerializer();//创建Json序列化器
    string sJson = javaScriptSerializer.Serialize(ListPersons);
    File.WriteAllText(path1, sJson);
}
//JavaScriptSerializer反序列化
{
    string sText = File.ReadAllText(path1);
    JavaScriptSerializer javaScriptSerializer = new JavaScriptSerializer();//创建Json序列化器
    List<Human> list = javaScriptSerializer.Deserialize<List<Human>>(sText);
    foreach (var item in list)
    {
        item.Action();
    }
}
//Newtonsoft.Json序列化
{//nuget
    string sJson = JsonConvert.SerializeObject(ListPersons);
    File.WriteAllText(path2, sJson);
}
//Newtonsoft.Json反序列化
{
    string sText = File.ReadAllText(path1);
    List<Human> list = JsonConvert.DeserializeObject<List<Human>>(sText);
    foreach (var item in list)
    {
        item.Action();
    }
}
```

> 常用 `Newtonsoft.Json` ，因其效率高于内置的 `JavaScriptSerializer` ，另如果使用内置序列化器，需要对应序列化的类型指定 `[Serializable]` 的特性。

