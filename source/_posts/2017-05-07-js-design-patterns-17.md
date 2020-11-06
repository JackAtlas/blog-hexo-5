title: 《JavaScript 设计模式与开发实战》读书笔记 17
date: 2017-05-07 08:02:08

---

第十七章 适配器模式
<!-- more -->

适配器模式的作用是解决两个软件实体间的接口不兼容的问题。使用适配器模式之后，原本由于接口不兼容而不能工作的两个软件实体可以一起工作。

假如正在编写一个渲染广东地图的页面。目前从第三方资源里获得了广东省的所有城市以及对应的 ID，并成功渲染到页面中：

```javascript
var getGuangdongCity = function () {
  var guangdongCity = [
    {
      name: 'shenzhen',
      id: 11
    },
    {
      name: 'guangzhou',
      id: 12
    }
  ];

  return guangdongCity;
};

var render = function (fn) {
  console.log('开始渲染广东省地图');
  document.write(JSON.stringify(fn()));
};

render(getGuangdongCity);
```

后来从另一个第三方资源获得了另外一些数据，但数据结构和正在运行的项目不一致：

```javascript
var guangdongCity = {
    shenzhen: 11,
    guangzhou: 12,
    zhuhai: 13
}
```

除了大动干戈地改写渲染页面的前端代码之外，更简便的方法是新增一个数据格式转换的适配器：

```javascript
var getGuangdongCity = function () {
  var guangdongCity = [
    {
      name: 'shenzhen',
      id: 11
    },
    {
      name: 'guangzhou',
      id: 12
    }
  ];

  return guangdongCity;
};

var render = function (fn) {
  console.log('开始渲染广东省地图');
  document.write(JSON.stringify(fn()));
};

var addressAdapter = function (oldAddressfn) {
  var address = {},
      oldAddress = oldAddressfn();

  for (var i = 0, c; c = oldAddress[i++];) {
    address[c.name] = c.id;
  }

  return function () {
    return address;
  }
};

render(addressAdapter(getGuangdongCity));
```

接下来就是把代码中调用 `getGuangdongCity` 的地方，用经过 `addressAdapter` 适配器转换之后的新函数来替代。

## 17.3 小结

适配器与装饰者模式、代理模式和外观模式之间的区别。

- 适配器模式主要用来解决两个已有接口之间不匹配的问题，不考虑这些接口是怎样实现的，也不考虑将来会如何演化。不改变已有接口，就能够使它们协同作用。
- 装饰者模式和代理模式也不改变原有对象的接口，但装饰者模式的作用是为了给对象增加功能。装饰者模式常常形成一条长的装饰链，而适配器模式通常只包装一次。代理模式是为了控制对对象的访问，通常也只包装一次。
- 有人把外观模式堪称一组对象的适配器，但外观模式最显著的特点是定义了一个新的接口。
