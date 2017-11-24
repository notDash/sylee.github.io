---
layout: default
title:  "浏览器是如何工作的!"
date:   2017-11-16 16:17:56 +0800
categories: jekyll update
---
# [{{page.title}}](http://taligarsiel.com/Projects/howbrowserswork1.htm)

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

## <span id="1"><b>1. 介绍</b></span>

浏览器可谓是使用最广泛的软件. 这篇文章我将要解释浏览器在底层是如何工作的. 我们将会了解当你在浏览器地址栏里输入'google.com'直到页面呈现出来这一过程都发生了什么。

### <span id="1.1"><b>1.1 本文涉及到的浏览器</b></span>

目前市面上主要有5款浏览器： Internet Explorer, Firefox, Safari, Chrome 以及 Opera。
我将使用开源的浏览器中进行举例，包含Firefox, Chrome 以及 部分开源的Fafari。
根据[W3C browser statistics](http://www.w3schools.com/browsers/browsers_stats.asp), 当前时间是2009年10月，使用Firefox, Safari 以及 Chrome 的比例占据将近60%。
所以目前开源浏览器占据的浏览器市场很大的份额。

### <span id="1.2"><b>1.2 浏览器的主要功能</b></span>

浏览器的主要功能是把你从服务器请求到的网络资源呈现在浏览器窗口上。资源通常包含了HTML，PDF， 图片等等。资源通常是由用户指定的URI(Unifor resource Identifier 统一资源定位符)来定位的。稍后的章节会介绍。

浏览器解释和呈现HTML文件的方式是通过HTML和CSS规范来实现的。 这些规范是由W3C组织进行维护的，该组织是互联网的标准制定者。长久以来各个浏览器厂商只实现了一部分规范，并且开发自己的扩展程序。这导致了在不同的浏览器当中很严重的兼容性问题。到目前为止，大部分的浏览器都大多实现了规范。
不同的浏览器UI有很多相同的部分:

* 输入URI的地址栏

* 前进和后退按钮

* 书签操作

* 操作当前加载文档的刷新和停止按钮

* 返回主页的主页按钮

但是比较奇怪的是， 浏览器的UI并没有一个通用的规范，它只是不同的浏览器厂商从长期的使用习惯中积累的经验。HTML5规范并没有规定浏览器的UI必须包含哪些元素，只是列出了一些通用的元素。地址栏、状态栏、工具栏以及各个浏览器指定的特定，例如Firefox的下载管理。更多参见用户界面章节。

### <span id="1.3"><b>1.3 浏览器的主要结构</b></span>

浏览器的主要组成部分:

1. 用户界面(The user interface) - 包含地址栏、前进/后退按钮、书签等等。除了主要的窗口之外你所看到的就是请求的页面。
1. 浏览器引擎 - 查询和操作渲染引擎的入口。
1. 渲染引擎 - 负责呈现请求内容。例如请求内容是HTML, 渲染引擎负责解析HTML以及CSS，并且渲染解析后的内容到屏幕上。
1. 网络链接 - 处理形如HTTP的网络请求。它有针对不同平台的实现接口。
1. 用户界面的后台处理程序(UI Backend) - 用于绘制类似于 bombo 盒子的小部件以及一些窗口。它抛出了各个平台通用的接口。它的底层是
    使用了操作系统的用户界面方法。
1. Javascript 解释器。 用于解析和执行Javascrip代码。
1. 数据存储。 这是一个持久层。浏览器需要在硬盘上保存各种各样的数据，比如coocies。HTNL5规范定义了'web database',针对浏览器的完整的数据库（尽管比较轻量）

    ![浏览器主要组成部分](/sylee.github.io/images/layers.png)

        Figure 1: Browser main components.

值得注意的是，Chrome不像其他的浏览器， 它给每一个tab分配一个渲染引擎的实例，每一个tab都是一个独立的进程。

### <span id="1.4"><b>1.4 组件间的通信</b></span>  

Firefox 和 Chrome 都独自开发了一套特别的通信机制。

## <span id="2"><b>2. 渲染引擎</b></span>  

渲染引擎的职责就是进行渲染， 也就是负责把请求到的内容呈现在浏览器屏幕上。

在默认情况下，渲染引擎能够展示HTML，XML以及image文档。也能通过插件来展示其他类型的文档。例如通过PDF视图插件可以展示PDF。我们将会在特定的章节讨论插件和扩展程序。本章节主要着重于主要的情况-如何展示由css格式化的HTML和images。

### <span id="2.1"><b>2.1 渲染引擎</b></span>  

我们参考的Firefox,Chrome,Safari浏览器都是基于两个渲染引擎建立的。 Firefox 使用 Gecko, 一个Mozilla自己开发的引擎。Safari 和 Chrome 都是使用的Webkit引擎。Webkit 引擎最开始是用于linux平台的开源引擎。后续被修改用于支持Apple的Mac以及 Windows系统。 详情移步[http://webkit.org/](http://webkit.org/)

### <span id="2.2"><b>2.2 主要渲染流程</b></span>  

渲染引擎将会从网络层请求到内容开始进行工作。这通常的大小在8k以内。

在这之后，以下就是渲染引擎基本的流程：

![渲染引擎基本流程](/sylee.github.io/images/flow.png)
    Figure 2: Render engine basic flow.  

解析HTML， 生成DOM tree -> 渲染render tree结构 -> 组织render tree 的布局  -> 在窗口绘制 render tree

渲染引擎会解析HTML文档，把HTML文档解析为“内容树(content tree)”， 并把HTML标签转换为树中的[DOM](http://taligarsiel.com/Projects/howbrowserswork1.htm#DOM)节点。渲染引擎还要解析样式文件，包含外链样式文件以及内联样式元素。样式信息和HTML当中可视化的指令将会用于创建另外一个树 - 渲染树 （[render tree](http://taligarsiel.com/Projects/howbrowserswork1.htm#Render_tree_construction)）。

渲染树包含了具有颜色以及尺寸等可视化属性的矩形盒子集合。这些矩形盒子都是按照在屏幕上的显示顺序排序的。

在构造晚渲染树之后，将会经过“[layout](http://taligarsiel.com/Projects/howbrowserswork1.htm#layout)”过程。意思就是给每一个节点设置在屏幕上显示的确切坐标位置。下一个阶段是绘制([painting](http://taligarsiel.com/Projects/painting)) - 渲染树将会通过UI的后台处理层，每一个节点都将会被绘制。

了解渲染的过程是一个循序渐进的过程很重要。为了达到更好的用户体验，渲染引擎将会尽可能快的把内容展示在屏幕上。它并不会等到所有的HTML都解析完之后才去构建和布局渲染树。当请求到一部分内容的时候，引擎将会解析和渲染这一部分内容，同时程序也将继续解析从网络中请求到的余下的内容。

### <span id="2.3"><b>2.3 渲染流程示例</b></span>  

![webkit flow](/sylee.github.io/images/webkitflow.png)
    Figure 3: Webkit main flow

![Gecko flow](/sylee.github.io/images/geckoflow.png)
    Figure 4: Mozilla's Gecko rendering engine main flow

从图3 和图4 中可以看到尽管Webkit 和 Gecko 使用了稍微不同的术语，但是流程是基本相同的。
Gecko 把格式化的元素形象的称为：Frame tree（结构树）。每一个元素都是一个框架。Webkit 使用术语：Render tree， 它由Render Object 组成。Webkit把设置元素的位置称为layout，而 Gecko称为Reflow。 Webkit 把连接DOM节点和视觉信息生成渲染树称为Attachment。另外一个较小的非语义上的差别是Gecko在HTML与DOM树之间多了额外的一层。叫做content sink, 它是创建DOM元素的工厂。我们将会逐个了解流程的每一部分。

<b>通常的解析</b>

既然解析在渲染引擎内是一个非常重要的过程，我们将会深入的了解它。
文档解析，亦即把它转换为一种代码可以理解和使用的结构。解析的结果通常是一个表示文档结构的<b>节点树</b>。它被称为解析树或者<b>语法树</b>。
例如：2 + 3 - 1 的表达式解析结果为

![运算表达式的树节点](/sylee.github.io/images/mathematical-expression.png)
    Figure 5: 运算表达式的树节点

<b>文法</b>

解析是基于创建文档语言所遵循的语法规则。每一个你能够解析的格式，都有一个由词法和语法规则组成的确切的文法。它被称为context free grammar(上下文无关的语法)。人类语言不是这样的语言，也就是说没法用常规的解析技术来进行解析。

<b>解析器 - 词法组合</b>

解析可以被分为两个步骤 - 词法分析 以及 语法分析。
词法分析是把输入的内容分解为很多符号的一个过程。这些符号是构成语言的词汇(构建语言有效的块集合)。在人类的语言中，它就是某种语言在字典中的所有单词所组成的。
语法分析就是语言语法规则的应用。
解析器的工作通常分为两个内容：<b>词法分析器</b>（有时称为 标记生成器）负责把输入分解为很多符号，<b>解析器</b>负责根据该语言的语法规则来分析文档结构，从而构建解析树。词法分析器知道如何区分和解释特殊的字符，例如空格和换行符。
![document to parse trees](/sylee.github.io/images/document-parsetree.png)
    Figure 6: from source document to parse trees

解析的过程是迭代式的。解析器通常会向词法分析器询问是否有新的符号，并且试图通过一条语法规则的来进行匹配。如果符合某条语法规则，该符号对应的节点将会被添加到解析树，紧接着解析器会询问另外一个符号就行解析。
如果没有规则匹配，解析器会在内部存储这个符号，并继续询问下一个符号直到某条规则匹配所有的内部存储的符号。如果没有找到对应的规则，解析器就回抛出一个异常。这意味着这个文档无效，并且包含语法错误。

<b>翻译</b>

通常解析树并不是最终的结果。解析结果通常被翻译-把文档翻译为另外一种格式。一个例子就是汇编。编译器会把源码编译为机器码，首先会把源码解析为解析树，然后再把解析树翻译为机器码文档。
![compilation flow](/sylee.github.io/images/compilation-flow.png)
    Figure 7: compilation flow

<b>解析实例</b>

在图5中，我们从一个数学表达式中创建了一个解析树。 让我们来定义一个简单的数学语言来了解解析过程。

词汇： 我们的语言包含整数，加法符号，减法符号

语法：  
    1. 构成语法的元素包含表达式，运算项，运算符。  
    2. 我们的语言能够包含任意数量的表达式。  
    3. 一个表达式定义为： 一个运算项 跟着一个 操作符，再跟着另外一个运算项。  
    4. 操作符为加号或者减号  
    5. 运算项为一个整数或者一个表达式。

分析下："2 + 3 - 1"。  
根据上面第5条规则，第一个匹配规则的子串是"2"。第二个匹配的的结果是"2 + 3"，它对应第二条规则。下一个匹配的结果已经到了该输入项的结尾。我们已经知道了形如?2 + 3?表示一个完整项，那么 "2 + 3 + 1"就是一个表达式。"2 + +" 是一个无效的输入，因为没有匹配任何规则。  

<b>正式的定义词汇和语法</b>  

词汇通常都通过常规的表达式来表示。  
例如我们将会像下面这样来定义我们的语言：  
INTER :0|[1-9][0-9]*  
PLUS : +  
MINUS : -  
如你所见，整数是通过常规的表达式来表达的。  
语法是遵循[BNF（Backus Naur form）](/sylee.github.io/2017/11/20/What-is-BNF(Backus-Naur-form).html)来定义的。我们的语言将会做如下的定义:   
expression := term operation tem  
operation := PLUS | MINUS  
term := INTEGER | expression  

我们之前说过，如果程序的语法是一个上下文无关的语法，就可以使用通常的解析器进行解析。上下文无关的语法，最直观的定义就是可以完全使用BNF来表示。可以参见[http://en.wikipedia.org/wiki/Context-free_grammar](http://en.wikipedia.org/wiki/Context-free_grammar)  

<b>解析器类型</b>  

解析器有两种类型： 自上而下 和 自下而上 的解析器。 自上而下的解析器是从语法层级比较高的地方着手进行匹配解析。自下而上的解析方式是从输入开始，逐级向上翻译为对应的语法规则，直到语法层级较高的规则为止。  

让我们结合实例来看看这两种解析方式：  

自上而下的解析将会从层级比较高的规则开始： 它将把 2 + 3 定义为一个表达式。然后再把 2 + 3 -1 定义为一个表达式。  
自下而上的解析将会扫描整个输入的字符串，如果有符合的规则， 则会根据规则替换匹配项，直到替换玩整个输入。匹配的表达式将会存储在解析器栈里。

| Stack                | Input         |
|:---------------------|:--------------|
|                      | 2 + 3 - 1     |
| term                 | + 3 - 1       |
| term operation       | 3 - 1         |
| expression           | - 1           |
| expression operation | 1             |
| expression           |               |

自下而上的解析方式又称之为移动减少解析器（shift reduce parser），因为输入是从左向右移动的，并且根据规则匹配主键减少。  

**自动生成解析器**

可以通过工具生成解析器，被称之为解析器生成器。 你只需要提供语言的词汇以及语法规则，它就能够生成一个可用的解析器。创建一个解析器徐傲对解析有深入的理解。不太容易手动创建一个解析器，所有解析器生成器会比较有用。  

Webkit 使用两个比较出名的解析器生成器： Flex 用于创建词法分析器， Bison用于创建解析器（你可以使用Lex 和 Yacc来运行）。Flex的输入是包含通常的表达式定义的一个文件。Bison 的输入是BNF格式的语法规则。  

**HTML 解析器**  

HTML解析器的职责是把HTML标记转换为解析树。  

**HTML 语法定义**  

HTML的词汇和语法在w3c创建的[规范](http://taligarsiel.com/Projects/howbrowserswork1.htm#w3c)里定义。  

**非上下文无关的程序语言**  

在解析一节的介绍里，我们知道程序语法可以通过BNF格式进行定义。  
但是不幸的是，所有常规的解析器都不适用于HTML。HTML不能够被轻易的定义为解析器需要的上下文无关的程序语法。  
有一个定义HTML的通用格式-DTD（Document Type Definition), 不过并不是上下文无关的语法。  
一眼看上去， HTML与XML非常的接近。 有很多的XML解析器。有一个HTML的XML变体-XXHTML。这二者有什么不用呢？  
不同之处在于HTML的目的在于非严谨的，它允许忽略你某些标签，并隐式的添加上，例如有时候允许忽略开始或者结束标签。不同于XML语法的严格和硬性要求，HTML整体上都是比较宽泛的。  
一方面这也是HTML如此浏览的一个原因，允许你犯错，让web开发更容易。另一方面，它导致很难定义一个语法格式。总结起来说， HTML比较难解析，由于并不是一个上下文无关的编程语法，它不能够被普通的解析器解析， 也不能被XML解析器解析。  

**HTML DTD**  

HTML是通过DTD来定义的。 这个格式用于定义SGML(Standard Generalized Markup Language)语言。 它定义了所有允许的元素，属性以及层级。正如我们之前所说的， HTML DTD 不能形成上下文无关的语言。  
DTD有一些变动，严格模式严格符合规范，其他的模式支持历史版本的浏览器。 目的也是为了兼容老版本的浏览器。 最新的严格DTD地址：[http://www.w3.org/TR/html4/strict.dtd](http://www.w3.org/TR/html4/strict.dtd)  

**DOM**  

解析树是由DOM元素以及属性节点组成的。 DOM是Document Objectd Model 的简称。 它是HTML文档的对象形式以及其他外部语言(形如Javascript)的接口。树的根节点是 [Document](https://www.w3.org/TR/1998/REC-DOM-Level-1-19981001/level-one-core.html#i-Document) 对象。  

DOM与标签之前有着一对一的对应关系。 例如： 

        <html>
	        <body>
		        <p>
			        Hello World
		        </p>
		        <div> <img src="example.png"/></div>
	        </body>
        </html>  

将会被翻译为以下的DOM树：  
    ![](/sylee.github.io/images/dom-tree-of-markup.png)  
    Figure 8: DOM tree of the example markup  

跟HTML一样， DOM也是被w3c组织定义和管理的。详见[http://www.w3.org/DOM/DOMTR](http://www.w3.org/DOM/DOMTR)。 它是操作文档的通用规范。 一个特定的模块描述了HTML特定的元素。 HTML定义可以参见[http://www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html](http://www.w3.org/TR/2003/REC-DOM-Level-2-HTML-20030109/idl-definitions.html)。  

当我说树包含了DOM节点， 意即树是由实现了DOM接口的元素构建的。 不同的浏览器使用了具体的实现，这些实现包含了浏览器内部使用的其他属性。  

**解析算法**  

在前几节中，我们知道， HTML不能够通过自上而下或者自下而上的解析器解析。  
原因如下：  
    1. 语言的非严谨性。  
    2. 浏览器具有传统的容错机制，用于支持很好的辨别无效的HTML。  
    3. 解析进程的反复迭代机制。 在解析的过程中，解析源通常都不会被改变， 但是在HTML中， script标签包含了document.write， 可以添    加额外的元素，所以解析过程中修改了原始输入。  

由于不能够使用通常的解析器就行解析， 浏览器为解析HTML创建了定制的解析器。  
解析算法在HTML5规范中有详细的描述。 算法由两步构成： 符号化 和 构建树。  
符号化即词法分析，把输入解析为一组符号。 HTML的符号包括开始标签， 结束标签， 属性名和属性值。  
标记生成器识别不同的标记， 并把它传递给树构造器，紧接着识别下一个标记， 周而复始， 直到结束。  
![](/sylee.github.io/images/html-parsing-flow.png)  
    Figure 6: HTML parsing flow (taken from HTML5 spec)  

**符号化的算法**  

算法的结果是一个HTML的标签。 算法被表示为状态机。 每一个状态消耗一个或者多个输入流的字符，然后根据选中的字符跟更新下一个状态。 当前的执行会被符号化的状态和构建树的状态所影响。 这意味着， 相同的符号处理，将会产生不同的结果，根据当前的状体来纠正下一个状态。这个算法太复杂了， 因此不能完整的呈现出来。 所有我们通过一个简单的实例来帮助我们理解这个原则。  

基础实例： 符号化已下的HTML：  

        <html>
	        <body>
		        Hello world
	        </body>
        </html>    

