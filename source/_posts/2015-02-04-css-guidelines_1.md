title: （译）CSS Guidelines (1)
date: 2015-02-04 20:12:03
tags: css
---
## 一、介绍 ##
CSS 不是一门优美的语言。尽管入门容易，但在任何合理的规模下都会产生问题。对于 CSS 的工作方式我们无法改变什么，但我们可以改变编写和构建 CSS 的方法。

当进行大规模、长周期的项目时，许多不同专长和能力的开发者一起工作，我们需要以统一的方式协作以达成以下目标：

- 保持样式表可维护；
- 保持代码透明、稳健、合理；
- 保持样式表的可拓展性。

为了达成以上目标我们需要应用许多技巧，本篇《CSS 指导》正是这些建议和方法的集合文档。

### 1.1 样式指导的重要性 ###
编码样式指导（注意，不是视觉样式指导）是对以下队伍有用的工具：

- 在一个合理的时间长度内建设及维护产品；
- 有不同能力和专长的开发者；
- 在任何给定的时间内都有一定数量的开发者在开发产品；
- 定期有新员工加入；
- 有一定数量可供开发者贡献和使用的代码库。

同时样式指导是对产品队伍——基于长生命周期且不断进化的项目的庞大代码库，长期有大量开发者为此贡献代码——所有开发者都应努力在代码中实现高度的标准化。

一个良好的样式指导将：

- 为代码库设定代码质量的标准；
- 促进代码库的一致性；
- 在整个代码库中给开发者以熟悉感；
- 提高生产力。

### 1.2 声明 ###
《CSS 指导》是一篇样式指导，而不是唯一的样式标准。包含了我会向客户和团队推荐的方法论、技巧以及提示，但你自己的品味和环境可能大不相同。效果可能不一样。

这些指导是可选的，但它们都在多年大大小小的项目中被屡次尝试、测试、施压、改善、废弃、重写以及重现。