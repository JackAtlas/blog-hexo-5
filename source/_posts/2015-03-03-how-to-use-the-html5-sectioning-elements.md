title: （译）如何使用HTML5章节标签
date: 2015-03-03 20:12:12
tags: html
---
本文由[小智](http://jackatlas.com/)根据 [Matt West](http://blog.teamtreehouse.com/author/mattwest) 的 《[How to Use The HTML5 Sectioning Elements](http://blog.teamtreehouse.com/use-html5-sectioning-elements)》所译。译文带有我自己的理解和思想，如需转载请注明相关信息：

> 原文地址：[http://blog.teamtreehouse.com/use-html5-sectioning-elements](http://blog.teamtreehouse.com/use-html5-sectioning-elements)
——作者：[Matt West](http://blog.teamtreehouse.com/author/mattwest)
——译者：[小智](http://jackatlas.com/)

HTML5 已经出现了许多可以用来标记网页的章节标签。使用这些元素能让你的页面更加“语义化”，让计算机程序能更好地读懂你的内容。

你可以从这篇文章中学到如何在你的网站中运用这些章节元素。我会解释为何用某个元素而不是另外一些，或者什么时候应该坚持使用经典的 `<div>`。

走起！

## main 元素 ##
`<main>` 元素应当包含网页中的主要内容。所有的这些内容都应该是独立页面中特有的，也不应出现在网站的其他地方。任何在复数页面中出现的内容（如logo、搜索框、底部链接等）都不应放置在 `<main>` 元素中。

下面的例子用 `<main>` 元素来代表页面的主要内容。

	<body>
		<header>
			<div id="logo">Rocking Stone</div>
			<nav>...</nav>
		</header>
		<main role="main">
			<h1>Guitars</h1>
			<p>The greatest guitars ever built.</p>

			<article>
				<h2>Gibson SG</h2>
				<p>...</p>
			</article>

			<article>
				<h2>Fender Telecaster</h2>
				<p>...</p>
			</article>
		</main>
	</body>

**注意**：这里我们使用 ARIA `role="main"` 属性，因为这能告诉不支持 `<main>` 元素的程序（比如某些屏幕阅读软件），这个标签是什么意义。

你只应在一个页面中使用**一个** `<main>` 元素，而且不能用 `<article>`、`<aside>`、`<header>`、`<footer>` 或者 `<nav>` 元素替代。

## article 元素 ##
`<article>` 元素应该包括一段即使脱离页面语境也能独立的内容，比如新闻、博客文章或者用户评论。

	<article>
		<header>
			<h1>Blog Post Title</h1>
			<p>Posted 13th February 2014</p>
		</header>
		<p>
		...
		</p>
	</article>

你可以在一个 `<article>` 元素中嵌套另一个。这就暗示着嵌套的 `<article>` 元素与外层的有关联。

	<article>
		<header>
			<h1>Blog Post Title</h1>
			<p>Posted 13th February 2014</p>
		</header>
		<p>...</p>
		<p>...</p>
		<p>...</p>
		<section>
			<h2>Comments</h2>
			<article>
				<footer>
					<p>Posted by: Joe Balochio</p>
				</footer>
				<p>This was a great article</p>
			</article>
			<article>
				<footer>
					<p>Posted by: Casey Brock</p>
				</footer>
				<p>How do you think this applies to the plan for world domination?</p>
			</article>
		</section>
	</article>

本例中我们用 `<article>` 元素来标记博客文章以及每条评论。这种嵌套结构表示这些评论与这篇博客文章的主题有关。

## section 元素 ##
`<section>` 用来表示一组相关联的内容。这很像 `<article>` 元素，主要的区别是 `<section>` 元素不必在脱离页面语境时也要有意义。

建议用标题元素（`<h1>` - `<h6>`）来定义 section 的主题。

就用这篇文章为例，用 `<section>` 元素来代表文章中每个独立的部分。

	<article>
		<h1>How to use HTML5 Sectioning Elements</h1>
		<p>...</p>

		<section>
			<h2>The <main> Element</h2>
			<p>...</p>
		</section>
		<section>
			<h2>The <article> Element</h2>
			<p>...</p>
		</section>
		<section>
			<h2>The <section> Element</h2>
			<p>...</p>
		</section>
		...
	</article>

这里我们用 `<article>` 元素来总括整篇文章，然后用几个 `<section>` 元素来包裹文章中讨论的每个小主题。

如果你仅仅是想包裹住一些内容来赋予样式，你应该用 `<div>` 元素而不是 `<section>`。

## nav 元素 ##
`<nav>` 元素用来标记一系列导向外部页面或者当前页面某个锚点的链接，也会用作站点的主导航、文章的目录或者博客列表。

	<nav>
		<ul>
			<li><a href="#chapter-one">Chapter One</a>
			<li><a href="#chapter-two">Chapter Two</a>
			<li><a href="#chapter-three">Chapter Three</a>
		</ul>
	</nav>

用列表来标记链接会让你的导航更易用，但这不是必须的。

## aside 元素 ##
`<aside>` 元素表示与主内容相关却相对独立的内容。包括侧边栏（就像书中那些）、`<nav>` 元素组、插图和醒目的引文。

	<article>
		<header>
			<h1>Google Buys Nest</h1>
			<p>Posted at 11:34am 13th January 2014</p>
		</header>
		<p>...</p>
		<p>...</p>

		<aside>
			<h1>Google (GOOG)</h1>
			<p>Google was founded in 1998 by Larry Page and Sergey Brin. The company...</p>
		</aside>
	</article>

我们在一篇新闻中使用 `<aside>` 元素来标记 Google 的信息。`<aside>` 中的公司信息可认为是对读者有用的，但不是与新闻直接相关。

## header 元素 ##
`<header>` 元素用于呈现文章或网页的介绍性内容。通常包含标题元素以及与内容相关的元数据比如新闻的发表日期。也可以包含长文档的目录（使用 `<nav>` 元素）。

`<header>` 元素会与最近的章节元素相关联，通常是页面结构中的父元素。

	<header>
		<h1>Google buys Nest</h1>
		<p>Posted at 11:34am 13th January 2014</p>
	</header>

本例是 `<header>` 元素带有新闻的标题和发表日期。

## footer 元素 ##
`<footer>` 元素用于呈现作者、版权、相关链接等信息。

	<footer>
		Copyright Matt West 2014
	</footer>

As with `<header>`, the `<footer>` element is associated with the nearest sectioning element.

如同 `<header>`，`<footer>` 元素会与最近的章节元素相关联。

## address 元素 ##
`<address>` 元素是其中一个最被误解的 HTML 元素。这个元素不是用来标记邮件地址，而是用来呈现文章或网页的联系信息。可以是通向作者网站的链接或者他们的电子邮件地址。

	<address>
		Contact <a href="mailto:matt@example.com">Matt West</a>
	</address>

This element is often used within the `<footer>` for an `<article>`.

这个元素经常用在 `<article>` 的 `<footer>` 中。

	<article>
		<header>
			<h1>Google buys Nest</h1>
			<p>Posted at 11:34am 13th January 2014</p>
		</header>
		<p>...</p>
		<p>...</p>
		<footer>
			<address>
				By <a href="mailto:matt@example.com">Matt West</a>
			</address>
			<p>Copyright Matt West 2014</p>
		</footer>
	</article>

## 关于章节元素的一些想法 ##
在文中你能学到在标记网页时如何使用 HTML5 章节元素。使用这些元素有很多好处。最大的好处就是让你的网页区域划分更加语义化，让计算机程序识别关键元素（比如主内容和页面导航）。这些信息对屏幕阅读器之类的应用非常有用。

**注意**：不是所有的屏幕阅读器都支持这些语义化的元素。你或许想继续使用 ARIA 以防万一（回复“aria”或者“20141010”可查看小智以前翻译的一篇关于“ARIA”的文章）。

使用这些章节元素也让开发者们更多地思考他们网页的结构。针对一段内容，对元素的选择不总是明显的，但让人更多地思考内容的意义。这是网络标准不仅提升标记语言质量，还整体上提升网页质量的例子。