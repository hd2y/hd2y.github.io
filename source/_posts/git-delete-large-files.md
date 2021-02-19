---
title: "Git 删除大文件"
date: "2020/04/20 16:14:16"
updated: "2020/04/20 17:19:07"
permalink: "git-delete-large-files"
tags:
 - Git
 - BFG
categories:
 - [开发, 工具]
---

之前创建的一个 git 仓库提交了几个大文件，影响小伙伴的下载体验，于是便想把大文件删除掉，大文件通过网盘等方式分享。

试过几种方案，最终使用 `BFG Repo-Cleaner` 解决，这里记录处理的过程。

## 查看仓库大文件

打开 Git Bash，进入到本地仓库，运行以下命令：

```bash
hd2y@DESKTOP-V2RPVP4 MINGW64 /e/LIS接口/Instruments (master)
$ git rev-list --all | xargs -rL1 git ls-tree -r --long | sort -uk3 | sort -rnk4 | head -10
100644 blob e212baea0de1eceda3fb219bb5ef65e51dcfc047 29208376   "dist/Tools/Access\346\225\260\346\215\256\345\272\223\351\251\261\345\212\250/AccessDatabaseEngine_2010_X64.exe"
100644 blob 89d525d7930c40f25dbc4a70fe9043a2d5a50a0e 27162256   "dist/Tools/Access\346\225\260\346\215\256\345\272\223\351\251\261\345\212\250/AccessDatabaseEngine_2010_x86.exe"
100644 blob d9b021ae60274f40bfec0e6c9c95c4be4be62f45 26481656   "dist/Tools/Access\346\225\260\346\215\256\345\272\223\351\251\261\345\212\250/AccessDatabaseEngine_2007_x86.exe"
100644 blob af48c267a4c9b21fbd0db6f6dcdafc38e3f0c4c0 10513579   dist/WPF/LIS.Connector.WpfApp.0.2.2.7z
100644 blob 76a62fd5d9bb82b7d6725c187f0a3031e152d561 3862528    "dist/\345\205\266\344\273\226/\351\230\264\351\201\223\345\276\256\347\224\237\346\200\201 BPR-2014A MDB/Data/BV.mdb"
100644 blob bdb57e3515850b78ec861b60cef54a105f43974a 1048495    "dist/\347\224\265\346\263\263\344\273\252/\350\265\233\346\257\224\344\272\232 HYDRASYS/Data/\345\234\260\350\264\253\347\224\265\346\263\263\345\233\276\350\260\261\345\210\244\350\257\273\350\247\204\345\210\231.pptx"
100644 blob 8a2b843f42381c22fe8574dce59fa11c09410b99  758498    "dist/\347\224\265\346\263\263\344\273\252/\350\265\233\346\257\224\344\272\232 HYDRASYS/Data/Phoresis Extended 5.6.x.pdf"
100644 blob ea69b14549302725315f28b4999c8cd4837e4519  709825    "dist/\350\241\200\346\260\224\344\273\252/GEM3500/Data/GEM 3500\351\200\232\350\256\257\345\215\217\350\256\256 Interface Spec 6.X.pdf"
100644 blob db4d2d0f6c405a7e980511d6a1a7ac1290ad350c  330416    "dist/\345\205\266\344\273\226/\351\230\264\351\201\223\345\276\256\347\224\237\346\200\201 BPR-2014A MDB/Data/\351\230\264\351\201\223\345\276\256\347\224\237\346\200\201.jpg"
100644 blob f2a467b4fe413cb6fbc10223c6ade54d789d5db8   23120    "dist/\347\224\265\346\263\263\344\273\252/\350\265\233\346\257\224\344\272\232 HYDRASYS/Data/OUT.DAT"
```

需要注意的是，Git Bash 中使用 `cd` 进入仓库的路径要调整，将 `\` 修改为 `/`。

## 尝试删除

开始是尝试使用博客园 [Git 删除大文件的方法](https://www.cnblogs.com/bigmango/p/11361344.html) 的方案删除，出现如下错误。

```bash
hd2y@DESKTOP-V2RPVP4 MINGW64 /e/LIS接口/Instruments (master)
$ git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -g | tail -10
fatal: Cannot open existing pack file '.git/objects/pack/pack-*.idx'
.git/objects/pack/pack-*.pack: bad
```

后搜索该错误，在 Google 发现了 [git瘦身：清除大文件或敏感文件记录](https://easeapi.com/blog/blog/62-git-delete-big-file.html) 这篇文章。

这里不得不说，用百度搜出来的内容经常都是 `Ctrl C + Ctrl V` 的 :poop:。

## 使用 BFG 删除

### 安装 JDK

首先我们需要安装 JDK，因为该工具是 Java 开发，下载地址：[Java SE Downloads](https://www.oracle.com/java/technologies/javase-downloads.html)。

安装后还需要配置环境变量，否则无法使用 `java` 命令，当然也可以直接用 `java.exe` 的绝对路径。

```js
// 用户变量
JAVA_HOME = 'C:\Program Files\Java\jdk-14.0.1'
// 系统变量
CLASSPATH = '.;%JAVA_HOME%\lib'
Path = '.;%JAVA_HOME%\bin;'
```

> 注意：Path 如果是单个编辑框，直接追加到最前，如果是一个列表，那就增加 `.` 与 `%JAVA_HOME%\bin`。以上内容复制的时候，不要复制单引号 `'`。

如果安装正确，运行 `java -version` 可以看到当前的版本信息，注意配置后命令行需要重启才能生效。

### 下载 BFG

JDK 安装完成后，需要下载 BFG 工具：[BFG Repo-Cleaner](https://www.oracle.com/java/technologies/javase-jdk14-downloads.html)。

需要注意的是，这里提供的是 `maven` 的下载，国内下载会非常慢，所以建议挂代理。

### 下载 Git 仓库

这里处理完成后，就可以操作删除大文件了，首先需要重新下载仓库：

```bash
$ git clone --mirror git://example.com/some-big-repo.git
```

> 注：需要重新下载的主要原因就是下载时需要使用 `--mirror`，所以不要漏掉该参数。

### 删除文件并提交

使用 BFG 删除指定文件名的文件：

```bash
$ java -jar bfg.jar --delete-files (filename) some-big-repo.git
```

使用 BFG 删除超出指定大小的文件：

```bash
$ java -jar bfg.jar --strip-blobs-bigger-than 100M some-big-repo.git
```

以上只是更新所有的分支与提交，并没有真正的删除，所以需要进入仓库，执行以下命令：

```bash
$ cd some-big-repo.git
$ git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

删除以后就可以推送到远程分支了：

```bash
$ git push
Enumerating objects: 71, done.
Writing objects: 100% (71/71), 2.61 MiB | 589.00 KiB/s, done.
Total 71 (delta 0), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To git://example.com/some-big-repo.git
 + 07534d2...717da32 master -> master (forced update)
```

## 完成

虽然提交时，可以看到仓库大小已经变成了 `2.61 MiB`，但是登录 Gitea 后查看仓库大小仍然是 `91 MiB`。

![91MiB](https://www.hd2y.net/upload/2020/04/91MiB-ecf86eb91bf448788d5ab83fda4f94e5.png)

不过不用担心，直接下载该仓库内容，可以看到下载文件大小是正常的。

![ZIP File](https://www.hd2y.net/upload/2020/04/ZIP%20File-cadfc1ec62b540a19a3bd4a15c706a95.png)

> 参考：
> + CSDN：[彻底删除git中的较大文件（包括历史提交记录）](https://blog.csdn.net/HappyRocking/article/details/89313501)
> + CSDN：[jdk13.0.1配置显示‘java'不是内部或外部命令解决方法](https://blog.csdn.net/weixin_44853744/article/details/103389161)
> + GitHub：[BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
