title: （译）异步加载 CSS 资源
date: 2015-04-21 23:58:08
tags:
- css
- js
---
本文由[小智](http://jackatlas.com/)根据 [Keith Clark](http://keithclark.co.uk/) 的 《[Loading CSS without blocking render](http://keithclark.co.uk/articles/loading-css-without-blocking-render/)》所译。译文带有我自己的理解和思想，如需转载请注明相关信息：

> 原文地址：[http://keithclark.co.uk/articles/loading-css-without-blocking-render/](http://keithclark.co.uk/articles/loading-css-without-blocking-render/)
——作者：[Keith Clark](http://keithclark.co.uk/)
——译者：[小智](http://jackatlas.com/)

本文演示如何在阻塞页面渲染之前通过异步加载尽快于用户前获取资源。

> 警告！我发出这篇文章后，迅速收到大量的社区反馈而且明显地这个技巧没有我想象中的好用。我自己测试以及在项目中使用都是成功的，但是许多开发者在 IE 和 Firefox 中发现问题（据说导致 Firefox beta 崩溃），另外一些开发者则报告 Chrome 和 Safari 中成功。

这些技巧背后的原则并不新，比如 *Filament group* 早已就载入 CSS 和字体写过很棒的文章。我写这篇文章为了记录我关于非阻塞加载资源的一些思考。

触发样式表异步加载的技巧是使用 `<link>` 标签并且给 `media` 属性设置无效值（我用 `media="none"`，但任何值都可以）。当一条媒体查询为 false 时，浏览器会加载样式表，但其中的资源则不会，直到需要用于渲染页面。

	<link rel="stylesheet" href="css.css" media="none">

一旦样式表加载完成，`media` 属性必须改设有效值以使其样式应用到页面中。`onload` 事件就是用来将 `media` 属性的值切换至 `all`的：

	<link rel="stylesheet" href="css.css" media="none" onload="if(media!='all')media='all'">

这种加载 CSS 的方法载入有用资源的速度会比一般方法迅速得多。关键的 CSS 仍能通过通常的阻塞载入路径（或者对于个别样式写成行内式），非关键 CSS 能渐进地下载并稍晚应用于浏览器分析渲染。

这个技巧用到 JavaScript，但对于禁用了 JS 的浏览器也能通过 `<noscript>` 标签载入 `<link>`。

	<link rel="stylesheet" href="css.css" media="none" onload="if(media!='all')media='all'">
	<noscript><link rel="stylesheet" href="css.css"></noscript>

下面是这个技巧的副作用。一旦非阻塞样式表加载完成，文档将会重新渲染以响应相关的新规则。插入新样式到页面中会触发内容重排，但仅会是首次页面加载没有缓存时才会出现的状况。你需要自行判断何时控制重排，何时加快速度。

## 使用非阻塞 CSS 加载字体 ##
字体是页面首次渲染时的大问题，它是阻塞型资源，下载时会使内容无法呈现。使用如上所示的非阻塞链接，能在背后下载包含字体数据的样式表，而不阻塞页面渲染。

	<link rel="stylesheet" href="main.css">
	<link rel="stylesheet" href="font.css" media="none" onload="if(media!='all')media='all'">

`font.css` 包含了 Merriweather 字体的 base64 编码 WOFF 版本。

	@font-face {
		font-family: Merriweather;
		font-style: normal;
		font-weight: 400;
		src: local('Merriweather'), url('data:application/x-font-woff;charset=utf-8;base64,...')
	}

`main.css` 包含了网站的所有样式，这里是关于字体的声明。

	body {
		font-family: Merriweather, "Lucida Grande", ...;
	}

当字体正在下载时，第一个符合的回调字体（本例中是 **Lucinda Granda**）会被用于渲染页面内容。一旦字体样式表加载好后，将转而使用 **Merriweather**。我尝试使用与所需字体样式相近的回调字体，从而使不可避免的重排影响最小。

我在 Chrome 中用我的谷歌分析工具（[https://keithclark.github.io/gadebugger/](https://keithclark.github.io/gadebugger/)）模拟 3G 环境测试阻塞和非阻塞两种方法。本地测试结果如下图所示，可以发现使用非阻塞方法时 `DOMContentLoaded` 事件早触发 450ms，而且静态资源也更早开始加载。

![图1](http://i2.tietuku.com/6d0d1fe437e1e35d.png)

模拟 3G 网络的示意图，上面是阻塞方法，下面是非阻塞方法。

在服务器上测试，使用 webpagetest（[http://www.webpagetest.org/](http://www.webpagetest.org/)）模拟 3G 环境的示意图如下：

![图2](http://i2.tietuku.com/682adfdff99d2e51.png)

上面是阻塞方法，下面是非阻塞方法。

两种方法都要用 2.8 秒来完全呈现页面，但是非阻塞方法中早一秒开始渲染。对于主样式表也进行了测试。

![图3](http://i2.tietuku.com/2c722ef79abec9a1.png)

上面是阻塞方法，下面是非阻塞方法。

对于字体，这个技巧使用良好，但我建议留意新的 CSS 字体加载模块（[http://dev.w3.org/csswg/css-font-loading/](http://dev.w3.org/csswg/css-font-loading/)），有非常好的字体加载控制。

## 总结 ##
加载字体只是非阻塞技巧的其中一个例子，这个技巧也能用于其他地方，比如从核心 CSS 中分离出针对 JavaScript 加强的样式。

我已经开始试验将 CSS 拆分成结构类（核心布局）和表现类（其他样式），让页面基础布局先渲染，而视觉样式稍后。

## 2015-04-01 更新 ##
- 本方法不适用低于 4.4 版本的安卓系统，`onload` 回调并没有触发。
- 有些浏览器会像阻塞方法一样加载 `media="none"` 的样式表，这意味着与平时一样渲染页面。