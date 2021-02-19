﻿---
title: "设计模式六大原则"
date: "2019/04/07 10:48:40"
updated: "2019/04/07 10:48:40"
permalink: "six-principles-of-design-mode"
tags:
 - 设计模式
categories:
 - [开发, 基础]
---

## 单一职责原则

（`Single Responsibility Principle` 简称 ：`SRP`）

定义：应该有且仅有一个原因引起类的变更。

接口的职责在设计时应该做到单一，降低类的复杂性，实现的职责都有明确的定义，提高了可读性、可维护性、可扩展性；

变更引起的风险降低，如果接口隔离性做的好，一个接口的修改只对相应实现类有影响，对其它接口没有影响，降低了耦合性。

> 单一职责提出了一个编写程序的标准，用“职责”或“变化原因”来衡量接口或类设计的是否优良，但是“职责”或“变化原因”都是不可度量的，因项目而异，因环境而异。

## 里氏替换原则

（`Liskov Substitution Principle` 简称：`LSP`）

定义：是面向对象设计的基本原则之一，只要父类能出现的地方子类就能出现，而且替换为子类也不会产生任何错误或异常。
- 子类必须完全实现父类的方法
- 子类可以有自己的个性
- 覆盖或实现父类的方法时输入参数可以被放大
- 覆盖或实现父类的方法时输出结果可以被缩小

优点：
- 代码共享，减少创建类的工作量
- 提高代码的重用性
- 子类可以形似父类，但又异于父类
- 提高代码的可扩展性
- 提高产品或项目的开放性

缺点：
- 继承是侵入性的，只要继承就必须拥有父类的所有属性和方法
- 降低代码的灵活性
- 增强了耦合性，父类的常量、变量、方法被修改时，要考虑到子类的修改，在缺乏规范的环境下，可能有许多代码需要重构。

> `LSP` 原则是继承复用的基石，只有当子类能够替换掉父类，软件单位功能不受影响时，父类才能被真正复用。`LSP` 原则是对开闭原则的补充，实现开闭原则的关键步骤就是抽象化，而父类与子类的继承关  系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。
  
## 依赖倒置原则

（`Dependence  Inversion Principle` 简称：`DIP`）

定义：是“面向接口编程” --面向对象设计的精髓之一，是六个原则中最难实现的一个原则，是实现开闭原则的重要途径。
- 模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，其依赖是通过接口和抽象类发生的
- 接口或抽象类不依赖于实现类
- 实现类依赖于接口或抽象类

优点：
- 可以减少类之间的耦合性，提高系统的稳定性
- 降低并行开发引起的风险
- 提高代码的可读性和可维护性

依赖的三种写法：
- 构造函数传递依赖对象
- setter方法传递依赖对象
- 接口声明依赖对象

本质是通过抽象使各个类或模块的实现彼此独立，不互相影响，实现模块间的松耦合，应遵循的原则：
- 每个类尽量都有接口或抽象类
- 变量的表面类型尽量是接口或者抽象类
- 任何类都不应从具体类中派生
- 尽量不要复写基类的方法
- 结合里氏替换原则使用

依赖倒置原则是设计模式；  
控制反转(`IoC`:将组件间的依赖关系从程序内部提到外部来管理)是目的；  
依赖注入(`DI`:将组件的依赖通过外部以参数或其他形式注入)是实现控制反转的方式。

## 接口隔离原则

（`Interface Segregation Principle` 简称：`ISP`）

定义：客户端不应依赖它不需要的接口，类间的依赖关系应建立在最小的接口上（降低耦合度）
- 接口尽量小（拆分接口时，首先必须满足单一职责原则）
- 接口要高内聚（尽量少公布 `public` 方法）
- 定制服务（模块间必然会有耦合，有耦合就要有相互访问的接口，设计时应为访问者提供定制服务）
- 接口的设计是有限度的（接口的设计粒度越小，接口越灵活，要把握一个度）

实践中可根据以下几个规则衡量：
- 一个接口只服务于一个子模块或业务逻辑
- 通过业务逻辑压缩接口中的public方法
- 已经被污染的接口，尽量去修改，若变更风险较大，则采用适配器模式进行转化处理
- 了解环境，拒绝盲从，环境不同，接口拆分的标准就不同，应深入了解业务逻辑，设计出灵活的接口

## 迪米特法则

（`Law of Demeter` 简称：`LoD`）又称最少知识原则

定义：一个类应该对自己需要耦合或调用的类知道的最少
- 只和直接的朋友交流
- 朋友之间也是有距离的
- 是自己的就是自己的（如果一个方法放在本类中，既不增加类间的关系，也对本类不产生负面影响，那就放在本类中）
- 谨慎使用 `Serializable` (远程调用传个vo，必须实现 `Serializable` 接口，如果有一天修改了访问权限，权限扩大了，但是服务器上没有做相应的变更，就会报序列化失败)

## 开闭原则

（Open-Closed Principle 简称：OCP）

定义：对扩展开放，对修改关闭，在程序需要扩展的时候，不能修改原有的代码，实现热插拔的效果。为了使程序的扩展性好，易于维护和升级，需要使用接口和抽象类。  

注意事项：
- 通过对接口或者抽象类约束扩展，对扩展进行边界限定，不允许出现在接口或抽象类中不存在的public方法
- 参数类型、引用对象尽量使用接口或抽象类，而不是实现类
- 抽象层尽量保持稳定，一旦确定不允许修改

如何使用：
- 抽象约束（实现对扩展开放的首要前提，对扩展进行边界限定）
- 元数据控制模块行为
- 制定项目章程（对项目来说，规定优于配置）
- 封装变化
  - 将相同的变化封装到一个接口或抽象类中；
  - 将不同的变化封装到不同的接口或抽象类中；