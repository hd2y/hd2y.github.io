---
title: "如何修复损坏的SQLite数据库文件"
date: "2019/05/21 11:39:38"
updated: "2019/07/10 18:43:16"
permalink: "how-to-repair-corrupted-sqlite-database-files/"
categories:
 - [开发, 数据库, SQLite]
---

最近处理过一个SQLite数据库因为未知原因损坏，无法进行数据操作的而导致软件无法正常使用的问题。

虽然工具软件是基于`Code First`开发，可以直接删除数据库来重新生成数据文件，但是因为里面存储了很多基本参数，运维人员认为重新设置这些参数比较麻烦，所以希望能够提供一个方案来修复已经损坏的数据库。

和运维沟通后将数据库文件导回，尝试跟踪问题。从网上查询到解决方案，这里简单做一个记录。

## 排查

使用SQLite数据库管理工具`SQLite Studio`进行数据查询操作，访问正常，尝试直接删除某一个表的数据出现如下错误提示：

```html
[10:42:22] Error while executing SQL query on database 'data': database disk image is malformed
```

从网上查找如何排查错误，发现一个语句可以对SQLite数据库进行检测：

```sql
PRAGMA integrity_check;
```

以上语句是执行整个库的完全性检查，会查看错序的记录、丢失的页，毁坏的索引等。 

执行后我获得的信息是：

```html
"*** in database main ***
Main freelist: freelist leaf count too big on page 138339
Main freelist: invalid page number 218103808
On tree page 42 cell 176: 2nd reference to page 138339
Page 18 is never used
Page 20 is never used
Page 23 is never used
Page 25 is never used
Page 26 is never used
Page 27 is never used
Page 28 is never used
Page 29 is never used
Page 30 is never used
Page 31 is never used
Page 32 is never used
Page 33 is never used
Page 34 is never used
Page 35 is never used
Page 36 is never used
Page 37 is never used
Page 38 is never used
Page 39 is never used
Page 40 is never used
Page 41 is never used
Page 43 is never used
Page 44 is never used
Page 45 is never used
Page 46 is never used
Page 47 is never used
Page 48 is never used
Page 49 is never used
Page 50 is never used
Page 51 is never used
Page 52 is never used
Page 53 is never used
Page 54 is never used
Page 55 is never used
Page 56 is never used
Page 57 is never used
Page 58 is never used
Page 59 is never used
Page 60 is never used
Page 61 is never used
Page 62 is never used
Page 63 is never used
Page 64 is never used
Page 65 is never used
Page 66 is never used
Page 67 is never used
Page 68 is never used
Page 69 is never used
Page 70 is never used
Page 71 is never used
Page 73 is never used
Page 74 is never used
Page 75 is never used
Page 76 is never used
Page 77 is never used
Page 78 is never used
Page 79 is never used
Page 80 is never used
Page 81 is never used
Page 82 is never used
Page 83 is never used
Page 84 is never used
Page 85 is never used
Page 86 is never used
Page 88 is never used
Page 89 is never used
Page 90 is never used
Page 91 is never used
Page 92 is never used
Page 93 is never used
Page 94 is never used
Page 95 is never used
Page 96 is never used
Page 97 is never used
Page 98 is never used
Page 99 is never used
Page 100 is never used
Page 101 is never used
Page 102 is never used
Page 103 is never used
Page 104 is never used
Page 105 is never used
Page 106 is never used
Page 107 is never used
Page 108 is never used
Page 110 is never used
Page 111 is never used
Page 112 is never used
Page 113 is never used
Page 114 is never used
Page 115 is never used
Page 116 is never used
Page 117 is never used
Page 118 is never used
Page 119 is never used
Page 120 is never used
Page 121 is never used
Page 122 is never used"
```

后面的问题就简单了，从网上搜索这个错误可以获得解决方案，基本原理就是将数据库内容导出脚本，然后通过这个脚本重新构建数据库。

可以参考：[https://techblog.dorogin.com/sqliteexception-database-disk-image-is-malformed-77e59d547c50](https://techblog.dorogin.com/sqliteexception-database-disk-image-is-malformed-77e59d547c50)

## 解决

首先我们需要下载SQLite数据库的命令行管理工具：sqlite3.exe，命令行工具如何使用可以参考：[https://www.sqlite.org/cli.html](https://www.sqlite.org/cli.html)

我们可以SQLite官网下载命令行工具，比如我下载的是`sqlite-tools-win32-x86-3280000.zip`，可以在这里找到：[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)

解压文件，并将数据文件拷贝到同级目录。

执行导出：
1. 双击`sqlite3.exe`运行SQLite数据库命令行工具。
2. 执行`.open data.db`打开我们需要导出数据脚本的数据文件。
3. 执行`.mode insert`将导出脚本模式修改为用于执行INSERT。
4. 执行`.output dump_all.sql`指定导出文件的未知与文件名。
5. 执行`.dump`执行导出。（需要注意的是如果数据库文件较大，可能需要耐心等待一段时间。）

如果内容较多，可能执行导出耗时会比较长，需要耐心等待。

执行完成后可以在文件夹下找到导出的脚本文件。

使用SQL文件生成数据库文件：
1. 关闭原有的命令行工具，重新打开。
2. 执行`.open data.fixed.db`，`data.fixed.db`就是我们恢复后数据库的数据库名。
3. 执行`.read dump_all.sql`将数据脚本加载到创建的数据库中，等待执行完成即完成数据库的修复。

执行成功会生成了一个数据库文件。

我们可以使用文章开篇所使用数据库管理工具尝试检索数据库的状态，如果反馈数据正常，则恢复工作完成。

```sql
PRAGMA integrity_check;
```
