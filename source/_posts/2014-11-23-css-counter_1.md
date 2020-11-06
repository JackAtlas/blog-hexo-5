title: CSS 计数器（上）
date: 2014-11-23 15:32:08
tags: css
---

## 摘要 ##
CSS 计数器，古老而又冷门的技术。

## 简介 ##
CSS 计数器不是什么新鲜玩意，早在小智上大学之前就已经出现了，只是比较冷门，开发中不常用到，但是随着 CSS 3 的支持日益增强，结合 CSS 3 的一些新特性，或许 CSS 计数器会迸发出不一样的光芒。

## 兼容性 ##
CSS 计数器只能跟 `content` 属性一起使用，而 `content` 属性只用于 `:before` / `:after` 伪元素上，因此IE6、7不支持 CSS 计数器，而其他浏览器支持良好，见下图：

![caniuse](http://i3.tietuku.com/f828b2fd35db2560.jpg)

## 技术要点 ##
CSS 计数器的关键是两个属性一个方法。

1. `counter-reset`
2. `counter-increment`
3. `counter()` / `counters()`

`counter-reset` 定义计数器的名字，文档流中需要出现在 `counter-increment` 之前，建议 `counter-reset` 所在元素为另外两个所在元素的父（祖）元素，原因见 `counters()` 部分。

`counter-increment` 使计数器累加。

`counter()` / `counters()`，输出计数器当前值。

### 计数规则 ###
计数器当前值是指，`counter-reset` 和 `counter()` 之间，每出现一个 `counter-increment` 累加一次，`counter()` 后面的与此 `counter()` 无关。请看下面的例子。

	/* css */
	.container {counter-reset: sum;} /* 定义计数器 */
	.item {counter-increment: sum;} /* 累加规则 */
	.monitor:after {content: counter(sum);} /* 显示计数器当前值 */

&nbsp;

	<!-- html -->
	<div class="container"> <!-- 当前值：0 -->
		<div class="item"></div> <!-- 当前值：1 -->
		<div class="item"></div> <!-- 当前值：2 -->
		<div class="monitor"></div> <!-- 输出：2 -->
		<div class="item"></div> <!-- 当前值：3 -->
		<div class="item"></div> <!-- 当前值：4 -->
		<div class="item"></div> <!-- 当前值：5 -->
		<div class="monitor"></div> <!-- 输出：5 -->
		<div class="item"></div> <!-- 与前面的 counter() 无关 -->
	</div>

### counter-reset ###
`counter-reset` 的格式如下：

	//css
	counter-reset: name1 [start1 name2 start2 ... ];
	// 中括号里的代码可选（下同）

- **name** 定义的计数器名；
- **start** 计数器的起始值，默认为0；
- 可同时定义多个计数器，中间用空格隔开。

### counter-increment ###
`counter-increment` 的格式如下：

	// css
	counter-increment: name1 [step1 name2 step2];

- **name** 对指定的计数器进行累加；
- **step** 每次累加的数值，可为负数；
- 可同时对多个计数器进行累加，中间用空格隔开。

### counter() / counters() ###
#### counter() ####
`counter()` 的格式如下：

	// css
	content: counter(name1[, style1]) [counter(name2, style2)]

- **name** 显示指定计数器的当前值；
- **style** 值的显示样式，与 `list-style-type` 的取值一样，见下文；
- 可同时显示多个计数器的当前值，中间用空格隔开。

> **list-style-type**：disc | circle | square | decimal | lower-roman | upper-roman | lower-alpha | upper-alpha | none | armenian | cjk-ideographic | georgian | lower-greek | hebrew | hiragana | hiragana-iroha | katakana | katakana-iroha | lower-latin | upper-latin

`counters()` 多了一个“s”，但是效果却大相径庭，`counters()`是表示**嵌套计数**，比如“1.1”，格式如下：

	// css
	content: counters(name1, string1[, style1]) [counters(name2, string2, style2)]

- **name** 和 **style** 与 `counter()` 中相同，不再重复；
- **string** 子序号的连接**字符串**，如 “1.1” 的 `string` 就是“.”，必须参数；
- 可同时显示多个计数器的嵌套计数，中间用空格隔开；
- 对应的 html 结构为 `counter-reset`层层嵌套。

		// css
		.reset { padding-left: 20px; counter-reset: sons;}
		.counter:before { content: counters(sons, '-') '. '; counter-increment: sons;}

		// html
		<div class="reset">
		    <div class="counter">大爷爷
		        <div class="reset">
		            <div class="counter">大伯</div>
		            <div class="counter">爸爸
		                <div class="reset">
		                    <div class="counter">大儿子</div>
		                    <div class="counter">二儿子</div>
		                    <div class="counter">小儿子</div>
		                </div>
		            </div>
		            <div class="counter">三叔</div>
		        </div>
		    </div>
		    <div class="counter">二爷爷</div>
		    <div class="counter">三爷爷
		        <div class="reset">
		            <div class="counter">什么叔</div>
		        </div>
		    </div>
	</div>

![嵌套计数](http://i3.tietuku.com/7e1bb002dbb33f5b.jpg)

**特别注意**

1. `counters()` 所在元素一定要嵌套在 `counter-reset` 所在元素内，否则会出现计数嵌套错误；
2. 多个计数器时，`counter-reset` 不要定义在同一个节点上，否则会因为 css 的特性被覆盖而保留一个。

**小细节**

上面三个关键点，有的用空格隔开，有的用逗号隔开，可能会混淆。

- 对于 `counter-reset` 和 `counter-increment`，它们是 css 属性，其值用空格隔开；
- 对于 `counter()`，它是 `content` 属性的值，因此 `counter()` 与 `counter()` 之间用空格隔开；
- `counter()` 本身是方法，其参数写在括号内，用逗号隔开；
- `counters()` 同 `counter()`。

《CSS 计数器（上）》到此为止，下一篇谈谈 CSS 计数器的用途。