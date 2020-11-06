title: 《JavaScript 设计模式与开发实战》读书笔记 4
date: 2017-01-28 14:06:49

---

## 第四章 单例模式
<!-- more -->

> 单例模式的定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

单例模式是一种常见的模式，有一些对象我们只需要一个，比如线程池、全局缓存、浏览器中的 `window` 对象等。

### 4.1 实现单例模式

用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。

```javascript
var Singleton = function (name) {
  this.name = name;
};

Singleton.prototype.getName = function () {
  alert(this.name);
};

Singleton.getInstance = (function () {
  var instance = null;
  return function (name) {
    if (!instance) {
      instance = new Singleton(name);
    }
    return instance;
  }
})();

var a = Singleton.getInstance('sven1');
var b = Singleton.getInstance('sven2');

alert(a === b); // true
```

通过 `Singleton.getInstance` 来获取 `Singleton` “类”的唯一对象。简单但增加了这个“类”的“不透明性”，`Singleton` “类”的使用者必须知道这是一个单例“类”，跟以往通过 `new XXX` 方式获取对象不同，这里偏要使用 `Singleton.getInstance` 来获取对象。

```javascript
var a = Singleton.getInstance('sven1');
var b = Singleton.getInstance('sven2');

alert( a === b); // true
```

### 4.2 透明的单例模式

```javascript
var CreateDiv = (function () {
  var instance;
  var CreateDiv = function (html) {
    if (instance) {
      return instance;
    }
    this.html = html;
    this.init();
    return instance = this;
  };

  CreateDiv.prototype.init = function () {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
  };

  return CreateDiv;
})();

var a = new CreateDiv('sven1');
var b = new CreateDiv('sven2');

alert(a === b);     // true
```

为了把 `instance` 封装起来，我们使用了自执行的匿名函数和闭包，并且让这个匿名函数返回真正的 Singleton 构造方法，这增加了一些程序的复杂度。

`CreateDiv` 的构造函数实际上负责了两件事情。第一是创建对象和执行初始化 `init` 方法，第二是保证只有一个对象。这是一种不好的做法。

假设某天需要利用这个类，在页面中创建千千万万的 `div`，即要让这个类从单例类变成一个普通的可产生多个实例的类，那就必须改写 `CreateDiv` 构造函数。

### 4.3 用代理实现单例模式

```javascript
var CreateDiv = function (html) {
  this.html = html;
  this.init();
}

CreateDiv.prototype.init = function () {
  var div = document.createElement('div');
  div.innerHTML = this.html;
  document.body.appendChild(div);
};

// 代理类
var ProxySingletonCreateDiv = (function () {
  var instance;
  return function (html) {
    if (!instance) {
      instance = new CreateDiv(html);
    }

    return instance;
  }
})();

var a = new ProxySingletonCreateDiv('sven1');
var b = new ProxySingletonCreateDiv('sven2');
console.log(a === b); // true
```

把负责管理单例的逻辑移到了代理类 `proxySingletonCreateDiv` 中。`CreateDiv` 变成普通的类，跟 `proxySingletonCreateDiv` 组合起来达到单例模式的效果。

### 4.4 JavaScript 中的单例模式

前面提到的实现更接近传统面向对象语言中的实现，单例对象从“类”中创建出来。在以类为中心的语言中，这是很自然的做法。但 JavaScript 是一门无类（class-free）语言，在 JavsScript 中创建对象的方法非常简单，既然只需要一个“唯一”的对象，为什么要先创建一个“类”呢？

> 单例模式的核心是确保只有一个实例，并提供全局访问。

全局变量不是单例模式，但在 JavaScript 开发中，经常会把全局变量当成单例来使用。例如：

```javascript
var a = {};
```

独一无二且提供全局访问，满足单例模式两个条件。

但全局变量，容易造成命名空间污染。在大中型项目中，如果不加以限制和管理，程序中可能会存在很多这样的变量。JavaScript 中的变量也很容易被不小心覆盖。JavaScript 的创造者 Brendan Eich 本人也承认全局变量是设计上的失误。

以下几种方式可以相对降低全局变量带来的命名污染。

##### 1. 使用命名空间

合理使用命名空间，并不会杜绝全局变量，但可以减少全局变量的数量。

```javascript
// 对象字面量
var namespace = {
  a: function () {
    alert(1);
  },
  b: function () {
    alert(2);
  }
};

// 动态创建命名空间
var MyApp = {};

MyApp.namespace = function (name) {
  var parts = name.split('.');
  var current = MyApp;
  for (var i in parts) {
    if (!current[parts[i]]) {
      current[parts[i]] = {};
    }
    current = current[parts[i]];
  }
};

MyApp.namespace('event');
MyApp.namespace('dom.style');

console.dir(MyApp);

// 上述代码等价于：
var MyApp = {
  event: {},
  dom: {
    style: {}
  }
};
```

##### 2. 使用闭包封装私有变量

把一些变量封装在闭包的内部，只暴露一些接口跟外界通信：

```javascript
var user = (function () {
  var __name = 'Sven',
      __age = 29;

  return {
    getUserInfo: function () {
      return __name + '-' + __age;
    }
  }
})();
```

用下划线来约定私有变量 `__name` 和 `__age`，封装在闭包产生的作用域中，外部是访问不到这两个变量的，避免对全局的命令污染。

### 4.5 惰性单例

惰性单例指的是在需要的时候才创建对象实例。惰性单例是单例模式的重点，这种技术在实际开发中非常有用，有用的程度可能超出了我们的想象。

```javascript
var createLoginLayer = (function () {
  var div;
  return function () {
    if (!div) {
      div = document.createElement('div');
      div.innerHTML = '我是登录窗';
      div.style.display = 'none';
      document.body.appendColor(div);
    }

    return div;
  }
})();

document.getElementById('loginBtn').onclick = function () {
  var loginLayer = createLoginLayer();
  loginLayer.style.display = 'block';
};
```

### 4.6 通用的惰性单例

上一节的惰性单例代码还有一些问题：

- 违反单一职责原则，创建对象和管理单例的逻辑都放在 createLoginLayer 对象内部。
- 如果下次需要创建页面中唯一的 `iframe` 或其他，就必须把 `createLoginLayer` 函数几乎照抄一遍。

管理单例的逻辑可以抽象出来：用一个变量来标识是否创建过对象，如果是，则下次直接返回这个已经创建好的对象：

```javascript
var obj;
if (!obj) {
  obj = xxx;
}
```

把管理单例的逻辑从原来的代码中抽离出来，这些逻辑被封装在 `getSingle` 函数内部，创建对象的方法 `fn` 被当成参数动态传入 `getSingle` 函数：

```javascript
var getSingle = function (fn) {
  var result;
  return function () {
    return result || (result = fn.apply(this, arguments));
  }
};
```

接下来将用于创建登录浮窗的方法用参数 `fn` 的形式传入 `getSingle`。之后再让 `getSingle` 返回一个新的函数，并且用一个变量 `result` 来保存 `fn` 的计算结果。`result` 变量因为身在闭包中，永远不会被销毁。

```javascript
var createLoginLayout = function () {
  var div = document.createElement('div');
  div.innerHTML = '我是登录浮窗';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
};

var createSingleLoginLayer = getSingle(createLoginLayer);

document.getElementById('loginBtn').onclick = function () {
  var loginLayer = createSingleLoginLayer();
  loginLayer.style.display = 'block';
};
```

把创建实例对象的职责和管理单例的职责分别放置在两个方法里，这两个方法可以独立变化而互不影响，当它们连接在一起的时候，就完成了创建唯一实例对象的功能。

单例模式的用途远不止创建对象，比如我们通常渲染完页面中的一个列表之后，接下来要给这个列表绑定 `click` 事件，如果是通过 `ajax` 动态往列表里追加数据，在使用事件代理的前提下，`click` 事件实际上只需要在第一次渲染列表的时候被绑定一次，但是我们不想去判断当前是否是第一次渲染列表，如果借助于 jQuery，通常选择给节点绑定 `one` 事件：

```javascript
var bindEvent = function () {
  $('div').one('click', function () {
    alert('click');
  });
};

var render = function () {
  console.log('开始渲染列表');
  bindEvent();
};

render();
render();
render();
```

如果利用 `getSingle` 函数，也能达到一样的效果。

```javascript
var bindEvent = getSingle(function () {
  document.getElementById('div1').onclick = function () {
    alert('click');
  }
  return true;
});

var render = function () {
  console.log('开始渲染列表');
  bindEvent();
};

render();
render();
render();
```

`render` 函数和 `bindEvent` 函数都分别执行了 3 次，但 div 实际上只被绑定了一个事件。
