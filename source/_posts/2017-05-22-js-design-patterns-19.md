title: 《JavaScript 设计模式与开发实战》读书笔记 19
date: 2017-05-22 20:08:27

---

第十九章 最少知识原则
<!-- more -->

最少知识原则（LKP）说的是一个软件实体应当尽可能少地与其他实体发生相互作用。这里的软件实体是一个广义的概念，不仅包括对象，还包括系统、类、模块、函数、变量等。

## 19.1 减少对象之间的联系

单一职责原则指导我们把对象划分成较小的粒度，可以提高对象的复用性。但越来越多的对象之间可能会产生错综复杂的联系，如果修改了其中一个对象，很可能会影响到跟它相互引用的其他对象。对象与对象耦合在一起，可能会降低复用性。

最少知识原则要求设计程序时，应当尽量减少对象之间的交互。如果两个对象之间不必彼此直接通信，那么这两个对象就不要发生直接的相互联系。常见的做法是引入一个第三者对象，来承担这些对象之间的通信作用。

## 19.2 设计模式中的最少知识原则

### 19.2.1 中介者模式

第十四章博彩公司的例子。博彩公司作为中介，每个人都只和博彩公司发生关联，博彩公司会根据所有人的投注情况计算好概率，彩民赢了钱从博彩公司拿，输了赔钱给博彩公司。

中介者模式很好地体现了最少知识原则。通过增加一个中介者对象，让所有的相关对象都通过中介者对象来通信，而不是互相引用。

### 19.2.2 外观模式

外观模式主要是为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使子系统更加容易使用。

外观模式的作用是对客户屏蔽一组子系统的复杂性。外观模式对客户提供一个简单易用的高层接口，高层接口会把用户的请求转发给子系统来完成具体的功能实现。但如果外观不能满足客户的个性化需求，客户也可以选择越过外观来直接访问子系统。

比如全自动洗衣机的“一键洗衣”功能，客户可以选择一键洗衣，也可以手动控制。

简单的外观模式示例：

```javascript
var A = function () {
  a1();
  a2();
};

var B = function () {
  b1();
  b2();
};

var facade = function () {
  A();
  B();
};

facade();
```

外观模式的作用主要有两点：

- 为一组子系统提供一个简单便利的访问入口。
- 隔离客户与复杂子系统之间的联系，客户不用去了解子系统的细节。

从第二点来看，外观模式是符合最少知识原则的。外观系统将客户和子系统隔开之后，如果修改子系统内部，只要外观不变，就不会影响客户的调用。同样，对外观的修改也不会影响到子系统，它们可以分别变化而互不影响。

## 19.3 封装在最少知识原则中的体现

封装在很大程度上表达的是数据的隐藏。一个模块或者对象可以将内部的数据或者实现细节隐藏起来，只暴露必要的接口 API 供外界访问。对象之间难免产生联系，当一个对象必须引用另外一个对象的时候，可以让对象只暴露必要的接口，让对象之间的联系限制在最小的范围之内。

封装也用来限制变量的作用域。在 JavaScript 中对变量作用域的规定是：

- 变量在全局声明，或者在代码的任何位置隐式声明（不用 var），则该变量全局可见；
- 变量在函数内显式声明（使用 var），则在函数内可见。

把变量的可见性限制在一个尽可能小的范围内，这个变量对其他不想关模块的影响就越小，变量被改写和发生冲突的机会也越小。这也是广义的最少知识原则的一种体现。

假如要编写一个具有缓存效果的计算乘积的函数 `function mult () {}`，需要一个对象 `var cache = {}` 来保存已经计算过的结果。`cache` 对象只对 `mult` 有用，把 `cache` 对象放在 `mult` 形成的闭包中，显然比把它放在全局作用域更加合适，代码如下：

```javascript
var mult = (function () {
  var cache = {};
  return function () {
    var args = Array.prototype.join.call(arguments, ',');
    if (cache[args]) {
      return cache[args];
    };
    var a = 1;
    for (var i = 0, l = arguments.length; i < l; i++) {
      a = a * arguments[i];
    };
    return cache[args] = a;
  };
})();

mult(1, 2, 3); // 输出：6
```