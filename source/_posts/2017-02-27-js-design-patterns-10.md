title: 《JavaScript 设计模式与开发实战》读书笔记 10
date: 2017-02-27 09:12:11

---

## 第十章 组合模式
<!-- more -->

> 组合模式就是用小的子对象来构建更大的对象，而这些小的子对象本身也许是由更小的“孙对象”构成的。

### 10.1 回顾宏命令

在第九章命令模式中，宏命令对象包含了一组具体的子命令对象，不管是宏命令对象，还是子命令对象，都有一个 `execute` 方法负责执行命令。

宏命令中包含了一组子命令，它们组成了一个树形结构，`macroCommand` 被称为组合对象，`closeDoorCommand` 等都是叶对象。在 `macroCommand` 的 `execute` 方法里，并不执行真正的操作，而是遍历它所包含的叶对象，把真正的 `execute` 请求委托给这些叶对象。

`macroCommand` 表现得像一个命令，但实际上只是一组真正命令的“代理”。并非真正的代理，虽然结构上相似，但 `macroCommand` 只负责传递请求给叶对象，目的不在于控制对叶对象的访问。

### 10.2 组合模式的用途

组合模式将对象组合成树形结构，以表示“部分-整体“的层次结构。通过对象的多态性表现，使得用户对单个对象和组合对象的使用具有一致性。

当宏命令和普通子命令接收到执行 `execute` 方法的请求时，宏命令和普通子命令都会做它们认为正确的事情。这些差异是隐藏在背后的，这种透明性让我们非常自由地扩展程序。

### 10.3 请求在树中传递的过程

请求从树最顶端的对象往下传递，如果当前处理请求的对象是叶对象，叶对象自身会对请求作出相应的处理；如果当前处理请求的对象是组合对象，组合对象则会遍历它属下的子节点，将请求继续传递给这些子节点。

作为客户，只需关心树最顶层的组合对象，只要请求这个组合对象，请求便会沿着树往下传递，依次到达所有的叶对象。

### 10.4 更强大的宏命令

```javascript
var MacroCommand = function () {
  return {
    commandList: [],
    add: function (command) {
      this.commandList.push(command);
    },
    execute: function () {
      for (var i = 0, command; command = this.commandList[i++];) {
        command.execute();
      }
    }
  }
};

var aCommand = {
  execute: function () {
    console.log('a');
  }
};

// ...

var macroCommand1 = MacroCommand();
macroCommand1.add(aCommand);
macroCommand1.add(bCommand);

var macroCommand2 = MacroCommand();
macroCommand2.add(cCommand);
macroCommand2.add(dCommand);
macroCommand2.add(eCommand);

var macroCommand = MacroCommand();
macroCommand.add(fCommand);
macroCommand.add(macroCommand1);
macroCommand.add(macroCommand2);

var setCommand = (function (command) {
  document.getElementById('button').onclick = function () {
    command.execute();
  }
})(macroCommand);
```

基本对象可以被组合成更复杂的组合对象，组合对象又可以被组合，这样不断递归下去，这棵树的结构可以支持任意多的复杂度。在树最终被构造完成之后，让整棵树运行的步骤，只是调用最上层对象的 `execute` 方法。每当对最上层对象进行一次请求时，实际上是在对整棵树进行深度优化的搜索，而创建组合对象的程序员并不关心这些内在的细节，往树里面添加新的节点对象是非常容易的。

### 10.5 抽象类在组合模式中的作用

组合模式最大的优点在于可以一致地对待组合对象和基本对象。客户不需要知道当前处理的是宏命令还是普通命令，只要是命令，并且有 `execute` 方法，就可以被添加到树中。

这种透明性带来的便利，在静态类型语言中体现得尤为明显。比如在 Java 中，实现组合模式的关键是 `Composite` 类和 `Leaf` 类都必须继承在一个 `Composite` 抽象类。这个 `Composite` 抽象类既代表组合对象，又代表叶对象，能保证组合对象和叶对象拥有同样名字的方法，从而可以对同一消息都做出反馈。组合对象和叶对象的具体类型被隐藏在 `Component` 抽象类身后。

针对 `Component` 抽象类来编写程序，客户操作的始终是 `Component` 对象，而不用去区分到底是组合对象还是叶对象。

然而在 JavaScript 这种动态类型语言中，对象的多态性是与生俱来的，也没有编译器去检查变量的类型，所以我们通常不会去模拟一个“怪异”的抽象类，JavaScript 中实现组合模式的难点在于要保证组合对象和叶对象拥有同样的方法，这通常需要鸭子类型的思想对它们进行接口检查。

在 JavaScript 中实现组合模式，看起来缺乏一些严谨性，代码算不上安全，但能更快速和自由地开发，这既是 JavaScript 的缺点，也是它的优点。

### 10.6 透明性带来的安全问题

组合模式的透明性使得发起请求的客户不用去顾忌树中组合对象和叶对象的区别，但它们本质上是有区别的。

组合对象可以拥有子节点，叶对象下面没有子节点，所以有可能会发生误操作，比如试图往叶对象中添加子节点。解决方案通常是给叶对象也增加 `add` 方法，并且在调用这个方法时，抛出一个异常来及时提醒客户：

```javascript
// ...

var openTvCommand = {
  execute: function () {
    console.log('打开电视');
  },
  add: function () {
    throw new Error('叶对象不能添加子节点');
  }
};

// ...
```

### 10.7 组合模式的例子——扫描文件夹

```javascript
var Folder = function (name) {
  this.name = name;
  this.files = [];
};

Folder.prototype.add = function (file) {
  this.files.push(file);
};

Folder.prototype.scan = function () {
  console.log('开始扫描文件夹：' + this.name);
  for (var i = 0, file, files = this.files; file = files[i++]) {
    file.scan();
  }
};

var File = function (name) {
  this.name = name;
};

File.prototype.add = function () {
  throw new Error('文件下面不能再添加文件');
};

File.prototype.scan = function () {
  console.log('开始扫描文件：' + this.name);
};
```

接下来创建一些文件夹和文件对象，并且组合成一棵树：

```javascript
var folder = new Folder('学习资料');
var folder1 = new Folder('JavaScript');
var folder2 = new Folder('jQuery');

var file1 = new File('JavaScript 设计模式与开发实践');
var file2 = new File('精通 jQuery');
var file3 = new File('重构与模式');

folder1.add(file1);
folder2.add(file2);

folder.add(folder1);
folder.add(folder2);
folder.add(file3);
```

我们改变了树的结构，增加了新的数据，却不用修改任何一句原有的代码，这是符合开放-封闭原则的。

运用了组合模式之后，扫描整个文件夹的操作也是轻而易举的，只需要操作树的最顶端对象：

```javascript
folder.scan();
```

### 10.8 注意

##### 1. 组合模式不是父子关系

组合模式是一种 HAS-A（聚合）的关系，而不是 IS-A。组合对象包含一组叶对象，但 Leaf 并不是 Composite 的子类。组合对象把请求委托给它所包含的所有叶对象，它们能够合作的关键是拥有相同的接口。

##### 2. 对叶对象操作的一致性

比如给全体员工发过节费，这个场景可以用组合模式，但如果公司给当天生日员工发祝福邮件，就不能用组合模式。

##### 3. 双向映射关系

给全体员工发过节费的通知步骤是从公司到部门到小组到个人，本身是组合模式的好例子。但有种情况是，也许某些员工属于多个组织架构，可能重复收过节费。

这种复合情况下必须给父节点和子节点建立双向映射关系，一个方法是给小组和员工对象都增加集合来保存对方的引用。但是这种互间的引用相当复杂，而且对象之间产生过多的耦合性，修改或删除一个对象都变得困难，此时可以引入中介者模式来管理这些对象。

##### 4. 用职责链模式提高组合模式性能

在组合模式中，如果树的结构比较复杂，节点数量很多，在遍历树的过程中，性能方面也许表现得不够理想。有时候可以借助一些技巧，在实际操作中避免遍历整棵树，比如借助职责链模式。职责链模式一般需要手动设置链条，但在组合模式中，父对象和子对象之间实际上形成了天然的职责链。让请求顺着链条从父对象往子对象传递，或者反过来从子对象往父对象传递，知道遇到可以处理该请求的对象为止。

### 10.9 引用父对象

有时候需要在子节点上保持对父节点的引用，比如在组合模式中使用职责链时，有可能需要让请求从子节点往父节点上冒泡传递。还有当我们删除某个文件的时候，实际上是从这个文件所在的上层文件夹中删除该文件的。

```javascript
var Folder = function (name) {
  this.name = name;
  this.parent = null;
  this.files = [];
};

Folder.prototype.add = function (file) {
  file.parent = this;
  this.files.push(file);
};

Folder.prototype.scan = function () {
  console.log('开始扫描文件夹：' + this.name);
  for (var i = 0, file, files = this.files; file = files[i++];) {
    file.scan();
  }
};

Folder.prototype.remove = function () {
  if (!this.parent) {
    return;
  }
  for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
    var file = files[i];
    if (file === this) {
      files.splice(l, 1);
    }
  }
};
```

在 `File.prototype.remove` 方法里，首先会判断 `this.parent`，如果为 `null`，那么这个文件夹是根节点或者游离节点，则让 `remove` 方法直接 `return`，表示不做任何操作。

如果该文件夹有父节点存在，此时遍历父节点中保存的子节点列表，删除想要删除的子节点。

`File` 类的实现基本一致：

```javascript
// ...
File.prototype.remove = function () {
  if (!this.parent) {
    return;
  }
  for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
    var file = files[l];
    if (file === this) {
      files.splice(l, 1);
    }
  }
};
```

### 10.10 何时使用组合模式

- 表示对象的部分-整体层次结构。组合模式可以方便地构造一棵树来表示对象的部分-整体结构。特别是在开发期间不确定这棵树存在多少层次时。树的构造最终完成后，只需请求树的最顶层对象，便能对整棵树做统一操作。在组合模式中增加和删除树的节点非常方便，并且符合开放-封闭原则。
- 客户希望统一对待树中的所有对象。组合模式使客户可以忽略组合对象和叶对象的区别，客户在面对这棵树的时候，不用关心当前正在处理的对象的对象是组合对象还是叶对象，也就不用写一堆 `if`、`else` 语句来分别处理它们。组合对象和叶对象会各自做自己正确的事情。

### 10.11 小结

组合模式可以让我们使用树形方式创建对象的结构。可以用一致的方式来处理组合组合对象和单个对象。

但可能会产生这样一个系统：系统中的每个对象看起来都差不多，只有在运行的时候才显现区别，这会使代码难以理解。此外，如果通过组合模式创建了太多的对象，可能让系统负担不起。

