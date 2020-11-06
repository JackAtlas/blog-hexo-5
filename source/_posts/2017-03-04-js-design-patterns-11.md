title: 《JavaScript 设计模式与开发实战》读书笔记 11
date: 2017-03-04 18:27:46

---

## 第十一章 模版方法模式
<!-- more -->

### 11.1 模版方法模式的定义和组成

模版方法模式是一种只需使用继承机制，但我们可以通过原型 `prototype` 来变相地实现集成。

模版方法模式由两部分结构组成，第一部分是抽象父类，第二部分是具体的实现子类。通常在抽象父类中封装了子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法。

在模版方法模式中，子类实现中的相同部分被上移到父类中，而将不同的部分留待子类来实现。这也很好地体现了泛化的思想。

### 11.2 Coffee or Tea

#### 11.2.1 先泡一杯咖啡

- (1) 把水煮沸
- (2) 用沸水冲泡咖啡
- (3) 把咖啡倒进杯子
- (4) 加糖和牛奶

```javascript
var Coffee = function () {};

Coffee.prototype.boilWater = function () {
  console.log('把水煮沸');
};

Coffee.prototype.brewCoffeeGriends = function () {
  console.log('用沸水冲泡咖啡');
};

Coffee.prototype.pourInCup = function () {
  console.log('把咖啡倒进杯子');
};

Coffee.prototype.addSugarAndMilk = function () {
  console.log('加糖和牛奶');
};

Coffee.prototype.init = function () {
  this.boilWater();
  this.brewCoffeeGriends();
  this.pourInCup();
  this.addSugarAndMilk();
};

var coffee = new Coffee();
coffee.init();
```

#### 11.2.2 泡一壶茶

- (1) 把水煮沸
- (2) 用沸水浸泡茶叶
- (3) 把茶水倒进杯子
- (4) 加柠檬

```javascript
var Tea = function () {};

Tea.prototype.boilWater = function () {
  console.log('把水煮沸');
};

Tea.prototype.steepTeaBag = function () {
  console.log('用沸水浸泡茶叶');
};

Tea.prototype.pourInCup = function () {
  console.log('把茶水倒进杯子');
};

Tea.prototype.addLemon = function () {
  console.log('加柠檬');
};

Tea.prototype.init = function () {
  this.boilWater();
  this.steepTeaBag();
  this.pourInCup();
  this.addLemon();
};

var tea = new Tea();
tea.init();
```

#### 11.2.3 分离出共同点

| 泡咖啡 | 泡茶 |
| ---| ---|
| 把水煮沸 | 把水煮沸 |
| 用沸水冲泡咖啡 | 用沸水浸泡茶叶 |
| 把咖啡倒进杯子 | 把茶水倒进杯子 |
| 加糖和牛奶 | 加柠檬 |

不同点：

- 原料不同，但都可以抽象成“饮料”。
- 泡的方式不同，但都可以抽象成“泡”。
- 加入的调料不同，但都可以抽象成“调料”。

整理成：

- (1) 把水煮沸
- (2) 用沸水冲泡饮料
- (3) 把饮料倒进杯子
- (4) 加调料

```javascript
var Beverage = function () {};

Beverage.prototype.boilWater = function () {
  console.log('把水煮沸');
};

Beverage.prototype.brew = function () {};             // 空方法，应该由子类重写

Beverage.prototype.pourInCup = function () {};        // 空方法，应该由子类重写

Beverage.prototype.addCondiments = function () {};    // 空方法，应该由子类重写

Beverage.prototype.init = function () {
  this.boilWater();
  this.brew();
  this.pourInCup();
  this.addCondiments();
};
```

#### 11.2.4 创建 Coffee 子类和 Tea 子类

```javascript
var Coffee = function () {};
Coffee.prototype = new Beverage();

Coffee.prototype.brew = function () {
  console.log('用沸水冲泡咖啡');
}

Coffee.prototype.pourInCup = function () {
  console.log('把咖啡倒进杯子');
}

Coffee.prototype.addCondiments = function () {
  console.log('加糖和牛奶');
}

var coffee = new Coffee();
coffee.init();
```

Tea 同理。

`Beverage.prototype.init` 被称为模版方法的原因是，该方法中封装了子类的算法框架，它作为一个算法的模版，指导子类以何种顺序去执行哪些方法。在 `Beverage.prototype.init` 方法中，算法内的每一个步骤都清楚地展示在我们眼前。


### 11.3 抽象类

模版方法模式是一种严重依赖抽象类的设计模式。JavaScript 在语言层面并没有提供对抽象类的支持，也很难模拟抽象类的实现，但可以进行一定的让步和变通。

#### 11.3.1 抽象类的作用

在 Java 中，类分为两种，一种为具体类，另一种为抽象类。具体类可被实例化（咖啡），抽象类不可被实例化（饮料）。因此抽象类是用类被具体类继承的。

抽象类和接口一样可用于向上转型，把对象的真正类型隐藏在抽象类或者接口之后，这些对象才可以被互相替换使用。

抽象类也可以表示一种契约。继承了这个抽象类的所有子类都将拥有跟抽象类一致的接口方法，抽象类的主要作用就是为它的子类定义这些公共接口。如果在子类中删掉了这些方法中的某一个，将不能通过编译器的检查。

#### 11.3.2 抽象方法和具体方法

抽象方法被声明在抽象类中，并没有具体的实现过程，是一些“哑”方法。当子类继承了这个抽象类时，必须重写父类的抽象方法。

除了抽象方法之外，如果每个子类中都有一些同样的具体实现方法，那这些方法也可以选择放在抽象类中，这可以节省代码以达到复用的效果，这些方法叫具体方法。当代码需要改变时，只需改动抽象类里的具体方法就可以了。

#### 11.3.4 JavaScript 没有抽象类的缺点和解决方案

JavaScript 并没有从语法层面提供对抽象类的支持。抽象类的第一个作用是隐藏对象的具体类型，由于 JavaScript 是一门“类型模糊”的语言，所以此作用并不重要。

另一方面，当我们在 JavaScript 中使用原型继承来模拟传统的类式继承时，并没有编译器进行检查，也无法保证子类会重写父类中的“抽象方法”。

两种变通的解决方案：

- 用鸭子类型来模拟接口检查，以便确保子类中确实重写了父类的方法。但模拟接口检查会带来不必要的复杂性，而且要求程序员主动进行这些接口检查，会在业务代码中添加一些与业务无关的代码。

- 让抽象类的方法直接抛出一个异常，如果没有在子类中编写相关方法则会在程序运行时得到一个错误。

第二种解决方案的优点是实现简单，付出的额外代价很少；缺点是我们得到错误信息时间点太靠后。

一共有 3 次机会得到这个错误信息，第 1 次是在编写代码的时候，通过编译器的检查来得到错误信息；第 2 次是在创建对象的时候用鸭子类型来进行“接口检查”；而目前我们不得不使用最后一次机会，在程序运行过程中才知道哪里发生了错误。

### 11.4 模版方法模式的使用场景

比如构建一系列的 UI 组件，过程一般如下：

1. 初始化一个 div 容器；
2. 通过 ajax 请求拉取相应的数据；
3. 把数据渲染到 div 容器里面，完成组件的构造；
4. 通知用户组建渲染完毕。

其中第 1 步和第 4 步是相同的。第 2 步不同的地方是请求 ajax 的远程地址，第 3 步不同的地方是渲染数据的方式。

于是可以把这 4 个步骤都抽象到父类的模版方法里面，父类中还可以顺便提供第 1 步和第 4 步的具体实现。当子类继承这个父类之后，会重写模版方法里面的第 2 步和第 3 步。

### 11.5 钩子方法

模版方法模式在父类中封装了子类的算法框架，在正常状态下适用于大多数子类，如果有一些子类是特殊的？比如有一些客人是不加调料的。

钩子方法（hook）可以用来解决这个问题，放置钩子是隔离变化的一种常见手段。我们在父类中容易变化的地方放置钩子，钩子可以有一个默认的实现。钩子方法的返回结果决定了模版方法后面部分的执行步骤。

```javascript
var Bevarage = function () {};

// ...

Beverage.prototype.customerWantsCondiments = function () {
  return true;     // 默认需要调料
}

Beverage.prototype.init = function () {
  this.boilWater();
  this.brew();
  this.pourInCup();
  if (this.customerWantsCondiments()) {   // 如果挂钩返回 true，则需要调料
    this.addCondiments();
  }
}

var CoffeeWithHook = function () {};

CoffeeWithHook.prototype = new Beverage();

// ...

CoffeeWithHook.prototype.customerWantsCondiments = function () {
  return window.confirm('请问需要调料吗？');
};

var coffeeWithHook = new CoffeeWithHook();
coffeeWithHook.init();
```

### 11.6 好莱坞原则

在这一原则的指导下，我们允许底层组件将自己挂钩到高层组件中，而高层组件决定什么时候、以何种方式去使用这些底层组件。

模版方法模式是好莱坞原则的一个典型使用场景，它与好莱坞原则的联系非常明显，当我们用模版方法模式编写一个程序时，就意味着子类放弃了对自己的控制权，改为父类通知子类，哪些方法应该在什么时候被调用。子类只负责提供设计上的细节。

好莱坞原则还常用于其他模式和场景，如发布-订阅模式和回调函数。

- 发布-订阅模式

  在发布订阅-订阅模式中，发布者会吧消息推送给订阅者，取代了原先不断去 fetch 消息的形式。

- 回调函数

  把需要执行的操作封装在回调函数里， 然后把主动权交给另外一个函数。

### 11.7 真的需要“继承”吗

模版方法是基于继承的设计模式，但 JavaScript 实际上没有提供真正的类式继承，继承是通过对象与对象之间的委托来实现的。

在好莱坞原则的指导下，可以这样写：

```javascript
var Beverage = function (param) {
  var boilWater = function () {
    console.log('把水煮沸')
  }

  var brew = param.brew || function () {
    throw new Error('必须传递 brew 方法')
  }

  var pourInCup = param.pourInCup || function () {
    throw new Error('必须传递 pourInCup 方法')
  }

  var addCondiments = param.addCondiments || function () {
    throw new Error('必须传递 pourInCup 方法')
  }

  var F = function () {}

  F.prototype.init = function () {
    boilWater()
    brew()
    pourInCup()
    addCondiments()
  }

  return F
}

var Coffee = Beverage({
  brew: function () {
    console.log('用沸水冲泡咖啡')
  },
  pourInCup: function () {
    console.log('把咖啡倒进杯子')
  },
  addCondiments: function () {
    console.log('加糖和牛奶')
  }
})

var Tea = Beverage({
  brew: function () {
    console.log('用沸水浸泡茶叶')
  },
  pourInCup: function () {
    console.log('把茶倒进杯子')
  },
  addCondiments: function () {
    console.log('加柠檬')
  }
})

var coffee = new Coffee()
coffee.init()

var tea = new Tea()
tea.init()
```

### 11.8 小结

在传统的面向对象语言中，运用了模版方法模式的程序中，子类的方法种类和执行顺序是不变的，所以把这部分逻辑抽象到父类的模版方法中，子类的方法具体实现是可变的。通过增加新的子类，就能给系统增加新的功能，不需要改动父类和其他子类，符合开放-封闭原则。

在 JavaScript 中，高阶函数是更好的选择。