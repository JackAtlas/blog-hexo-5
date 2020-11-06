title: 《JavaScript 设计模式与开发实战》读书笔记 8
date: 2017-02-20 22:56:01

---

## 第八章 发布-订阅模式
<!-- more -->

> 发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。

在 JavaScript 开发中，我们一般用事件模型来替代传统的发布-订阅模式。

### 8.2 发布-订阅模式的作用

发布-订阅模式可以广泛应用于异步编程中，是替代传统回调函数的方案。在异步编程中使用发布-订阅模式，无需过多关注对象在异步运行期间的内部状态，只需订阅感兴趣的事件发生点。

发布-订阅模式可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口。发布-订阅模式让两个对象松耦合地联系在一起。当有新的订阅者出现时，发布者的代码不需要任何修改；同样发布者需要改变时，也不会影响到之前的订阅者。只要之前约定的事件名没有变化，就可以自由使用它们。

### 8.3 DOM 事件

```javascript
document.body.addEventListener('click', function () {
  alert(2);
}, false);
```

订阅 `document.body` 上的 `click` 事件，当 `body` 节点被点击时，`body` 节点便会向订阅者发布这个消息。

### 8.4 自定义事件

1. 指定谁充当发布者；
2. 给发布者添加一个缓存列表，用于存放回调函数以便通知订阅者；
3. 发布消息的时候，发布者会遍历这个缓存列表，依次触发里面存放的订阅者回调函数。

还可以往回调函数里填入一些参数，订阅者可以接收这些参数。

同时也有必要增加一个标示 `key`，让订阅者只订阅自己感兴趣的消息。

```javascript
var salesOffices = {};

salesOffices.clientList = [];

salesOffices.listen = function (key, fn) {
  if (!this.clientList[key]) {
    this.clientList[key] = [];
  }
  this.clientList[key].push(fn);
};

salesOffices.trigger = function () {
  var key = Array.prototype.shift.call(arguments),  // 取出消息类型
      fns = this.clientList[key];                   // 取出该消息类型的回调函数集合

  if (!fns || fns.length === 0) {
    return false;
  }

  for (var i = 0, fn; fn = fns[i++];) {
    fn.apply(this, arguments);                    // arguments 是发布消息时附带的参数
  }
}

// 小明订阅 88 平方米房子的消息
salesOffices.listen('squareMeter88', function (price) {
  console.log('价格= ' + price);
});

salesOffice.trigger('squareMeter88', 2000000);        // 发布 88 平方米房子的价格
```

### 8.5 发布-订阅模式的通用实现

现在把发布-订阅的功能提取出来，放在一个单独的对象内：

```javascript
var event = {
  clientList: [],
  listen: function (key, fn) {
    if (!this.clientList[key]) {
      this.clientList[key] = [];
    }
    this.clientList[key].push(fn);     // 订阅的消息添加进缓存列表
  },
  trigger: function () {
    var key = Array.prototype.shift.call(arguments),
        fns = this.clientList[key];

    if (!fns || fns.length === 0) {
      return false;
    }

    for (var i = 0, fn; fn = fns[i++];) {
      fn.apply(this, arguments);
    }
  }
};
```

再定义一个 `installEvent` 函数，这个函数可以给所有的对象都动态安装发布-订阅功能：

```javascript
var installEvent = function (obj) {
  for (var i in event) {
    obj[i] = event[i];
  }
};
```

给某个对象增加发布-订阅功能：

```javascript
var obj = {};
installEvent(obj);

// ...
```

### 8.6 取消订阅的事件

```javascript
event.remove = function (key, fn) {
  var fns = this.clientList[key];

  if (!fns) {
    return false;
  }

  if (!fn) {     // 如果没有传入具体的回调函数，表示需要取消 key 对应消息的所有订阅
    fns && (fns.length = 0);
  } else {
    for (var l = fns.length - 1; l >= 0; l--) {
      var _fn = fns[l];
      if (_fn === fn) {
        fns.splice(l, 1);   // 删除订阅者的回调函数
      }
    }
  }
};
```

### 8.7 网站登陆

假设页面各个模块的渲染都有一个共同的前提条件，就是必须先用 ajax 异步请求获取用户的登录信息。

异步的问题通常可以用回调函数来解决。

但此处用发布-订阅模式的原因是，我们不知道除了现有的模块，将来还有哪些模块需要使用这些用户信息。如果它们和信息模块产生了强耦合，比如这样：

```javascript
login.succ(function () {
  header.setAvatar(data.avatar);     // 设置 header 模块的头像
  nav.setAvatar(data.avatar);        // 设置导航模块的头像
  message.refresh();                 // 刷新消息列表
  cart.refresh();                    // 刷新购物车列表
});
```

编写者必须了解 header 模块里设置头像的方法叫 `setAvatar`、购物车模块里刷新的方法叫 `refresh`，这种耦合性会使程序变得僵硬，header 模块不能随意再改变 `setAvatar` 的方法名，它自身也不能改名。这是针对具体实现编程的典型例子，针对具体实现编程是不被赞同的。

用发布-订阅模式重写之后，对用户信息感兴趣的业务模块将自行订阅登录成功的消息事件。当登录成功时，登录模块只需要发布登录成功的消息，而不需要关心业务方要做什么，也不需要了解它们的内部细节。

```javascript
$.ajax('http://xxx.com?login', function (data) {
  // 登录成功
  login.trigger('loginSucc', data); // 发布登录成功的消息
})
```

各模块监听登录成功的消息：

```javascript
var header = (function () {
  login.listen('loginSucc', function (data) {
    header.setAvatar(data.avatar);
  });
  return {
    setAvatar: function (data) {
      console.log('设置 header 模块的头像');
    }
  }
})();
```

我们随时可以把 `setAvatar` 的方法改名。如果在登录完成后增加一个刷新收货地址列表的行为，只要在收货地址模块里加上监听消息的方法即可。

### 8.8 全局的发布-订阅对象

两个小问题：

- 给每一个发布者都添加了 `listen` 和 `trigger` 方法，以及一个缓存列表 `clientList`，资源浪费。
- 订阅者与发布者仍有一定的耦合性，订阅者至少要知道发布者的名字，才能订阅到事件。

发布-订阅模式可以用一个全局的 `Event` 对象来实现，订阅者不需要了解消息来自哪个发布者，发布者也不需要知道消息推送给哪些订阅者。

```javascript
var Event = (function () {
  var clientList = {},
      listen,
      trigger,
      remove;

  listen = function (key, fn) {
    if (!clientList[key]) {
      clientList[key] = [];
    }
    clientList[key].push(fn);
  };

  trigger = function () {
    var key = Array.prototype.shift.call(arguments),
        fns = clientList[key];
        if (!fns || fns.length === 0) {
          return false;
        }
        for (var i = 0, fn; fn = fns[i++];) {
          fn.apply(this, arguments);
        }
  };

  remove = function (key, fn) {
    var fns = clientList[key];
    if (!fns) {
      return false;
    }
    if (!fn) {
      fns && (fns.length = 0);
    } else {
      for (var l = fns.length - 1; l >= 0; l--) {
        var _fn = fns[l];
        if (_fn === fn) {
          fns.splice(l, 1);
        }
      }
    }
  };

  return {
    listen: listen,
    trigger: trigger,
    remove: remove
  }
})();
```

### 8.9 模块间通信

上一节的发布-订阅模式的实现，是基于一个全局的 `Event` 对象，利用它可以在两个封装良好的模块中进行通信，这两个模块可以完全不知道对方的存在。

但是，模块之间如果用了太多的全局发布-订阅模式来通信，那么模块与模块之间的联系就会被隐藏到了背后。我们最终会搞不清楚消息来自哪个模块，或者消息会流向哪些模块。

### 8.10 必须先订阅再发布吗

之前实现的订阅-发布模式，都是订阅者必须先订阅一个消息，随后才能接收到发布者发布的消息。

某些情况下，需要先将消息保存下来，等到有对象来订阅它的时候，再重新把消息发布给订阅者。

建立一个存放离线事件的堆栈，当事件发布的时候，如果此时还没有订阅者来订阅这个事件，我们暂时把发布事件的动作包裹在一个函数里，这些包裹函数将被存入堆栈，等到有对象来订阅此事件的时候，我们将遍历堆栈并依次执行这些包裹函数。

### 8.11 全局事件的命名冲突

全局的发布-订阅对象里只有一个 `clientList` 来存放消息名和回调函数，会出现命名冲突的情况，因此可以给 `Event` 对象提供创建命名空间的功能。

```javascript
var Event = (function () {
  var global = this,
      Event,
      _default = 'default';

  Event = function () {
    var _listen,
        _trigger,
        _remove,
        _slice = Array.prototype.slice,
        _shift = Array.prototype.shift,
        _unshift = Array.prototype.unshift,
        namespaceCache = {},
        _create,
        find,
        each = function (ary, fn) {
          var ret;
          for (var i = 0, l = ary.length; i < l; i++) {
            var n = ary[i];
            ret = fn.call(n, i, n);
          }
          return ret;
        };

        _listen = function (key, fn, cache) {
          if (!cache[key]) {
            cache[key] = [];
          }
          cache[key].push(fn);
        };

        _remove = function (key, cache, fn) {
          if (cache[key]) {
            if (fn) {
              for (var i = cache[key].length; i >= 0; i--) {
                if (cache[key] === fn) {
                  cache[key].splice(i, 1);
                }
              }
            } else {
              cache[key] = [];
            }
          }
        };

        _trigger = function () {
          var cache = _shift.call(arguments),
              key = _shift.call(arguments),
              args = arguments,
              _self = this,
              ret,
              stack = cache[key];

          if (!stack || !stack.length) {
            return;
          }

          return each(stack, function () {
            return this.apply(_self, args);
          });
        };

        _create = function (namespace) {
          var namespace = namespace || _default;
          var cache = {},
              offlineStack = [],   // 离线事件
              ret = {
                listen: function (key, fn, last) {
                  _listen(key, fn, cache);
                  if (offlineStack === null) {
                    return;
                  }
                  if (last === 'last') {
                    offlineStack.length && offlineStack.pop()();
                  } else {
                    each(offlineStack, function () {
                      this();
                    });
                  }

                  offlineStack = null;
                },
                one: function (key, fn, last) {
                  _remove(key, cache);
                  this.listen(key, fn, last);
                },
                remove: function (key, fn) {
                  _remove(key, cache, fn);
                },
                trigger: function () {
                  var fn,
                      args,
                      _self = this;

                  _unshift.call(arguments, cache);
                  args = arguments;
                  fn = function () {
                    return _trigger.apply(_self, args);
                  };

                  if (offlineStack) {
                    return offlineStack.push(fn);
                  }
                  return fn();
                }
              };

              return namespace ? (namespaceCache[namespace] ? namespaceCache[namespace] : namespaceCache[namespace] = ret) : ret;
        };

        return {
          create: _create,
          one: function (key, fn, last) {
            var event = this.create();
            event.one(key, fn, last);
          },
          remove: function (key, fn) {
            var event = this.create();
            event.remove(key, fn);
          },
          listen: function (key, fn, last) {
            var event = this.create();
            event.listen(key, fn, last);
          },
          trigger: function () {
            var event = this.create();
            event.trigger.apply(this, arguments);
          }
        };
  }();

  return Event;
})();
```

客户调用：

```javascript
// 先发布后订阅
Event.trigger('click', 1);
Event.listen('click', function (a) {
  console.log(a);  // 输出：1
});

// 使用命名空间
Event.create('namespace1').listen('click', function (a) {
  console.log(a);  // 输出：1
});

Event.create('namespace1').trigger('click', 1);

Event.create('namespace2').listen('click', function (a) {
  console.log(a);  // 输出：2
});

Event.create('namespace2').trigger('click', 2);
```

### 8.12 JavaScript 实现发布-订阅模式的便利性

在 Java 中实现一个自己的订阅-发布模式，通常会把订阅者对象自身当成引用传入发布者对象中，同时订阅者对象还需提供一个名为诸如 `update` 的方法，供发布者在适合的时候调用。在 JavaScript 中，用注册回调函数的形式来代替传统的发布-订阅模式，更加优雅和简单。

在 JavaScript 中，我们无需去选择使用推模型还是拉模型。推模型是指在事件发生时，发布者一次性把所有变更的状态和数据都推送给订阅者。拉模型中，发布者仅仅通知订阅者事件已经发生，此外发布者要提供一些公开的接口供订阅者主动拉取数据。拉模型的好处是可以让订阅者“按需获取”，但同时会增加代码量和复杂度。

在 JavaScript 中，`arguments` 可以很方便地表示参数列表，所以一般会选择推模型，使用 `Function.prototype.apply` 方法把所有参数推送给订阅者。

### 8.13 小结

优点：一为时间上的解耦，二为对象之间的解耦。既可以用在异步编程中，也可以帮助完成更松耦合的代码编写，还可以用来实现一些别的设计模式，比如中介者模式。架构上看，MV* 少不了发布-订阅模式，JavaScript 本身也是一门基于事件驱动的语言。

缺点：创建订阅者本身消耗一定的时间和内存，当订阅一个消息后，也许此消息最后都未发生，但这个订阅者会始终存在于内存中。另外，发布-订阅模式虽可以弱化对象之间的联系，但如果过度使用的话，对象和对象之间的必要联系也会被深埋在背后，导致程序难以跟踪维护和理解，特别是有多个发布者和订阅者嵌套到一起的时候。


