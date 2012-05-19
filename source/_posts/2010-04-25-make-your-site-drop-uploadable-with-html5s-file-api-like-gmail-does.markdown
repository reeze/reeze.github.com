---
date: '2010-04-25 00:14:29'
layout: post
slug: make-your-site-drop-uploadable-with-html5s-file-api-like-gmail-does
status: publish
title: 让你的网站也像Gmail一样支持文件拖放上传-HTML5之File API
wordpress_id: '175'
categories:
- Javascript
- Web技术
tags:
- File API
- Gmail
- HTML5
- 托放上传
---

> 如果你比较好奇，可以先从[这里下载所有代码](http://labs.reeze.cn/labs/HTML5/FileAPI/FileAPI_Test.zip)，也可以点击这里[查看chrome下上传的demo](http://labs.reeze.cn/labs/HTML5/FileAPI/chrome_drop_upload.html)，点这里[查看firefox下的demo](http://labs.reeze.cn/labs/HTML5/FileAPI/firefox36_drop_upload.html)


前不久[Gmail推出了支持拖拽的附件上传功能](http://net.chinabyte.com/395/11221395.shtml)，试用了下还真不错，其实很久以前就在想能有直接拖拽附件的功能，多亏有了HTML5，Web应用越来越像客户端的应用了。

在好奇心驱使下，想了解一下Gmail到底是怎么做到的，了解了一下最新的[HTML5 File API草案](http://www.w3.org/TR/FileAPI)，这个接口主要提供的就是提供对文件对象的访问，别想歪了，这个接口是无法随意的访问系统里的文件的。他能做的就是访问<input type="file" />标签里所选择的文件，这些文件可以通过用户手动选择，或者是HTML5的拖放接口选中的文件。有兴趣的童鞋可以看看这个规范，还算比较简单。

下面简单看看接口定义几个对象。

FileList、File对象。

在HTML5中的<input type="file"  />标签中增加了mutilple属性，允许进行多文件选择。大家应该都知道一般上传标签中是不允许选择多个文件的。 新增的这个属性就是允许进行多个文件的选择（这个在桌面应用中也很常见）。<input type="file" **multiple="multiple"** id="file" />

下面是在Firebug中的输出
`
>>> var f = document.getElementById("file")
>>> f.files
FileList0=File length=1 // 选中的文件数量
>>> f.files[0]
FilefileName=es.dll fileSize=271360
`

FileList对象就是用户选择的所有文件的对象表示，如果是通过input标签选择的，就可以通过上面代码所示的方法进行访问，File对象就可以刚才选择的某个文件的信息，如上面的代码所示，主要可以得到所选中的文件名以及文件大小信息。

你可能在想只能得到这些信息到底有什么用呢？都没有办法读取文件内容，这就得提到规范中的FileReader接口了，这个接口就是用来读取File对象文件的。

在[File API规范](http://www.w3.org/TR/FileAPI)中提到File API主要是和其他的接口协同合作。比如XMLHttpRequest (这个新接口支持通过xhr的send()方法发送File对象)， DataTransfer(也就是HTML5中的拖拽接口 )， 以及[Web Worksers](http://dev.w3.org/html5/workers)(这个主要是异步脚本执行，相当于给JS提供了“多线程”脚本执行能力，并且支持通过postMessage()进行“线程间通信”)，感兴趣的，可以看看[这篇日志](http://www.v-ec.com/dh20156/article.asp?id=242)，以及[这篇](http://feedproxy.google.com/~r/webbang/~3/_usD4yheDqI)。

目前能实现这样的效果的方式主要有如下几种：




  * Gmail中提到的这两个浏览器都支持拖放接口，托放以后可以直接通过托放事件的DataTransfer属性访问到本次托放是关联的文件对象列表FileList,然后通过XMLHttpRequest的send方法将File对象发送到服务器


  * 在Chrome下支持直接将文件拖放到文件选择控件上，就相当于直接选择了文件。这时可以通过<input type="file" />DOM对象的files属性访问到被托放进来的文件列表对象，然后也可以通过Ajax将文件对象发送到服务器，通过将文件选择控件透明度降低也可以实现Gmail类似的效果。 在Chrome因为可以直接通过托拽的方式让文件选择控件“赋值”，此时也可以通过一个iframe加表单的方式将数据发送到服务器。


  * 在Firefox3.6下可以通过FileReader直接读取到文件的内容，然后直接将文件内容发送到服务器端(可以参考[这个例子](http://labs.reeze.cn/labs/HTML5/FileAPI/index.html)，这是个不完整的例子，直接浏览是看不到效果的，查看源代码你就会懂的。）


下面就来看看Gmail到底是怎么做到的吧。

本来想通过Firebug的概况功能来捕捉到在托拽期间的脚本执行情况，比如：

[![](/uploads/2010/04/screenshot1.png)](/uploads/2010/04/screenshot1.png)[
](/uploads/2010/04/screenshot.png)

但是脚本执行里压根没有找到ajax相关的函数调用，可能是因为firebug还不支持监控页面里嵌入的iframe中的脚本执行跟踪，这也说明本次上传肯定是在某个iframe中完成的。，那就直接监听网络吧，托拽上传一个附件时查看网络情况，发现附件是通过下面的ajax post过去的：


[![](/uploads/2010/04/Compose-Mail-reeze.xia@gmail.com-Gmail1.jpg)](/uploads/2010/04/Compose-Mail-reeze.xia@gmail.com-Gmail1.jpg)


大家注意看，是通过ajax post方式将附件POST到服务器的，

[![](/uploads/2010/04/Gmail-11.jpg)](/uploads/2010/04/Gmail-11.jpg)

[![](/uploads/2010/04/Compose-Mail-reeze.xia@gmail.com-Gmail-31.jpg)](/uploads/2010/04/Compose-Mail-reeze.xia@gmail.com-Gmail-31.jpg)[
](/uploads/2010/04/Compose-Mail-reeze.xia@gmail.com-Gmail-3.jpg)

可以看出Gmail在firefox下不是通过表单直接提交实现的。在chrome下的开发人员工具有点简单，无法看到网络情况，我也懒的再去抓包看了，估计是使用透明<input type="file" />+ajax方式实现的。

在Gmail支持托拽的声明中提到目前只支持Chrome 2+以及FireFox3.6+。虽然这两个浏览器都支持HTML5，但是对于所有规范的支持程度都是不一样的，并且规范也还不是正式规范。在Firefox3.6的release note中提到：

Support for new DOM and HTML5 specifications including the Drag & Drop API and the File API, which allow for more interactive web pages.

开始支持了HTML5的拖拽接口以及File API。



* * *

下面根据浏览器以及HTML5的规范整理出两个浏览器下实现类似Gmail 上传附件的代码。

[![](/uploads/2010/04/14.jpg)](/uploads/2010/04/14.jpg)

点击[这里下载所有代码](http://labs.reeze.cn/labs/HTML5/FileAPI/FileAPI_Test.zip)，有兴趣的童鞋查看源代码就知道怎么回事了，有一定的注释：）

也可以点击这里[查看chrome下上传的demo](http://labs.reeze.cn/labs/HTML5/FileAPI/chrome_drop_upload.html)，点这里[查看firefox下的demo](http://labs.reeze.cn/labs/HTML5/FileAPI/firefox36_drop_upload.html)，之所以分开是为了简单起见，当然你真的想要给你的网站提供托拽上传功能，你就得自己去同时兼容这两个浏览器啦，相信这也不是件困难的事情：）
