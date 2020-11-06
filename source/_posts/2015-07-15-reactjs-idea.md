title: 使用 React JS 的一些心得
date: 2015-07-15 18:29:08
tags: ReactJS

---

　　公司的某个项目二期前端使用了 ReactJS 库，略有心得，在此记录。望有缘人共同交流。

## 项目背景
　　本项目改造之前，前后端协作都是前端写页面 demo，后端写模版。缺点基本上有:

- 使用 jQuery 编写页面逻辑会涉及非常多 DOM 操作，尤其是写后台界面这种富交互的 web 应用时，费时费力；
- 前端页面和后端模版有许多重复工作；
	- 效率低
	- 前端的修改后端不一定能完全覆盖
- 后端要参与到视图层面的开发；
- 后端要等前端出 demo 才能进行模版编写/修改。

　　改造前的项目是一个用 zepto（一个类似 jquery 的库）编写的单页面应用，流程只能从固定的入口进入，固定的出口离开（如同一个黑盒）。而新的需求要能直接到达其中某一步，因此决定将其拆分，由前端编写锚点路由进行流程的引导。

## 目录结构
　　改造后的目录结构如下（省略部分项目构建文件）：

|- assets  
|　　|- css  
|　　|- img  
|　　|- fonts  
|　　|- lib  
|  
|- scripts  
|　　|- build  
|　　|- components  
|　　|- views  
|　　　　|- 各模块文件夹  
|　　|- app.js  
|  
|- index.html

- index.html 项目入口，除此之外没有其他 html 文件；
- assets 存放静态资源；
- build 存放工程化后的 js 代码，被 index 所引用；
- components 通用 react 组件
- views 里面按业务模块分文件夹，存放了一系列的 react 组件，与 components 文件夹里的相比，只是通用和不通用；
- app.js 路由及各板块主脚本的引用。

　　由于是从一期项目改造，样式方面变化不大，而且项目比较紧急，因此 css 部分没有用预处理器重新编写，然而也并不是本篇文章的重点。

## 锚点路由
　　本项目中，路由的作用就是监视浏览器地址栏，根据预设的规则调用不同的 react 组件。比如地址是 `/#/usercenter` 就调用 `<UserCenter />` 组件进行页面的渲染。为什么叫“锚点”路由？因为路由规则所监视的形如 `/#/usercenter` 的地址，就是用的浏览器锚点的功能，这样也是为了能使浏览器的“前进”“后退”这样的 location 操作可以生效，也避免了 SEO 无力的问题。
　　
　　
## 组件树
　　React 的一大特点就是组件化，组件内部的改变可以通过组件 state 的设置实现，父子组件之间的通讯由子组件的 props 实现。组件层层嵌套，形成一个类似 DOM 树一般的**组件树**。而数据在“树根”处通过 Ajax 获取，层层传递往“树梢”处“流动”。而与服务器的交互（典型的比如增删改查，由 ajax 实现）。
　　
　　
## 前后端分离
　　本次项目中，后端不再参与到表现方面的开发，可以专注于业务逻辑，通过合理的约定，前后端可以“异步”开发。许多人将 React 看作是 MVC 中的 V，我之前也是这么认为的。但实际上 React 遵循的并不是典型的 MVC 之道，甚至不太赞同 MVC。前后端分离近年来是业界热议的话题，React 的出现让人们看到了 MV* 以外的可能。
　　
## 一些问题
### 组件树间通讯
　　刚开始改写的时候，我是将一些关键数据写在 localStorage 中，供组件树间共用。但有从别的域名跳转至某棵“组件树”中这样的需求实现，而 localStorage 是不能在不同的域名间共享，因此作废，改为路由传递参数，也方便其他域名作跳转。但不同的路由规则所对应的组件是相对独立的，即有若干棵“组件树”。同一棵组件树内，组件间的通讯如上文所说，但组件树之间的通讯并不方便，暂时没发现比较符合 React 之道的方法，我个人是更希望一整个 web app 可以成为一棵庞大的“组件树”。
　　
### Flux/Reflux
　　尝试过使用 Reflux + React 搭建框架，后来发现本项目中不是很有必要，可能是我对 Flux/Reflux + React 的数据流模型理解不够深刻，有待学习。
　　
### 目录结构
　　目录结构比较粗糙，可以根据前端工程化的原则进行适度的优化。
　　
## 小结
　　ReactJS 入门不难，坑也不多，编写代码结构比较清晰，在移动端上兼容性良好（本项目只在移动端），据说性能比其他 JS 框架高出不少（主要是因为 Virtual DOM），开发体验也不错，不妨一试。