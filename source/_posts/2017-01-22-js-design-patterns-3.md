title: 《JavaScript 设计模式与开发实战》读书笔记 3
date: 2017-01-22 12:32:06

---

## 第三章 闭包和高阶函数
<!-- more -->

### 3.1 闭包
#### 3.1.1 变量的作用域

带 `var`，局部变量；不带 `var`，全局变量。

在 JavaScript 中，函数可以用来创造函数作用域。变量的搜索是从内到外而飞从外到内的。

#### 3.1.2 变量的生存周期

全局变量的生存周期是永久，除非主动销毁之。

局部变量随着函数调用的结束而被销毁。注意：

```javascript
var f = function () {
  var a = 1;
  return function () {
    a++;
    alert(a);
  }
};

f(); // 输出：2
f(); // 输出：3
f(); // 输出：4
```

当执行 `var f = func()` 时，`f` 返回了一个匿名函数的引用，它可以访问到 `func()` 被调用时产生的环境，而局部变量 `a` 一直处在这个环境里。既然局部变量所在的环境还能被外界访问，这个局部变量就有了不被销毁的理由。在这里产生了一个闭包结构，局部变量的生命看起来被延续了。

假设页面上有 5 个 `div` 节点，通过循环为每个 div 绑定 `onclick` 事件，按照索引顺序，点击第 1 个 `div` 时弹出 0，点击第 2 个 `div` 时弹出1。

```html
<html>
<body>
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4</div>
  <div>5</div>
<script>
var nodes = document.getElementsByTagName( 'div' );
for ( var i = 0, len = nodes.length; i < len; i++ ) {
  nodes[ i ].onclick = function () {
    alert( i );
  }
};
</script>
</body>
</html>
```

测试发现无论点击哪个 `div`，最后弹出的结果都是 5。因为 `div` 节点的 `onclick` 事件是被异步触发的，当事件被触发的时候，`for` 循环早已结束，此时变量 `i` 的值已经是 5，所以在 `div` 的 `onclick` 事件函数中顺着作用域链从内到外查找变量 `i` 时，查找到的值总是 5。

解决办法是在闭包的帮助下，把每次循环的 `i` 值都封闭起来。当在事件函数中顺着作用域链中从内到外查找变量 `i` 时，会先找到被封闭在闭包环境中的 `i`：

```javascript
for ( var i = 0, len = nodes.length; i < len; i++ ) {
  (function ( i ) {
    nodes[ i ].onclick = function () {
      console.log(i);
    }
  })( i )
};
```

同样的道理：

```javascript
var Type = {}

for ( var i = 0, type; type = [ 'String', 'Array', 'Number' ][ i++ ];) {
  (function ( type ) {
    Type[ 'is' + type ] = function ( obj ) {
      return Object.prototype.toString.call( obj ) === '[object ' + type + ']';
    }
  })( type )
};

Type.isArray( [] ); // 输出：true
Type.isString( 'str' ); // 输出：true
```

#### 3.1.3 闭包的更多作用

##### 1. 封装变量

假设有一个计算乘积的简单函数：

```javascript
var mult = function () {
  var a = 1;
  for (var i = 0, l = arguments.length; i < 1; i++) {
    a = a * arguments[i];
  };
  return a;
};
```

现在我们觉得对于那些相同的参数来说，每次都进行计算是一种浪费，我们可以加入缓存机制来提高这个函数的性能：

```javascript
var cache = {}

var mult = function () {
  var args = Array.prototype.join.call( arguments, ',' );
  if ( cache[ args ] ) {
    return cache[ args ];
  }

  var a = 1;
  for ( var i = 0, l = arguments.length; i < 1; i++ ) {
    a = a * arguments[i];
  }

  return cache[ args ] = a;
}

alert ( mult( 1, 2, 3 ) ); // 输出：6
alert ( mult( 1, 2, 3 ) ); // 输出：6，从缓存中输出
```

我们看到 `cache` 这个变量仅仅在 `mult` 函数中被使用，与其让 `cache` 变量跟 `mult` 函数一起平行地暴露在全局作用域下，不如把它封闭在 `mult` 函数内部，这样可以减少页面中的全局变量，以避免这个变量在其他地方被不小心修改而引发错误。

```javascript
var mult = (function () {
  var cache = {};
  return function () {
    var args = Array.prototype.join.call( arguments, ',' );
    if ( args in cache ) {
      return cache[ args ];
    }
    var a = 1;
    for ( var i = 0, l = arguments.length; i < 1; i++ ) {
      a = a * arguments[i];
    }
    return cache[ args ] = a;
  }
})()
```

> 提炼函数是代码重构中的一种常见技巧。如果在一个大函数中有一些代码块能够独立出来，我们常常把这些代码块封装在独立的小函数里面。独立出来的小函数有助于代码复用，如果这些小函数有一个良好的命名，它们本身也起到了注释的作用。如果这些小函数不需要在程序的其他地方使用，最好是把它们用闭包封装起来。

```javascript
var mult = (function () {
  var cache = {}
  var calculate = function () { // 封闭 calculate 函数
    var a = 1;
    for ( var i = 0, l = arguments.length; i < 1; i++ ) {
      a = a * arguments[i];
    }
    return a;
  }

  return function () {
    var args = Array.prototype.join.call( arguments, ',' );
    if (args in cache) {
      return cache[ args ];
    }
    return cache[ args ] = calculate.apply( null, arguments );
  }
})();
```

##### 2. 延续局部变量的寿命

`img` 对象经常用于进行数据上报：

```javascript
var report = function ( src ) {
  var img = new Image();
  img.src = src;
};

report( "http://xxx.com/getUserInfo" );
```

一些低版本浏览器的实现存在 bug，在这些浏览器下使用 `report` 函数进行数据上报会丢失 30% 左右的数据。原因是 `img` 是 `report` 函数中的局部变量，当 `report` 函数的调用结束后，`img` 局部变量随即被销毁，而此时或许还没来得及发出 HTTP 请求，所以此次请求就会丢失。

把 `img` 变量用闭包封闭起来，便能解决请求丢失的问题：

```javascript
var report = (function () {
  var imgs = [];
  return function ( src ) {
    var img = new Image();
    imgs.push( img );
    img.src = src;
  }
})
```

#### 3.1.4 闭包与面向对象设计

过程与数据结合是形容面向对象中的“对象”时经常使用的表达。对象以方法的形式包含了过程，而闭包则是在过程中以环境的形式包含了数据。通常用面向对象思想能实现的功能，用闭包也能实现。反之亦然。在 JavaScript 语言的祖先 Scheme 语言中，甚至都没有提供面向对象的原生设计，但可以使用闭包来实现一个完整的面向对象系统。

下面来看看这段跟闭包相关的代码：

```javascript
var extent = function () {
  var value = 0;
  return {
    call: function () {
      value ++;
      console.log( value )
    }
  }
};

var extent = extent();

extent.call(); // 输出：1
extent.call(); // 输出：2
extent.call(); // 输出：3
```

如果换成面向对象的写法，就是：

```javascript
var extent = {
  value: 0,
  call: function () {
    this.value ++;
    console.log( this.value )
  }
};

extent.call(); // 输出：1
extent.call(); // 输出：2
extent.call(); // 输出：3
```

或者：

```javascript
var Extent = function () {
  this.value = 0;
};

Extent.prototype.call = function () {
  this.value ++;
  console.log( this.value );
};

var extent = new Extent();

extent.call();
extent.call();
extent.call();
```

#### 用闭包实现命令模式

在 JavaScript 版本的各种设计模式实现中，闭包的运用非常广泛。

在完成闭包实现的命令模式之前，我们先用面向对象的方式来编写一段命令模式的代码。

```javascript
var Tv = {
  open: function () {
    console.log('打开电视机');
  },
  close: function () {
    console.log('关上电视机'); }
};

var OpenTvCommand = function ( receiver ) {
  this.receiver = receiver;
};

OpenTvCommand.prototype.execute = function () {
  this.receiver.open(); // 执行命令，打开电视机
};

OpenTvCommand.prototype.undo = function () {
  this.receiver.close(); // 撤销命令，关闭电视机
};

var setCommand = function ( command ) {
  document.getElementById( 'execute' ).onclick = function () {
  command.execute(); // 输出：打开电视机
  }
  document.getElementById( 'undo' ).onclick = function () {
    command.undo(); // 输出：关闭电视机
  }
};

setCommand( new OpenTvCommand( Tv ) )
```

命令模式的意图是把请求封装为对象，从而分离请求的发起者和请求的接收者（执行者）之间的耦合关系。在命令被执行之前，可以预先往命令对象中植入命令的接收者。

但在 JavaScript 中，函数作为一等对象，本身就可以四处传递，用函数对象而不是普通对象来封装请求显得更加简单和自然。如果需要往函数对象中预先植入命令的接收者，那么闭包可以完成这个工作。在面向对象版本的命令模式中，预先植入的命令接收者被当成对象的属性保存起来；而在闭包版本的命令模式中，命令接收者会被封闭在闭包形成环境中：

```javascript
var Tv = {
  open: function () {
    console.log( '打开电视机' );
  },
  close: function () {
    console.log( '关上电视机' );
  }
};

var createCommand = function ( receiver ) {
  var execute = function () {
    return receiver.open(); // 执行命令，打开电视机
  }

  var undo = function () {
    return receiver.close(); // 执行命令，关闭电视机
  }

  return {
    execute: execute,
    undo: undo
  }
};

var setCommand = function ( command ) {
  document.getElementById( 'execute' ).onclick = function () {
    command.execute(); // 输出：打开电视机
  }
  document.getElementById( 'undo' ).onclick = function () {
    command.undo(); // 输出：关闭电视机
  }
};

setCommand( createCommand( Tv ) );
```

#### 3.1.6 闭包与内存管理

局部变量本来应该在函数退出的时候被解除引用，但如果局部变量被封闭在闭包形成的环境中，那么这个局部变量就能一直生存下去。使用闭包的一部分原因是我们选择主动把一些变量封闭在闭包中，因为可能在以后还需要使用这些变量，把这些变量放在闭包中和放在全局作用域，对内存方面的影响是一致的，这里并不能说成是内存泄漏。如果在将来需要回收这些变量，我们可以手动把这些变量设为 `null`。

使用闭包的同时比较容易形成循环引用，如果闭包的作用域链中保存着一些 DOM 节点，这时候就有可能造成内存泄漏。但这本身并非闭包的问题，也并非 JavaScript 的问题。在 IE 浏览器中，由于 BOM 和 DOM 中的对象是使用 C++ 以 COM 对象的方式实现的，而 COM 对象的垃圾收集机制采用的是引用计数策略。在基于引用计数策略的垃圾回收机制中，如果两个对象之间形成了循环引用，那么这两个对象都无法被回收，但循环引用造成的内存泄漏在本质上也不是闭包造成的。

如果要解决循环引用带来的内存泄漏问题，我们只需要把循环引用中的变量设为 `null` 即可。将变量设置为 `null` 意味着切断变量与它此前引用的值之间的连接。当垃圾收集器下次运行时，就会删除这些值并回收它们占用的内存。

### 3.2 高阶函数

高阶函数是指至少满足以下条件之一的函数：

- 函数可以作为参数被传递；
- 函数可以作为返回值输出。

#### 3.2.1 函数作为参数传递

把参数作为函数传递，可以抽离出一部分容易变化的业务逻辑，把这部分业务逻辑放在函数参数中，这样一来我们可以分离业务代码中变化与不变的部分。其中一个重要应用场景就是回调函数。

##### 1. 回调函数

在 ajax 异步请求的应用中，回调函数的使用非常频繁。当我们想在 ajax 请求返回之后做一些事情，但又并不知道请求返回的确切时间时，最常见的方案就是把 callback 函数当作参数传入发起 ajax 请求的方法中，待请求完成之后执行 callback 函数。

另外，当一个函数不适合执行一些请求时，我们也可以把这些请求封装成一个函数，并把它作为参数传递给另外一个函数，“委托”给另外一个函数来执行。

##### 2. Array.prototype.sort

`Array.prototype.sort` 接受一个函数当作参数，这个函数里面封装了数组元素的排序规则。从 `Array.prototype.sort` 的使用可以看到，我们的目的是对数组进行排序，这是不变的部分；而使用什么规则去排序，则是可变的部分。把可变的部分封装在函数参数里，动态传入 `Array.prototype.sort`，使 `Array.prototype.sort` 方法成为了一个非常灵活的方法：

```javascript
// 从小到大排列
[1, 4, 3].sort(function(a, b){
  return a - b;
});
// 输出：[1,3,4]

// 从大到小排列
[1, 4, 3].sort(function(a, b){
  return b - a;
});
// 输出：[4, 3, 1]
```

#### 3.2.2 函数作为返回值输出

##### 1. 判断数据的类型

判断一个数据是否数组，在以往的实现中，可以基于鸭子类型的概念来判断，比如判断这个数据有没有 `length` 属性，有没有 `sort` 方法或者 `slice` 方法等。更好的方式是用 `Object.prototype.toString` 来计算。`Object.prototype.toString.call(obj)` 返回一个形如 `[object Array]` 的字符串。所以我们可以编写一系列的 `isType` 函数。

```javascript
var isString = function (obj) {
  return Object.prototype.toString.call(obj) === '[object String]';
}

var isArray = function (obj) {
  return Object.prototype.toString.call(obj) === '[object Array]';
}

...
```

可以发现，这些函数的大部分实现是相同的，不同的只是 `Object.prototype.toString.call(obj)` 返回的字符串，为了避免多余的代码，我们尝试把这些字符串作为参数提前值入 `isType` 函数：

```javascript
var isType = function (type) {
  return function (obj) {
    return Object.prototype.toString.call(obj) === '[object ' + type + ']';
  }
}

var isString = isType('String')
var isArray = isType('Array')
var isNumber = isType('Number')

console.log(isArray([1, 2, 3])); // 输出：true
```

可以用循环语句注册这些 `isType` 函数：

```javascript
var Type = {};

for (var i = 0, type; type = ['String', 'Array', 'Number'][i++]) {
  (function (type) {
    Type['is' + type] = function (obj) {
      return Object.prototype.toString.call(obj) === '[object ' + type + ']';
    }
  })(type)
}

Type.isArray([]);     // 输出：true
Type.isString('str'); // 输出：true
```

##### 2. getSingle

单例模式将在后面章节学习，此处仅作举例。

```javascript
var getString = function (fn) {
  var ret;
  return function () {
    return ret || (ret = fn.apply(this, arguments));
  }
}
```

这个例子既把函数当作参数传递，又让函数执行后返回了另外一个函数。我们可以看看 `getSingle` 函数的效果：

```javascript
var getSingle = getSingle(function () {
  return document.createElement('script');
});

var script1 = getScript();
var script2 = getScript();

alert(script1 === script2); // 输出：true
```

#### 3.2.3 高阶函数实现 AOP

AOP（面向切面编程）主要作用是把一些跟核心业务无关的功能抽离出来，这些跟业务无关的功能通常包括日志统计、安全控制、异常处理等。把这些功能抽离出来之后，再通过“动态织入”的方式掺入业务逻辑模块中。这样的好处首先是可以保持业务逻辑模块的纯净和高内聚性，其次是可以很方便地复用日志统计等功能模块。

通常，在 JavaScript 中实现 AOP，都是指把一个函数“动态织入”到另外一个函数之中，具体的实现技术有很多，下面是通过扩展 `Function.prototype` 来做到的例子。

```javascript
Function.prototype.before = function (beforefn) {
  var __self = this; // 保存原函数的引用
  return function () { // 返回包含了原函数和新函数的“代理”函数
    beforefn.apply(this, arguments); // 执行新函数，修正 this
    return __self.apply(this, arguments)
  }
}

Function.prototype.after = function (afterfn) {
  var __self = this;
  return function () {
    var ret = __self.apply(this, arguments);
    afterfn.apply(this, arguments);
    return ret;
  }
}

var func = function () {
  cons ole.log(2)
}

func = func.before(function () {
  console.log(1);
}).after(function () {
  console.log(3);
});

func();

// 1
// 2
// 3
```

这种使用 AOP 的方式来给函数添加职责，也是 JavaScript 语言中一种非常特别和巧妙的装饰者模式实现。

#### 3.2.4 高阶函数的其他应用

##### 1. function currying（函数柯里化）

currying 又称部分求值。一个 currying 的函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到函数真正需要求值的时候，之前传入的所有参数都会被一次性用于求值。

```javascript
var currying = function (fn) {
  var args = [];

  return function () {
    if (arguments.length === 0) {
      return fn.apply(this, args);
    } else {
      [].push.apply(args, arguments);
      return arguments.callee;
    }
  }
}

var cost = (function () {
  var money = 0;

  return function () {
    for (var i = 0, l = arguments.length; i < 1; i++) {
      money += arguments[i];
    }
    return money;
  }
})();

var cost = currying(cost); // 转化成 currying 函数
cost(100); // 未真正求值
cost(200); // 未真正求值

alert(cost()); // 求值并输出
```

这是一个 currying 函数。当调用 `cost()` 时，如果明确地带上了一些参数，表示此时并不进行真正的求值计算，而是把这些参数保存起来，此时让 `cost` 函数返回另外一个函数。只有当我们以不带参数的形式执行 `cost()` 时，才利用前面保存的所有参数，真正开始进行求值计算。

##### 2. uncurrying

```javascript
Function.prototype.uncurrying = function () {
  var self = this;
  return function () {
    var obj = Array.prototype.shift.call( arguments );
    return self.apply( obj, arguments );
  };
};
```

作用：

在类数组对象 `arguments` 借用 `Array.prototype` 的方法之前，先把 `Array.prototype.push.call` 这句代码转换为一个通用的 `push` 函数：

```javascript
var push = Array.prototype.push.uncurrying();

(function () {
  push( arguments, 4 );
  console.log( arguments ); // 输出：[1, 2, 3, 4]
})(1, 2, 3);
```

通过 uncurrying 的方式，`Array.prototype.push.call` 变成了一个通用的 `push` 函数。这样一来 `push` 的作用就跟 `Array.prototype.push` 一样了，同样不仅仅局限于只能操作 `array` 对象。而对于使用者而言，调用 `push` 函数的方式也显得更加简洁和意图明了。

还可以一次性把 `Array.prototype` 上的方法“复制”到 `array` 对象上，同样这些方法可操作的对象也不仅仅只是 `array` 对象：

```javascript
for (var i = 0, fn, ary = ['push', 'shift', 'forEach']; fn = ary[i++];) {
  Array[fn] = Array.prototype[fn].uncurrying();
};
```

上面是 `Function.prototype.uncurrying` 的一种实现。下面是分析：

```javascript
Function.prototype.uncurrying = function () {
  var self = this;     // self 此时是 Array.prototype.push
  return function () {
    var obj = Array.prototype.shift.call( arguments );
    // obj 是 {
    //     "length": 1,
    //     "0": 1
    // }
    // arguments 对象的第一个元素被截去，剩下[2]
    return self.apply( obj, arguments );
    // 相当于 Array.prototype.push.apply( obj, 2 )
  };
};

var push = Array.prototype.push.uncurrying();
var obj = {
  "length": 1,
  "0": 1
};

push( obj, 2 );
console.log(obj); // 输出：{0: 1, 1: 2, length: 2}
```

另一种实现方式：

```javascript
Function.prototype.uncurrying = function () {
  var self = this;
  return function () {
    return Function.prototype.call.apply( self, arguments );
  };
};
```

##### 3. 函数节流

少数情况下，函数的触发不是由用户直接控制的。在这些场景下，函数有可能会被非常频繁地调用，而造成大的性能问题。

- `window.onresize` 事件。
- `mouseover` 事件。
- 上传进度。

**原理**

按时间段忽略掉一些请求，比如确保在 500ms 内只打印一次。可以借助 `setTimeout`。

将即将被执行的函数用 `setTimeout` 延迟一段时间执行。如果该次延迟执行还没有完成，则忽略接下来调用该函数的请求。 `throttle` 函数接受 2 个参数，第一个参数为需要被延迟执行的函数，第二个参数为延迟执行的时间。

```javascript
var throttle = function (fn, interval) {
  var __self = fn,                    // 保存需要被延迟执行的函数引用
      timer,                          // 定时器
      firstTime = true;               // 是否是第一次调用

  return function () {
    var args = arguments,
        __me = this;

    if (firstTime) {                 // 如果是第一次调用，不需延迟执行
      __self.apply(__me, args);
      return firstTime = false;
    }

    if (timer) {                     // 如果定时器还在，说明前一次延迟执行还没有完成
      return false
    }

    timer = setTimeout(function () { // 延迟一段时间执行
      clearTimeout(timer);
      timer = null;
      __self.apply(__me, args);
    }, interval || 500);
  };
};

window.onresize = throttle(function () {
  console.log( 1 );
}, 500);
```

##### 4. 分时函数

某些函数是用户主动调用的，但因为一些客观的原因，这些函数会严重影响页面性能，如创建长列表。

在短时间内往页面中大量添加 DOM 节点显然也会让浏览器吃不消，结果往往是浏览器的卡顿甚至假死。

```javascript
var ary = [];

for (var i = 1; i <= 1000; i++) {
  ary.push(i);     // 假设 ary 装载了 1000 个好友的数据
};

var renderFriendList = function (data) {
  for (var i = 0, l = data.length; i < l; i++) {
    var div = document.createElement('div');
    div.innerHTML = i;
    document.body.appendChild(div);
  }
};
```

`timeChunk` 函数让创建节点的工作分批进行，比如把 1 秒钟创建 1000 个节点，改为每隔 200 毫秒创建 8 个节点。

```javascript
var timeChunk = function (ary, fn, count) {
  var obj,
      t;
  var len = ary.length;
  var start = function () {
    for (var i = 0; i < Math.min(count || 1, ary.length); i++) {
      var obj = ary.shift();
      fn(obj);
    }
  };

  return function () {
    t = setInterval(function () {
      if (ary.length === 0) {     // 如果全部节点都已经被创建好
        return clearInterval(t);
      }
    }, 200);     // 分批执行的时间间隔，也可以用参数的形式传入
  };
};
```

使用 `timeChunk` 函数，每一批只往页面中创建 8 个节点：

```javascript
var renderFriendList = timeChunk(ary, function (n) {
  var div = document.createElement('div');
  div.innerHTML = i;
  document.body.appendChild(div);
}, 8);
```

##### 5. 惰性加载函数

由于浏览器间的实现差异，一些嗅探工作在所难免。比如一个在各浏览器中通用的事件绑定函数 `addEvent`，常见的写法如下：

```javascript
var addEvent = function (elem, type, handler) {
  if (window.addEventListener) {
    return elem.addEventListener(type, handler, false);
  }
  if (window.attachEvent) {
    return elem.attachEvent('on' + type, handler);
  }
};

// 每次被调用都会执行 `if` 条件分支。
```

把嗅探浏览器的操作提前到代码加载的时候，返回一个包裹了正确逻辑的函数。

```javascript
var addEvent = (function () {
  if (window.addEventListener) {
    return function (elem, type, handler) {
      elem.addEventListener(type, handler, false);
    }
  }
  if (window.attachEvent) {
    return function (elem, type, handler) {
      elem.attachEvent('on' + type, handler);
    }
  }
})();
```
也许从头到尾都没有使用过 `addEvent` 函数，这样看来，前一次的浏览器嗅探就是完全多余的动作，也会稍稍延长页面 `ready` 的时间。

**惰性载入方案：**

```javascript
var addEvent = function (elem, type, handler) {
  if (window.addEventListener) {
    addEvent = function (elem, type, handler) {
      elem.addEventListener(type, handler, false);
    }
  } else if (window.attachEvent) {
    elem.attachEvent('on' + type, handler);
  }

  addEvent(elem, type, handler);
}
```
