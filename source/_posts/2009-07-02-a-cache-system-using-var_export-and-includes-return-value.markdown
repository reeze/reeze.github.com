---
date: '2009-07-02 12:55:41'
layout: post
slug: a-cache-system-using-var_export-and-includes-return-value
status: publish
title: 基于var_export 和 include返回值的缓存方案
wordpress_id: '9'
categories:
- PHP
- Web技术
---

[前一篇文章](http://reeze.yo2.cn/2009/07/01/php-include-function-return-value-and-method-of-void-serialize/)我们研究了include调用返回值的问题，并指出可以通过这种方式来完成序列化相同的功能，现在我就来研究一下这种方法的可行性和效率，因为直接的返回php值肯定是比unserialize()函数要快。


第一步我们来研究下怎么将php对象持久化的保存起来。下面是我定义的一些变量：


`
private $_var;
public $pub = array('pub value', 3, 4);
public function __constructor($var)
{
$this->_var = $var;
}`

`public function show()
{
echo $this->_var;
}
}

$string = "It's a string...";
$array = array(1, 2, 'key' => 'value', array('sub-array'));
$number = 135345.55;
$class = new MyClass('class var');

`

`//通过serialize()方法我们可以将他们持久化比如：
echo serialize($string); //s:16:"It's a string...";
echo serialize($array); //a:4:{i:0;i:1;i:1;i:2;s:3:"key";s:5:"value";i:2;a:1:{i:0;s:9:"sub-array";}}
echo serialize($number); //d:135345.5499999999883584678173065185546875;
echo serialize($class); //O:7:"MyClass":2:{s:13:"MyClass_var";N;s:3:"pub";a:3:{i:0;s:9:"pub value";i:1;i:3;i:2;i:4;}}
// 我们可以将这些序列化的结果存到文件中，在需要的时候unserialize()返回得到相应的值，但是现在我不会这么做。
`

前篇文章提到了通过include返回值来直接取得php值对象，首先我们要把值保存起来，因为我们要通过include来包含它，首先遇到的问题就是我们的序列化函数必须要生成合法的php表达式才行，否则include是无法得到相应的返回值的
比如我们要序列化 字符串 "abcd" 我们可以这么做
`
file_puts_content("data.php", "return 'abcd';");
//然后这样取得相应的值
$string = include "data.php";
echo $string; // 它应该输出 abcd
`
那数组怎么办呢？比如上面的数组。我们可以自己编写这个序列化函数
`
function encode($var){
if (is_array($var)) {
$code = 'array(';
foreach ($var as $key => $value) {
$code .= "'$key'=>".encode($value).',';
}
$code = chop($code, ','); //remove unnecessary coma
$code .= ')';
return $code;
} else {
if (is_string($var)) {
return "'".$var."'";
} elseif (is_bool($var)) {
return ($var ? 'TRUE' : 'FALSE');
} elseif (is_numeric($var)) {
return "$var";
}
else
{
return 'NULL';
}
}
}`
这个函数可以将字符串，数组以及数字变成合法的php表达式。
比如：
file_put_contents("data.php", "<?phpn return " . encode($array) . ";n");
data.php文件的结果是：
`
return array ( 0 => 1, 1 => 2, 'key' => 'value', 2 => array ( 0 => 'sub-array', ), )array(4) { [0]=> int(1) [1]=> int(2) ["key"]=> string(5) "value" [2]=> array(1) { [0]=> string(9) "sub-array" } }
`
我们的目的达到了。可以直接的通过include这个文件来得到我们的值。
但是现在有个问题，我们没有序列化对象类型的值，这个该怎么处理呢？
一个类对象有对象的状态和对象的行为，行为在类定义完以后就确定了，所以每个类的实例的行为都是一样的。所以我们可以不考虑，我们只需要考虑类对象的状态就可以了，简单来讲就是类的属性状态需要保存起来。那怎么样得到一个类的属性呢？
经过一番搜寻以后发现一个函数
get_object_vars
(PHP 4, PHP 5)get_object_vars -- 返回由对象属性组成的关联数组
这个函数可以获得对象的属性关联数组，也就只可以得到对象的状态，但是对象的属性有各种访问控制，get_object_vars()函数在对象外访问只能得到对象的公开属性，而无法得到私有属性，这样的话我们就无法得到对象的全部状态，不可行，但是在对象内可以得到对象的所有属性，那我们可不可以在对象内定义一个___get_properties()方法来返回这些状态呢。
给类增加这样一个方法
`
public function __get_properties()
{
return get_obj_vars($this);
}
`
这样我们就可以得到类的所有属性了。第一步算是完成了，我们得到状态该怎么重新恢复出来呢？要在对象外部给对象设置属性我们只有两种情况：一种是这个属性是公开属性，我们可以直接赋值 比如： $obj->prop = $value; 如果是私有属性我们则需要自己增加setter方法比如 setProp($value);方法，来设置。但是这就会遇到一个问题，我们的属性要么是公开属性，要么必须要有setter方法来设置，很多情况下我们不希望给类增加这么多没有实际用处的方法，也为了封装性，不会有这么多的setter方法。虽然我们能得到对象的状态，但是却无法恢复对象状态，这样的话，我们的序列化方法也就没有什么意义了。我们探索到现在算是失败了。
解决办法：var_export()函数。
在看symfony代码的时候发现了这个函数，手册是这么描述的：
---------
var_export
(PHP 4 >= 4.2.0, PHP 5)var_export -- 输出或返回一个变量的字符串表示
描述
mixed var_export ( mixed expression [, bool return] )

此函数返回关于传递给该函数的变量的结构信息，它和 var_dump() 类似，不同的是其返回的表示是合法的 PHP 代码。

您可以通过将函数的第二个参数设置为 TRUE，从而返回变量的表示。

这个就是我们想要的那个函数,我们来看看这个函数是怎么使用的。
`
//变量继续使用上面定义的变量
echo var_export($string); //'It's a string...'
echo var_export($array); /* array (0 => 1, 1 => 2, 'key' => 'value', 2 => array (0 => 'sub-array', ),)*/
echo var_export($number); // 135345.55
echo var_export($class); /* MyClass::__set_state(array('_var' => NULL, 'pub' => array (0 => 'pub value', 1 => 3, 2 => 4)) */
`
我们可以看到生成的都是合法的PHP表达式。通过设置第二个参数为true，就可以将返回结果赋值给变量比如$new_array_string = var_export($array, TRUE);然后将这个结果写入文件持久化

file_put_contents("data.php", "public function __constructor($var)
{
$this->_var = $var;
}

public function show()
{
echo $this->_var;
}
public static function __set_state(array $array)
{
$tmp = new MyClass();
foreach($array as $key => $value)
{
$this->$key = $value;
}
}
}

// 一些变量
$string = "It's a string...";
$array = array(1, 2, 'key' => 'value', array('sub-array'));
$number = 135345.55;
$class = new MyClass('class var')

// 缓存
cache("string.data.php", $string); // 当然扩展名不一定非得php， 文件名我也只是简单的处理
cache("array.data.php", $array);
// 等等。。。

// 获取数据,当然也可以在其他文件中来获取。
$class = cache_get("class.data.php");

参考：http://www.thoughtlabs.com/2008/02/02/phps-mystical-__set_state-method/?dsq=12016456
