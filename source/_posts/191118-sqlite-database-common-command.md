---
title: "SQLite 数据库常用命令"
date: "2019/11/18 10:04:45"
updated: "2019/11/18 10:04:45"
permalink: "sqlite-database-common-command/"
categories:
 - [开发, 数据库, SQLite]
---

## 压缩数据库

删除数据或更新数据后，数据库不断增大，可以使用一下命令对数据库进行压缩。

```sql
VACUUM;
```
