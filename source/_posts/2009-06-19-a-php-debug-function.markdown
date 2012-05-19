---
date: '2009-06-19 20:05:43'
layout: post
slug: a-php-debug-function
status: publish
title: PHP调试函数
wordpress_id: '15'
categories:
- PHP
- Web技术
tags:
- debug
- PHP
---

在项目中经常要调试程序，但是我电脑上的ZendStudio总是没配置好，不能单步调试，不过有时候不一定需要让ZendStudio来帮我们调试，所以写了下面这个辅助函数来方便调试，因为有时候调试的位置加多了自己也不知道到底是加在什么地方了，下面的函数就是方便的dump对象信息，同时显示调试的问题和所在的行数。

`
//调试函数,方便显示调试函数的位置和文件
function p(){
  $args = func_get_args();

   // 调用栈,debug_backtrace()可以返回调用栈。这样 我们就可以方便的知道函数在哪里调用的。
  $backtrace = debug_backtrace();

  $file = $backtrace[0]['file'];
  $line = $backtrace[0]['line'];
  echo "

    ";
      echo "$file:$linen";
      foreach ($args as $arg)
      {
        var_dump($arg);
      }
      echo "

";
  exit;
}
`
