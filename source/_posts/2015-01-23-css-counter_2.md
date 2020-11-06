title: CSS 计数器（下）
date: 2015-1-23 22:02:38
tags: css
---
## 摘要 ##
技术文，不想细看可直接跳至最后的总结部分。

承接上篇《CSS 计数器（上）》，回复“20141123”、“CSS计数器”、“CSS”可查看。

上篇我们了解了 CSS 计数器的三个关键点，本篇我们来了解 CSS 计数器的用途。

## 计数器 ##
既然名为“CSS 计数器”，那么最基本的应用就是计数器了。

### 计数器1 ###

	// css
	section.characters {counter-reset: characters;}
	section.characters input:checked {counter-increment: characters;}
	section.total1:after {content: counter(characters);}
	// 结构和样式代码就省略了，这里只写计数器相关代码，下同

效果：

1. 什么都没选择时：

	![demo-1-1.png](http://i3.tietuku.com/6e9ef5e11469380b.png)

2. 选择两个：

	![demo-1-2.png](http://i3.tietuku.com/e052e3580c837811.png)

3. 选择四个：

	![demo-1-3.png](http://i3.tietuku.com/7b9a8620eb7b32be.png)

### 计数器2 ###

	// css
	.container-2 {counter-reset: sections boxes;}
	
	.container-2 section {counter-increment: sections;}
	
	.container-2 section::before {content: 'Section ' counter(sections);}
	
	.container-2 .box {counter-increment: boxes;}
	
	.container-2 .box::before {content: counter(boxes, upper-roman);}

效果：

![demo-2.png](http://i3.tietuku.com/fc73b58a72f995e0.png)

## 求和 ##
改变每个 `counter-increment` 的值（默认1）可以实现求和的功能。

	section {counter-reset: sum;}
	
	.a:checked {counter-increment: sum 64;}	
	.b:checked {counter-increment: sum 16;}	
	.c:checked {counter-increment: sum -32;}	
	.d:checked {counter-increment: sum 128;}	
	.e:checked {counter-increment: sum 4;}	
	.f:checked {counter-increment: sum -8;}
	
	.sum::before {content: '=' counter(sum);}

效果：

1. 未选中：

	![demo-3-1.png](http://i3.tietuku.com/ac825cba317fcee1.png)

2. 16 = 16：

	![demo-3-2.png](http://i3.tietuku.com/41bae5c2158221c3.png)

3. 16 - 32 + 4 = -12：

	![demo-3-3.png](http://i3.tietuku.com/9b80e3f70d4d0bec.png)

## 总结 ##
CSS 计数器有时候确实会带来一些便利，但是为什么诞生这么多年来几乎无人问津？

业界中种种模式都尝试让开发中的各种角色职责更清晰，分工更合理高效，比如前后端分离，甚至前端之中就会按“MVC”、“MV\*”等原则分割，每一部分下面又再细分。而 CSS 计数器的工作方式则刚好相反，负责视图的 CSS 越权把本应由脚本语言做的控制也做了。

比如上面的求和 demo，把数据放到了 css 里面。

总而言之，CSS 计数器是一项有趣的技术，但使用场景有限且有时会造成开发职责混乱，比较鸡肋。

## 其他思考 ##
有时语言并非越强大越好，限制什么，留下哪些自由，代码应该如何组织，这些设计思想其实挺有意思。