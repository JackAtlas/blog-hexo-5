title: 《JavaScript 设计模式与开发实战》读书笔记 13
date: 2017-04-08 00:12:17

---

## 第十三章 职责链模式

<!-- more -->

> 使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

职责链模式的最大优点：请求发送者只需要知道链中的第一个节点，从而弱化了发送者和一组接收者之间的强联系。

## 13.4

链中的各个节点需要可以灵活拆分和重组。

```javascript
var order500 = function(orderType, pay, stock) {
    if (orderType === 1 && pay === true) {
        console.log('500元定金预购，得到100优惠券')
    } else {
        return 'nextSuccessor'   // 我不知道下一个节点是谁，反正把请求往后传递
    }
}

var order200 = function(orderType, pay, stock) {
    if (orderType === 2 && pay === true) {
        console.log('200元定金预购，得到50优惠券')
    } else {
        return 'nextSuccessor'   // 我不知道下一个节点是谁，反正把请求往后传递
    }
}

var orderNormal = function(orderType, pay, stock) {
    if (stock > 0) {
        console.log('普通购买，无优惠券')
    } else {
        console.log('手机库存不足')
    }
}
```

接下来把函数装进职责链节点，定义一个构造函数 Chain，在 new Chain 的时候传递的参数即为需要被包装的函数，同时它还有一个实例属性 this.successor，表示在链中的下一个节点。

此外 Chain 的 prototype 中还有两个函数，作用如下：

```javascript
// Chain.prototype.setNextSuccessor 指定在链中的下一个节点
// Chain.prototype.passRequest 传递请求给某个节点

var Chain = function(fn) {
    this.fn = fn
    this.successor = null
}

Chain.prototype.setNextSuccessor = function(successor) {
    return this.successor = successor
}

Chain.prototype.passRequest = function() {
    var ret = this.fn.apply(this, arguments)

    if (ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor, arguments)
    }
}
```

把3个订单函数分别包装成职责链的节点：

```javascript
var chainOrder500 = new Chain(order500)
var chainOrder200 = new Chain(order200)
var chainOrderNormal = new Chain(orderNormal)
```

然后指定节点在职责链中的顺序：

```javascript
chainOrder500.setNextSuccessor(chainOrder200)
chainOrder200.setNextSuccessor(chainOrderNormal)
```

把请求传递给第一个节点

```javascript
chainOrder500.passRequest(1, true, 500) // 输出：500元定金预购，得到100优惠券
```

如此，可灵活增加、删除和修改链中的节点顺序。


## 13.5 异步的职责链

给 Chain 类再增加一个原型方法 Chain.prototype.next，表示手动传递请求给职责链中的下一个节点：

```javascript
Chain.prototype.next = function() {
    return this.successor && this.successor.passRequest.apply(this.successor, arguments)
}
```

例子：

```javascript
var fn1 = new Chain(function() {
    console.log(1)
    return 'nextSuccessor'
})

var fn2 = new Chain(function() {
    console.log(2)
    var self = this
    setTimeout(function() {
        self.next()
    }, 1000)
})

var fn3 = new Chain(function() {
    console.log(3)
})

fn1.setNextSuccessor(fn2).setNextSuccessor(fn3)
fn1.passRequest()
```

请求在链中的节点里传递，但节点有权决定什么时候把请求交给下一个节点。可以想象，一部的职责链加上命令模式（把ajax请求封装成命令对象，详见第九章），可以很方便地创建一个ajax队列库。

## 13.6 职责链模式的优缺点

职责链模式的最大优点就是解耦了请求发送者和 N 个接收者之间的复杂关系，由于不知道链中的哪个节点可以处理你发出的请求，所以你只需把请求传递给第一个节点即可。

使用了职责链模式后，链中的节点对象可以灵活地拆分重组。增加或者删除一个节点，或者改变节点在链中的位置都是轻而易举的事情。

还有一个优点，可以手动指定起始节点。

这种模式并非没有弊端。首先我们不能保证某个请求一定会被链中的节点处理，此时的请求就得不到答复，而是径直从链尾离开，或者抛出一个异常错误。在这种情况下，可以在链尾增加一个保底的接受者来处理这种即将离开链尾的请求。

另外，职责链模式使得程序中多了一些节点对象，可能在某一次的请求传递过程中，大部分节点并没有起到实质性的作用，仅仅是让请求传递下去，从性能方面考虑，要避免过长的职责链带来的性能损耗。

## 13.7 用 AOP 实现职责链

之前的实现中，我们用一个 Chain 类来把普通函数包装成职责链的节点。利用 JavaScript 的函数式特性，有一种更方便的方法来创建职责链。

改写一下第三章 3.2.3 的 `Function.prototype.after` 函数，使得第一个函数返回 'nextSuccessor' 时，将请求继续传递给下一个函数。

```javascript
Function.prototype.after = function(fn) {
    var self = this;
    return function() {
        var ret = self.apply(this, arguments)
        if (ret === 'nextSuccessor') {
            return fn.apply(this, arguments)
        }

        return ret
    }
}

var order = order500yuan.after(order200yuan).after(orderNormal)

order(1, true, 500)   // 输出：500元定金预购，得到100优惠券
```

## 13.8 用职责链模式获取文件上传对象

在第七章有一个用迭代器获取文件上传对象的例子，其实用职责链模式可以更简单。

```javascript
var getActiveUploadObj = function() {
    try {
        return new ActiveXObject('TXFNActiveX.FTNUpload') // IE
    } catch (e) {
        return 'nextSuccessor'
    }
}

var getFlashUploadObj = function() {
    if (supportFlash()) {
        var str = '<object type="application/x-shockwave-flash"></object>'
        return $(str).appendTo($('body'))
    }
    return 'nextSuccessor'
}

var getFormUploadObj = function() {
    return $('<form><input name="file" type="file" /></form>').appendTo($('body'))
}

var getUploadObj = getActiveUploadObj.after(getFlashUploadObj).after(getFormUploadObj)

console.log(getUploadObj())
```

## 13.9 小结

无论是作用域链、原型链，还是 DOM 节点中的事件冒泡，我们都能从中找到职责链模式的影子。职责链模式还可以和组合模式结合在一起，用来连接部件和父部件，或是提高组合对象的效率。