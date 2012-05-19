---
date: '2009-07-01 18:24:21'
layout: post
slug: a-way-use-include-to-avoid-unserialize
status: publish
title: PHP中include语句的返回值及避免序列化开销的方法
wordpress_id: '11'
categories:
- PHP
- Web技术
---

以前乱翻symfony生成的缓存文件的时候看到很多类似：



`
"value"); ?>
`


这种表达式，当时并没有怎么在意，今天研究symfony代码的时候看到这样一句代码

`
classes = include($file); ?>;
`



一直都是通过include require来包含文件，但是从来没有使用过他的返回值，印象中include返回的无非是true或者false吧。今天自己看了下文档，发现在php文件中是可以直接调用return的。比如：


`
// return.php

// get_return.php



echo $value; // 输出 Array, 因为$value 是从return.php返回的一个数组
?>
`
其实项目中很少情况需要这样的返回方法。如果想要从return.php中得到返回值一般是通过调用return.php中所调用的函数来得到。

在symfony中这种方式就很合理，如果大家熟悉symfony的话，应该知道，symfony运行起来以后会在cache目录下生成系统配置文件的缓存,诚然可以通过序列化的方式来缓存这些信息，但是反序列化是需要消耗资源的。通过这种方式来做持久化也是个不错的选择.
