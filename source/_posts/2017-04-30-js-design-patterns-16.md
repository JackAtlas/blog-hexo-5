title: 《JavaScript 设计模式与开发实战》读书笔记 16
date: 2017-04-30 12:36:11

---

第十六章 状态模式
<!-- more -->

## 16.1 模拟面向对象的状态模式实现

### 16.1.1 电灯程序

```javascript
var Light = function () {
  this.state = 'off';
  this.button = null;
}

Light.prototype.init = function() {
  var button = document.createElement('button'),
      self = this;

  button.innerHTML = '开关';
  this.button = document.body.appendChild(button);
  this.button.onclick = function () {
    self.buttonWasPressed();
  };
};

Light.prototype.buttonWasPressed = function() {
  if (this.state === 'off') {
    console.log('开灯');
    this.state = 'off';
  } else if (this.state === 'on') {
    console.log('关灯');
    this.state = 'off';
  };
};

var light = new Light();
light.init();
```

上述程序的缺点：

- `buttonWasPressed` 违反“开放-封闭”原则，每次新增或者修改 `light` 的状态，都需要改动其中的代码，使其非常不稳定。
- 所有和状态有关的行为，都封装在 `buttonWasPressed` 方法中，无法预计以后会膨胀到什么地步。
- 状态的切换不明显，仅仅表现为对 `state` 变量赋值。在实际开发中，这样的操作容易被不小心漏掉。也无法一目了然知道一共有多少种状态，除非读完 `buttonWasPressed` 里的所有代码。
- 状态之间的切换关系，不过是往 `buttonWasPressed` 方法里堆砌 if、else 语句，增加或修改一个状态可能需要改变若干操作，使其难以阅读和维护。

### 16.1.2 状态模式改进电灯程序

通常谈到封装，一般优先封装对象的行为，而不是对象的状态。但状态模式的关键是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部，所以 `button` 被按下的时候，只需在上下文中，把这个请求委托给当前的状态对象即可，该状态对象负责渲染它自身的行为。

定义三个状态类，每个类都有一个原型方法 `buttonWasPressed`，代表在各自状态下，按钮被按下时将发生的行为。

```javascript
// OffLightState
var OffLightState = function (light) {
  this.light = light;
};

OffLightState.prototype.buttonWasPressed = function() {
  console.log('弱光'); // offLightState 对应的行为
  this.light.setState(this.light.weakLightState); // 切换状态到 weakLightState
};

// WeakLightState
var WeakLightState = function (light) {
  this.light = light;
};

WeakLightState.prototype.buttonWasPressed = function() {
  console.log('强光'); // weakLightState 对应的行为
  this.light.setState(this.light.strongLightState); // 切换状态到 strongLightState
};

// StrongLightState
var OffLightState = function (light) {
  this.light = light;
};

StrongLightState.prototype.buttonWasPressed = function() {
  console.log('弱光'); // strongLightState 对应的行为
  this.light.setState(this.light.offLightState); // 切换状态到 offLightState
};
```

在 `Light` 类的构造函数里为每个状态类都创建一个状态对象，可以明显看到一共有多少种状态。

```javascript
var Light = function () {
  this.offLightState = new OffLightState(this);
  this.weakLightState = new WeakLightState(this);
  this.strongLightState = new StrongLightState(this);
  this.button = null;
};
```

在 `button` 按钮被按下的事件里，`Context` 也不再直接进行任何实质性的操作，而是通过 `self.currState.buttonWasPressed()` 将请求委托给当前持有的状态对象去执行：

```javascript
Light.prototype.init = function() {
  var button = document.createElement('button'),
      self = this;

  this.button = document.body.appendChild(button);
  this.button.innerHTML = '开关';

  this.currState = this.offLightState; // 设置当前状态
  this.button.onClick = function () {
    self.currState.buttonWasPressed();
  };
};
```

状态对象可以通过 `setState` 方法来切换 `light` 对象的状态。

```javascript
Light.prototype.setState = function(newState) {
  this.currState = newState;
};
```

上述写法的好处是，可以使每一种状态和对应的行为之间的关系局部化，这些行为被分散和封装在各自对应的状态类中，便于阅读和管理代码。

## 16.2 状态模式的定义

> 允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

前半句的意思是将状态封装成独立的类，并将请求委托给当前的状态对象，当对象的内部状态改变时，会带来不同的变化。

后半句的是从客户的角度来看，我们使用的对象，在不同的状态下具有截然不同的行为，这个对象看起来是从不同的类中实例化而来，实际上是使用了委托的效果。

## 16.3 状态模式的通用结构

首先定义了 `Light` 类，也被称为上下文（`Context`）。在 `Light` 的构造函数中，要创建每一个状态类的实例对象，`Context` 将持有这些对象的引用，以便把请求委托给状态对象。用户的请求（即点击 `button` 的动作）也是实现在 `Context` 中的。

接下来需要编写各种状态类，`light` 对象被传入状态类的构造函数，状态对象也需要持有 `light` 对象的引用，以便调用 `light` 中的方法或者直接操作 `light` 对象。

## 16.4 缺少抽象类的变通方式

JavaScript 既不支持抽象类，也没有接口的概念。所以在使用状态模式的时候需要格外小心，如果编写一个状态子类时，忘记给其实现 `buttonWasPressed` 方法，则会在状态切换时抛出异常。因为 `Context` 总会将请求委托给状态对象的 `buttonWasPressed` 方法。

解决方案是让抽象父类的抽象方法直接抛出一个异常，这个异常至少会在程序运行期间就被发现。

```javascript
var State = function () {}

State.prototype.buttonWasPressed = function() {
  throw new Error('父类的 buttonWasPressed 方法必须被重写。')
};

var SuperStrongLightState = function (light) {
  this.light = light;
};

SuperStrongLightState.prototype = new State();  // 继承抽象父类

SuperStrongLightState.prototype.buttonWasPressed = function() {
  console.log('关灯')
  this.light.setState(this.light.offLightState)
};
```

## 16.5 文件上传示例

### 16.5.1 更复杂的切换条件

- 文件在扫描状态中，不能进行任何操作。扫描完成后，根据文件的 md5 值判断，若确认该文件已经存在于服务器，则直接跳到上传完成状态。如果该文件的大小超过允许上传的最大值，或者该文件已经损坏，则跳往上传失败状态。剩下的情况才进入上传中状态。
- 上传过程中可以点击暂停按钮来暂停上传，暂停后点击同一个按钮会继续上传。
- 扫描和上传过程中，点击删除按钮无效。只有在暂停、上传完成、上传失败之后，才能删除文件。

### 16.5.2 一些准备工作

上传是一个异步过程，所以控件会不停调用 JavaScript 提供的一个全局函数 `window.external.upload`，来通知 JavaScript 目前的上传进度，控件会把当前的文件状态作为参数 `state` 塞进 `window.external.upload`。

```javascript
// 模拟的方法
window.external.upload = function (state) {
  console.log(state); // 可能为 sign、uploading、done、error
};
```

需要在页面中放置一个用于上传的插件对象：

```javascript
var plugin = (function () {
  var plugin = document.createElement('embed');
  plugin.style.display = 'none';

  plugin.type = 'application/txftn-webkit';

  plugin.sign = function () {
    console.log('开始文件扫描');
  };

  plugin.pause = function () {
    console.log('暂停文件上传');
  };

  plugin.uploading = function () {
    console.log('开始文件上传');
  };

  plugin.del = function () {
    console.log('删除文件上传');
  };

  plugin.done = function () {
    console.log('文件上传完成');
  };

  document.body.appendChild(plugin);

  return plugin;
})();
```

### 16.5.3 具体代码

先定义 `Upload` 类，控制上传过程的对象将从 `Upload` 类中创建而来：

```javascript
var Upload = function (fileName) {
  this.plugin = plugin;
  this.fileName = fileName;
  this.button1 = null;
  this.button2 = null;
  this.state = 'sign'; // 设置初始状态为 waiting
};

Upload.prototype.init = function() {
  var that = this;
  this.dom = document.createElement('div');
  this.dom.innerHTML =
  '<span>文件名称：' + this.fileName + '</span>\
  <button data-action="button1">扫描中</button>\
  <button data-action="button2">删除</button>';

  document.body.appendChild(this.dom);
  this.button1 = this.dom.querySelector('[data-action="button1"]');
  this.button2 = this.dom.querySelector('[data-action="button2"]');
  this.bindEvent();
};

Upload.prototype.bindEvent = function() {
  var self = this;
  this.button1.onclick = function () {
    if (self.state === 'sign') { // 扫描状态下，任何操作无效
      console.log('扫描中，点击无效……');
    } else if (self.state === 'uploading') { // 上传中，点击切换到暂停
      self.changeState('pause');
    } else if (self.state === 'pause') { // 暂停中，点击切换到上传中
      self.changeState('uploading');
    } else if (self.state === 'done') {
      console.log('文件上传完成，点击无效');
    } else if (self.state === 'error') {
      console.log('文件上传失败，点击无效');
    };
  };

  this.button2.onclick = function () {
    if (self.state === 'done' || self.state === 'error' || self.state === 'pause') {
      // 上传完成、上传失败和暂停状态下可以删除
      self.changeState('del');
    } else if (self.state === 'sign') {
      console.log('文件正在扫描中，不能删除');
    } else if (self.state === 'uploading') {
      console.log('文件正在上传中，不能删除');
    };
  };
};

Upload.prototype.changeState = function(state) {
  switch (state) {
    case 'sign':
      this.plugin.sign();
      this.button1.innerHTML = '扫描中，任何操作无效';
      break;
    case 'uploading':
      this.plugin.uploading();
      this.button1.innerHTML = '正在上传中，点击暂停';
      break;
    case 'pause':
      this.plugin.pause();
      this.button1.innerHTML = '已暂停，点击继续上传';
      break;
    case 'done':
      this.plugin.done();
      this.button1.innerHTML = '上传完成';
      break;
    case 'error':
      this.button1.innerHTML = '上传失败';
      break;
    case 'del':
      this.plugin.del();
      this.dom.parentNode.removeChild(this.dom);
      console.log('删除完成');
      break;
  };

  this.state = state;
};
```

测试：

```javascript
var uploadObj = new Upload('JavaScript设计模式与开发实践');

uploadObj.init();

window.external.upload = function (state) { // 插件调用 JavaScript 的方法
  uploadObj.changeState(state);
};

window.external.upload('sign');

setTimeout(function () {
  window.external.upload('uploading'); // 1 秒后开始上传
}, 1000);

setTimeout(function () {
  window.external.upload('done'); // 5 秒后上传完成
}, 5000);
```

### 16.5.4 状态模式重构

第一步提供 `window.external.upload` 函数，在页面中模拟创建上传插件 `plugin`，这部分代码没有变化。

第二步改造 `Upload` 构造函数，在构造函数中为每种状态子类都创建一个实例对象：

```javascript
var Upload = function (fileName) {
  this.plugin = plugin;
  this.fileName = fileName;
  this.button1 = null;
  this.button2 = null;
  this.signState = new SignState(this);  // 设置初始状态为 waiting
  this.uploadingState = new UploadingState(this);
  this.pauseState = new PauseState(this);
  this.doneState = new DoneState(this);
  this.errorState = new ErrorState(this);
  this.currState = this.signState; // 设置当前状态
};
```

第三步，`Upload.prototype.init` 方法无需改变。

第四步，负责具体的按钮事件实现，在点击了按钮之后，`Context` 并不做任何具体的操作，而是把请求委托给当前的状态类来执行：

```javascript
Upload.prototype.bintEvent = function() {
  var self = this;
  this.button1.onclick = function () {
    self.currState.clickHandler1();
  };
  this.button2.onclick = function () {
    self.currState.clickHandler2();
  };
};
```

把状态对应的逻辑行为放在 `Upload` 类中：

```javascript
Upload.prototype.sign = function() {
  this.plugin.sign;
  this.currState = this.signState;
};

Upload.prototype.uploading = function() {
  this.button1.innerHTML = '正在上传，点击暂停';
  this.plugin.uploading();
  this.currState = this.uploadingState;
};

Upload.prototype.pause = function() {
  this.button1.innerHTML = '已暂停，点击继续上传';
  this.plugin.pause();
  this.currState = this.pauseState;
};

Upload.prototype.done = function() {
  this.button1.innerHTML = '上传完成';
  this.plugin.done();
  this.currState = this.doneState;
};

Upload.prototype.error = function() {
  this.button1.innerHTML = '上传失败';
  this.currState = this.errorState;
};

Upload.prototype.del = function() {
  this.plugin.del();
  this.dom.parentNode.removeChild(this.dom);
};
```

第五步，编写各个状态类的实现。这里使用了 `StateFactory`，从而避免 JavaScript 中没有抽象类所带来的问题。

```javascript
var StateFactory = (function () {
  var State = function () {};

  State.prototype.clickHandler1 = function() {
    throw new Error('子类必须重写父类的 clickHandler1 方法');
  };

  State.prototype.clickHandler2 = function() {
    throw new Error('子类必须重写父类的 clickHandler2 方法');
  };

  return function (param) {
    var F = function (uploadObj) {
      this.uploadObj = uploadObj;
    };

    F.prototype = new State();

    for (var i in param) {
      F.prototype[i] = param[i];
    };

    return F;
  };
})();

var SignState = StateFactory({
  clickHandler1: function () {
    console.log('扫描中，点击无效……');
  },
  clickHandler2: function () {
    console.log('文件正在上传中，不能删除');
  };
});

var UploadingState = StateFactory({
  clickHandler1: function () {
    this.uploadObj.pause();
  },
  clickHandler2: function () {
    console.log('文件正在上传中，不能删除');
  };
});

var PauseState = StateFactory({
  clickHandler1: function () {
    this.uploadObj.uploading();
  },
  clickHandler2: function () {
    this.uploadObj.del();
  };
});

var DoneState = StateFactory({
  clickHandler1: function () {
    console.log('文件已完成上传，点击无效');
  },
  clickHandler2: function () {
    this.uploadObj.del();
  };
});

var ErrorState = StateFactory({
  clickHandler1: function () {
    console.log('文件上传失败，点击无效');
  },
  clickHandler2: function () {
    this.uploadObj.del();
  };
});
```

测试：

```javascript
var uploadObj = new Upload('JavaScript设计模式与开发实践');

window.external.upload = function (state) {
  uploadObj[state]();
};

window.external.upload('sign');

setTimeout(function () {
  window.external.upload('uploading'); // 1秒后开始上传
}, 1000);

setTimeout(function () {
  window.external.upload('done'); // 5 秒后上传完成
}, 5000);
```

## 16.6 状态模式的优缺点

优点：

- 状态模式定义了状态与行为之间的关系，并将它们封装在一个类里。通过增加新的状态类，很容易增加新的状态和转换。
- 避免 `Context` 无限膨胀，状态切换的逻辑被分布在状态类中，也去掉了 `Context` 中原本过多的条件分支。
- 用对象代替字符串来记录当前状态，使得状态的切换一目了然。
- `Context` 中的请求动作和状态类中封装的行为可以非常容易地独立变化而互不影响。

缺点是会在系统中定义许多状态类，编写状态类是一个枯燥的工作，而且会因此增加不少对象。另外，由于逻辑分布在状态类中，虽然避开了条件分支语句，但也造成了逻辑分散的问题，无法在一个地方看到整个状态机的逻辑。

## 16.7 状态模式中的性能优化点

- 有两种选择来管理 `state` 对象的创建和销毁。第一种是仅当 `state` 对象被需要时才创建并随后销毁，另一种是一开始就创建好所有的状态对象，并且始终不销毁它们。如果 `state` 对象比较庞大，可以用第一种方式来节省内存，可以避免创建不会用到的对象并及时回收。但如果状态的改变很频繁，最好一开始就把这些 `state` 对象都创建出来，也没必要销毁因为很快将再次用到。
- 在本章例子中，每个 `Context` 对象都创建了一组 `state` 对象，实际上这些对象之间是可以共享的，各 `Context` 对象共享一个 `state` 对象，这也是享元模式的应用场景之一。

## 16.8 状态模式与策略模式

策略模式和状态模式的相同点是，都有一个上下文、一些策略或者状态类，上下文把请求委托给这些类来执行。

区别是策略模式中的各个策略之间是平等有平行的，没有任何联系，所以客户必须熟知这些策略类的作用，以便随时主动切换算法；状态模式中，状态和状态对应的行为早已被封装好，状态之间的切换也早已被规定完成，“改变行为”这件事情发生在状态模式内部，对客户来说，不需要了解这些细节。

## 16.9 JavaScript 版本的状态机

前面的示例都是模拟传统面向对象语言的状态模式实现，为每个状态定义一个状态子类，然后在 `Context` 中持有这些状态对象的引用，以便把 `currState` 设置为当前的状态对象。

在 JavaScript 这种“无类”语言中，没有规定让状态对象一定要从类中创建出来。且 JavaScript 可以非常方便使用委托技术，并不需要事先让一个对象持有另一个对象。

通过 `Function.prototype.call` 方法把请求委托给某个字面量对象来执行。

```javascript
var Light = function () {
  this.currState = FSM.off;  // 设置当前状态
  this.button = null;
};

Light.prototype.init = function() {
  var button = document.createElement('button'),
      self = this;

  button.innerHTML = '已关灯';
  this.button = document.body.appendChild(button);

  this.button.onclick = function () {
    self.currState.buttonWasPressed.call(self);   // 把请求委托给 FSM 状态机
  };
};

var FSM = {
  off: {
    buttonWasPressed: function () {
      console.log('关灯');
      this.button.innerHTML = '下一次按我是开灯';
      this.currState = FSM.on;
    }
  },
  on: {
    buttonWasPressed: function () {
      console.log('开灯');
      this.button.innerHTML = '下一次按我是关灯';
      this.currState = FSM.off;
    }
  }
};

var light = new Light();
light.init();
```

利用下面的 `delegate` 函数来完成状态机编写。这是面向对象和闭包互换的一个例子，前者把变量保存为对象的属性，后者把变量封闭在闭包形成的环境中：

```javascript
var delegate = function (client, delegation) {
  return {
    buttonWasPressed: function () {
      // 将客户的操作委托给 delegation 对象
      return delegation.buttonWasPressed.apply(client, arguments);
    };
  };
};

var FSM = {
  off: {
    buttonWasPressed: function () {
      console.log('关灯');
      this.button.innerHTML = '下一次按我是开灯';
      this.currState = this.onState;
    }
  },
  on: {
    buttonWasPressed: function () {
      console.log('开灯');
      this.button.innerHTML = '下一次按我是关灯';
      this.currState = this.offState;
    }
  }
};

var Light = function () {
  this.offState = delegate(this, FSM.off);
  this.onState = delegate(this, FSM.on);
  this.currState = this.offState; // 设置初始状态为关闭
  this.button = null;
};

Light.prototype.init = function() {
  var button = document.createElement('button'),
      self = this;
  button.innerHTML = '已关灯';
  this.button = document.body.appendChild(button);
  this.button.onclick = function () {
    self.currState.buttonWasPressed();
  }
};

var light = new Light();
light.init();
```

## 16.10 表驱动的有限状态机

| |状态A|状态B|状态C|
|---|---|---|---|
|条件X|...|...|...|
|条件Y|...|状态C|...|
|条件Z|...|...|...|

```javascript
var fsm = StateMachine.create({
  initial: 'off',
  events: [
    { name: 'buttonWasPressed', from: 'off', to: 'on' },
    { name: 'buttonWasPressed', from: 'on', to: 'off' }
  ],
  callbacks: {
    onButtonWasPressed: function (event, from, to) {
      console.log(arguments);
    }
  },
  error: function (eventName, from, to, args, errorCode, errorMessage) {
    console.log(arguments); // 从一种状态试图切换到一种不可能到达的状态的时候
  }
});

button.onclick = function () {
  fsm.buttonWasPressed();
};
```

github 上有一个对应的库实现：[https://github.com/jakesgordon/javascript-state-machine][1]。


  [1]: https://github.com/jakesgordon/javascript-state-machine
