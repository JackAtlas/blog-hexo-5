title: 《JavaScript 设计模式与开发实战》读书笔记 6
date: 2017-02-13 13:01:56

---

## 第六章 代理模式
<!-- more -->

> 代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

代理模式的关键是，当客户不方便直接访问一个对象或者不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际上访问的是替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象。

### 6.2 保护代理和虚拟代理

代理可以帮助本体过滤掉一些请求，这种代理叫做*保护代理*。把一些开销很大的对象，延迟到真正需要它的时候才去创建，这种代理叫做*虚拟代理*。

保护代理用于控制不同权限的对象对目标对象的访问，但在 JavaScript 并不容易实现保护代理，因为我们无法判断谁访问了某个对象。而虚拟代理是最常用的一种代理模式。

### 6.3 虚拟代理实现图片预加载

场景：先用一张 loading 图片占位，然后用异步的方式加载图片，等图片加载好了再把它填充到 img 节点里。

首先创建一个普通的本体对象，这个对象负责往页面中创建一个 `img` 标签，并且提供一个对外的 `setSrc` 接口，外界调用这个接口，便可以给该 `img` 标签设置 `src` 属性：

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

myImage.setSrc('a.jpg');
```

代理对象 `proxyImage`，在图片被真正加载好之前，页面中将出现一张占位的菊花图 loading.gif 来提示用户图片正在加载。

```javascript
var proxyImage = (function () {
  var img = new Image;
  img.onload = function () {
    myImage.setSrc(this.src);
  }
  return {
    setSrc: function (src) {
      myImage.setSrc('loading.gif');
      img.src = src;
    }
  }
})();

proxyImage.setSrc('a.jpg');
```

现在我们通过 `proxyImage` 间接地访问 `MyImage`。`proxyImage` 控制了客户对 `MyImage` 的访问，并且在此过程中加入一些额外的操作，比如在真正的图片加载好之前，先把 `img` 节点的 `src` 设置为一张本地的 `loading` 图片。

## 6.4 代理的意义

不用代理的预加载图片函数：

```javascript
var MyImage = (function () {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);
  var img = new Image;

  img.onload = function () {
    imgNode.src = img.src;
  };

  return {
    setSrc: function (src) {
      imgNode.src = 'loading.gif';
      img.src = src;
    }
  }
})();

MyImage.setSrc('a.jpg');
```

> 单一职责原则：就一个类（通常也包括对象和函数等）而言，应该仅有一个引起它变化的原因。

如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起它变化的原因可能会有多个。面向对象设计鼓励将行为分布到细粒度的对象之中，如果一个对象承担的职责过多，等于把这些职责耦合到了一起，这种耦合会导致脆弱和低内聚的设计。

上段代码中的 `MyImage` 对象除了负责给 `img` 节点设置 `src` 外，还要负责预加载图片。我们在处理其中一个职责时，有可能因为其强耦合性影响另外一个职责的实现。

另外，在面向对象的程序设计中，大多数情况下，若违反其他任何原则，同时将违反开放-封闭原则。如果只是从网络上获取一些体积很小的图片，或者以后的网速快到根本不需要预加载，可能希望把预加载图片的代码从 `MyImage` 对象里删除，这时就不得不改动 `MyImage` 对象了。

纵观整个程序，我们没有改变或者增加 `MyImage` 的接口，但是通过代理对象，实际上给系统添加了新的行为。这是符合开放-封闭原则的。给 `img` 节点设置 `src` 和图片预加载这两个功能，被隔离在两个对象里，它们可以各自变化而不影响对方。如果有一天不需要预加载，只需改成请求本体而不是请求代理对象即可。

### 6.5 代理和本体接口的一致性

如果有一天不需要预加载，就不需要代理对象，直接请求本体。关键是代理对象和本体都对外提供了 `setSrc` 方法，在客户看来，代理对象和本体是一致的，代理接受请求的过程对于用户来说是透明的，用户并不清楚代理和本体的区别。

- 用户可以放心地请求代理，只关心能否得到想要的结果。
- 在任何使用本体的地方都可以替换成使用代理。

在 Java 等语言中，代理和本体都需要显式地实现同一个接口，一方面接口保证了它们会拥有同样的方法，另一方面，面向接口编程迎合依赖倒置原则，通过接口向上转型，从而避开编译器的类型检查，代理和本体将来可以被替换使用。

在 JavaScript 这种动态类型语言中，我们有时通过鸭子类型来检测代理和本体是否都实现了 `setSrc` 方法，大多数时候干脆不做检测，全部依赖程序猿的自觉性，这对于程序的健壮性是有影响的。不过对于一门快速开发的脚本语言，这些影响还是在可以接受的范围内。

如果代理对象和本体对象都为一个函数（函数也是对象），函数必然都能被执行，则可以认为它们也具有一致的“接口”：

```javascript
var myImage = (function () {
  var imgNode = document.createElement('img');
  document.body.appendChild(imgNode);

  return function (src) {
    imgNode.src = src;
  }
})();

var proxyImage = (function () {
  var img = new Image;
  img.onload = function () {
    myImage(this.src);
  }

  return function (src) {
    myImage('loading.gif');
    img.src = src;
  }
})();

proxyImage('a.jpg');
```

### 6.6 虚拟代理合并 HTTP 请求

在 Web 开发中，也许最大的开销就是网络请求。假设我们在做一个文件同步的功能，当我们选中一个 checkbox 的时候，它对应的文件就会被同步到另外一台备用服务器上面。给这些 checkbox 绑定点击事件，并在点击的同时往另一台服务器同步文件：

```javascript
var synchronousFile = function (id) {
  console.log('开始同步文件，id 为：' + id);
};

var checkbox = document.getElementsByTagName('input');
for (var i = 0, c; c = checkbox[i++];) {
  c.onclick = function () {
    if (this.checked === true) {
      synchronousFile(this.id);
    }
  }
};
```

当选中多个 checkbox 的时候，依次往服务器发送了多次同步文件的请求，频繁的网络操作将会带来巨大的开销。

我们可以通过一个代理函数 `proxySynchronousFile` 来收集一段时间之内的请求，最后一次性发送给服务器。如果不是对实时性要求非常高的系统，短时间的延迟不会带来太大的副作用，却能大大减轻服务器的压力。

```javascript
var synchronousFile = function (id) {
  console.log('开始同步文件，id 为：' + id);
};

var proxySynchronousFile = (function () {
  var cache = [],                // 保存一段时间内需要同步的 ID
      timer;                     // 定时器

  return function (id) {
    cache.push(id);
    if (timer) {               // 保证不会覆盖已经启动的定时器
      return;
    }

    timer = setTimeout(function () {
      synchronousFile(cache.join(',')); // 2 秒后向本体发送需要同步的 ID 集合
      clearTimeout(timer);   // 清空定时器
      timer = null;
      cache.length = 0;      // 清空 ID 集合
    }, 2000);
  }
})();

var checkbox = document.getElementsByTagName('input');

for (var i = 0, c; c = checkbox[i++];) {
  c.onclick = function () {
    if (this.checked === true) {
      proxySynchronousFile(this.id);
    }
  }
};
```

### 6.7 虚拟代理在惰性加载中的应用

假设存在一个仿控制台的组件 miniConsole.js，代码量在 1000 行左右，我们并不想一开始就加载这么大的 JS 文件，因为也许不是每个用户都需要打印 log。我们希望在有必要的时候才开始加载它，比如当用户按下 F2 主动唤出控制台的时候。

在 miniConsole.js 加载之前，为了能够让用户正常使用里面的 API，通常是用一个占位 miniConsole 代理对象来给用户提前使用，这个代理对象提供给用户的接口，跟实际的 miniConsole 是一样的。

用户使用这个代理对象来打印 log 的时候，并不会真正在控制台内打印日志，更不会在页面中创建任何 DOM 节点，真正的 miniConsole.js 还没有被加载。

可以把打印 log 的请求都包裹在一个函数里面，这个包装了请求的函数就相当于其他语言中命令模式中的 `Command` 对象。随后这些函数将全部被放到缓存队列中，这些逻辑都是在 miniConcole 代理对象中完成实现的。等用户唤出控制台的时候，才开始加载真正的 miniConcole.js 的代码，加载完成之后将遍历 miniConcole 代理对象中的缓存函数队列，同时依次执行它们。

```javascript
var cache = [];

var miniConsole = {
  log: function () {
    var args = arguments;
    cache.push(function () {
      return miniConcole.log.apply(miniConcole, args);
    });
  }
};

miniConcole.log(1);
```

当用户按下 F2 时，开始加载真正的 miniConsole.js：

```javascript
var handler = function (ev) {
  if (ev.keyCode == 113) {
    var script = document.createElement('script');
    script.onload = function () {
      for (var i = 0, fn; fn = cache[i++];) {
        fn();
      }
    };
    script.src = 'miniConsole.js';
    document.getElementsByTagName('head')[0].appendChild(script);
  }
};

document.body.addEventListener('keydown', handler, false);

// miniConsole.js 代码：
miniConsole = {
  log: function () {
    //真正代码略
    console.log(Array.prototype.join.call(arguments));
  }
};
```

这里还要注意一个问题，就是我们要保证在 F2 被重复按下的时候，miniConsole.js 只被加载一次。另外整理一下 minConsole 代理对象的代码，使它成为一个标准的虚拟代理对象：

```javascript
var miniConcole = (function () {
  var cache = [];
  var handler = function (ev) {
    if (ev.keyCode === 113) {
      var script = document.createElement('script');
      script.onload = function () {
        for (var i = 0, fn; fn = cache[i++];) {
          fn();
        }
      };
      script.src = 'miniConsole.js';
      document.getElementsByTagName('head')[0].appendChild(script);
      document.body.removeEventListener('keydown', handler); // 只加载一次 miniConsole.js
    }
  };

  document.body.addEventListener('keydown', handler, false);

  return {
    log: function () {
      var args = arguments;
      cache.push(function () {
        return miniConsole.log.apply(miniConsole, args);
      });
    }
  }
})();
```

### 6.8 缓存代理

缓存代理可以为一些开销大的运算结果提供暂时的缓存，在下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。

### 6.8.1 计算乘积

```javascript
var mult = function () {
  console.log('开始计算乘积');
  var a = 1;
  for (var i = 0, l = arguments.length; i < l; i++) {
    a = a * arguments[i];
  }
  return a;
};

mult(2, 3);     // 输出：6
mult(2, 3, 4);  // 输出：24
```

加入缓存代理函数：

```javascript
var proxyMult = (function () {
  var cache = {};
  return function () {
    var args = Array.prototype.join.call(arguments, ',');
    if (args in cache) {
      return cache[args];
    }
    return cache[args] = mult.apply(this, arguments);
  }
})();

proxyMult(1, 2, 3, 4);  // 输出：24
proxyMult(1, 2, 3, 4);  // 输出：24
```

当我们第二次调用 `proxyMult(1, 2, 3, 4)` 的时候，本体 `mult` 函数并没有被计算，`proxyMult` 直接返回了之前缓存好的计算结果。


#### 6.8.2 缓存代理用于 ajax 异步请求数据

实现方式跟计算计算乘积的例子差不多，唯一不同的是，请求数据是个异步的操作，我们无法直接把计算结果放倒代理对象的缓存中，而是要通过回调的方式。

## 6.9 用高阶函数动态创建代理

通过传入高阶函数这种更加灵活的方式，可以为各种计算方法创建缓存代理。现在这些计算方法被当作参数传入一个专门用于创建缓存代理的工厂中。

```javascript
var mult = function () {
  // 乘法
  // ...
};

var plus = function () {
  // 加法
  // ...
};

var createProxyFactory = function (fn) {
  var cache = [];
  return function () {
    var args = Array.prototype.join.call(arguments, ',');
    if (args in cache) {
      return cache[args];
    }
    return cache[args] = fn.apply(this, arguments);
  }
};

var proxyMult = createProxyFactory(mult),
    proxyPlus = createProxyFactory(plus);
```

### 6.10 其他代理模式

- 防火墙代理：控制网络资源的访问，保护主题不让“坏人”接近。
- 远程代理：为一个对象在不同的地址空间提供局部代表。
- 保护代理：用于对象应该有不同访问权限的情况。
- 智能引用代理：取代了简单的指针，它在访问对象时执行一些附加操作，比如计算一个对象被引用的次数。
- 写时复制代理：通常用于复制一个庞大对象的情况。写时复制代理延迟了复制的过程，当对象被真正修改时，才对它进行复制操作。DLL（操作系统中的动态链接库）是其典型运用场景。

### 6.11 小结

代理模式包括许多小分类，在 JavaScript 开发中最常用的是虚拟代理和缓存代理。虽然代理模式非常有用，但我们在编写业务代码的时候，往往不需要去预先猜测是否需要使用代理模式，当真正发现不方便直接访问某个对象的时候，再编写代理也不迟。

