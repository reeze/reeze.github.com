---
layout: post
title: "PHP中的NOP及为什么有这个opcode"
slug: "what-and-why-opcode-zend-nop-in-php"
date: 2012-05-28 14:19
comments: true
categories: 
  - PHP内核
---

## 什么是NOP 
NOP 是一个特殊的opcode，表示空操作，在很多语言中存在，汇编中的NOP含义也是一样，
机器指令中的空操作通常用来将内存地址进行对齐，以提高CPU访问内存的效率，
GCC等编译器也会将特定的语句进行优化而产生空操作。

同样，在Zend引擎中也有类似的opcode: NOP，官方文档有对此的[简单说明](http://cn.php.net/manual/en/internals2.opcodes.nop.php)：

	[php]
	<?php
	/*
	 * no operation
	 * opcode number: 0
	 */
	function A(){}; 
	?>

	/* VLD 的输出结果 */
	line	#	op	fetch	ext	return	operands
	6	0	NOP	 	 	 	 
	7	1	RETURN	 	 	 	1
	Function name: A

	Compiled variables: none

	line	#	op	fetch	ext	return	operands
	6	0	RETURN	 	 	 	null

上面的VLD结果可以看出，函数A()的声明编译后变成了NOP操作。

## 为什么PHP中需要空操作opcode NOP
Zend虚拟机是高级抽象，不要考虑内存对齐等的问题，为什么还需要空操作这样的opcode呢？

在没有使用opcode缓存的时候，PHP脚本的执行都需要经过opcode编译及执行两个环节。
先看看下面这段代码的VLD输出吧：

	filename:       /Users/reeze/Opensource/php-test/php-src-5.4/test.php
	function name:  (null)
	number of ops:  8
	compiled vars:  none
	line     # *  op                           fetch          ext  return  operands
	---------------------------------------------------------------------------------
	   2     0  > > JMPZ                                                     true, ->3
	   3     1  >   ZEND_DECLARE_CLASS                               $0      '%00foo%2FUsers%2Freeze%2FOpensource%2Fphp-test%2Fphp-src-5.4%2Ftest.php0x106cd601f', 'foo'
	   4     2    > JMP                                                      ->3
	   6     3  >   ZEND_FETCH_CLASS                              4  :1      'Bar'
			 4      NEW                                                      :1
			 5      DO_FCALL_BY_NAME                              0          
	   7     6      NOP                                                      
	   9     7    > RETURN  

和函数定义一样，上面VLD输出的第7行看到类Bar的opcode变成了NOP，不过请留意第3行，
这一行类Foo定义的opcode是ZEND_DECLARE_CLASS，也就是类的声明。

*为什么同样是类的声明，第一个类声明的OPCODE和第二个的不一样呢？*

上例中的代码，在Bar类声明之前的`new Bar`是可以的，但是此时Bar类的声明并没有执行到，
那为什么可以访问到Bar类呢，*这是因为Bar类的声明在编译时就已经完成了*，
因为Bar类已经在编译时声明好了，所以在真正执行的时候就不需要再次执行声明类的操作了，
所以它所对应的opcode被优化成NOP了，

而Foo这个类由于处在条件快之中，编译器无法确定Foo类是否一定会被执行，所以还是需要在
执行时来声明这个类，所以opcode没有改变。

对于使用了opcode缓存的代码来说，把函数和类的声明移到了编译时，也就减少了执行时的opcode执行，
这能加快代码的执行。

你可能回想，既然这种类及函数的声明可以优化掉，为什么不直接丢弃这个opcode呢？
Zend引擎在实现时的确采用了这种方法进行了优化，比如在Zend/zend_compile.c 中的`void zend_do_assign()`函数
就采用了这中方式，那为什么在opcode中还是存在NOP opcode呢？
Zend引擎在编译时会申请一块连续的内存保存opcode数组，在进行这类的优化时，
opcode数组已经编译好了，这时可以选择将这个opcode删掉，也就是从一个数组中删掉一个元素，
为了保证数组连续，这就需要将后面所有的元素向前移动，如果这样做的话带来的开销是非常大的。
为此只是简单的就优化掉的OPCODE置为NOP，这也是空间换时间的一种做法。

在实际代码中，函数及类的声明占总的代码量的比率是比较小的，为了不必要的增加代码复杂度，
所以选择继续保留这个opcode所占用的空间。



