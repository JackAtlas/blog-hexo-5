title: （译）React AJAX 最佳实践
date: 2017-01-18 19:49:06

---

当你开始注意 AJAX 与 React 问题时，专家们告诉你的第一件事就是 React 是个视图类库，没有网络、AJAX 功能。

虽然知道这点挺好的，但没什么用，尤其是当你只想在 React 组件里向服务器获取数据的时候。

真相是，有很多方法可以做到。你自己可能就想到几种，但要是你选择了错误的方法，你的代码就回变得一团糟。

因此你就有疑问：什么事“对的”或者“首选”的方法？

> 在 React 组件中向服务器获取数据的最佳实践是怎样的？

答案是……看情况。

## 四种方法

我列举了四种在 React 中使用 AJAX 的不错的方法。

你使用哪种方法取决于你 app 的体量及复杂程度，以及你已经在使用的类库和技术。

1. 根组件

	![根组件](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-root-component.png)

2. 容器组件

	![容器组件](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-container-components.png)

3. Redux 异步 Actions

	![Redux 异步 Actions](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-redux-async-actions.png)

4. Relay

	![Relay](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-relay.png)

## 1. 根组件

![根组件](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-root-component.png)

这是最简单的方法，因此很适合原型或者小型应用。

在此方法中，你构建一个单一的根/父组件，用它来执行你所有的 AJAX 请求。

做为例子，可以看看 React 官方的教程。CommentBox 组件就是一个发送所有 AJAX 请求的根组件。

我不喜欢官方例子的原因：他们使用 jQuery 发送 AJAX 请求。jQuery 是一个很庞大有很多功能的类库，仅用于 AJAX 并不合理。

我推荐使用 `fetch()`。这是一个简单、标准化的 AJAX API。它已经被 Chrome 和 Firefox 所支持，node 和其他浏览器也有“补丁”。具体细节，或者供选择 AJAX 类库时参考，请看我的《[AJAX 库对比](http://andrewhfarmer.com/ajax-libraries/)》（原作者的）。

**其他说明**：如果你有一个很深的组件树（子组件甚至后代组件）那么你就需要从根组件经过一条很长的路将数据传递给深层的组件。

什么时候适合在根组件发起 AJAX 请求？

- app 的组件树不深；
- app 中没有使用 Redux 或者 Flux。

## 2. 容器组件

![容器组件](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-container-components.png)

一个容器组件“给表现层或其他容器组件提供数据和行为”。如果你从没听过这个说法，我建议你读一下 Dan Abramov 的《[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)》。

对于我们的目标，容器组件方法和根组件方法非常相似，除了复数可与服务器交互的组件。

它的工作原理是这样的：给每一个需要从服务器获取数据的表现层组件创建一个容器组件，用此容器组件发送 AJAX 请求获取数据，通过属性传递给子组件。

举个具体例子，想象你想展示一个用户简介，有名字和一张照片。

首先创建一个 `<UserProfile />` 表现层组件，其接收一个 `name` 和 `profileImage` 属性。这个组件不应该有任何的 AJAX 代码。

然后创建一个 `<UserProfileContainer />` 组件接收 `userId`。它会下载关于那个用户的数据，然后通过参数传递给 `<UserProfile />` 组件。

容器组件可以通过一个简单的 AJAX 库来发起 AJAX 请求。我推荐 `fetch()`。

什么时候适合在容器组件发起 AJAX 请求？

- app 的组件树很深；
- 大部分组件不从服务端请求数据，只有一部分需要；
- 要从不同的 API 或者终端请求数据；
- app 中没有使用 Redux 或者 Flux，或者相比 "异步 actions" 你更喜欢容器组件。

## 3. Redux 异步 Actions

![Redux 异步 Actions](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-redux-async-actions.png)

Redux 管理数据，AJAX 从服务端获取数据，由 Redux 代码来处理网络请求是合理的。

如果你正在使用 Redux，别把 AJAX 放在你的 React 组件中。而是放在你的异步 Actions 中。

我推荐使用 `fetch()` 来发起真正的网络请求。幸运地，这也是 Redux 官方文档中使用的。他们甚至写了一个含 Redux、React、`fetch()` 和 reddit API 的例子。

如果你正在使用另外一种 flux 库，方法是简单的——在 actions 中发起网络请求。

什么时候适合在 Redux 异步 Actions 中发起 AJAX 请求？

- 如果你在使用 Redux，这就是合适的方法；
- 如果你在使用其他的 flux 库，会有相似能用的方法。

## 4. Relay

![Relay](http://7xrurp.com1.z0.glb.clouddn.com/react-ajax-relay.png)

通过 GraphQL 声明 React 组件所需的数据，[Relay](http://facebook.github.io/relay/) 就会自动下载数据并会填充到组件属性。

Relay 在大型应用中使用良好，但需要较大的学习成本。你需要：

- 学习 Relay 和 GraphQL；
- 用 GraphQL（而不是用 `propTypes`）指出 React 组件所需的数据；
- 搭建一个 GraphQL 服务器；

Relay 只用于与 GraphQL 服务器通信，对于任何第三方 API 将无能为力。

目前，**Relay 只能与一个 GraphQL 服务器通信**，因此如果你从多个源获取数据，这个方法不适合你。未来将实现与多个服务器通信的功能，这个 [github issue](https://github.com/facebook/relay/issues/114) 中有更深入的讨论。

如果你想用这个方法，[Relay Playground](https://facebook.github.io/relay/prototyping/playground.html) 是一个非常好的地方让你弄明白 Relay 是如何工作的。

什么时候适合使用 Relay？

- 你在构建大型应用，而且你所担忧的问题正好是 Relay 所解决的；
- 还没构建 JSON API；
- 你愿意搭建一个 GraphQL 服务器；
- 你的 app 仅仅与单一服务端通信。

## 反面模式

如果上述所有的方法都是正确的，那么哪些方法是错误的？这里有两种典型的方法是我所反对的。

### 反面模式1: 在表现层组件中发起 AJAX 请求

不要在已经有其他职责的组件中添加 AJAX 代码——比如复杂的交互渲染。这样做会违反[关注分离](https://en.wikipedia.org/wiki/Separation_of_concerns)的设计原则。

### 反面模式2: ReactDOM.render()

你可以将 AJAX 逻辑与 React 完全分离，从服务器获取到数据之后再执行 [ReactDOM.render()](https://facebook.github.io/react/docs/top-level-api.html#reactdom.render) 方法。

这个方法或许可行，我将之列为反面模式是因为我相信根组件方法与之类似且更简洁。

## 结束语

用 React 构建的应用是*模块化*的。React 只是其中的一个模块，AJAX 库也是。这不是 Rails 或 Angular。

