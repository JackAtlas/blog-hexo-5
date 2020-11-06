title: 《JavaScript 设计模式与开发实战》读书笔记 9
date: 2017-02-24 14:31:06

---

## 第九章 命令模式
<!-- more -->

### 9.1 命令模式的用途

命令模式最常见的应用场景：需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道请求的操作是什么。希望用一种松耦合的方式来设计程序，使得请求发送者和接收者能够消除彼此之间的耦合关系。

相对于过程化的请求调用，`command` 对象拥有更长的生命周期。对象的生命周期是跟初始请求无关的，因为这个请求已经被封装在 `command` 对象的方法中，成为了这个对象的行为，可以在程序运行的任意时刻调用这个方法。

命令模式还支持撤销、排队等操作。

### 9.2 菜单程序

`setCommand` 函数负责往按钮上安装命令，可以肯定的是，点击按钮会执行某个 `command` 命令，执行命令的动作被约定为调用 `command` 对象的 `execute()` 方法。

负责绘制按钮的程序员：

```javascript
var setCommand = function (button, command) {
  button.click = function () {
    command.execute();
  }
};
```

负责编写点击按钮之后具体行为的程序员：

```javascript
var MenuBar = {
  refresh: function () {
    console.log('刷新菜单目录');
  }
};

var SubMenu = {
  add: function () {
    console.log('增加子菜单');
  },
  del: function () {
    console.log('删除子菜单');
  }
};
```

先把这些行为都封装在命令类中：

```javascript
var RefreshMenuBarButton = function (receiver) {
  this.receiver = receiver;
};

RefreshMenuBarCommand.prototype.execute = function () {
  this.receiver.refresh();
};

var AddSubMenuCommand = function (receiver) {
  this.receiver = receiver;
};

AddSubMenuCommand.prototype.execute = function () {
  this.receiver.add();
};

var DelSubMenuCommand = function (receiver) {
  this.receiver = receiver;
};

DelSubMenuCommand.prototype.execute = function () {
  console.log('删除子菜单');
};
```

把命令接收者传入到 `command` 对象中，并且把 `command` 对象安装到 `button` 上面：

```javascript
var refreshMenuBarCommand = new RefreshMenuBarCommand(MenuBar);
var addSubMenuCommand = new AddSubMenuCommand(SubMenu);
var delSubMenuCommand = new DelSubMenuCommand(SubMenu);

setCommand(button1, refreshMenuBarCommand);
setCommand(button2, addSubMenuCommand);
setCommand(button3, delSubMenuCommand);
```

### 9.3 JavaScript 中的命令模式

上节是模拟传统面向对象语言的命令模式实现。命令模式将过程式的请求调用封装在 `command` 对象的 `execute` 方法里，通过封装方法调用。

命令模式的由来，其实是*回调（callback）*函数的一个面向对象的替代品。

JavaScript 将函数作为一等对象，命令模式已融入到语言之中。运算块不一定要封装在 `command.execute` 方法中，也可以封装在普通函数中。即使我们依然需要请求“接收者”，也未必适用面向对象的方式，闭包可以完成。

面向对象设计中，命令模式的接收者被当成 `command` 对象的属性保存起来，同时约定执行命令的操作调用 `command.execute` 方法。在使用闭包的命令模式实现中，接收者被封闭在闭包产生的环境中，执行命令的操作可以更加简单，仅仅执行回调函数即可。

闭包实现的命令模式：

```javascript
var setCommand = function (button, func) {
  button.onclick = function () {
    func();
  }
};

var MenuBar = {
  refresh: function () {
    console.log('刷新菜单界面');
  }
};

var RefreshMenuBarCommand = function (receiver) {
  return function () {
    receiver.refresh();
  }
};

var refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);

setCommand(button1, refreshMenuBarCommand);
```

如果想更明确地表达当前正在使用命令模式，活着除了执行命令之外，将来有可能要提供撤销命令等操作，最好还是把执行函数改为调用 `execute` 方法：

```javascript
var RefreshMenuBarCommand = function (receiver) {
  return {
    execute: function () {
      receiver.refresh();
    }
  }
};

var refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);
setCommand(button1, refreshMenuBarCommand);
```

### 9.4 撤销命令

给 5.4 节中小球动画添加撤销动作，点击后小球回到上一次的位置。

先把代码改为命令模式：

```javascript
var ball = document.getElementById('ball');
var pos = document.getElementById('pos');
var moveBtn = document.getElementById('moveBtn');

var MoveCommand = function (receiver, pos) {
  this.receiver = receiver;
  this.pos = pos;
};

MoveCommand.prototype.execute = function () {
  this.receiver.start('left', this.pos, 1000, 'strongEaseOut');
};

var moveCommand;

moveBtn.onclick = function () {
  var animate = new Animate(ball);
  moveCommand = new MoveCommand(animate, pos.value);
  moveCommand.execute();
};
```

撤销操作的实现一般是给命令对象增加一个名为 `unexecute` 或者 `undo` 的方法来执行 `execute` 的反向操作。在 `command.execute` 方法让小球开始运动之前，需要先记录小球的当前位置，在 `unexecute` 或者 `undo` 操作中，再让小球回到刚刚记录下的位置。

```javascript
var ball = document.getElementById('ball');
var pos = document.getElementById('pos');
var moveBtn = document.getElementById('moveBtn');
var cancelBtn = document.getElementById('cancelBtn');

var MoveCommand = function (receiver, pos) {
  this.receiver = receiver;
  this.pos = pos;
  this.oldPos = null;
};

MoveCommand.prototype.execute = function () {
  this.receiver.start('left', this.pos, 1000, 'strongEaseOut');
  this.oldPos = this.receiver.dom.getBoundingClientRect()[this.receiver.propertyName]; // 记录小球开始移动前的位置
};

MoveCommand.prototype.undo = function () {
  this.receiver.start('left', this.oldPos, 1000, 'strongEaseOut');
  // 回到小球移动前记录的位置
};

var moveCommand;

moveBtn.onclick = function () {
  var animate = new Animate(ball);
  moveCommand = new MoveCommand(animate, pos.value);
  moveCommand.execute();
};

cancelBtn.onclick = function () {
  moveCommand.undo();  // 撤销命令
}
```

### 9.5 撤销和重做

上一节是撤销一个命令，如果需要撤销一系列的命令，在这之前，可以把所有执行过的命令都储存在一个历史列表中，然后倒序循环来依次执行这些命令的 `undo` 操作。

但是在某些情况下无法顺利利用 `undo` 操作让对象回到 `execute` 之前的状态。比如在 `Canvas` 中，很难为命令对象定义一个擦除某条线的 `undo` 操作，因为在 `Canvas` 画图中，擦除一条线相对不容易实现。

这时最好的方法是清除画布，然后把刚才执行的命令全部重新执行一遍，这一点同样可以利用一个历史列表堆栈办到。记录命令日志，然后重复执行它们，这是逆转不可逆命令的一个好办法。

如 HTML5 版游戏中，命令模式可用来实现播放录像功能。原理同上，把用户的输入封装成指令，执行过的命令存放在堆栈中，播放录像时只需要从头开始依次执行这些命令。

```javascript
var Ryu = {
  attack: function () {
    console.log('攻击');
  },
  defense: function () {
    console.log('防御');
  },
  jump: function () {
    console.log('跳跃');
  },
  crouch: function () {
    console.log('蹲下');
  }
};

var makeCommand = function (receiver, state) {
  return function () {
    receiver[state]();
  }
};

var commands = {
  '119': 'jump',
  '115': 'crouch',
  '97': 'defense',
  '100': 'attack'
};

var commandStack = [];

document.onkeypress = function (ev) {
  var keyCode = ev.keyCode,
      command = makeCommand(Ryu, commands[keyCode]);

  if (command) {
    command();    // 执行命令
    commandStack.push(command);    // 将刚刚执行过的命令保存进堆栈
  }
};

document.getElementById('replay').onclick = function () {
  var command;
  while (command = commandStack.shift()) { // 从堆栈里依次取出命令并执行
    command();
  }
};
```

### 9.6 命令队列

把请求封装成命令对象的优点在这里再次体现，对象的生命周期几乎是永久的，除非主动去回收它。也就是说，命令对象的生命周期跟初始请求发生的时间无关，`command` 对象的 `execute` 方法可以在程序运行的任何时刻执行。

所以可以把运动过程封装成命令对象，压进一个队列堆栈，当动画执行完，也就是当前 `command` 对象的职责完成之后，会主动通知队列，此时取出正在队列中等待的第一个命令对象，并且执行它。

比较关注的问题是，一个动画结束后如何通知队列。通常可以使用回调函数来通知队列，还可以使用订阅-发布模式。

### 9.7 宏命令

宏命令是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。

宏命令 `MacroCommand`，其 `add` 方法表示把子命令添加进宏命令对象，当调用宏命令对象的 `execute` 方法时，会迭代这一组子命令对象，并依次执行它们的 `execute` 方法：

```javascript
var closeDoorCommand = {
  execute: function () {
    console.log('关门');
  }
};

var MacroCommand = function () {
  return {
    commandList: [],
    add: function (command) {
      this.commandList.push(command),
    },
    execute: function () {
      for (var i = 0, command; command = this.commandList[i++];) {
        command.execute();
      }
    }
  }
};

var macroCommand = MacroCommand();
macroCommand.add(closeDoorCommand);
// ...

macroCommand.execute();
```

还可以为宏命令添加撤销功能，跟 `macroCommand.execute` 类似，当调用 `macroCommand.undo` 方法时，宏命令里包含的所有子命令对象要依次执行各自的 `undo` 操作。

宏命令是命令模式与组合模式的联用产物。

### 9.8 智能命令与傻瓜命令

回看 9.7 节中 `closeDoorCommand` 中没有包含任何 `receiver` 的信息，它本身就包揽了执行请求的行为，这跟之前看到的命令对象都包含一个 `receiver` 是矛盾的。

一般来说，命令模式都会在 `command` 对象中保存一个接收者来负责真正执行客户的请求，这种情况下命令对象是“傻瓜式”的，它只负责把客户的请求转交给接收者来执行，好处是请求发起者和请求接收者之间尽可能地解耦。

但也可以定义一些直接实现请求的命令对象，这就不需要接收者的存在，这种命令对象也叫智能命令。没有接收者的智能命令，退化到和策略模式非常接近，从代码结构上无法分辨，只有通过意图。策略模式指向的问题域更小，所有策略对象的目标总是一致的，它们只是达到这个目标的不同手段，它们的内部实现是针对“算法”而言的。而智能命令模式指向的问题域更广，`command` 对象解决的目标更具发散性。命令模式还可以完成撤销、排队等功能。

