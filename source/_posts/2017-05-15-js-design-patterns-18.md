title: 《JavaScript 设计模式与开发实战》读书笔记 18
date: 2017-05-15 18:32:44

---

第十八章 单一职责原则
<!-- more -->

> 就一个类而言，应该仅有一个引起它变化的原因。

在 JavaScript 中，需要用到类的场景并不太多，单一职责原则更多地是被运用在对象或者方法级别上。

单一职责（SRP）原则体现为：一个对象（方法）只做一件事情。

## 18.1 设计模式中的 SRP 原则

### 18.1.1 代理模式

如第六章的图片预加载例子。通过增加代理的方式，把预加载图片的职责放到代理对象中，而本体仅仅负责往页面中添加 `img` 标签。

`myImage` 负责往页面中添加 `img` 标签：

```javascript
var myImage = (function () {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  return {
    setSrc: function (src) {
      imgNode.src = src;
    }
  }
})();
```

`proxyImage` 负责预加载图片，并在预加载完成之后把请求交给本体 `myImage`：

```javascript
var proxyImage = (function () {
  var img = new Image;
  img.onload = function () {
    myImage.setSrc(this.src);
  };
  return {
    setSrc: function (src) {
      myImage.setSrc('xxx.gif');
      img.src = src;
    }
  };
})();

proxyImage.setSrc('abc.jpg');
```

### 18.1.2 迭代器模式

有一段这样的代码，先遍历一个集合，然后往页面中添加一些 `div`，这些 `div` 的 `innerHTML` 分别对应一个集合里的元素：

```javascript
var appendDiv = function (data) {
  for (var i = 0, l = data.length; i < l; i++) {
    var div = document.createElement('div');
    div.innerHTML = data[i];
    document.body.appendChild(div);
  }
};

appendDiv([1, 2, 3, 4, 5, 6]);
```

`appendDiv` 函数承担了遍历聚合对象和渲染数据的职责，有必要把遍历 `data` 的职责提取出来，这正是迭代器模式的意义，迭代器模式提供了一种方法来访问聚合对象，而不用暴露这个对象的内部表示。

当把迭代聚合对象的职责单独封装在 `each` 函数中后，即使以后还要增加新的迭代方式，我们只需要修改 `each` 函数即可，`appendDiv` 函数不会受到牵连：

```javascript
var each = function (obj, callback) {
  var value,
      i = 0,
      length = obj.length,
      isArray = isArrayLike(obj); // isArrayLike 函数此处未实现，仅作示意

  if (isArray) { // 迭代 object 对象
    for (; i < length; i++) {
      callback.call(obj[i], i, obj[i]);
    }
  } else {
    for (i in obj) { // 迭代 object 对象
      value = callback.call(obj[i], i, obj[i]);
    }
  }

  return obj;
};

var appendDiv = function (data) {
  each(data, function (i, n) {
    var div = document.createElement('div');
    div.innerHTML = n;
    document.body.appendChild(div);
  });
};

appendDiv([1, 2, 3, 4, 5, 6]);
appendDiv({ a: 1, b: 2, c: 3, d: 4 });
```

### 18.1.3 单例模式

第四章实现过一个惰性单例：

```javascript
var createLoginLayer = (function () {
  var div;
  return function () {
    if (!div) {
      div = document.createElement('div');
      div.innerHTML = '我是登录浮窗';
      div.style.display = 'none';
      document.body.appendChild(div);
    };

    return div;
  }
})();
```

现在可以把管理单例的职责和创建登录浮窗的职责分别封装在两个方法里：

```javascript
var getSingle = function (fn) {
  var result;
  return function () {
    return result || (result = fn.apply(this, arguments));
  };
};

var createLoginLayer = function () {
  var div = document.createElement('div');
  div.innerHTML = '我是登录浮窗';
  document.body.appendChild(div);
  return div;
};

var createSingleLoginLayer = getSingle(createLoginLayer);

var loginLayer1 = createSingleLoginLayer();
var loginLayer2 = createSingleLoginLayer();

console.log(loginLayer1 === loginLayer2); // 输出：true
```

### 18.1.4 装饰者模式

使用装饰者模式的时候，通常让类或者对象一开始只具有一些基础的职责，更多的职责在代码运行时被动态装饰到对象上面。装饰者模式可以为对象动态增加职责。

第十五章的例子，把数据上报的功能单独放在一个函数里，然后把这个函数动态装饰到业务函数上面：

```javascript
Function.prototype.after = function(afterfn) {
  var __self = this;
  return function () {
    var ret = __self.apply(this, arguments);
    afterfn.apply(this, arguments);
    return ret;
  };
};

var showLogin = function () {
  console.log('打开登录浮窗');
};

var log = function () {
  console.log('上报标签为：' + this.getAttribute('tag'));
};

document.getElementById('button').onclick = showLogin.after(log);
```

## 18.2 何时应该分离职责

并不是所有的职责都应该一一分离。

一方面，如果随着需求的变化，有两个职责总是同时变化，那就不必分离他们。比如在 `ajax` 请求的时候，创建 `xhr` 对象和发送 `xhr` 请求几乎总是在一起的，那么创建 `xhr` 对象的职责和发送 `xhr` 请求的职责就没必要分开。

另一方面，职责的变化轴线仅当它们确定会发生变化时才具有意义，即使两个职责已经被耦合在一起，但还没有发生改变的征兆，也许没有必要主动分离它们，在代码需要重构的时候再分离也不迟。

## 18.4 SRP 原则的优缺点

SRP 原则的优点是降低了单个类或者对象的复杂度，按照职责把对象分解成更小的粒度，有助于代码的复用，也有利于单元测试。

最明显的缺点是增加编写代码的复杂度。当把对象分解成更小的粒度之后，实际上也增大了对象之间相互联系的难度。
