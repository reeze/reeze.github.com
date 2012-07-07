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
NOP 是一个特殊的opcode，表示空操作，在很多地方存在，汇编中的NOP含义也一样，
机器指令中的空操作通常用来将内存地址进行对齐，以提高CPU访问内存的效率，
GCC等编译器也会将特定的语句进行优化而产生空操作。

## PHP中的空操作opcode NOP: ZEND\_NOP
PHP基于Zend虚拟机，其他基于虚拟机的语言中大都会有类似NOP的指令，
PHP文档有对此的[简单说明](http://cn.php.net/manual/en/internals2.opcodes.nop.php)：

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

上面的VLD结果可以看出，函数A()的声明编译后变成了`NOP`操作。

Zend虚拟机是高级抽象，不要考虑内存对齐等的问题，为什么还需要空操作这样的opcode呢？

原因简单讲就是：编译过程优化的结果。有的内容由于可以在编译时就可确定，那么一部分opcode可以
在编译时替换成空操作。

我们来看看这段代码的编译结果：

	<?php
	if(true) {
		class Foo {}
	}

	new Bar();
	class Bar {}

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

和前面官方函数定义代码一样，上面VLD输出的第7行看到类Bar的opcode变成了NOP，不过请留意第3行，
这一行类Foo定义的opcode是ZEND_DECLARE_CLASS，也就是类的声明。

*为什么同样是类的声明，第一个类声明的OPCODE和第二个的不一样呢？*

上例中的代码，在Bar类声明之前是可以执行的`new Bar`，但是此时Bar类的声明并没有执行到，
那为什么可以访问到Bar类呢，*这是因为Bar类的声明在编译时就已经完成了*，
因为Bar类已经在编译时声明好了，所以在真正执行的时候就不需要再次执行声明类的操作了，
所以它所对应的opcode被替换成NOP了。

而Foo这个类由于处在条件判断块之中，编译期无法确定Foo类是否一定会被执行，所以还是需要在
执行时来声明这个类，所以opcode没有改变。

对于使用了opcode缓存的代码来说，把函数和类的声明移到了编译时，也就减少了执行时的opcode执行，
这能加快代码的执行。

## 优化
你可能会想，既然类及函数的声明可以优化掉，为什么不能直接丢弃这个opcode呢？
一能减少opcode占用的内容，比如很多的框架中有大量的类定义，也能减少执行时间，
因为空操作并不是0成本的，执行NOP的时候还是需要消耗CPU的。

先看看opcode编译过程的一个重要函数：

	zend_op *get_next_op(zend_op_array *op_array TSRMLS_DC)
	{
		zend_uint next_op_num = op_array->last++;
		zend_op *next_op;
		if (next_op_num >= CG(context).opcodes_size) {
			if (op_array->fn_flags & ZEND_ACC_INTERACTIVE) {
				/* we messed up */
				zend_printf("Ran out of opcode space!\n"
							"You should probably consider writing this huge script into a file!\n");
				zend_bailout();
			}
			CG(context).opcodes_size *= 4;
			op_array_alloc_ops(op_array, CG(context).opcodes_size);
		}
	 
		next_op = &(op_array->opcodes[next_op_num]);
	 
		init_op(next_op TSRMLS_CC);

		return next_op;
	}

这个函数每次会返回一个zend\_op(也就是opcode一个最小单位)，opcode的存储空间申请和哈希表类似，
通过预先申请空间的方式，如果空间不足则适当扩容。在编译时，opcode是以文件为单位的，而通常
在一个文件中函数或类声明的个数是不会太多的。而在编译时opcode数组已经是预先申请好的，所以
及时优化掉这个opcode，而实际在编译时的内存占用也不会有任何的优化。


目前只有少数几处使用了ZEND\_NOP这个opcode。读者可以参考Zend/zend\_compile.c: zend_do_early\_binding()，
这个函数进行就是在确定在编译时能确定的函数以及类声明，在完成函数或类的声明后将当前编译的opcode设置为ZEND\_NOP。
因为后续在执行是并需要再次对该函数或类进行声明了。


在这个函数中其实可以将生成的ZEND\_NOP优化掉的，比如eAccelarator扩展中就对opcode进行了优化，将ZEND\_NOP从
opcode\_array数组中移除了，因为使用了opcode cache扩展优化只进行一次，而执行对多次执行，
这样的优化是值得的。目前Zend引擎并没有进行任何的优化，首先从代码上来看，类和函数声明的数量和其他指令的数量
之间差很多个等级，所以至少这个地方优化的收益是有限的，为了保证Zend引擎的简洁它没有进行优化。

目前APC扩展已经基本确定为将要进入PHP默认opcode缓存的官方扩展了，那么这些优化都可以在扩展中进行，
保证Zend引擎的简单易维护更为重要。
