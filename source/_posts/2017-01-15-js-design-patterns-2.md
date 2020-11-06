title: 《JavaScript 设计模式与开发实战》读书笔记 2
date: 2017-01-15 00:11:58

---

## 第二章 this、call 和 apply

<!-- more -->

### 2.1 this
JavaScript 的 `this` 总是指向一个对象，而具体指向哪个对象是在运行时基于函数的执行环境动态绑定的，而非函数被声明时的环境。

#### 2.1.1 this 的指向
除去不常用的 `with` 和 `eval` 的情况：

- 作为对象的方法调用
- 作为普通函数调用
- 构造器调用
- `Function.prototype.call` 或 `Function.prototype.apply` 调用

##### 1. 作为对象的方法调用

```javascript
var obj = {
  a: 1,
  getA: function () {
    alert ( this === obj ); // 输出：true
    alert ( this.a ); // 输出：1
  }
};

obj.getA();
```

##### 2. 作为普通函数调用
指向全局对象，在浏览器的 JavaScript 中，这个全局对象是 `window` 对象。

```javascript
window.name = 'globalName';

var getName = function () {
  return this.name;
};

console.log( getName() ); // 输出：globalName
```

或者：

```javascript
window.name = 'globalName';

var myObject = {
  name: 'sven',
  getName: function () {
    return this.name;
  }
};

var getName = myObject.getName;
console.log( getName() ); // globalName
```

有时，在 `div` 节点的事件函数内部，有一个局部的 `callback` 方法，`callback` 被作为普通函数调用时，`callback` 内部的 `this` 指向了 `window`，但我们往往是想让它指向该 `div` 节点，此时可以用一个变量保存 `div` 节点的引用。在 ECMAScript 5 的 `strict` 模式下，这种情况下的 `this` 已经被规定为不会指向全局对象，而是 `undefined`。

##### 3. 构造器调用

除了宿主提供的一些内置函数，大部分 JavaScript 函数都可以当作构造器使用。构造器的外表跟普通函数一模一样，它们的区别在于被调用的方式。当用 `new` 运算符调用函数时，该函数总会返回一个对象，通常情况下，构造器里的 `this` 就指返回的这个对象。

```javascript
var MyClass = function () {
  this.name = 'sven';
}

var obj = new MyClass();
console.log( obj.name ); // 输出：sven
```

但用 `new` 调用构造器时，如果构造器显式地返回了一个 `sven` 类型的对象，那么此次运算结果最终会返回这个对象，而不是我们之前期待的 `this`：

```javascript
var MyClass = function () {
  this.name = 'sven';
  return { // 显式地返回一个对象
    name: 'anne'
  }
};

var obj = new MyClass();
console.log( obj.name ); // 输出：anne
```

如果构造器不显式地返回任何数据，或者是返回一个非对象类型的数据，就不会造成上述问题。

##### 5. Function.prototype.call 或 Function.prototype.apply 调用
跟普通的函数调用相比，用 `Function.prototype.call` 或 `Function.prototype.apply` 可以动态地改变传入函数的 `this`：

```javascript
var obj1 = {
  name: 'sven',
  getName: function () {
    return this.name;
  }
};

var obj2 = {
  name: 'anne'
};

console.log( obj1.getName() ); // 输出：sven
console.log( obj1.getName().call( obj2 ) ); // 输出：anne
```

#### 2.1.2 丢失的 this

```javascript
var obj = {
  myName: 'sven',
  getName: function () {
    return this.myName
  }
};

console.log( obj.getName() ); // 输出：sven

var getName2 = obj.getName;
console.log( getName2() ); // 输出：undefined
```

当调用 `obj.getName` 时，`getName` 方法是作为 `obj` 对象的属性被调用的，此时的 `this` 指向 `obj` 对象，所以 `obj.getName()` 输出 `sven`。当用另外一个变量 `getName2` 来引用 `obj.getName`，并且调用 `getName2` 时，此时是普通函数调用方式，`this` 是指向全局 `window` 的，所以程序执行的结果是 `undefined`。

```javascript
var getId = document.getElementById;
getId('div1');
```

上面这段代码抛出了一个错误，因为许多浏览器引擎的 `getElementById` 方法的内部实现中需要用到 `this`。这个 `this` 本来被期望指向 `document`，当 `getElementById` 方法作为 `document` 对象的属性被调用时，方法内部的 `this` 确实是指向 `document` 的。但当用 `getId` 来引用 `document.getElementById` 之后，再调用 `getId`，此时就成了普通函数调用，函数内部的 `this` 指向了 `window`，而不是原来的 `document`。我们可以尝试利用 `apply` 把 `document` 当作 `this` 传入 `getId` 函数，帮助“修正” `this`：

```javascript
document.getElementById = (function (fun) {
  return function () {
    return func.apply( document, arguments );
  }
})( document.getElementById );

var getId = document.getElementById;
var div = getId( 'div1' );

alert(div.id); // 输出：div1
```

## 2.2 call 和 apply
### 2.2.1 call 和 apply 的区别
`Function.prototype.call` 和 `Function.prototype.apply` 都是非常常用的方法。它们的作用一模一样，区别仅在于传入参数形式的不同。

`apply` 接受两个参数，第一个参数指定了函数体内 `this` 对象的指向，第二个参数为一个带下表的集合（数组或类数组），`apply` 方法把这个集合中的元素作为参数传递给被调用的函数：

```javascript
var func = function ( a, b, c ) {
  alert( [a, b, c] ) // 输出 [ 1, 2, 3]
}

func.apply( null, [ 1, 2, 3 ] );
```

`call` 传入的参数数量不固定，跟 `apply` 相同的是，第一个参数也是代表函数体内的 `this` 指向，从第二个参数开始往后，每个参数被依次传入函数：

```javascript
var func = function ( a, b, c ) {
  alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};

func.call( null, 1, 2, 3 );
```

当调用一个函数时，JavaScript 的解释器并不会计较形参和实参在数量、类型以及顺序上的区别，JavaScript 的参数在内部就是用一个数组来表示的。从这个意义上说，`apply` 比 `call` 的使用率更高。`call` 是包装在 `apply` 上的一颗语法糖。

当传入的第一个参数为 `null`，函数体内的 `this` 会指向默认的宿主对象，在浏览器中就是 `window`：

```javascript
var func = function ( a, b, c ) {
  alert ( this === window ); // 输出 true
};

func.apply( null, [ 1, 2, 3 ] );
```

但如果在严格模式下，函数体内的 `this` 还是为 `null`：

```javascript
var func = function ( a, b, c ) {
  "use strict";
  alert ( this === window ); // 输出 true
};

func.apply( null, [ 1, 2, 3 ] );
```

有时候我们使用 `call` 或者 `apply` 的目的不在于指定 `this` 指向，而是另有用途，比如借用其他对象的方法。那么我们可以传入 `null` 来代替某个具体的对象：

```javascript
Math.max.apply( null, [ 1, 2, 5, 3, 4 ] ) // 输出：5
```

### 2.2.2 call 和 apply 的用途
#### 1. 改变 this 指向
`call` 和 `apply` 最常见的用途是改变函数内部的 `this` 指向：

```javascript
var obj1 = {
  name: 'sven'
};

var obj2 = {
  name: 'anne'
};

window.name = 'window';

var getName = function () {
  alert ( this.name );
};

getName(); // 输出：window
getName.call( obj1 ); // 输出：sven
getName.call( obj2 ); // 输出：anne
```

#### 2. Function.prototype.bind
大部分高级浏览器都实现了内置的 `Function.prototype.bind`，用来指定函数内部的 `this` 指向，即使没有原生的实现，也可以模拟一个：

```javascript
Function.prototype.bind = function ( context ) {
  var self = this; // 保存原函数
  return function () {
    return self.apply( context, arguments ); // 执行新的函数的时候，会把之前传入的 context 当作新函数体内的 this
  }
};

var obj = {
   name: 'sven'
};

var func = function () {
  alert ( this.name ); // 输出：sven
}.bind(obj);

func();
```

通常会实现得稍微复杂一些：

```javascript
Function.prototype.bind = function () {
  var self = this, // 保存原函数
      context = [].shift.call( arguments ), // 需要绑定的 this 上下文
      args = [].slice.call( arguments ); // 剩余的参数转成数组
  return function () { // 返回一个新的函数
    return self.apply( context, [].concat.call( args, [].slice.call( arguments ) ) );
    // 执行新的函数的时候，会把之前传入的 `context` 当作新函数体内的 this
    // 并且组合两次分别传入的参数，作为新函数的参数
  };
};

var obj = {
  name: 'sven'
};

var func = function ( a, b, c, d ) {
  alert ( this.name ); // 输出：sven
  alert ( [ a, b, c, d ] ) // 输出：[ 1, 2, 3, 4 ]
}.bind( obj, 1, 2 );

func( 3, 4 );
```

#### 3. 借用其他对象的方法
借用构造函数可以实现一种类似继承的效果：
```javascript
var A = function ( name ) {
  this.name = name;
};

var B = function () {
  A.apply( this, arguments );
};

B.prototype.getName = function () {
  return this.name;
};

var b = new B( 'sven' );
console.log( b.getName() ); // 输出：'sven'
```

函数的参数列表 arguments 是一个类数组对象，虽然它也有“下标”，但它并非真正的数组，所以也不能像数组一样，进行排序操作或者往集合里添加一个新的元素。在操作 `arguments` 时，我们常常会借用 `Array.prototype` 对象上的方法。

```javascript
(function () {
  Array.prototype.push.call( arguments, 3 );
  console.log( arguments ); // 输出[1,2,3]
})( 1, 2 );
```

以 `Array.prototype.push` 为例，看看 V8 引擎中的具体实现：

```javascript
function ArrayPush () {
  var n = TO_UINT32( this.length ); // 被 push 的对象的 length
  var m = %_ArgumentsLength(); // push 的参数个数
  for (var i = 0; i < m; i++) {
    this[ i + n ] = %_Arguments( i ); // 复制元素
  }
  this.length = n + m; // 修正 length 属性的值
  return this.length;
};
```

`Array.prototype.push` 实际上是一个属性复制的过程，把参数按照下标一次添加到被 `push` 的对象上面，顺便修改了这个对象的 `length` 属性。至于被修改的对象是数组还是类数组对象，并不重要。

```javascript
var a = {};
Array.prototype.push.call( a, 'first' );

alert( a.length ); // 输出：1
alert( a[ 0 ] ); // first
```

如果在低版本的 IE 浏览器中执行，必须显式地给对象 `a` 设置 `length` 属性：

```javascript
var a = {
  length: 0
};
```

可以借用 `Array.prototype.push` 方法的对象还要满足以下两个条件：

- 对象本身要可以存取属性；
- 对象的 `length` 属性可读写。