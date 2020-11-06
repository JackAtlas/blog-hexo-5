title: 《JavaScript 设计模式与开发实战》读书笔记 1
date: 2017-01-12 21:02:14

---

## 第一章 面向对象的 JavaScript

<!-- more -->

### 1.1 动态类型语言和鸭子类型
- 静态类型语言在编译时便已确定变量的类型
- 动态类型语言的变量类型要到程序运行的时候，待变量被赋予某个值之后，才会具有某种类型

> 鸭子类型：“如果它走起路来像鸭子，叫起来也是鸭子，那么它就是鸭子。”

即我们只关注对象的行为（HAS-A），而不关注对象本身（IS-A）。利用鸭子类型的思想，不必借助超类型的帮助，就能轻松地在动态类型语言中实现一个原则：“面向接口编程，而不是面向实现编程”。

### 1.2 多态

> 多态：同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。换句话说，给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的反馈。

多态背后的思想是将“做什么”和“谁去做以及怎样去做”分离开来，也就是将“不变的事物”与“可能改变的事物”分离开来。在这个故事中，动物都会叫，这是不变的，但是不同类型的动物具体怎么叫是可变的。把不变的部分隔离出来，把可变的部分封装起来，这给予了我们扩展程序的能力，程序看起来是可生长的，也是符合开放-闭合原则的，相对于修改代码来说，仅仅增加代码就能完成同样的功能，这显然优雅和安全得多。

### 1.2.5 JavaScript的多态
JavaScript 的变量类型在运行期是可变的，这意味着 JavaScript 对象的多态性是与生俱来的。JavaScript 作为一门动态类型语言，它在编译时没有类型检查的过程，既没有检查创建的对象类型，又没有检查传递的参数类型。并不存在任何程度上的“类型耦合”。在 JavaScript 中，并不需要诸如向上转型之类的技术来取得多态的效果。

### 1.2.6 多态在面向对象程序设计中的作用
多态最根本的作用就是通过把过程化的条件分支语句转化为对象的多态性，从而消除这些条件分支语句。

将行为分布在各个对象中，并让这些对象各自负责自己的行为，这正是面向对象设计的优点。

### 1.2.7 设计模式与多态
在 JavaScript 这种将函数作为一等对象的语言中，函数本身也是对象，函数用来封装行为并且能够被四处传递。当我们对一些函数发出“调用”的消息时，这些函数会返回不同的执行结果，这是“多态性”的一种体现，也是很多设计模式在 JavaScript 中可以用高阶函数来代替实现的原因。

## 1.3 封装
封装的目的是将信息隐藏。一般而言，我们讨论的封装是封装数据和封装实现。这一节将讨论更广义的封装，不仅包括封装数据和封装实现，还包括封装类型和封装变化。
### 1.3.1 封装数据
在许多语言的对象系统中，封装数据是由语法解析来实现的，这些语言也许提供了 `private`、`public`、`protected` 等关键字来提供不同的访问权限。

但 JavaScript 并没有提供对这些关键字的支持，我们只能依赖变量的作用域来实现封装特性，而且只能模拟出 `public` 和 `private` 这两种封装性。

除了 ECMAScrip 6 中提供的 let 之外，一般我们通过函数来创建作用域：

```javascript
var myObject = (function () {
  var __name = 'sven'; // 私有（private）变量
  return {
    getName: function () { // 公开（public）变量
      return __name;
    }
  }
})();

console.log( myObject.getName() ); // 输出：sven
console.log( myObject.__name ); // 输出：undefined
```

### 1.3.2 封装实现
从封装实现细节来讲，封装使得对象内部的变化对其他对象而言是透明的，也就是不可见的。对象对它自己的行为负责。其他对象或者用户都不关心它的内部实现。封装使得对象之间的耦合变松散，对象之间只通过暴露的 API 接口来通信。当我们修改一个对象时，可以随意地修改它的内部实现，只要对外的接口没有变化，就不会影响到程序的其他功能。

### 1.3.3 封装类型
封装类型是静态类型语言中一种重要的封装方式。一般而言，封装类型是通过抽象类和接口来进行的。把对象的真正类型隐藏在抽象类或接口之后，相比对象的类型，客户更关心对象的行为。在 JavaScript 中，并没有对抽象类和接口的支持。JavaScript 本身也是一门类型模糊的语言。在封装类型方面，JavaScript 没有能力，也没有必要做得更多。对于 JavaScript 的设计模式实现来说，不区分类型是一种失色，也可以说是一种解脱。

### 1.3.4 封装变化
从设计模式的角度出发，封装在更重要的层面体现为**封装变化**。

从意图上区分，设计模式可被划分为创建型模式、结构型模式和行为型模式。

创建型模式封装创建对象的变化，结构型模式封装的是对象之间的组合关系，行为型模式封装的是对象的行为变化。

通过封装变化的方式，把系统中稳定不变的部分和容易变化的部分隔离开来，在系统的演变过程中，我们只需要替换那些容易变化的部分，如果这些部分是已经封装好的，替换起来也相对容易。这可以最大程度地保证程序的稳定性和可扩展性。

## 1.4 原型模式和基于原型继承的 JavaScript 对象系统

在以类为中心的面向对象编程语言中，类和对象的关系可以想象成铸模和铸件的关系，对象总是从类中创建而来。而在原型编程的思想中，一个对象是通过克隆另外一个对象所得到的。

原型模式不单是一种设计模式，也被称为一种编程范型。

### 1.4.1 使用克隆的原型模式
原型模式的实现关键，是语言本身是否提供了 `clone` 方法。ECMAScript 5 提供了 `Object.create` 方法，可以用来克隆对象。

```javascript
var Plane = function () {
  this.blood = 100;
  this.attackLevel = 1;
  this.defenseLevel = 1;
};

var plane = new Plane();
plane.blood = 500;
plane.attackLevel = 10;
plane.defenseLevel = 7;

var clonePlane = Object.create(plane);
console.log(clonePlane); // 输出：Object {blood: 500, attackLevel: 10, defenseLevel: 7}
```

在不支持 `Object.create` 方法的浏览器中，则可以使用以下代码：

```javascript
Object.create = Object.create || function (obj) {
  var F = function () {};
  F.prototype = obj;
  
  return new F();
}
```

### 1.4.5 JavaScript 中的原型继承

原型编程的基本规则。

- 所有的数据都是对象。
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
- 对象会记住它的原型。
- 如果对象无法响应某个请求，它会把这个请求委托给它自己的原型。

#### 1. 所有的数据都是对象
在 JavaScript 中不能说所有的数据都是对象，但可以说绝大部分数据都是对象。JavaScript 中的根对象是 `Object.prototype` 对象。`Object.prototype` 对象是一个空对象。我们在 JavaScript 中遇到的每个对象，实际上都是以 `Object.prototype` 对象克隆而来的，`Object.prototype` 对象就是它们的原型。

```javascript
var obj1 = new Object();
var obj2 = {};
```

可以利用 ECMAScript 5 提供的 `Object.getPrototypeOf` 来查看这两个对象的原型：

```javascript
console.log( Object.getPrototypeOf(obj1) === Object.prototype ); // 输出：true
console.log( Object.getPrototypeOf(obj2) === Object.prototype ); // 输出：true
```

#### 2. 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它

```javascript
function Person (name) {
  this.name = name;
};

Person.prototype.getName = function () {
  return this.name;
};

var a = new Person ('sven');

console.log( a.name ); // 输出：sven
console.log( a.getName() ); // 输出：sven
console.log( Object.getPrototypeOf(a) === Person.prototype ); // 输出：true
```

在这里 `Person` 并不是类，而是函数构造器，JavaScript 的函数既可以作为普通函数被调用，也可以作为构造器被调用。当使用 `new` 运算符来调用函数时，此时的函数就是一个构造器。用 `new` 运算符来创建对象的过程，实际上也只是先克隆 `Object.prototype` 对象，再进行一些其他额外操作的过程。

#### 3. 对象会记住它的原型
就 JavaScript 的真正实现来说，其实并不能说对象有原型，而只能说对象的构造器有原型。对于“对象把请求委托给它自己的原型”这句话，更好的说法是对象把请求委托给它的构造器的原型。

JavaScript 给对象提供了一个 `__proto__` 的隐藏属性，某个对象的 `__proto__` 属性默认会指向它的构造器的原型对象，即 `{Constructor}.prototype`。在一些浏览器中，`__proto__` 被公开出来。

```javascript
var a = new Object();
console.log( a.__proto__ === Object.prototype ); // 输出：true
```

`__proto__` 就是对象跟“对象构造器的原型”联系起来的纽带。

#### 4. 如果对象无法响应某个请求，它会把这个请求委托给它的构造器的原型
这条规则即是原型继承的精髓所在。

虽然 JavaScript 的对象最初都是由 `Object.prototype` 对象克隆而来的，但对象构造器的原型并不仅限于 `Object.prototype` 上，而是可以动态指向其他对象。当对象 a 需要借用对象 b 的能力时，可以有选择性地把对象 a 的构造器的原型指向对象 b，从而达到继承的效果。

```javascript
var obj = { name: 'sven' };

var A = function () {};
A.prototype = obj;

var a = new A();
console.log( a.name ); // 输出： sven
```

当我们期望得到一个“类”继承自另外一个“类”的效果时，往往会用下面的代码来模拟实现：

```javascript
var A = function () {};
A.prototype = { name: 'sven' };

var B = function () {};
B.prototype = new A();

var b = new B();
console.log( b.name ); // 输出：sven
```

`Object.prototype` 的原型是 `null`。

### 1.4.6 原型继承的未来
`Object.create` 是原型模式的天然实现，目前大多数主流浏览器都提供了 `Object.create` 方法。但在当前的 JavaScript 引擎下，通过 Object.create 来创建对象的效率并不高，通常比通过构造函数创建对象要慢。通过设置构造器的 `prototype` 来实现原型继承的时候，除了根对象 `Object.prototype` 本身之外，任何对象都会有一个原型。而通过 `Object.create(null)` 可以创建出没有原型的对象。

ECMAScript 6 带来新的 `class` 语法，背后仍是通过原型机制来创建对象。

```javascript
class Animal {
  constructor (name) {
    this.name = name;
  }
  getName () {
    return this.name
  }
}

class Dog extends Animal {
  constructor (name) {
    super(name);
  }
  speak () {
    return "woof";
  }
}

var dog = new Dog("Scamp");
console.log(dog.getName() + ' says ' + dog.speak())
```