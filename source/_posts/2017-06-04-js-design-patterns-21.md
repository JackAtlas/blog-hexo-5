title: 《JavaScript 设计模式与开发实战》读书笔记 21
date: 2017-06-04 11:11:11

---

第二十一章 接口和面向接口编程
<!-- more -->

接口通常会涉及以下几种含义：

1. 一个库或者模块通过主动暴露的接口来通信，可以隐藏软件系统内部的工作细节。
2. 一些语言提供的关键字，比如 Java 的 `interface`。
3. 面向接口编程中的接口。接口是对象能响应的请求的集合。

本章主要讨论的是第二和第三种。

## 21.1 Java 的抽象类

抽象类的一些作用：

- 向上转型。让 `Duck` 对象和 `Chicken` 对象的类型都隐藏在 `Animal` 类型身后，隐藏对象的具体类型之后，`duck` 对象和 `chicken` 对象才能被交换使用。
- 建立一些契约。继承自抽象类的具体类都会继承对象类里的 `abstract` 方法，并且要求覆盖它们。比如在命令模式中，各个子命令类都必须实现 `execute` 方法，才能保证在调用 `command.execute` 的时候不会抛出异常。

不关注对象的具体类型，而仅仅针对超类型中的“契约方法”来编写程序，可以产生可靠性高的程序，也可以极大地减少子系统实现之间的相互依赖关系：

> 面向接口编程，而不是面向实现编程。

从过程上看，“面向接口编程”其实是“面向超类型编程”。当对象的具体类型被隐藏在超类型身后，这些对象就可以相互替换使用，我们的关注点才能从对象的类型上转移到对象的行为上。“面向接口编程”也可以看成面向抽象编程，即针对超类型中的 `abstract` 方法编程，接口在这里被当成 `abstract` 方法中约定的契约行为。这些契约行为暴露了一个类或者对象能够做什么，但是不关心具体如何去做。

## 21.2 interface

使用 `interface` 实际上也是继承的一种方式，叫做接口继承。

相对于单继承的抽象类，一个类可以实现多个 `interface`。抽象类中除了 `abstract` 方法外还有一些供子类公用的具体方法。`interface` 则产生一个完全抽象的类，不提供具体实现和方法体，允许创建者确定方法名、参数列表和返回类型。

`interface` 同样可用于向上转型。

## 21.3 JavaScript 语言是否需要抽象类和 interface

抽象类和 `interface` 的作用主要是两点：

- 通过向上转型来隐藏对象的真正类型，以表现对象的多态性。
- 约定类与类之间的一些契约行为。

JavaScript 是一门动态类型语言，除了 number、string、boolean 等基本数据类型之外，其他的对象都可以被堪称“天生”被“向上转型”成 Object 类型。

因为不需要进行向上转型，接口在 JavaScript 中的最大作用就退化到了检查代码的规范性。比如检查某个对象是否实现了某个方法，或者检查是否给函数传入了预期类型的参数。

作为一门解释执行的动态类型语言，没有编译器帮助我们检查代码的规范性，只能手动编写一些接口检查的代码。

## 21.4 用鸭子类型进行接口检查

鸭子类型是动态类型语言面向对象设计的一个重要概念。利用鸭子类型的思想，不必借助超类型的帮助，就能在动态类型语言中轻松地实现本章提到的设计原则：面向接口编程，而不是面向实现编程。

当然在 JavaScript 开发中，总是进行接口检查是不明智的，也是没必要的。

## 21.5 用 TypeScript 编写基于 interface 的命令模式

```typescript
interface Command {
  execute: Function;
}

class RefreshMenuBarCommand implements Command {
  
  constructor() {}
  execute() {
    console.log('刷新菜单界面')
  }
}

class AddSubMenuCommand implements Command {
  
  constructor() {}
  execute() {
    console.log('增加子菜单')
  }
}

class DelSubMenuCommand implements Command {
  constructor() {}
  // 忘记重写 execute 方法
}

var refreshMenuBarCommand = new RefreshMenuBarCommand(),
    addSubMenuCommand = new AddSubMenuCommand(),
    delSubMenuCommand = new DelSubMenuCommand();

refreshMenuBarCommand.execute(); // 输出：刷新菜单界面
addSubMenuCommand.execute(); // 输出：增加子菜单
delSubMenuCommand.execute(); // 输出：Uncaught TypeError:undefined is not a function
```

如图，TypeScript 提供的编译器及时给出了错误信息。
