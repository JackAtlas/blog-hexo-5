title: 《JavaScript 设计模式与开发实战》读书笔记 7
date: 2017-02-17 00:04:18

---

## 第七章 迭代器模式
<!-- more -->

> 迭代器模式指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

### 7.1 jQuery 中的迭代器

```javascript
$.each([1, 2, 3], function (i, n) {
  console.log('当前下标为：' + i);
  console.log('当前值为：' + n);
});
```

### 7.2 实现自己的迭代器

```javascript
var each = function (ary, callback) {
  for (var i = 0, l = ary.length; i < l; i++) {
    callback.call(ary[i], i, ary[i]);     // 把下标和元素当作参数传给 callback 函数
  }
};

each([1, 2, 3], function (i, n) {
  alert([i, n]);
});
```

### 7.3 内部迭代器和外部迭代器

##### 1. 内部迭代器

上节的 `each` 函数属于内部迭代器，`each` 函数的内部已经定义好了迭代规则，完全接手整个迭代过程，外部只需要一次初始调用。内部迭代器调用非常方便，外界不用关心迭代器内部实现，跟迭代器的交互也仅是一次初始调用，但这也刚好是内部迭代器的缺点。由于内部迭代器的迭代规则已经被提前规定，上面的 `each` 函数就无法同时迭代 2 个数组了。

```javascript
// 比较两个数组是否相等
var compare = function (ary1, ary2) {
  if (ary1.length !== ary2.length) {
    throw new Error('ary1 和 ary2 不相等');
  }
  each(ary1, function (i, n) {
    if (n !== ary2[i]) {
      throw new Error('ary1 和 ary2 不相等');
    }
  });
  alert('ary1 和 ary2 相等');
};

compare([1, 2, 3], [1, 2, 4]); // throw new Error('ary1 和 ary2 不相等');
```

这个函数不好看，能顺利完成需求，得益于 JavaScript 里可以把函数当作参数传递的特性。在一些没有闭包的语言中，内部迭代器本身的实现也相当复杂。比如 C 语言中的内部迭代器是用函数指针来实现的，循环处理所需要的数据都要以参数的形式明确地从外部传递进去。

##### 2. 外部迭代器

外部迭代器必须显式地请求迭代下一个元素。外部迭代器增加了一些调用的复杂度，但相对也增强了迭代器的灵活性，可以手工控制迭代的过程或顺序。

```javascript
var Iterator = function (obj) {
  var current = 0;

  var next = function () {
    current += 1;
  };

  var isDone = function () {
    return current >= obj.length;
  };

  var getCurrItem = function () {
    return obj[current];
  };

  return {
    next: next,
    isDone: isDone,
    getCurrItem: getCurrItem
  }
};

var compare = function (iterator1, iterator2) {
  while (!iterator1.isDone() && !iterator2.isDone()) {
    if (iterator1.getCurrItem() !== iterator2.getCurrItem()) {
      throw new Error('iterator1 和 iterator2 不相等');
    }
    iterator1.next();
    iterator2.next();
  }

  alert('iterator1 和 iterator2 相等');
}

var iterator1 = Iterator([1, 2, 3]);
var iterator2 = Iterator([1, 2, 3]);

compare(iterator1, iterator2);   // 输出：iterator1 和 iterator2 相等
```

外部迭代器虽然调用方式相对复杂，但适用面更广，也能满足更多变的需求。

两种迭代器在实际生产中没有优劣之分，究竟使用哪个要根据需求场景而定。

### 7.4 迭代类数组对象和字面量对象

只要被迭代的聚合对象拥有 `length` 属性而且可以用下标访问，就可以被迭代。

在 JavaScript 中，`for in` 语句可以用来迭代普通字面量对象的属性。jQuery 中提供了 `$.each` 函数来封装各种迭代行为：

```javascript
$.each = function (obj, callback) {
  var value,
      i = 0,
      length = obj.length,
      isArray = isArraylike(obj);

  if (isArray) {         // 迭代类数组
    for (; i < length; i++) {
      value = callback.call(obj[i], i, obj[i]);

      if (value === false) {
        break;
      }
    }
  } else {
    for (i in obj) {   // 迭代 object 对象
      value = callback.call(obj[i], i, obj[i]);
      if (value === false) {
        break;
      }
    }
  }

  return obj;
};
```

### 7.5 倒序迭代器

GoF 中对迭代器模式的定义非常松散，因此可以有多种迭代器的实现。

如倒序迭代器：

```javascript
var reverseEach = function (ary, callback) {
  for (var l = ary.length - 1; l >= 0; l--) {
    callback(l, ary[l]);
  }
};

reverseEach([0, 1, 2], function (i, n) {
  console.log(n);  // 分别输出：2，1，0
});
```

### 7.6 中止迭代器

迭代器可以像普通 for 循环中的 `break` 一样，提供一种跳出循环的方法。

```javascript
var each = function (ary, callback) {
  for (var i = 0, l = ary.length; i < l; i++) {
    if (callback(i, ary[i]) === false) {     // callback 的执行结果返回 false，提前终止迭代
      break;
    }
  }
};

each([1, 2, 3, 4], function (i, n) {
  if (n > 3) {     // n 大于 3 的时候终止循环
    return false;
  }
  console.log(n);  // 分别输出：1，2，3
});
```

### 7.7 迭代器模式的应用举例

假设需求：根据不同的浏览器获取相应的上传组件对象：

```javascript
// 旧代码
var getUploadObj = function () {
  try {
    return new ActiveXObject('TXFTNActiveX.FTNUpload');  // IE 上传控件
  } catch (e) {
    if (supportFlash()) {  // supportFlash 函数未提供
      var str = '<object type="application/x-shockwave-flash"></object>';
      return $(str).appendTo($('body'));
    } else {
      var str = '<input name="file" type="file" />'; // 表单上传
      return $(str).appendTo($('body'));
    }
  }
};
```

以上代码为了得到一个 `upload` 对象，函数里充斥着 `try`，`catch` 以及 `if` 条件分支。缺点第一难以阅读，第二严重违反封闭-开放原则。在开发和调试过程中，需要来回切换不同的上传方式，每次改动都很痛苦。如果要增加支持另外的上传方式，唯一的方法是继续往函数里增加条件分支。

重构代码，把每种获取 `upload` 对象的方法都封装在各自的函数里，然后使用一个迭代器，迭代获取这些 `upload` 对象，知道获取到一个可用的为止：

```javascript
var getActiveUploadObj = function () {
  try {
    return new ActiveXObject('TXFTNActiveX.FTNUpload');
  } catch (e) {
    return false;
  }
}

var getFlashUploadObj = function () {
  if (supportFlash()) {
    var str = '<object type="application/x-shockwave-flash"></object>';
    return $(str).appendTo($('body'));
  }
  return false;
}

var getFormUploadObj = function () {
  var str = '<input name="file" type="file" />';
  return $(str).appendTo($('body'));
};
```

以上三个函数都有同一个约定：如果该函数里面的 `upload` 对象是可用的，则让函数返回该对象，反之返回 `false`，提示迭代器继续往后面进行迭代。

迭代器的工作：

- 提供一个可以被迭代的方法，使得 `getActiveUploadObj`、`getFlashUploadObj` 以及 `getFormUploadObj` 依照优先级被循环迭代。
- 如果正在被迭代的函数返回一个对象，则表示找到了正确的 `upload` 对象，反之如果该函数返回 `false`，则让迭代器继续工作。

```javascript
var iteratorUploadObj = function () {
  for (var i = 0, fn; fn = arguments[i++];) {
    var uploadObj = fn();
    if (uploadObj !== false) {
      return uploadObj;
    }
  }
};

var uploadObj = iteratorUploadObj(getActiveUploadObj, getFlashUploadObj, getFormUploadObj);
```

重构代码后，获取不同上传对象的方法被隔离在各自的函数里互不干扰，`try`、`catch` 和 `if` 分支不再纠缠在一起，使得我们可以很方便地维护和扩展代码。比如后来再增加 Webkit 控件上传和 HTML5 上传，只需：

```javascript
var getWebkitUploadObj = function () {
  // 具体代码略
}

var getHtml5UploadObj = function () {
  // 具体代码略
}

var uploadObj = iteratorUploadObj(getActiveUploadObj, getWebkitUploadObj, getFlashUploadObj, getHtml5UploadObj, getFormUploadObj);
```


