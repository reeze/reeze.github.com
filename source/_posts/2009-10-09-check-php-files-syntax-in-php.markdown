---
date: '2009-10-09 19:21:32'
layout: post
slug: check-php-files-syntax-in-php
status: publish
title: 在PHP中检查PHP文件是否有语法错误
wordpress_id: '49'
categories:
- PHP
- Web技术
tags:
- PHP
- syntax check
---

之前在当当的时候的一个项目中用到了一个简单的模板引擎，其实也是借鉴discuz来做的模板引擎，很简单，它所作的事情就是把一些自定义的标签编译成php代码。已经说了很简单了，所以编译的时候也名优进行模板语法的检查，那么在开发过程中就会出现编译出来的php文件有语法问题，有语法问题没有关系，我修改重新编译一下就好了。首先不能在每次请求的时候都把php模板重新编译一下，会严重影响性能，折中的处理时在每个编译好的php文件末尾检查一下该模板文件是否已经修改过，根据设定的更新频率，如果又需要则重新编译模板文件，现在的问题是编译出来的php文件自己有语法错误，根本执行不到模板检查那一步，所以即使修改了模板文件中的问题也不会重新编译。 所以我想寻找一种简单的方法来检查生成的php文件是否合法。不合法就重新编译，这样开发过程中就不用出现错误就得手动删除缓存文件了。

在网上找了一下。刚开始以为 token_get_all()函数能处理语法错误的问题，结果发现，它只是做简单的词法分析。没有办法。后来到论坛上去问了一下
http://groups.google.com/group/professional-php/browse_thread/thread/b8581f6b07b10ff0/2601a63c406bb1c1?lnk=gst&q=reeze#2601a63c406bb1c1

有人告诉我有这样一个函数 php_check_syntax() http://www.php.net/manual/en/function.php-check-syntax.php 我想问题就这么坚决了。。我真应该RTF(Read The Fuck Mannual). 仔细一看。这个函数已近被弃用了：
Note: For technical reasons, this function is deprecated and removed from PHP. Instead, use php -l somefile.php from the commandline.

这个technical reason 到底是什么呢？ 先不管了，以后再慢慢研究，反正不能使用这个方法就对了。
他们的建议是使用命令行$php -l filename.php 来检查语法。
Gary Every给了我一个代码片段参考：

在命令行下检查问题也不大。如果我要放在在线应用呢？ 这就涉及到可移植性的问题了。首先是操作系统，然后就是环境变量。这样的话就会依赖于服务器端的配置。在http://www.php.net/manual/en/function.php-check-syntax.php 上有人贴出了自己的php_check_syntax()函数实现。
有的采用的就是上面的命令行的方法。
后面有提到使用eval的方法来验证。eval方法会执行传入的代码， 如果代码有语法错误则会抛出parser error, 可以使用'@'错误抑制符去掉错误信息，eval和echo一样并不是函数，不能使用变量函数的方法调用比如：
$func = 'eval'
$func()这样的调用就是无效的。它会提示没有eval函数，如果你自己定义这么一个函数也是有问题的。因为eval是一个关键字。
eval调用和include差不多，如果被包含文件中没有明确return就返回null。如果直接eval我们需要检查的文件会造成被检查的文件内代码被执行，这可不是我们想要的，我们只需要检查一下这个文件的语法是否正确。 我们可以在要检查的文件之前添加return 语句，让代码提前跳出，那么后面的代码就不会执行了。好的，就这么干。代码如下：
checker.php
`
";
        $file_content = $check_code . $file_content . "

file.php
`
b
?>
`
因为Parse error 是没法被 set_error_handler处理函数处理的。这个异常没办法catch到。所以才使用了@来抑制错误。这带来的问题就是我们无法得到详细的错误信息。 不过目前我需要的功能也只是检查语法是否正确。不正确的话重新编译模板文件，就这么简单，至于语法错误，在显示网页的时候自然会看得到。
最好的办法就是这个被遗弃的php_check_syntax这个方法回到php中。下次再研究下他们是出于什么原因把这个函数去掉的。
