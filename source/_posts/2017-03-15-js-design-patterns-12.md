title: 《JavaScript 设计模式与开发实战》读书笔记 12
date: 2017-03-15 21:02:46

---

## 第十二章 享元模式

<!-- more -->

> 享元（flyweight）模式是一种用于性能优化的模式，“fly”在这里是苍蝇的意思，意为蝇量级。享元模式的核心是运用共享技术来有效支持大量细粒度的对象。

如果系统中因为创建了大量类似的对象而导致内存占用过高，享元模式就非常有用了。

### 12.1 初识享元模式

假设有 50 种男士内衣和 50 种女士内衣，需生产一些塑料模特穿内衣拍照片。

```javascript
var Model = function (sex, underwear) {
  this.sex = sex;
  this.underwear = underwear;
};

Model.prototype.takePhoto = function () {
  console.log('sex= ' + this.sex + ' underwear=' + this.underwear);
};

for (var i = 1; i <= 50; i++) {
  var maleModel = new Model('male', 'underwear' + i);
  maleModel.takePhoto();
};

for (var j = 1; j <= 50; j++) {
  var femaleModel = new Model('female', 'underwear' + j);
  femaleModel.takePhoto();
};
```

要得到一张照片，每次都需要传入 sex 和 underwear 参数，现有 50 种男内衣和 50 种女内衣，一共产生 100 个对象。如果有 10000 种内衣，程序可能因为存在太多对象而崩溃。

优化：男模特和女模特各有一个就够了。

```javascript
// 先把 underwear 参数从构造函数中移除，构造函数只接收 sex 参数。
var Model = function (sex) {
  this.sex = sex;
};

Model.prototype.takePhoto = function () {
  console.log('sex= ' + this.sex + ' underwear=' + this.underwear);
};

// 分别创建一个男模特对象和一个女模特对象。
var maleModel = new Model('male'),
    femaleModel = new Model('female');

// 给男模特一次穿上所有男装并拍照。
for (var i = 1; i <= 50; i++) {
  maleModel.underwear = 'underwear' + i;
  maleModel.takePhoto();
};

// 给女模特一次穿上所有女装并拍照。
for (var j = 1; j <= 50; j++) {
  femaleModel.underwear = 'underwear' + j;
  femaleModel.takePhoto();
};
```

### 12.2 内部状态与外部状态

享元模式要求将对象的属性划分为内部状态与外部状态（状态在这里通常指属性）。享元模式的目标是尽量减少共享对象的数量。

- 内部状态存储于对象内部。
- 内部状态可以被一些对象共享。
- 内部状态独立于具体的场景，通常不会改变。
- 外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享。

把所有内部状态相同的对象都指定为同一个共享的对象。而外部状态可以从对象身上剥离出来，并储存在外部。

剥离了外部状态的对象称为共享对象，外部状态在必要时被传入共享对象来组装成一个完整的对象。虽然组装外部状态成为一个完整对象需要花费一定的时间，但却可以大大减少系统中的对象数量，相比之下，这点时间或许是微不足道的。享元模式是一种用时间换空间的优化模式。

通常来讲，内部状态有多少种组合，系统中便存在多少个对象。

### 12.3 享元模式的通用结构

上述例子中还存在以下问题：

- 通过构造函数显式 `new` 出了男女两个 model 对象，在其他系统中，也许并不是一开始就需要所有的共享对象。
- 给 model 对象手动设置了 `underwear` 外部状态，在更复杂的系统中，不是最好的方法，因为外部状态可能会相当复杂，与共享对象之间的联系会变得困难。

通过对象工厂来解决第一个问题，只有当某种共享对象被真正需要时，才从工厂中被创建出来。第二个问题，可以用一个管理器来记录对象相关的外部状态，使这些外部状态通过某个钩子和共享对象联系起来。

### 12.4 文件上传的例子

#### 12.4.1 对象爆炸

每一个文件对应一个 JavaScript 上传对象的创建，若同时往程序里 new 了极多个对象，则会造成浏览器崩溃。

当用户选择文件之后，插件和 Flash 都会通知 Window 下的一个全局 JavaScript 函数 startUpload，文件列表被组合成一个数组 files 塞进该函数的参数列表里：

```javascript
var id = 0;

window.startUpload = function (uploadType, file) { // upload 区分是控件还是 flash
  for (var i = 0, file; file = files[i++];) {
    var uploadObj = new Upload(uploadType, file.fileName, file.fileSize);
    uploadObj.init(id++);   // 给 upload 对象设置一个唯一的 id
  }
}
```

当用户选择完文件之后，startUpload 函数会遍历 files 数组来创建对应的 upload 对象。Upload 构造函数接受 3 个参数，分别是插件类型、文件名和文件大小。这些信息已经被组装在 files 数组里返回：

```javascript
var Upload = function (uploadType, fileName, fileSize) {
  this.uploadType = uploadType;
  this.fileName = fileName;
  this.fileSize = fileSize;
  this.dom = null;
};

Upload.prototype.init = function (id) {
  var that = this;
  this.id = id;
  this.dom = document.createElement('div');
  this.dom.innerHTML = '<span>文件名称：' + this.fileName + '，文件大小：' + this.fileSize + '</span>' + '<button class="delFile">删除</button>';
  this.dom.querySelector('.delFile').onclick = function () {
    that.delFile();
  }
  document.body.appendChild(this.dom);
};
```

为了简化示例，该对象只有删除文件的功能，对应方法是 `Upload.prototype.delFile`。该方法中又一个逻辑：当被删除的文件小于 3000KB 时，该文件将被直接删除。否则页面中会弹出一个提示框，询问用户是否确认删除。

```javascript
Upload.prototype.delFile = function () {
  if (this.fileSize < 3000) {
    return this.dom.parentNode.removeChild(this.dom);
  }

  if (window.confirm('确定要删除该文件吗？' + this.fileName) {
    return this.dom.parentNode.removeChild(this.dom);
  }
};
```

创建上传对象：

```javascript
startUpload('plugin', [
  {
    fileName: '1.txt',
    fileSize: 1000
  },
  {
    fileName: '2.html',
    fileSize: 3000
  }
]);

startUpload('flash', [
  {
    fileName: '4.txt',
    fileSize: 1000
  },
  {
    fileName: '5.html',
    fileSize: 3000
  }
]);
```

#### 12.4.2 享元模式重构文件上传

`upload` 对象必须依赖 `uploadType` 属性才能工作，这是因为插件上传、Flash 上传、表单上传的实际工作原理有很大的区别，各自调用的接口也是完全不一样的，必须在对象创建之初就明确其上传类型，才可以在程序的运行过程中，分别调用各自的 `start`、`pause`、`cancel`、`del` 等方法。

一旦明确了 `uploadType`，无论使用什么方法上传，这个上传对象都可被人和文件共用的。而 `fileName` 和 `fileSize` 是根据场景变化的，每个文件的 `fileName` 和 `fileSize` 不一样，无法共享，划分为外部状态。

#### 12.4.3 剥离外部状态

```javascript
var Upload = function (uploadType) {
  this.uploadType = uploadType;
};
```

`Upload.prototype.init` 函数也不再需要，因为 `upload` 对象初始化的工作被放在了 `uploadManager.add` 函数里面，接下来只需定义 `Upload.prototype.del` 函数即可：

```javascript
Upload.prototype.delFile = function (id) {
  uploadManager.setExternalState(id, this);  // (1)
  if (this.fileSize < 3000) {
    return this.dom.parentNode.removeChild(this.dom);
  }

  if (window.confirm('确定要删除该文件吗？' + this.fileName)) {
    return this.dom.parentNode.removeChild(this.dom);
  }
};
```

在删除文件前，需读取文件大小，而文件大小的信息被储存在外部管理器 `uploadManager` 中，所以需要通过 `uploadManager.setExternalState` 方法给共享对象设置正确的 `fileSize`，(1) 中表示把当前 id 对应的对象的外部状态都组装到共享对象中。

#### 12.4.4 工厂进行对象实例化

如果某种内部状态对应的共享对象已经被创建过，则直接返回这个对象，否则创建一个新的对象：

```javascript
var UploadFactory = (function () {
  var createdFlyWeightObjs = {};

  return {
    create: function (uploadType) {
      if (createFlyWeightObjs[uploadType]) {
        return createFlyWeightObjs[uploadType];
      }

      return createdFlyWeightObjs[uploadType] = new Upload(uploadType);
    }
  }
})();
```

#### 12.4.5 管理器封装外部状态

`uploadManager` 对象负责向 `UploadFactory` 提交创建对象的请求，并用一个 `uploadDatabase` 对象保存所有 `upload` 对象的外部状态，以便在程序运行过程中给 `upload` 共享对象设置外部状态：

```javascript
var uploadManager = (function () {
  var uploadDatabase = {};

  return {
    add: function (id, uploadType, fileName, fileSize) {
      var flyWeightObj = UploadFactory.create(uploadType);

      var dom = document.createElement('div');
      dom.innerHTML = '<span>文件名称：' + fileName + '，文件大小：' + fileSize + '</span>' + '<button class="delFile">删除</button>';

      dom.querySelector('.delFile').onclick = function () {
        flyWeightObj.delFile(id);
      }

      document.body.appendChild(dom);

      uploadDatabase[id] = {
        fileName: fileName,
        fileSize: fileSize,
        dom: dom
      };

      return flyWeightObj;
    },
    setExternalState: function (id, flyWeightObj) {
      var uploadData = uploadDatabase[id];
      for (var i in uploadData) {
        flyWeightObj[i] = uploadData[i];
      }
    }
  }
})();
```

触发上传动作的 `startUpload` 函数：

```javascript
var id = 0;

window.startUpload = function (uploadType, files) {
  for (var i = 0, file; file = files[i++];) {
    var uploadObj = uploadManager.add(++id, uploadType, file.fileName, file.fileSize);
  }
};
```

运行同样的代码创建上传对象，结果与之前的一致：

```javascript
startUpload('plugin', [
  {
    fileName: '1.txt',
    fileSize: 1000
  },
  {
    fileName: '2.html',
    fileSize: 3000
  }
]);

startUpload('flash', [
  {
    fileName: '4.txt',
    fileSize: 1000
  },
  {
    fileName: '5.html',
    fileSize: 3000
  }
]);
```

重构前的代码里一共创建了 4 个 `upload` 对象，用享元模式重构后，对象数量减为 2，就算现在同时上传 2000 个文件，需要创建的 `upload` 对象数量依然是 2。

## 12.5 享元模式的适用性

享元模式是一种比较好的性能优化方案，但也会带来一些复杂性的问题，前面的例子中，使用享元模式后需要分别多维护一个 `factory` 对象和一个 `manager` 对象，在大部分不必要使用享元模式的情境下，这些开销是可以避免的。

一般来说，以下情况发生时便可以使用享元模式：

- 一个程序中使用了大量的相似对象。
- 由于使用了大量对象，造成很大的内存开销。
- 对象的大多数状态都可以变为外部状态。
- 剥离出对象的外部状态后，可以用相对较少的共享对象取代大量对象。

## 12.6 再谈内部状态和外部状态

实现享元模式的关键是把内部状态和外部状态分离开来。有多少种内部状态的组合，系统中便存在多少个共享对象，而外部状态储存在共享对象的外部，必要时被传入共享对象来组装成一个完整的对象。现在来考虑两种极端的情况，即对象没有内部状态和没有外部状态的情况。

### 12.6.1 没有内部状态的享元

在前面的例子中，如果一个网站只支持单一上传方式，这意味着之前代码中作为内部状态的 `uploadType` 属性是可以删除掉的。

```javascript
var Upload = function () {};
```

其他属性依然可以作为外部状态保存在共享对象外部。

```javascript
var UploadFactory = (function () {
  var uploadObj;
  return {
    create: function () {
      if (uploadObj) {
        return uploadObj;
      }

      return uploadObj = new Upload();
    }
  }
})();
```

管理器部分的代码不需要改动，还是负责剥离和组装外部状态。可以看到，当对象没有内部状态的时候，生产共享对象的工厂实际上变成了一个单例工厂。这时的共享对象没有内部状态的区分，但还是有剥离外部状态的过程，仍倾向于称之为享元模式。

#### 12.6.2 没有外部状态的享元

没有剥离外部状态的过程，a1 和 a2 指向同一个对象，所以即使使用了共享的技术，也不是纯粹的享元模式。

### 12.7 对象池

对象池维护一个装载空闲对象的池子，如果需要对象的时候，不是直接 new，而是转从对象池里获取。如果对象池里没有空闲对象，则创建一个新的对象，当获取出的对象完成它的职责后，再进入池子等待被下一次获取。

在 Web 前端开发中，对象池使用最多的场景大概就是跟 DOM 有关的操作。很多时间和空间都消耗在了 DOM 节点上。

#### 12.7.1 对象池实现

以地图为例，某两次搜索结果数量分别为 2 和 4。按照对象池的思想，第二次搜索开始前，并不会把第一次创建的 2 个气泡删除掉，而是放进对象池，这样在现实第二次搜索结果时，只需再创建 4 个气泡而不是 6 个。

先定义一个获取小气泡节点的工厂，作为对象池的数组称为私有属性被包含在工厂闭包里，这个工厂有两个暴露对外的方法，`create` 表示获取一个 `div` 节点，`recover` 表示回收一个节点：

```javascript
var toolTipFactory = (function () {
  var toolTipPool = [];     // toolTip 对象池

  return {
    create: function () {
      if (toolTipPool.length == 0) {     // 如果对象池为空
        var div = document.createElement('div');   // 创建一个节点
        document.body.appendChild(div);
        return div;
      } else {
        return toolTipPool.shift();     // 则从对象池中取出一个dom
      }
    },
    recover: function (tooltipDom) {
      return toolTipPool.push(tooltipDom); // 对象池回收 dom
    }
  }
})
```

第一次搜索时需要创建 2 个小气泡节点，为方便回收，用一个数组 `ary` 来记录它们：

```javascript
var ary = [];
for (var i = 0, str; str = ['A', 'B'][i++];) {
  var toolTip = toolTipFactory.create();
  toolTip.innerHTML = str;
  ary.push(toolTip);
};
```

接下来假设地图要开始重新绘制，在此之前把这两个节点回收进对象池：

```javascript
for (var i = 0, toolTip; toolTip = ary[i++];) {
  recover(toolTip);
};
```

再创建 6 个气泡：

```javascript
for (var i = 0, str; str = ['A', 'B', 'C', 'D', 'E', 'F'][i++];) {
  var toolTip = toolTipFactory.create();
  toolTip.innerHTML = str;
};
```

对象池和享元模式的思想有点相似，虽然 `innerHTML` 的值 A、B、C、D 等也可看成节点的外部状态，但在这里没有主动分离内部状态和外部状态。

#### 12.7.2 通用对象池实现

可以在对象池工厂里，把创建对象的具体过程封装起来，实现一个通用的对象池：

```javascript
var objectPoolFactory = function (createObjFn) {
  var objectPool = [];

  return {
    create: function () {
      var obj = objectPool.length === 0 ? createObjFn.apply(this, arguments) : objectPool.shift();

      return obj;
    },
    recover: function (obj) {
      objectPool.push(obj);
    }
  }
};
```

现在利用 `objectPoolFactory` 来创建一个装载一些 `iframe` 的对象池：

```javascript
var iframeFactory = objectPoolFactory(function () {
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);

  iframe.onload = function () {
    iframe.onload = null;     // 防止 iframe 重复加载的 bug
    iframeFactory.recover(iframe);     // iframe 加载完成之后回收节点
  }

  return iframe;
});

var iframe1 = iframeFactory.create();
iframe.src = 'http://baidu.com';

var iframe2 = iframeFactory.create();
iframe.src = 'http://QQ.com';

setTimeout(function () {
  var iframe3 = iframeFactory.create();
  iframe3.src = http://163.com
}, 3000)
```

对象池是另一种性能优化方案，和享元模式有一些相似之处，但没有分离内部状态和外部状态这个过程。

