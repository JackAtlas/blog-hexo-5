title: 《JavaScript 设计模式与开发实战》读书笔记 15
date: 2017-04-22 23:11:46

---

第十五章 装饰者模式
<!-- more -->

在程序开发中，很多时候并不希望某个类天生就非常庞大，一次性包含许多职责。装饰者模式可以动态给某个对象添加一些额外的职责，不会影响从这个类中派生出的其他对象。

在传统的面向对象语言中，给对象添加功能常常使用继承的方式，但是继承的方式并不灵活，还会带来许多问题：以方便导致超类和子类之间存在强耦合性，当超类改变时，子类也会随之改变；另一方面，超类的内部细节是对子类可见的，继承这种功能复用方式常常被认为破坏了封装性。

使用继承完成一些功能复用时，有可能创建出大量的子类，使子类的数量爆炸性增长。

装饰者模式能在不改变对象自身的基础上，在程序运行期间给对象动态地添加职责。

## 15.1 模拟传统面向对象语言的装饰者模式

作为一门解释执行的语言，给 JavaScript 中的对象动态添加或者改变职责是一件简单的事情，虽然这种做法改动了对象自身，跟传统定义中的装饰者模式不一样，但更符合 JavaScript 的语言特色。

```javascript
var obj = {
    name: 'sven',
    address: '深圳市'
}

obj.address = obj.address + '福田区'
```

传统面向对象语言的装饰者模式模拟：

```javascript
var Plane = function(){}

Plane.prototype.fire = function() {
    console.log('发射普通子弹')
}

var MissileDecorator = function(plane) {
    this.plane = plane
}

MissileDecorator.prototype.fire = function() {
    this.plane.fire()
    console.log('发射导弹')
}

var AtomDecorator = function(plane) {
    this.plane = plane
}

AtomDecorator.prototype.fire = function() {
    this.plane.fire()
    console.log('发射原子弹')
}
```

导弹类和原子弹类的构造函数都接受参数 `plane` 对象，并且保存好这个对象，在它们的 `fire` 方法中，除了执行自身的操作之外，还调用 `plane` 对象的 `fire` 方法。

这种给对象动态增加职责的方式，没有真正改动对象自身，而是将对象放入另一个对象中，这些对象以一条链的方式进行引用，形成一个聚合对象。这些对象都有相同的接口（`fire` 方法），当请求到达链中的某个对象时，这个对象会执行自身的操作，随后把请求转发给链中的下一个对象。

因为装饰者和它所装饰的对象有一致的接口，所以对使用该对象的客户来说是透明的，被装饰的对象也不需要了解它曾被装饰过，这种透明性使得我们可以递归地嵌套任意多个装饰者对象。

```javascript
var plane = new Plane()
plane = new MissileDecorator(plane)
plane = new AtomDecorator(plane)

plane.fire() // 分别输出：发射普通子弹、发射导弹、发射原子弹
```

## 15.2 装饰者也是包装器

从功能上看，decorator 能很好地描述这个模式，从结构上看，wrapper 的说法更加贴切。装饰者模式将一个对象嵌入另一个对象中，实际上相当于这个对象被另一个对象包装起来，形成一条包装链。请求随着这条链依次传递给所有对象，每个对象都有处理这条请求的机会。

## 15.3 JavaScript 的装饰者

JavaScript 语言动态改变对象相当容易，可以直接改写对象或者对象的某个方法，不需要用“类”来实现装饰者模式：

```javascript
var plane = {
    fire: function() {
        console.log('发射普通子弹')
    }
}

var missileDecorator = function() {
    console.log('发射导弹')
}

var atomDecorator = function() {
    console.log('发射原子弹')
}

var fire1 = plane.fire

plane.fire = function() {
    fire1()
    missileDecorator()
}

var fire2 = plane.fire

plane.fire = function() {
    fire2()
    atomDecorator()
}

plane.fire() // 分别输出：发射普通子弹、发射导弹、发射原子弹
```

## 15.4 装饰函数

需要一个办法，在不改变原函数代码的情况下增加功能。

可通过保存原引用的方式改写某个函数：

```javascript
window.onload = function() {}

var _onload = window.onload || function() {}

window.onload = function() {
    _onload()
    alert(1)
}
```

但是这种方式存在两个问题：

- 必须维护 `_onload` 中间变量，如果函数的装饰链较长，或者需要装饰的函数变多，中间变量的数量也会变多。
- `this` 被劫持。在 `window.onload` 中没有，是因为调用普通函数 `_onload` 时，`this` 也指向 `window`，跟调用 `window.onload` 时一样。现在把 `window.onload` 换成 `document.getElementById`

```javascript
var _getElementById = document.getElementById

document.getElementById = function(id) {
    alert(1)
    return _getElementById(id)
}

var button = document.getElementById('button')

// 输出：Uncaught TypeError: Illegal invocation
```

此时的 `_getElementById` 是一个全局函数，`this` 指向 `window`，而 `document.getElementById` 方法的内部实现需要使用 `this` 引用，`this` 在这个方法内部预期是指向 `document` 而不是 `window`。

需要手动把 `document` 当作上下文 `this` 传入 `_getElementById`：

```javascript
var _getElementById = document.getElementById
document.getElementById = function() {
    alert(1)
    return _getElementById.apply(document, arguments)
}

var button = document.getElementById('button')
```

## 15.5 用 AOP 装饰函数

```javascript
Function.prototype.before = function(beforefn) {
    var __self = this  // 保存原函数的引用
    return function() {  // 返回包含了原函数和新函数的“代理”函数
        beforefn.apply(this, arguments) // 执行新函数，且保证 this 不被劫持，新函数接受的参数
                                        // 也会被原封不动地传入原函数，新函数在原函数之前执行
        return __self.apply(this, arguments)  // 执行原函数并返回原函数的执行结果，
                                              // 并且保持 this 不被劫持
    }
}

Function.prototype.after = function(afterfn) {
    var __self = this
    return function() {
        var ret = __self.apply(this, arguments)
        afterfn.apply(this, arguments)
        return ret
    }
}
```

回到之前的例子：

```javascript
window.onload = function() {
    alert(1)
}

window.onload = (window.onload || function() {}).after(function() {
    alert(2)
}).after(function() {
    alert(3)
}).after(function() {
    alert(4)
})
```

上面的实现是在 `Function.prototype` 上添加 `before` 和 `after` 方法，但许多人不喜欢这种污染原型的方法，可以把原函数和新函数都作为参数传入：

```javascript
var before = function(fn, beforefn) {
    return function() {
        beforefn.apply(this, arguments)
        return fn.apply(this, arguments)
    }
}

var a = function() {
    function() {alert(3)}
    function() {alert(4)}
}

a = before(a, function() {alert(1)})

a()
```

## 15.6 应用实例

### 15.6.1 数据统计上报

```javascript
var showLogin = function() {
    console.log('打开登录浮层')
}

var log = function() {
    console.log('上报标签为：' + this.getAttribute('tag'))
}

showLogin = showLogin.after(log)
```

### 15.6.2 用 AOP 动态改变函数的参数

```javascript
Function.prototype.before = function(beforefn) {
    var __self = this
    return function() {
        beforefn.apply(this, arguments)   // (1)
        return __self.apply(this, arguments) // (2)
    }
}
```

在 1 和 2 处可以看到，`beforefn` 和原函数 `__self` 共用一组参数列表 `arguments`，当在 `beforefn` 函数体内改变 `arguments` 的时候，原函数 `__self` 接收的参数列表也会变化。

下面例子展示如何通过 `Function.prototype.before` 方法给函数 `func` 的参数 `param` 动态添加属性 b：

```javascript
var func = function(param) {
    console.log(param)
}

func = function(function(param) {
    param.b = 'b'
})

func({ a: 'a' })
```

如，用 AOP 方法给 ajax 函数动态装饰上 `Token` 参数，保证 ajax 函数是一个相对纯净的函数，提高其复用性。

```javascript
var ajax = function(type, url, param) {
    console.log(param)
}

var getToken = function() {
    return 'Token'
}

ajax = ajax.before(function(type, url, param) {
    param.Token = getToken()
})

ajax('get', 'http://xx.com/userinfo', { name: 'sven' })

// 输出：{ name: 'sven', Token: 'Token' }
```

### 15.6.3 插件式的表单验证

分离校验输入和提交 ajax 请求的代码。

```javascript
Function.prototype.before = function(beforefn) {
    var __self = this
    return function() {
        if (beforefn.apply(this, arguments) === false) {
            // beforefn 返回 false 的情况则 return，不再实行后面的原函数
            return
        }
        return __self.apply(this, arguments)
    }
}

var validate = function() {
    if (username.value === '') {
        alert('用户名不能为空')
        return false
    }
    if (password.value === '') {
        alert('密码不能为空')
        return false
    }
}

var formSubmit = function() {
    var param = {
        username: username.value,
        password: password.value
    }
    ajax('http://xx.com/login', param)
}

formSubmit = formSubmit.before(validate)
```

`validate` 成为一个即插即用的函数，甚至可以被写成配置文件的形式，有利于我们分开维护。再利用策略模式稍加改造，就可以把这些校验规则都写成插件的形式，用在不同的项目中。

因为新函数通过 `Function.prototype.before` 或者 `Function.prototype.after` 被装饰之后，返回的实际上是一个新函数，如果在原函数上保存了一些属性，那么这些属性会丢失。

另外，这种装饰方式也叠加了函数的作用域，如果装饰的链条过长，性能上也会有影响。

## 15.7 装饰者模式和代理模式

装饰者模式和代理模式最重要的区别在于它们的意图和设计目的。

代理模式的目的是，当直接访问本体不方便或者不符合需求时，为这个本体提供一个替代者。本体定义了关键功能，代理提供或者拒绝对它的访问，或者在访问本体之前做一些额外的事情。装饰者模式的作用就是对对象动态加入行为。代理模式强调一种关系，这种关系可以静态的表达，即一开始就可以被确定。装饰者模式用于一开始不能确定对象的全部功能时。代理模式通常只有一层“代理-本体”的引用，装饰者模式经常会形成一条装饰链。