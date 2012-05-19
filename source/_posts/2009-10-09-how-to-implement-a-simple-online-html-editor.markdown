---
date: '2009-10-09 15:11:26'
layout: post
slug: how-to-implement-a-simple-online-html-editor
status: publish
title: 实现一个简单在线HTML编辑器
wordpress_id: '23'
categories:
- PHP
- Web技术
tags:
- editor
- html
- JavaScript
---

一直没有仔细研究过在线HTML编辑器，以前以为编辑功能很复杂，需要用大量的JavaScript来模拟编辑器的效果，以前都是使用一些开源的HTML编辑器，HTML在各网上随处可见，发表文章，评论。最近自己想做一个类似Things这样的Web版的应用，需要一个想Google  Notebook(可惜的是现在已经停止开发了) 那样的编辑功能，看看现在网上的这些编辑器都庞大了，都是一些自己根本用不到的功能，其实我的需求很简单：简单的编辑既可以，并且需要轻量级一些，因为页面上可能需要开很多个编辑器实例。 所有想自己也来研究一下，看看能不能自己开发一个。

几天前花了一个晚上用firebug看了下Google Docs是怎么做。第一个遇到的问题就是如何让光标停在鼠标点击所在得地方。我刚开始一味都是js模拟出来的，这得有多复杂啊。并且还要兼容各个浏览器，天啊！后来上网一搜发现，浏览器早就想到了我们会有这样的一个需求。其实很简单，两条语句就可以说明HTML编辑器的最为核心的部分：
`
document.designMode = 'On';
document.contentEditable = true;
`
参考Mozilla上的[这篇](https://developer.mozilla.org/en/Rich-Text_Editing_in_Mozilla)文章，介绍了HTML编辑的基本信息，要自己DIY一个常用功能的HTML编辑，这篇文章已经够你用的了。

在你的网页中嵌入这两条语句试试看：），你就会发现你的网站整个得都变的可以编辑了。可以随便乱修改。 不过放心，这样修改并不会破坏你的网站， 当然你也不希望你的整个网站是可以编辑的。例如我们只希望别人发布一条评论，只希望评论输入框可以输入。 要实现这样的效果可以有两种做法：

第一种就是使用直接让某个元素变成可编辑的例如：

index.html
`
Test TextEditor


# Hi, HTML Editor!


Hello, you comment please


Your comment


Get HTML
`
点击该区域后该区域的内容就变成可以编辑的了，这是我们就是对其进行简单的编辑。你可能会觉得直接编辑的功能太简单了，比如想要插入链接或者图片什么的。就没有办法了。这些功能浏览器并没有帮我们做好，不过实现这些功能也不麻烦， 参考上面Mozilla的文档。浏览器都提供了常用的功能API。

一般的编辑器都会提供一个工具栏之类的按钮来编辑文本内容。比如我使用的WordPress提供的编辑器：

[caption id="attachment_42" align="alignnone" width="936" caption="wordpress提供的编辑器"][![wordpress提供的编辑器](/uploads/2009/10/5b9e5c0b92f7fdf042ea3993165b80ec.png)](/uploads/2009/10/5b9e5c0b92f7fdf042ea3993165b80ec.png)[/caption]

可以对文字内容进行操作，加粗下划线，字体，对齐等等，并且提供可视化以及HTML编辑模式。

这个和FCKeditor 以及tinymce之类的编辑器使用的编辑方式和上面我提到的直接编辑html对象的方法不一样，他们使用的是iframe，使用iframe有很多好处，iframe中的文档和当前文档并不会因为样式或者HTML结构而影响到彼此，所以大部分的编辑器都是使用这种方式。它们基本的方式都是：




  1. 在页面中使用一个不可见的字段比如:“input, textarea"之类表单字段，他们的值就是需要编辑的内容。


  2. 页面初始化好，比如载入编辑器相关的一些脚本，也是就是window.onload好以后。创建一个iframe来显示可编辑区域，iframe初始化好以后读取父窗(相对于这个iframe)口的这个不可见表单值的内容，使之成为iframe中的body的html，也就是把所有需要编辑的内容添加到iframe中


  3. iframe中的内容初始化好以后，在iframe中执行上面提到的：
`
document.designMode = 'On';
document.contentEditable = true;
`
把iframe整个窗口变成可以编辑的


  4. 进行编辑，这时候的编辑可能需要一个工具栏，基于同样的原因，一般工具栏也会是一个iframe，显示它们自己的编辑按钮。编辑是就利用浏览器提供的接口来对ifame中的内容进行编辑


  5. 父窗口中提交表单之前或者你需要的地方需要把编辑器中编辑完的内容回写到你的表单字段中去，否则编辑结果没有保存写来就没有意义了。


基本原理就是这样。要做出这样一个东西来，需要的就是一些细活了。要想做出一个FCk这样好用的编辑器也不是那么简单的。但是至少我们知道它是怎么运作的。 这就够了。

花了点时间做了一个简单的编辑器，真正要用的话很多的细节还是需要好好处理的，代码没有怎么清理，是变想边写，不是很完整。

需要的同学可以下来参考参考。
猛击 [ >> 这里 << ](/uploads/2009/10/editor.rar)下载代码。
