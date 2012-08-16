---
layout: post
title: "怎么样获取PHP函数默认参数的常量名"
date: 2012-07-22 20:40
comments: true
categories: 
  - PHP实现
  - Reflection
  - PHP内核
  - 反射
---

好久没有更新了，发篇占位文：如果某个函数的默认参数是个常量，那么怎么样获取这个参数的常量名称？见代码：

```php
<?php

function new_blog($title = DEFAULT_TITLE) {
	// blahblah
}
```

在上面的代码中，怎么样获取函数new_blog函数的参数$title所对应的默认值常量名： `DEFAULT_TITLE`。这个问题和以前我曾写过的一篇
关于[如何获取变量名称](http://reeze.cn/2010/12/26/how-to-get-a-php-variables-name-a-custom-php-syntax-implementation/)的博文相似。

这个问题，在PHP5.4.6之前基本上没有解决方法了，因为函数定义是编译时的信息，在PHP运行时是获取不到的。
当然这里说的无法实现是指的使用官方PHP版本时没法搞定。

在PHP中类似的需求，一般都可以使用PHP的反射扩展。


### PHP的反射(Reflection)
反射是PHP5中提供的用于获取或操作PHP内部信息的标准扩展，可能写应用代码的用户使用的较少一些，
编写框架或者平台性的系统会使用到。

比如你的框架需要实现一种插件机制，而你可能需要利用反射来获取类或者函数的元信息。
这里就不对Reflection的使用做过多的介绍了，详细信息见官方文档： <http://cn.php.net/manual/en/book.reflection.php>


### 新的函数ReflectionParameter::getDefaultValueConstantName()
不过在PHP5.4.6之前，Reflection是没有实现该功能的。这个需求其实来自PHPUnit的作者[Sebastian Bergmann](https://github.com/sebastianbergmann)。
因为这个需求在Reflection模块来说是一个缺失，不属于大功能的升级，所以直接进入了目前的最新分支PHP-5.4。
同时这个功能在PHP-5.4.6中可用了。<https://github.com/php/php-src/blob/PHP-5.4.6/NEWS#L41>

实现代码见：<https://github.com/php/php-src/commit/13a9555342a4156a6150818234639b49a596ccd6>,
这个方法目前没有使用说明，不过看名字应该也能明白。不过可以参考[测试用例](https://github.com/php/php-src/commit/13a9555342a4156a6150818234639b49a596ccd6#diff-1)。

这个提交给ReflectionParameter类增加了两个函数：

1. ReflectionParameter::isDefaultValueConstant() 用于判断函数的这个参数是否是常量默认参数
1. ReflectionParameter::getDefaultValueConstantName() 用于获取这个常量默认参数的参数名称


```php
<?php
define("CONST_TEST_1", "const1");

function ReflectionParameterTest($test1=array(), $test2 = CONST_TEST_1) {
	echo $test;
}
$reflect = new ReflectionFunction('ReflectionParameterTest');
foreach($reflect->getParameters() as $param) {
	if($param->getName() == 'test1') {
		var_dump($param->isDefaultValueConstant());
	}
	if($param->getName() == 'test2') {
		var_dump($param->isDefaultValueConstant());
	}
	if($param->isDefaultValueAvailable() && $param->isDefaultValueConstant()) {
		var_dump($param->getDefaultValueConstantName());
	}
}

class Foo2 {
	const bar = 'Foo2::bar';
}

class Foo {
	const bar = 'Foo::bar';

	public function baz($param1 = self::bar, $param2=Foo2::bar, $param3=CONST_TEST_1) {
	}
}

$method = new ReflectionMethod('Foo', 'baz');
$params = $method->getParameters();

foreach ($params as $param) {
    if ($param->isDefaultValueConstant()) {
        var_dump($param->getDefaultValueConstantName());
    }
}
?>
// 运行结果
bool(false)
bool(true)
string(12) "CONST_TEST_1"
string(9) "self::bar"
string(9) "Foo2::bar"
string(12) "CONST_TEST_1"
```
