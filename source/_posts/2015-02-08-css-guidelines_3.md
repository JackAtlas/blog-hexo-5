title: （译）CSS Guidelines (3)
date: 2015-02-08 14:22:54
tags: css
---
## 注释 ##
编写 CSS 之前的认知工作是非常巨大的。有很多事情要注意，不同项目中有很多细微的差别要记住，最糟糕的情况是大部分开发者发现他们“不是写这代码的人”。**在一定程度上**记忆你自己写的类名、规则、对象、辅助方法是可行的，但接受 CSS 的开发者则不然。

**CSS 需要更多的注释。**

CSS 是一种不会留下太多痕迹的声明式语言，但看 CSS 通常很难辨别：

- 一段 CSS 是否与其他地方的代码相关联；
- 一段 CSS 修改了其它地方会有什么影响；
- 有没有别的 CSS 会用到；
- 有何样式会被继承（有意无意的）；
- 有何样式会被忽略（有意无意的）；
- 某段 CSS 作者计划用在何处。

我们甚至无需重视一些会让接手项目的开发者更棘手的诡异地方（比如 `overflow` 会触发 BFC，或者某些 `transform` 属性会触发硬件加速）。

由于 CSS 没有将自己的特性表述清楚，因此需要大量的注释。

一条规则是，你应该在任何有不直接明显信息的代码处写注释。意思是说，没必要告诉别人 `color: red;` 是用来变红的，但如果你用 `overflow: hidden;` 来清除浮动（和闭合浮动相对），这可能是值得记入文档的。

### 高级 ###

我们用一块80字符宽的多行注释来为整个小节或组件写注释。

这是一个真实的例子，CSS Wizardry 的页头样式注释：

	/**
	 * The site’s main page-head can have two different states:
	 *
	 * 1) Regular page-head with no backgrounds or extra treatments; it just
	 *    contains the logo and nav.
	 * 2) A masthead that has a fluid-height (becoming fixed after a certain point)
	 *    which has a large background image, and some supporting text.
	 *
	 * The regular page-head is incredibly simple, but the masthead version has some
	 * slightly intermingled dependency with the wrapper that lives inside it.
	 */

这种级别的细节应是对规定、序列、条件、处理方案等进行代码描述的样板。

#### 对象扩展指针 （Objective-Extension Pointers） ####

当你要维护众多的模块，或者应用 OOCSS 的概念，你会发现相关联的 CSS 规则不总是在同一个文件或位置。例如，你会有一个按钮类的对象（纯粹提供结构样式），在皮肤组件部分中会有所扩展。我们用简单的对象扩展指针来记录这些跨文件的关系。在对象文件中：

	/**
	 * Extend `.btn {}` in _components.buttons.scss.
	 */

	.btn {}

在主题文件中：

	/**
	 * These rules extend `.btn {}` in _objects.buttons.scss.
	 */

	.btn--positive {}

	.btn--negative {}

这种简单的注释能极大地方便那些不知道项目间关系，或者想知道样式是如何、为何、从何继承的开发者。


### 低级（Low-level） ###

很多时候我们想对一条规则中的多条声明进行注释。我们用一种颠倒的脚注。下面是一段更复杂的关于生面说到的网站头的注释。

	/**
	 * Large site headers act more like mastheads. They have a faux-fluid-height
	 * which is controlled by the wrapping element inside it.
	 *
	 * 1. Mastheads will typically have dark backgrounds, so we need to make sure
	 *    the contrast is okay. This value is subject to change as the background
	 *    image changes.
	 * 2. We need to delegate a lot of the masthead’s layout to its wrapper element
	 *    rather than the masthead itself: it is to this wrapper that most things
	 *    are positioned.
	 * 3. The wrapper needs positioning context for us to lay our nav and masthead
	 *    text in.
	 * 4. Faux-fluid-height technique: simply create the illusion of fluid height by
	 *    creating space via a percentage padding, and then position everything over
	 *    the top of that. This percentage gives us a 16:9 ratio.
	 * 5. When the viewport is at 758px wide, our 16:9 ratio means that the masthead
	 *    is currently rendered at 480px high. Let’s…
	 * 6. …seamlessly snip off the fluid feature at this height, and…
	 * 7. …fix the height at 480px. This means that we should see no jumps in height
	 *    as the masthead moves from fluid to fixed. This actual value takes into
	 *    account the padding and the top border on the header itself.
	 */

	.page-head--masthead {
		margin-bottom: 0;
		background: url(/img/css/masthead.jpg) center center #2e2620;
		@include vendor(background-size, cover);
		color: $color-masthead; /* [1] */
		border-top-color: $color-masthead;
		border-bottom-width: 0;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1) inset;

		@include media-query(lap-and-up) {
			background-image: url(/img/css/masthead-medium.jpg);
		}

		@include media-query(desk) {
			background-image: url(/img/css/masthead-large.jpg);
		}

		> .wrapper { /* [2] */
			position: relative; /* [3] */
			padding-top: 56.25%; /* [4] */

			@media screen and (min-width: 758px) { /* [5] */
				padding-top: 0; /* [6] */
				height: $header-max-height - double($spacing-unit) - $header-border-width; /* [7] */
			}

		}

	}

这类注释允许我们将所有的文档写到一起，并且指向各自标注的地方。

### 预处理注释 ###

在大部分的预处理程序中，我们可以通过配置项使注释不会在编译时被省略掉。用这种注释来记录不需被省略的代码。如果有些代码要在编译时被省略则使用会被省略的注释。例如：

	// Dimensions of the @2x image sprite:
	$sprite-width:  920px;
	$sprite-height: 212px;

	/**
	 * 1. Default icon size is 16px.
	 * 2. Squash down the retina sprite to display at the correct size.
	 */
	.sprite {
		width:  16px; /* [1] */
		height: 16px; /* [1] */
		background-image: url(/img/sprites/main.png);
		background-size: ($sprite-width / 2 ) ($sprite-height / 2); /* [2] */
	}

我们用预处理注释来记录变量（这些代码不会被写进 CSS 文件），而 CSS 则使用 CSS 注释。这意味着当我们 debug 样式表的时候只有正确的相关联的信息。

### 删除注释 ###
产品环境中应该没有注释，发布前所有的 CSS 都经过压缩。