---
layout: default
title:  "浏览器是如何工作的!"
date:   2017-11-16 16:17:56 +0800
categories: jekyll update
---
## [{{page.title}}](http://taligarsiel.com/Projects/howbrowserswork1.htm)  

* [1. 介绍](#1)  
    * [1.1 本文涉及到的浏览器](#1.1)  
    * [1.2 浏览器的主要功能](#1.2)  
    * [1.3 浏览器的主要结构](#1.3)  
    * [1.4 组件之前的通信](#1.4)  

* [2. 渲染引擎](#2)
    * [2.1 渲染引擎](#2.1)
    * [2.2 主要的流程](#2.2)
    * [2.3 流程示例](#2.3)
    * [2.4 解析以及DOM树的结构](#2.4)
    * [2.5 渲染树(Render tree)的结构](#2.5)
    * [2.6 布局](#2.6)
    * [2.7 绘制（Painting）](#2.7)
    * [2.8 动态改变](#2.8)
    * [2.9 渲染引擎的线程](#2.9)
    * [2.10 css2 虚拟模型](#2.10)
    * [2.11 资源](#2.11)  


<span id="1"><b>1. 介绍</b></span>  

浏览器可谓是使用最广泛的软件. 这篇文章我将要解释浏览器在底层是如何工作的. 我们将会了解当你在浏览器地址栏里输入'google.com'直到页面呈现出来这一过程都发生了什么。

<span id="1.1"><b>1.1 本文涉及到的浏览器</b></span>  

目前市面上主要有5款浏览器： Internet Explorer, Firefox, Safari, Chrome 以及 Opera。
我将使用开源的浏览器中进行举例，包含Firefox, Chrome 以及 部分开源的Fafari。
根据[W3C browser statistics](http://www.w3schools.com/browsers/browsers_stats.asp), 当前时间是2009年10月，使用Firefox, Safari 以及 Chrome 的比例占据将近60%。
所以目前开源浏览器占据的浏览器市场很大的份额。

<span id="1.2"><b>1.2 浏览器的主要功能</b></span>  

浏览器的主要功能是把你从服务器请求到的网络资源呈现在浏览器窗口上。资源通常包含了HTML，PDF， 图片等等。资源通常是由用户指定的URI(Unifor resource Identifier 统一资源定位符)来定位的。稍后的章节会介绍。

浏览器解释和呈现HTML文件的方式是通过HTML和CSS规范来实现的。 这些规范是由W3C组织进行维护的，该组织是互联网的标准制定者。长久以来各个浏览器厂商只实现了一部分规范，并且开发自己的扩展程序。这导致了在不同的浏览器当中很严重的兼容性问题。到目前为止，大部分的浏览器都大多实现了规范。
不同的浏览器UI有很多相同的部分:  
* 输入URI的地址栏  
* 前进和后退按钮  
* 书签操作  
* 操作当前加载文档的刷新和停止按钮  
* 返回主页的主页按钮  

但是比较奇怪的是， 浏览器的UI并没有一个通用的规范，它只是不同的浏览器厂商从长期的使用习惯中积累的经验。HTML5规范并没有规定浏览器的UI必须包含哪些元素，只是列出了一些通用的元素。地址栏、状态栏、工具栏以及各个浏览器指定的特定，例如Firefox的下载管理。更多参见用户界面章节。

<span id="1.3"><b>1.3 浏览器的主要结构</b></span> 

浏览器的主要组成部分:  
    1. 用户界面(The user interface) - 包含地址栏、前进/后退按钮、书签等等。除了主要的窗口之外你所看到的就是请求的页面。  
    2. 浏览器引擎 - 查询和操作渲染引擎的入口。  
    3. 渲染引擎 - 负责呈现请求内容。例如请求内容是HTML, 渲染引擎负责解析HTML以及CSS，并且渲染解析后的内容到屏幕上。  
    4. 网络链接 - 处理形如HTTP的网络请求。它有针对不同平台的实现接口。  
    5. 用户界面的后台处理程序(UI Backend) - 用于绘制类似于 bombo 盒子的小部件以及一些窗口。它抛出了各个平台通用的接口。它的底层是使用了操作系统的用户界面方法。  
    6. Javascript 解释器。 用于解析和执行Javascrip代码。  
    7. 数据存储。 这是一个持久层。浏览器需要在硬盘上保存各种各样的数据，比如coocies。HTNL5规范定义了'web database',针对浏览器的完整的数据库（尽管比较轻量）  
    ![浏览器主要组成部分](/images/layers.png)  
        Figure 1: Browser main components.  

值得注意的是，Chrome不像其他的浏览器， 它给每一个tab分配一个渲染引擎的实例，每一个tab都是一个独立的进程。

<span id="1.4"><b>1.4 组件间的通信</b></span>  

Firefox 和 Chrome 都独自开发了一套特别的通信机制。

<span id="2"><b>渲染引擎</b></span>  

渲染引擎的职责就是进行渲染， 也就是负责把请求到的内容呈现在浏览器屏幕上。

在默认情况下，渲染引擎能够展示HTML，XML以及image文档。也能通过插件来展示其他类型的文档。例如通过PDF视图插件可以展示PDF。我们将会在特定的章节讨论插件和扩展程序。本章节主要着重于主要的情况-如何展示由css格式化的HTML和images。

<span id="2.1"><b>渲染引擎</b></span>  

我们参考的Firefox,Chrome,Safari浏览器都是基于两个渲染引擎建立的。 Firefox 使用 Gecko, 一个Mozilla自己开发的引擎。Safari 和 Chrome 都是使用的Webkit引擎。Webkit 引擎最开始是用于linux平台的开源引擎。后续被修改用于支持Apple的Mac以及 Windows系统。 详情移步[http://webkit.org/](http://webkit.org/)

<span id="2.2"><b>主要渲染流程</b></span>  

渲染引擎将会从网络层请求到内容开始进行工作。这通常的大小在8k以内。

在这之后，以下就是渲染引擎基本的流程：

![渲染引擎基本流程](/images/flow.png)
    Figure 2: Render engine basic flow.  

解析HTML， 生成DOM tree -> 渲染render tree结构 -> 组织render tree 的布局  -> 在窗口绘制 render tree

渲染引擎会解析HTML文档，把HTML文档解析为“内容树(content tree)”， 并把HTML标签转换为树中的[DOM](http://taligarsiel.com/Projects/howbrowserswork1.htm#DOM)节点。渲染引擎还要解析样式文件，包含外链样式文件以及内联样式元素。样式信息和HTML当中可视化的指令将会用于创建另外一个树 - 渲染树 （[render tree](http://taligarsiel.com/Projects/howbrowserswork1.htm#Render_tree_construction)）。

渲染树包含了具有颜色以及尺寸等可视化属性的矩形盒子集合。这些矩形盒子都是按照在屏幕上的显示顺序排序的。

在构造晚渲染树之后，将会经过“[layout](http://taligarsiel.com/Projects/howbrowserswork1.htm#layout)”过程。意思就是给每一个节点设置在屏幕上显示的确切坐标位置。下一个阶段是绘制([painting](http://taligarsiel.com/Projects/painting)) - 渲染树将会通过UI的后台处理层，每一个节点都将会被绘制。

了解渲染的过程是一个循序渐进的过程很重要。为了达到更好的用户体验，渲染引擎将会尽可能快的把内容展示在屏幕上。它并不会等到所有的HTML都解析完之后才去构建和布局渲染树。当请求到一部分内容的时候，引擎将会解析和渲染这一部分内容，同时程序也将继续解析从网络中请求到的余下的内容。

<span id="2.3"><b>渲染流程示例</b></span>  

![webkit flow](/images/webkitflow.png)
    Figure 3: Webkit main flow

![Gecko flow](/images/geckoflow.png)
    Figure 4: Mozilla's Gecko rendering engine main flow

从图3 和图4 中可以看到尽管Webkit 和 Gecko 使用了稍微不同的术语，但是流程是基本相同的。
Gecko 把格式化的元素形象的称为：Frame tree（结构树）。每一个元素都是一个框架。Webkit 使用术语：Render tree， 它由Render Object 组成。Webkit把设置元素的位置称为layout，而 Gecko称为Reflow。 Webkit 把连接DOM节点和视觉信息生成渲染树称为Attachment。另外一个较小的非语义上的差别是Gecko在HTML与DOM树之间多了额外的一层。叫做content sink, 它是创建DOM元素的工厂。我们将会逐个了解流程的每一部分。


<b>通常的解析</b>

... 未完待续
