---
title: "解决 Access 数据文件打开报错：无法打开应用程序较早版本创建的数据库"
date: "2019/09/15 12:55:49"
updated: "2020/02/11 13:26:38"
permalink: "resolve-access-data-file-opening-error-unable-to-open-database-created-by-earlier-version-of-the-application/"
tags:
 - DBeaver
categories:
 - [开发, 数据库]
---

处理 LIS 接口，经常会遇到需要和 Access 数据库对接读写数据，可能需要打开对方数据库查看数据库表结构以及数据。

正常我们安装 Office 以后会附带有 Access，可以打开数据库，浏览数据以及执行 SQL。

但是我并不推荐使用 Office 里的 Access 软件，原因如下：

1. 安装麻烦，新版本需要购买（我订阅了 Office 365 这倒没什么）；
2. 软件本身操作不友好，和其他 Office 软件类似的工具栏，在数据库操作中需要经常使用的一些功能下略显尴尬与低效，例如切换窗口或者创建查询；
3. 虽然支持 mac，但是不能在 Linux 下使用；
4. 对旧版本，例如 97 之前创建的数据库不支持会提示：无法打开应用程序较早版本创建的数据库；
   > ![20190915111827.png](https://hd2y.oss-cn-beijing.aliyuncs.com/20190915111827_1568523773853.png)

特别是针对第四点，网上去搜索 Access 数据库管理软件，能找到的资料其实很少。

开始，我会使用 Office 的 Access 软件，搭配一个简陋的从网上找到的工具来处理 Access 数据库文件，但是这就相对比较麻烦，特别是教同事怎么处理 Access 数据库文件的时候，他们经常会问“怎么创建一个查询窗”以及“这个数据库文件怎么打不开，是不是损坏了”？

这个问题曾经困扰我很长一段时间，不过因为有解决方案，就没有继续追究，直到我遇到了“DBeaver”。

开始接触这个软件，只是因为需要找一个可以连接各种数据库的工具，开始是用 Navicat，但是因为其需要收费，并且只支持五种数据库的连接：Oracle、MSSQL、SQLite、MySQL、PostgreSQL。

后来无意中打开了这个软件的官网：[https://dbeaver.io/](https://dbeaver.io/)，惊讶于居然可以支持那么多种数据库，便下载下来试了一下，真的是只能说：真香。

1. 安装软件不到 50MB（不过连接不同的数据库需要安装不同的“驱动”），并且国内下载速度也很快（我试了一下科学上网反而速度慢了很多）；
2. 社区版免费，企业完整版可以按月付费，每个月 19USD，虽然略贵，但是社区版已经支持“MySQL, PostgreSQL, SQLite, Oracle, DB2, SQL Server, Sybase, MS Access, Teradata, Firebird, Hive, Presto”等等数据库类型，所以免费版已经很香了（注意看下图的滚动条）；
   > ![20190915121837.png](https://hd2y.oss-cn-beijing.aliyuncs.com/20190915121837_1568523644651.png)
3. 在各种数据库下均有一致的体验，和其他数据库管理软件体验也很类似，所以上手很简单；
4. 支持查看数据库对象表、视图、存储过程等等；
5. SQL 编辑有智能提示，除关键字外表和视图名称也有；
6. SQL 编辑右键可以看到有 SQL 美化；
7. 查询数据会自动分页，并且查询的数据可以实时的修改并提交；
8. 如果是二进制数据例如文本和图片也可以很方便的查看；
9. 跨平台，出 Windows 以外还支持 mac 以及 linux；
10. 支持导入/导出表数据以及查询数据、支持查看数据表 DDL；

![20190915121018.png](https://hd2y.oss-cn-beijing.aliyuncs.com/20190915121018_1568523760385.png?x-oss-process=image/auto-orient,1/resize,p_50/quality,q_50)

因为优点太多就不一一列举了，可以自己下载体验。

另外回到文章标题，Access97 以前的版本也是支持打开的，可以进行查询，但是因为 jdbc 的问题，旧版本的 Access 仅支持查询，不支持增删改，不过这也基本上够用了，后面的问题就用代码来解决吧。

```csharp
using System;
using System.Data.OleDb;

namespace JohnSun.Tests
{
    class Program
    {
        static void Main(string[] args)
        {
            // 需要注意以下代码需要x86平台运行
            using (OleDbConnection conn = new OleDbConnection(@"Provider=Microsoft.Jet.OLEDB.4.0;Data Source=C:\Users\hd2y\Downloads\Data .mdb;"))
            {
                conn.Open();
                using (OleDbCommand comm = conn.CreateCommand())
                {
                    comm.CommandText = "select count(1) from items";
                    int count = (int)comm.ExecuteScalar();
                    Console.WriteLine($"Items表共有{count}条数据！");
                }

                using (OleDbCommand comm = conn.CreateCommand())
                {
                    comm.CommandText = "update items set MRU=@mru";
                    comm.Parameters.Add(new OleDbParameter("@mru", "test"));
                    int count = comm.ExecuteNonQuery();
                    Console.WriteLine($"共更新Items表{count}条记录！");
                }
            }
        }
    }
}
```

> 注意：偶然间发现有些 Access 数据库，DBeaver 打开也会报错，建议使用 Visual Studio 的数据库连接工具，可以通过 `工具` - `连接到数据库` 打开。

> 附录：
> + DBeaver下载地址：[https://dbeaver.io/download/](https://dbeaver.io/download/)
> + 连接字符串检索：[https://www.connectionstrings.com/](https://www.connectionstrings.com/)
