title: （译）CSS Guidelines (2)
date: 2015-02-05 14:22:54
tags: css
---
## 语法及格式 ##
样式指导的一种最简单的形式是关于语法和格式的一系列规则。以标准方法书写 CSS 意味着对于团队的所有成员来说代码看起来总是很熟悉。

另外，整洁的代码让人感觉清爽。这是个更容易投入工作的环境，促进其他团队成员去维持他们发现的整洁代码标准。丑陋的代码则会造成糟糕的先例。

### 代码分割 ###
伴随着最近急速发展的预处理器，开发者通常将 CSS 分割成若干文件。

尽管不使用预处理器，将不关联的代码快分割到独立的文件中也不失为一个好方法，在构建这一步中会重新拼凑起来。

出于某些原因，如果你不希望代码分割，那么下一节内容可能需要做些调整来满足你的设置。

### 目录 ###
目录是需要经常维护管理的，但由此带来的好处大大超出其代价。目录需要一位勤奋的开发者去持续更新，但这很值得。一个最新的目录能让团队知道 CSS 项目里有什么、做什么、按什么顺序排列。

一个简单的目录会（按顺序）列出小节的名字以及简要描述，例如：

```css
	/**
	 * CONTENTS
	 *
	 * SETTINGS
	 * Global.................Globally-available variables and config.
	 *
	 * TOOLS
	 * Mixins.................Useful mixins.
	 *
	 * GENERIC
	 * Normalize.css..........A level playing field.
	 * Box-sizing.............Better default 'box-sizing'.
	 *
	 * BASE
	 * Headings...............H1-H6 styles.
	 *
	 * OBJECTS
	 * Wrappers...............Wrapping and constraining elements.
	 *
	 * COMPONENTS
	 * Page-head..............The main page header.
	 * Page-foot..............The main page footer.
	 * Buttons................Button elements.
	 *
	 * TRUMPS
	 * Text...................Text helpers.
	 */
```

每一项对应一小节及/或其内容。

当然，多数项目中这些小节会非常庞大，但我们可以看到这些小结给开发者们提供了一个纵观全局的概览（在主样式表），可以看到哪里写了什么，为什么这么写。

### 80字符宽度 ###
可以的话，将 CSS 文件的宽度限制在80个字符内，原因如下：

- 能并排打开多个文件；
- 在线（如 Github）或终端中查看 CSS；
- 这种长度的注释看起来更舒服。

```css
	/**
	 * I am a long-form comment. I describe, in detail, the CSS that follows. I am
	 * such a long comment that I easily break the 80 character limit, so I am
	 * broken across several lines.
	 */
```

有些不可避免的例外，比如 URL，或者渐变语法，这些都不必担心。

### 标题 ###
在 CSS 里每个主要部分之前都写一个标题：

```css
	/*------------------------------------*\
	  #SECTION-TITLE
	\*------------------------------------*/

	.selector {}
```

标题加上一个 `#` 前缀让我们搜索的时候更容易命中，单纯搜索标题可能会有很多结果。

在标题和代码（另一段注释、Sass 或 CSS）之间留一个空行。

如果每一小节代码在不同的文件中，标题应该出现在文件的最上面。如果一个文件中含有多个小节，则每个标题上面都应该有5个空行。这样当在大文件中快速下拉时能迅速分辨出不同的小节。

```css
	/*------------------------------------*\
	  #A-SECTION
	\*------------------------------------*/

	.selector {}





	/*------------------------------------*\
	  #ANOTHER-SECTION
	\*------------------------------------*/

	/**
	* Comment
	*/

	.another-selector {}
```

### 规则的结构 ###
在讨论怎样写我们的规则之前，先来熟悉一下相关的术语：

```css
	[selector] {
	  [property]: [value];
	  [<--declaration--->]
	}
```

比如：

```css
	.foo, .foo--bar,
	.baz {
	  display: block;
	  background-color: green;
	  color: red;
	}
```

这里我们可以看到：

- 相关的选择器在同一行，不相关的选择器再另一行；
- 花括号（{）之前有个空格；
- 属性和值在同一行；
- 冒号（:）之后有个空格；
- 每条声明独立一行；
- 花括号（{）与最后一个选择器在同一行；
- 第一条声明在花括号（{）的下一行；
- 花括号（}）独立一行；
- 每条声明有4个空格的缩进；
- 最后一条声明后面也有分号（;）。

这种格式似乎是比较通用的标准（除了缩进的空格数，很多开发者倾向于2个空格）。

因此，下面的代码是不正确的：

```css
	.foo, .foo--bar, .baz
	{
	display:block;
	background-color:green;
	color:red }
```

这里的问题有：

- tab 缩进而不是空格缩进；
- 无关的选择器在同一行；
- 花括号（{）独立一行；
- 花括号（}）没有独立一行；
- 最后一个分号（;）缺失；
- 冒号（:）后面没有空格。

### 多行 CSS ###
CSS 应该分成多行书写，特别是在某些特定的环境下。这样会有很多好处：

- 代码合并时冲突的概率降低，因为每一条功能独立一行；
- 更“真实”可靠的文件比较，因为每一行只有一个变化。

这条规则的特例显而易见，比如只有一条声明的相似规则：

```css
	.icon {
	  display: inline-block;
	  width:  16px;
	  height: 16px;
	  background-image: url(/img/sprite.svg);
	}

	.icon--home     { background-position:   0     0  ; }
	.icon--person   { background-position: -16px   0  ; }
	.icon--files    { background-position:   0   -16px; }
	.icon--settings { background-position: -16px -16px; }
```

这种规则比单行写法更好，因为：

- 依然服从“一个改变一行”的原则；
- 这几行代码有足够的相似度，因为阅读它们不像阅读其他代码那样仔细，更容易看到它们的选择器，这是我们更感兴趣的。

### 缩进 ###
就像突出独立声明一样，将关联的规则通过缩进来展现其相关性，例如：

```css
	.foo {}

		.foo__bar {}

			.foo__baz {}
```

这样做，开发者一看就能知道 `.foo__baz {}` 在 `.foo__bar {}` 里，而 `.foo__bar {}` 又在 `.foo {}` 里。

这种像 DOM 折叠结构的写法告诉开发者们这些类应当在哪里使用，而不必回头去看 HTML。

#### Sass缩进 ####
Sass 支持嵌套，如：

```css
	.foo {
		color: red;

		.bar {
			color: blue;
		}

	}
```

编译后的 CSS：

```css
	.foo { color: red; }
	.foo .bar { color: blue; }
```

书写 Sass 的缩进时，我们仍坚持4个空格，我们也会在每个嵌套的前后各加一个空行。

#### 对齐 ####
试着将声明内一些共有的、关联的字符串对齐，比如：

```css
	.foo {
		-webkit-border-radius: 3px;
		   -moz-border-radius: 3px;
		        border-radius: 3px;
	}

	.bar {
		position: absolute;
		top:    0;
		right:  0;
		bottom: 0;
		left:   0;
		margin-right: -10px;
		margin-left:  -10px;
		padding-right: 10px;
		padding-left:  10px;
	}
```

使用能支持多光标编辑的编辑器会更轻松，开发者们可以一次修改若干相同而对齐的代码行。

### 有意义的空行 ###
好比缩进，我们可以巧妙地利用规则间的空行来呈现许多信息，比如：

- 紧密关联的规则之间空1行；
- 不紧密关联的规则之间空2行；
- 小节之间空5行。

例如：

```css
	/*------------------------------------*\
	#FOO
	\*------------------------------------*/

	.foo {}

		.foo__bar {}


		.foo--baz {}





	/*------------------------------------*\
	#BAR
	\*------------------------------------*/

	.bar {}

		.bar__baz {}

		.bar__foo {}
```

千万不要在两条规则之间不留空，这是不正确的：

```css
	.foo {}
		.foo__bar {}
	.foo--baz {}
```

### HTML ###
基于 HTML 和 CSS 相互关联的天性，我若是不谈谈标记语言的语法和格式指导这说不过去。

将属性值用引号包裹，尽管没有引号也能工作。这能减少意外的可能性，也是大部分开发者惯用的格式。下面的写法能工作（也是有效的）：

```html
	<div class=box>
```

更倾向于这种：

```html
	<div class="box">
```

这里要求写引号，

当 `class` 属性里有多个值时，用2个空格隔开：

```html
	<div class="foo  bar">
```

当多个 `class` 之间有关联，用方括号（`[` 和 `]`）包裹，就像这样：

```html
	<div class="[ box  box--highlight ]  [ bio  bio--long ]">
```

这一条不是强烈推荐，并且我也不太肯定，但确实带来了不少好处。详见[《在标记语言中给相关的类分组》](http://csswizardry.com/2014/05/grouping-related-classes-in-your-markup/)。

如同我们的规则，在 HTML 中使用有意义的空行也是可能的。你可以用5个空行表示主题间的隔断，例如：

```html
	<header class="page-head">
	  ...
	</header>





	<main class="page-content">
	  ...
	</main>





	<footer class="page-foot">
	  ...
	</footer>
```

用1个空行将独立却稍有关联的片段隔开，例如：

```html
	<ul class="primary-nav">

		<li class="primary-nav__item">
			<a href="/" class="primary-nav__link">Home</a>
		</li>

		<li class="primary-nav__item  primary-nav__trigger">
			<a href="/about" class="primary-nav__link">About</a>

			<ul class="primary-nav__sub-nav">
				<li><a href="/about/products">Products</a></li>
				<li><a href="/about/company">Company</a></li>
			</ul>

		</li>

		<li class="primary-nav__item">
			<a href="/contact" class="primary-nav__link">Contact</a>
		</li>

	</ul>
```

这让开发者一眼就能看出 DOM 结构中的不同部分，同时也能让某些编辑器（如 Vim）去处理空行分界的代码区域。

### 深度阅读 ###

- [《在标记语言中给相关的类分组》](http://csswizardry.com/2014/05/grouping-related-classes-in-your-markup/)