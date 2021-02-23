---
title: "PlantUML 的安装与使用"
date: "2020/05/16 14:27:44"
updated: "2020/05/16 14:36:16"
permalink: "plantuml-installation-and-use/"
tags:
 - UML
categories:
 - [开发, 工具]
---

工作中经常需要输出一些文档，需要叙述产品的设计、研发、使用等。

如果只是文字，肯定显得很晦涩，所以最好是结合一些图表，方便读者理解。

之前绘图主要是使用 `Visio`，提供的模板很多，搭配 Office 套件使用很舒服。但是 `Visio` 需要单独购买安装，并且价格略贵。

之前在微信看到有人在讨论 `PlantUML`，就搜索试用了一下，虽然有学习成本，但是相比 `Visio` 的拖拽，我更喜欢 `PlantUML` 编码生成图形的方式。

## 安装

`PlantUML` 的示例可以参考官方文档，这里建议使用 `VS Code` 搭配对应的扩展进行绘图。

`PlantUML` 官方中文文档：[https://plantuml.com/zh/](https://plantuml.com/zh/)。

### 安装 VS Code 插件

直接在插件里搜索 `PlantUML`，然后进行安装即可。

![plantuml extension](https://hd2y.oss-cn-beijing.aliyuncs.com/plantuml%20extension-e98aebd405bf42aabaa6140a51aff533.png)

### 安装 Java JDK

这里建议安装 `jdk8`，否则生成预览时，可能会有一些警告信息。

`jdk8` 下载路径：[https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)。

安装以后还需要配置系统环境变量：

```js
JAVA_HOME = 'C:\Program Files\Java\jdk1.8.0_251'
CLASSPATH = '.;%JAVA_HOME%\bin'
// 双击后选择“编辑文本” 然后将内容追加到最前
Path = 'C:\Program Files (x86)\Common Files\Oracle\Java\javapath;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;'
```

如果没有问题，可以在 `cmd` 中使用 `java -version` 看看能否正常使用 `java` 命令。

### 安装 Graphviz

虽然安装以后已经可以完成部分图形的绘制，但是测试发现部分图例还需要使用 `Graphviz`。

`Graphviz` 下载链接：[https://graphviz.gitlab.io/download/](https://graphviz.gitlab.io/download/)。

安装以后一样需要配置系统环境变量：

```js
GRAPHVIZ_DOT = 'C:\Program Files (x86)\Graphviz2.38\bin\dot.exe'
GRAPHVIZ_INSTALL_DIR = 'C:\Program Files (x86)\Graphviz2.38'
// 双击后新建
Path = 'C:\Program Files (x86)\Graphviz2.38\bin\'
```

## 绘制预览与导出

绘制可以参考前文提到的 `PlantUML` 的 [中文文档](https://plantuml.com/zh/)。

首先需要创建文件，需要注意的是支持的文件后缀名是 `*.wsd, *.pu, *.puml, *.plantuml, *.iuml`，否则没有高亮显示关键字、不能使用智能提示、无法使用格式化。

![edit uml diagram](https://hd2y.oss-cn-beijing.aliyuncs.com/edit%20uml%20diagram-1e44caaf640d4f90aaef16fde4c58743.png)

预览需要使用快捷键：`Alt + D`。
内容格式化：`Shift + Alt + F`。

导出首先使用 `Ctrl + Shift + P`，输入 `PlantUML`，使用 `PlantUML: Export Current File Diagrams`。

![export diagram](https://hd2y.oss-cn-beijing.aliyuncs.com/export%20diagram-cdff58005c58456f894b49ee7924820c.png)

> 参考：
> + PlantUML 官网：[https://plantuml.com/zh/](https://plantuml.com/zh/)
> + GitHub：[qjebbs / vscode-plantuml](https://github.com/qjebbs/vscode-plantuml)
