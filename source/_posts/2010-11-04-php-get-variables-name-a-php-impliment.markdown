---
date: '2010-11-04 11:07:37'
layout: post
slug: php-get-variables-name-a-php-impliment
status: publish
title: 怎样获取PHP变量的变量名之PHP实现
wordpress_id: '214'
categories:
- PHP
- PHP内核
- Web技术
tags:
- PHP
- 变量
- 获取变量名字
---

[上一篇文章](http://reeze.cn/2010/10/30/php-internal-how-to-get-variables-name-an-extension-implement/)里提到是用PHP扩展实现获取变量的变量名的方法. 今天发现有一个PHP实现的版本 . 实现方法来自:http://mach13.com/how-to-get-a-variable-name-as-a-string-in-php

**刚开始以为这个方法好使, 仔细想想其实也是有问题的.**
<del>
这个解决方法是用的PHP里的get_defined_vars()方法,该方法返回当前作用域内的所有变量信息.也是和$GLOBALS一样,以变量名 => 值的方式返回.
他的代码很简单:

`
$v)
        $aDefinedVars_0[$k] = $v;

    $iVarSave = $iVar;
    $iVar     =!$iVar;   // 将当前变量的值取反

    $aDiffKeys = array_keys (array_diff_assoc ($aDefinedVars_0, $aDefinedVars));  // 对比取反前后的变量
    $iVar      = $iVarSave; // 恢复当前变量的值

    return $aDiffKeys[0];
    }

?>

`

它通过引用的方式改变当前变量的值, 然后通过对比前后两个数组的差异来获取值被改变了的变量.然后返回其名字.经过测试这的确是一个方法.相对我实现的方法. 它提供的方法移植性较好, 不需要赖以扩展. 而这个php版本的实现, 必须传递一个get_defined_vars()的参数, 我实现的那个扩展,则不需要. 对于类似 var_name($a=10,get_defined_vars()); 的调用,该方法无法正常获得变量名.
</del>

这个今天又仔细想了想,下面提供的方法是有问题的.. 他解决问题的方法是通过修改变量的值, 并对比前后所有的变量来找出值发生变化的变量. 而实际上.修改了其中一个变量另一个变量的值也会发生变化: 这就是引用, 如下
`



PS: 如果你真的需要这种方法. 请重新思考一下你的需要真的需要这样的方法么?
