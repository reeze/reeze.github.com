---
layout: post
title: "LevelDB扩展发布V0.1版本"
date: 2012-09-12 23:16
comments: true
categories: 
---

很开心，第一个提交的PHP扩展已经在PECL官方发布了，这是一个[Google LevelDB](http://code.google.com/p/leveldb/)的PHP
封装，主要用于对LevelDB的访问，目前已经实现了LevelDB最具价值的一些特性：迭代器，快照等。


LevelDB数据的设计是只能单进程访问的（多线程没有问题），所以通常这个扩展不合适作为普通的Web应用数据存储，
可以作为离线的数据存储用，或者只是方便读取现有leveldb的数据。

如果有需要可以前去 <http://pecl.php.net/package/leveldb> 下载。基本的使用说明在<http://reeze.cn/php-leveldb/>
详细的API文档由勤劳高效的胖胖<http://www.phppan.com/>编写，不过还没有发布。

同时，还有了@php.net的马甲一枚: reeze(at)php.net。MARK一下。
