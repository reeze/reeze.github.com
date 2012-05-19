---
date: '2010-11-14 15:38:32'
layout: post
slug: php-puzzle-1-the-variables-scope-in-php-and-javascript
status: publish
title: 'PHP Puzzle(一): 有趣的变量作用域-PHP中global和Javascript中的var关键字'
wordpress_id: '657'
categories:
- Javascript
- PHP
- PHP内核
tags:
- JavaScript
- PHP Puzzle
- PHP变量
- 作用域
- 全局变量
- 局部变量
---

昨天在网上看到几道有意思的PHP题, 下面这道题让我想起了对应的Javascript版本.
`

这段代码运行结果是什么呢? 别急着执行这段代码,先想想你的结果.然后再对比一下吧.


我们看先看看global的定义 http://www.php.net/manual/en/language.variables.scope.php 这里也没有太为规范的解释.只是说可以通过global关键字来访问全局变量. 这里还涉及到一个类型转换的问题.

大家都知道PHP脚本是编译为opcode逐语句执行的. 那么现在要一句语句解释就很容易了.
`

这里可能比较困惑的的是现在变量$a到底是局部变量还是全局变量了.因为global在定义局部变量之后.所以$a变为了全局变量,而在最后输出结果的时候$a并没有值.所以最后在相乘的时候是 NULL * 100; 也就是0了;可能会有人有疑问, 后面只是把$a变为了全局变量, 他的值应该不变的啊. 让我通过下面的例子来看把:
`

  int(0)
  ["a"]=>
  &NULL;
}
`
变量a是NULL的一个引用,因为全局作用域内没有a这个变量. 所以即使在函数前面定义了一个a变量,但是它的值已经指向了全局作用域了.
实际上 global关键字首先从全局符号表中查找变量名叫做a的变量,并把这个变量值设置为当前作用域的符号表中的a变量(更新了当前变量的值). 如果全局作用域内没有这个变量则会在全局作用域内增加这个变量, 实现代码见: $PHP_SRC/Zend/zend_vm_execute.h
`
static int ZEND_FASTCALL zend_fetch_var_address_helper_SPEC_CONST(int type, ZEND_OPCODE_HANDLER_ARGS) {

        // ...
        if (zend_hash_find(target_symbol_table, varname->value.str.val, varname->value.str.len+1, (void **) &retval;) == FAILURE) {
            switch (type) {
                case BP_VAR_R:
                case BP_VAR_UNSET:
                    zend_error(E_NOTICE,"Undefined variable: %s", Z_STRVAL_P(varname));
                    /* break missing intentionally */
                case BP_VAR_IS:
                    retval = &EG;(uninitialized_zval_ptr);
                    break;
                case BP_VAR_RW:
                    zend_error(E_NOTICE,"Undefined variable: %s", Z_STRVAL_P(varname));
                    /* break missing intentionally */
                case BP_VAR_W: {
                        zval *new_zval = &EG;(uninitialized_zval);

                        Z_ADDREF_P(new_zval);
                        zend_hash_update(target_symbol_table, varname->value.str.val, varname->value.str.len+1, &new;_zval, sizeof(zval *), (void **) &retval;);
                    }
                    break;
                EMPTY_SWITCH_DEFAULT_CASE()
            }
        }

     //...
}
`
看了这个解释大家可能觉得理所当然.一句一句执行的嘛. 看完了PHP中全局作用域的例子,咱们再看看类似的Javascript中的局部变量的版本吧
`
var a = 1;
function multiply(b)
{
     a = 100;
     var a;

     return a * b;
}
alert(a);
alert(multiply(100));
`
那这段代码的输出将会是多少呢?
如果还是同样的思路,结果可能是你的期望完全不一样的结果. **这里的var定义变量和php中global不是一样的东西, php中的global是会在运行时执行的.而Javascript中的var在运行之前就已经"处理"好了**.在运行之前的"语法分析"(没有看过Javascript引擎的实现.姑且这么分把)过程中,multiply函数中出现了var a;则把变量a加到函数体内的"局部变量表"中了.在运行过程中并不会执行var a;这一句.  这也是Javascript"怪异"的地方.定义变量的位置并没有关系.所以在函数内定义局部变量最好放在函数体的前面.

所以第一个alert输出的1, 函数的执行并没有改版全局范围内的a变量; 第二就没有什么问题了, 是10000;
