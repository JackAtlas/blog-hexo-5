title: （译）CSS Guidelines (0)
date: 2015-02-01 15:32:08
tags: css
---
本文由[小智](http://jackatlas.com/)根据 [Harry Roberts](http://csswizardry.com/work/) 的 《[CSS Guidelines](http://cssguidelin.es/)》所译。译文带有我自己的理解和思想，如需转载请注明相关信息：

> 原文地址：[http://cssguidelin.es/](http://cssguidelin.es/)  
——作者：[Harry Roberts](http://csswizardry.com/work/)  
——译者：[小智](http://jackatlas.com/)

## 译者的话 ##
最好还是阅读原文，因为译文毕竟经过译者的再加工，受限于译者的英语水平和国语水平，或许原作者的意思不能完全理解，理解的部分书写出来也可能辞不达意。

**重要说明**

1. 原文中 `rule` 指作者行文中的一些条目，而 `ruleset` 指 CSS 规则，文中暂时都翻译为“规则”，可能会造成一些表达上的误会，如果想到更合适的词语会替换；
2. 欢迎各位指点。

## 前言 ##
> 编写稳健、可管理、可拓展 CSS 的高级指导。

## 关于作者 ##
[Harry Roberts](http://csswizardry.com/work/)

## 支持捐助 ##
请到原文中查找。

## 目录 ##
1. 介绍
  1. 样式指导的重要性
  2. 声明
2. 语法及格式
  1. 代码分割
  2. 目录
  3. 80个字宽
  4. 标题
  5. 规则的结构
  6. 多行 CSS
  7. 缩进
    1. Sass 缩进
    2. 对齐
  8. 有意义的空行
  9. HTML
3. 注释
  1. 高级
    1. 对象扩展指针
  2. 低级
  3. Proprocessor Comments
  4. Removing Comments
4. 命名规则
  1. Hyphen Delimited
  2. BEM-like Naming
    1. Starting Context
    2. More Layers
    3. Modifying Elements
  3. HTML 命名规则
  4. JavaScript 钩子
    1. `data-*` 属性
  5. Taking It Further
5. CSS 选择器
  1. Selector Intent
  2. 重用性
  3. Location Independence
  4. Portability
    1. Quasi-Qualified Selectors
  5. 命名
    1. UI 组件命名
  6. Selector Performance
    1. The Key Selector
  7. General Rules
6. 特殊性
  1. ID 选择器
  2. Nesting
  3. `!important`
  4. Hacking Specificity
7. 工程化原则
  1. High-level Overview
  2. Object-orientation
  3. The Single Responsibility Principle
  4. The Open/Closed Principle
  5. DRY
  6. Composition over Inheritance
  7. The Separation of Concerns
    1. Misconceptions