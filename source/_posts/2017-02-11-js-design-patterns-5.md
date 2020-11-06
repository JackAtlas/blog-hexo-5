title: 《JavaScript 设计模式与开发实战》读书笔记 5
date: 2017-02-11 09:46:12

---

## 第五章 策略模式
<!-- more -->

> 策略模式的定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

### 5.1 使用策略模式计算奖金

##### 1. 最初的代码实现

```javascript
var calculateBonus = function (performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return salary * 4;
  }

  if (performanceLevel === 'A') {
    return salary * 3;
  }

  if (performanceLevel === 'B') {
    return salary * 2;
  }
};

calculateBonus('B', 20000); // 输出：40000
calculateBonus('S', 6000);  // 输出：24000
```

这段代码十分简单，但是缺点显而易见：

- `calculateBonus` 函数比较庞大，包含了很多 `if-else` 语句，这些语句需要覆盖所有的逻辑分支。
- `calculateBonus` 函数缺乏弹性，如果增加新的绩效等级，或者修改奖金系数，必须深入 `calculateBonus` 函数的内部实现，违反开放-封闭原则。
- 算法复用性差，如果在程序其他地方需要重用这些算法，只能复制粘贴。

##### 2. 使用组合函数重构代码

把各种算法封装到一个个的小函数里面，这些小函数有着良好的命名，可以一目了然知道其对应的算法，也可以复用在程序的其他地方。

```javascript
var performanceS = function (salary) {
  return salary * 4;
}

var performanceA = function (salary) {
  return salary * 3;
}

var performanceB = function (salary) {
  return salary * 2;
}

var calculateBonus = function (performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return performanceS(salary);
  }

  if (performanceLevel === 'A') {
    return performanceA(salary);
  }

  if (performanceLevel === 'B') {
    return performanceB(salary);
  }
};
```

程序得到了有限的改善，依然没有解决最重要的问题：`calculateBonus` 函数又可能越来越庞大，而且在系统变化的时候缺乏弹性。

##### 3. 使用策略模式重构代码

策略模式的目的是将算法的使用与算法的实现分离开来。

一个基于策略模式的程序至少由两部分组成。第一部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程。第二部分是环境类 Context，Context 接受客户的请求，随后把请求委托给某一个策略类。要做到这点，Context 中要维持对某个策略对象的引用。

第一个版本是模仿传统面向对象语言中的实现。

策略类：

```javascript
var performanceS = function () {};

performanceS.prototype.calculate = function (salary) {
  return salary * 4;
};

var performanceA = function () {};

performanceA.prototype.calculate = function (salary) {
  return salary * 3;
};

var performanceB = function () {};

performanceB.prototype.calculate = function (salary) {
  return salary * 2;
};
```

奖金类：

```javascript
var Bonus = function () {
  this.salary = null;        // 原始工资
  this.strategy = null;      // 绩效等级对应的策略对象
};

Bonus.prototype.setSalary = function (salary) {
  this.salary = salary;      // 设置员工的原始工资
};

Bonus.prototype.setStrategy = function (strategy) {
  this.strategy = strategy;  // 设置员工绩效等级对应的策略对象
};

Bonus.prototype.getBonus = function () {  // 取得奖金数额
  return this.strategy.calculate(this.salary);  // 把计算奖金的操作委托给对应的策略对象
};
```

使用：

```javascript
var bonus = new Bonus();

bonus.setSalary(10000);
bonus.setStrategy(new performanceS());    // 设置策略对象

console.log(bonus.getBonus());            // 输出：40000

bonus.setStrategy(new performanceA());    // 设置策略对象
console.log(bonus.getBonus());            // 输出：30000
```

> 定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

详细说就是：定义一系列的算法，把它们各自封装成策略类，算法被封装在策略类内部的方法里。在客户对 Context 发起请求的时候，Context 总是把请求委托给这些策略对象中间的某一个进行计算。

“并且使它们可以相互替换”，这句话很大程度上是相对于静态类型语言而言的。因为静态类型语言中有类型检查机制，所以各个策略类需要实现同样的接口。当它们的真正类型被隐藏在接口后面时，它们才能被相互替换。而在 JavaScript 这种“类型模糊”的语言中没有这种困扰，任何对象都可以被替换使用。因此，JavaScript 中的“可以相互替换使用”表现为它们具有相同的目标和功能。

### 5.2 JavaScript 版本的策略模式

上一节的代码中，strategy 是对象从各个策略类中创建而来。在 JavaScript 语言中，函数也是对象，所以更简单和直接的做法是把 strategy 定义为函数。同样，Context 也没有必要用 Bonus 类来表示。

```javascript
var strategies = {
  "S": function (salary) {
    return salary * 4;
  },
  "A": function (salary) {
    return salary * 3;
  },
  "B": function (salary) {
    return salary * 2;
  }
};

var calculateBonus = function (level, salary) {
  return strategies[level](salary);
};

console.log(calculateBonus('S', 20000)); // 输出：80000
console.log(calculateBonus('A', 10000)); // 输出：30000
```

### 5.3 多态在策略模式中的体现

通过使用策略模式重构代码，我们消除了原程序中大片的条件分支语句。所有跟计算奖金有关的逻辑不再放在 Context 中，而是分布在各个策略对象中。Context 并没有计算奖金的能力，而是把这个职责委托给了某个策略对象。每个策略对象负责的算法已被各自封装在对象内部。当我们对这些策略对象发出“计算奖金”的请求时，它们会返回不同的计算结果，这正是对象多态性的体现，也是“它们可以相互替换”的目的。替换 Context 中当前保存的策略对象，便能执行不同的算法来得到我们想要的结果。

### 5.4 使用策略模式实现缓动动画

#### 5.4.1 实现动画效果的原理

动画片是把一些差距不大的原画以较快的帧数播放，来达到视觉上的动画效果。在 JavaScript 中，可以通过连续改变元素的某个 CSS 属性，比如 `left`、`top`、`background-position` 来实现动画效果。

#### 5.4.2 思路和一些准备工作

编写一个动画类和一些缓动算法，让小球以各种缓动效果在页面中运动。

在运动开始之前，需要提前记录一些有用的信息，至少包括：

- 动画开始时，小球所在的原始位置；
- 小球移动的目标位置；
- 动画开始时的准确时间点；
- 小球运动持续的时间。

随后会用 `setInterval` 创建一个定时器，定时器每隔 19ms 循环一次。在定时器的每一帧里，我们会把动画已消耗的时间、小球原始位置、小球目标位置和动画持续的总时间等信息传入缓动算法。该算法会根据这几个参数，计算出小球当前应该所在的位置。最后更新该 `div` 对应的 CSS 属性。

#### 5.4.3 让小球运动起来

常见运动算法，这些算法接受 4 个参数，分别是动画已消耗的时间、小球原始位置、小球目标位置、动画持续的总时间，返回的值则是动画元素应该处在的当前位置。

```javascript
var tween = {
  linear: function (t, b, c, d) {
    return c * t / d + b;
  },
  easeIn: function (t, b, c, d) {
    return c * (t /= d) * t + b;
  },
  strongEaseIn: function (t, b, c, d) {
    return c * (t /= d) * t * t * t * t + b;
  },
  strongEaseOut: function (t, b, c, d) {
    return c * ((t = t / d + 1) * t * t * t * t + 1) + b;
  },
  sineaseIn: function (t, b, c, d) {
    return c * (t /= d) * t * t + b;
  },
  sineaseOut: function (t, b, c, d) {
    return c * ((t = t / d - 1) * t * t + 1) + b;
  }
};
```

接下来定义 Animate 类，Animate 的构造函数接受一个参数：即将运动起来的 dom 节点。Animate 类的代码如下：

```javascript
var Animate = function (dom) {
  this.dom = dom;             // 进行动画的 dom 节点
  this.startTime = 0;         // 动画开始时间
  this.startPos = 0;          // 动画开始时，dom 节点的位置，即 dom 的初始位置
  this.endPos = 0;            // 动画结束时，dom 节点的位置，即 dom 的目标位置
  this.propertyName = null;   // dom 节点需要被改变的 css 属性名
  this.easing = null;         // 缓动算法
  this.duration = null;       // 动画持续时间
};
```

`Animate.prototype.start` 方法负责启动这个动画，动画启动的瞬间要记录一些信息，供缓动算法在以后计算小球当前位置的时候使用。在记录完这些信息后启动定时器：

```javascript
Animate.prototype.start = function (propertyName, endPos, duration, easing) {
  this.startTime = +new Date;              // 动画启动时间
  this.startPos = this.dom.getBoundingClientRect()[propertyName]; // dom 节点初始位置
  this.propertyName = propertyName;        // dom 节点需要被改变的 CSS 属性名
  this.endPos = endPos;                    // dom 节点目标位置
  this.duration = duration;                // 动画持续时间
  this.easing = tween[easing];             // 缓动算法

  var self = this;
  var timeId = setInterval(function () {   // 启动定时器，开始执行动画
    if (self.step() === false) {          // 如果动画已结束，则清除定时器
      clearInterval(timeId);
    }
  }, 19);
};
```

`Animate.prototype.start` 方法接受 4 个参数：

- `propertyName`：要改变的 CSS 属性名，比如 'left'、'top'。
- `endPos`：小球运动的目标位置。
- `duration`：动画持续时间。
- `easing`：缓动算法。

`Animate.prototype.step` 方法代表小球运动的每一帧要做的事。在此处，这个方法负责计算小球的当前位置和调用更新 CSS 属性值的方法 `Animate.prototype.update`。

```javascript
Animate.prototype.step = function () {
  var t = +new Date;              // 取得当前时间
  if (t >= this.startTime + this.duration) { // (1)
    this.update(this.endPos);    // 更新小球的 CSS 属性值
    return false;
  }
  var pos = this.easing(t - this.startTime, this.startPos, this.endPos - this.startPos, this.duration);    // 计算小球当前位置
  this.update(pos);               // 更新小球的 CSS 属性值
};
```

(1) 处的意思是，如果当前时间大于动画开始时间加上动画持续时间之和，说明动画已经结束，此时要修正小球的位置。因为在这一帧开始之后，小球的位置已经接近了目标位置，但很可能不完全等于目标位置。此时我们要主动修正小球的当前位置为最终的目标位置。此外让 `Animate.prototype.step` 方法返回 false，可以通知 `Animate.prototype.start` 方法清除计时器。

负责更新小球 CSS 属性值的 `Animate.prototype.update` 方法：

```javascript
Animate.prototype.update = function (pos) {
  this.dom.style[this.propertyName] = pos + 'px';
};
```

测试：

```javascript
var div = document.getElementById('div');
var animate = new Animate(div);

animate.start('left', 500, 1000, 'strongEaseOut');
```

### 5.5 更广义的“算法”

从定义上看，策略模式就是用来封装算法的。在实际开发中，通常会把算法的含义扩散开来，使策略模式也可以用来封装一系列的“业务规则”。只要这些业务规则指向的目标一致，并且可以被替换使用，就可以用策略模式来封装它们。

### 5.6 表单校验

假设我们正在编写一个注册页面，在点击注册按钮之前，有如下几条校验逻辑：

- 用户名不能为空
- 密码长度不能少于 6 位
- 手机号码必须符合格式

#### 5.6.1 第一个版本

没有引入策略模式：

```javascript
var registerForm = document.getElementById('registerForm');

registerForm.onsubmit = function () {
  if (registerForm.userName.value === '') {
    alert('用户名不能为空');
    return false;
  }

  if (registerForm.password.value.length < 6) {
    alert('密码长度不能少于 6 位');
    return false;
  }

  if (!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
    alert('手机号码格式不正确');
    return false;
  }
}
```

缺点：

- `registerForm.onsubmit` 函数比较庞大，包含了很多 `if-else` 语句，这些语句需要覆盖所有的校验规则。
- `registerForm.onsubmit` 函数缺乏弹性，如果增加或修改校验规则，都必须深入 `registerForm.onsubmit` 函数的内部实现，违反开放-封闭原则。
- 算法复用性差。

#### 5.6.2 策略模式重构表单校验

将校验逻辑封装成策略对象：

```javascript
var strategies = {
  isNonEmpty: function (value, errorMsg) {
    if (value === '') {
      return errorMsg;
    }
  },
  minLength: function (value, length, errorMsg) {
    if (value.length < length) {
      return errorMsg;
    }
  },
  isMobile: function (value, errorMsg) {
    if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
      return errorMsg;
    }
  }
};
```

Validator 类作为 Context，负责接收用户的请求并委托给 strategy 对象。在编写 Validator 类的代码之前，有必要先了解用户是如何向 Validator 类发送请求的，这有助于我们知道如何去编写 Validator 类的代码。

```javascript
var validateFunc = function () {
  var validator = new Validator();   // 创建一个 validator 对象
  validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');
  validator.add(registerForm.password, 'minLength: 6', '密码长度不能少于 6 位');
  validator.add(registerForm.phoneNumber, 'isMobile', '手机号码格式不正确');

  var errorMsg = validator.start();  // 获得校验结果
  return errorMsg;                   // 返回校验结果
}

var registerForm = document.getElementById('registerForm');
registerForm.onsubmit = function () {
  var errorMsg = validateFunc();     // 如果 errorMsg 有确切的返回值，说明未通过校验
  if (errorMsg) {
    alert(errorMsg);
    return false;                   // 阻止表单提交
  }
}
```

先创建一个 `validator` 对象，然后通过 `validator.add` 方法，往 `validator` 对象中添加一些校验规则。`validator.add` 方法接受 3 个参数：

```javascript
validator.add(registorForm.password, 'minLength:6', '密码长度不能少于6位');
```

- `registerForm.password` 为参与校验的 `input` 输入框。
- 'minLength:6' 是一个以冒号隔开的字符串。冒号前面的 `minLength` 代表客户挑选的 `strategy` 对象，冒号后面的数字 6 表示在校验过程中所必须的一些参数。
- 第 3 个参数是当校验未通过时返回的错误信息。

当添加完一系列的校验规则之后，会调用 `validator.start()` 方法来启动校验。如果 `validator.start()` 返回了一个确切的 `errorMsg` 字符串当作返回值，说明该次校验没有通过，此时需让 `registerForm.onsubmit` 方法返回 `false` 来阻止表单的提交。

```javascript
var Validator = function () {
  this.cache = [];               // 保存校验规则
};

Validator.prototype.add = function (dom, rule, errorMsg) {
  var ary = rule.split(':');     // 把 strategy 和参数分开
  this.cache.push(function () {  // 把校验的步骤用空函数包装起来，并且放入 cache
    var strategy = ary.shift(); // 用户挑选的 strategy
    ary.unshift(dom.value);     // 把 input 的 value 添加进参数列表
    ary.push(errorMsg);         // 把 strategy 添加进参数列表
    return strategies[strategy].apply(dom, ary);
  });
};

Validator.prototype.start = function () {
  for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
    var msg = validatorFunc();  // 开始校验，并取得校验后的返回信息
    if (msg) {                  // 如果有确切的返回值，说明校验没有通过
      return msg;
    }
  }
};
```

使用策略模式重构代码之后，仅通过“配置”的方式就可以完成一个表单的校验，这些校验规则可以复用在程序的任何地方，还能作为插件的形式，方便地被移植到其他项目中。

在修改某个校验规则的时候，只需要编写或者改写少量的代码。比如我们想将用户名输入框的校验规则改成用户名不能少于 10 个字符：

```javascript
validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空');

// 改成：
validator.add(registerForm.userName, 'minLength:10', '用户名长度不能少于10位');
```

#### 5.6.3 给某个文本输入框添加多种校验规则

```javascript
// ...
Validator.prototype.add = function (dom, rules) {
  var self = this;
  for (var i = 0, rule; rule = rules[i++];) {
    (function (rule) {
      var strategyAry = rule.strategy.split(':');
      var errorMsg = rule.errorMsg;

      self.cache.push(function () {
        var strategy = strategyAry.shift();
        strategyAry.unshift(dom.value);
        strategyAry.push(errorMsg);
        return strategies[strategy].apply(dom, strategyAry);
      });
    })(rule)
  }
};

//...
var validateFunc = function () {
  var validator = new Validator();

  validator.add(registerForm.userName, [{
    strategy: 'isNonEmpty',
    errorMsg: '用户名不能为空'
  }, {
    strategy: 'minLength:10',
    errorMsg: '用户名长度不能少于10位'
  }]);

  //...
}
```

### 5.7 策略模式的优缺点

**优点：**

- 策略模式利用组合、委托和多态等技术和思想，有效避免多重条件选择语句。
- 策略模式提供了对开放-封闭原则的完美支持，将算法封装在独立的 `strategy` 中，使得它们易于切换，易于理解，易于扩展。
- 策略模式中的算法也可以复用在系统的其他地方。
- 策略模式中利用组合和委托来让 Context 拥有执行算法的能力，这也是继承的一种更轻便的替代方案。

**缺点：**

- 在程序中增加许多策略类或者策略对象，但实际上比把它们负责的逻辑堆砌在 Context 中要好。
- 使用策略模式，必须了解所有的 `strategy`，必须了解各个 `strategy` 之间的不同点。这样才能选择一个合适的 `strategy`。此时 `strategy` 要向用户暴露它的所有实现，这是违反最少知识原则的。

### 5.8 一等函数对象与策略模式

在以类为中心的传统面向对象语言中，不同的算法或者行为被封装在各个策略类中，Context 将请求委托给这些策略对象，这些策略对象会根据请求返回不同的执行结果，这样便能表现出对象的多态性。

> 在函数作为一等对象的语言中，策略模式是隐形的。`strategy` 就是值为函数的变量。

在 JavaScript 中，除了使用类来封装算法和行为之外，使用函数当然也是一种选择。这些“算法”可以被封装到函数中并且四处传递，也就是我们常说的“高阶函数”。实际上在 JavaScript 这种讲函数作为一等对象的语言里，策略模式已经融入到了语言本身当中，我们经常用高阶函数来封装不同的行为，并且把它传递到另一个函数中。当我们对这些函数发出“调用”的消息时，不同的函数会返回不同的执行结果。在 JavaScript 中，“函数对象的多态性”来得更加简单。

### 5.9 小结

本章既有接近传统面向对象语言的策略模式实现，也有更适合 JavaScript 语言的策略模式版本。在 JavaScript 语言的策略模式中，策略类往往被函数所代替，这时策略模式就成为一种“隐形”的模式。


