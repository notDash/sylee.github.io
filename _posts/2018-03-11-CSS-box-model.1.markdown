---
layout: default
title:  "CSS 盒模型!"
date:   2018-03-11
---
# CSS 盒模型

<!-- **[集中式工作流](http://blog.jobbole.com/76847/)** -->

---------------------------------------------------

好久没有更新东西，正好在看【精通CSS高级Web标准解决方案(第二版)】一书，回顾盒模型的知识。仅以此记录。

盒模型，粗略的来讲，就是元素在页面上的显示形式：被看做一个矩形框。这个矩形框包含：元素的**内容**、元素的**内边距**、元素的**边框**和元素的**外边距**。

## outline 属性

## 标准盒模型

## 非标准盒模型

IE早期版本，包括IE6，在**混杂**模式中使用自己的非标准盒模型。width = 内容 + 内边距 + 边框。
但是IE的各个版本之间没有一个统一的规范：IE5.x 会认为padding的宽度包含在content的宽度里。

## CSS3 box-sizing

## 外边距叠加

只有普通文档流中块级框的垂直外边距才会发生叠加。行内框、浮动框或者绝对定位框之间的外边距不会叠加。

- 两个元素垂直排列外边距叠加
- 一个元素包含另一个元素，内部元素没有border以及padding时候的叠加
- 自身外边距叠加


