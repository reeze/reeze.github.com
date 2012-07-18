---
layout: post
title: "初试Travis-CI"
date: 2012-07-15 20:26
comments: true
categories: 
  - PHP
  - CI
  - Travis
---

Travis CI是一个基于云的[持续集成](http://en.wikipedia.org/wiki/Continuous_integration)项目，
目前已经支持大部分主流语言了，比如：C，PHP，Ruby，Python, Nodejs等等。和[Jenkins](http://jenkins-ci.org/)类似，
Travis CI也是开源的，不过Travis和Github集成非常紧密，官方的集成测试托管只支持Github项目，
不过你也可以搭建一套自己的方案。 这里有一篇比较详实的[对Travis-CI的介绍](http://www.juvenxu.com/2012/03/06/travis-ci/)，
同时InfoQ上也有一篇[关于Travic\_CI的报道](http://www.juvenxu.com/2012/03/06/travis-ci/)，

如果你有开源项目，那么Travis绝对值得一试，目前托管在Github上的大部分知名项目都使用了Travis来做集成测试。
比如Ruby语言的：Rails, Rack, Sinatra, RSpec, Cumber, Node.js, PHP的：Symfony2, Doctrine2, Zend Framework 2。

PHP语言也[使用了Travis做集成测试](https://github.com/php/php-src)，不过目前由于PHP的扩展众多，
很多的测试用例本身也不够健壮，PHP的测试经常会失败。

使用Travis-CI的项目可以在说明文件中增加目前版本的构建状态。Travis为每个项目提供一个图片地址，比如PHP的：
https://secure.travis-ci.org/php/php-src.png?branch=master, 
![php-src](https://secure.travis-ci.org/php/php-src.png?branch=master)，之所以是构建失败，是因为前面提到的原因。

如果构建成功，图片将显示：![php-leveldb](https://secure.travis-ci.org/reeze/php-leveldb.png?branch=master)，
这是我最近在写的一个扩展：[php-leveldb](https://github.com/reeze/php-leveldb)的构建状态<http://travis-ci.org/reeze/php-leveldb>

下面简单介绍一下，如果你在编写一个PHP扩展，该怎么样使用Travis-CI来做持续集成。
当然，你的代码需要在Github上进行托管。

## Travis-CI配置文件
要使用Travis，首先需要在你的代码根目录下包含一个叫做.travis.yml的文件，这是一个配置文件，
为[yaml格式](http://www.yaml.org)，

``yml
language: php
php:
  - 5.2
  - 5.3
  - 5.4

env:
  - REPORT_EXIT_STATUS=1 NO_INTERACTION=1

before_script: sh travis/prepare.sh

script: sh travis/run-test.sh

notifications:
  email:
    - reeze.xia@gmail.com
```

配置简单明了，选择需要的语言，同时可以设置需要测试的语言版本，因为php-leveldb支持5.2 ~ 5.4, 
所以这里设置了3个需要测试的版本。

env 设置为执行测试时需要设置的环境变量，因为php 执行`make test`测试时如果测试失败会提示
用户是否将测试结果发送给PHP官方。因为测试是自动进行的，如果不设置NO_INTERACTION=1会导致
测试失败后一直等待用户输入而hang住直到测试超时。

before_script 可以用来进行一些准备工作，例如php-leveldb扩展需要先安装leveldb才能编译。

travis/prepare.sh文件做的工作也就是从leveldb官方下载代码并编译，
最后编译扩展。
```
#!/bin/sh

wget http://leveldb.googlecode.com/files/leveldb-1.5.0.tar.gz
tar zxvf leveldb-1.5.0.tar.gz
cd leveldb-1.5.0 && make
cd ..
phpize && ./configure --with-leveldb=$PWD/leveldb-1.5.0 && make
```

travis可以执行任何脚本。因为travis在执行测试之前会建立一个虚拟机用于测试。

script属性就是测试的入口，可以是任何的sh命令。测试结果到底是成功还是失败会依据这个命令的
返回值，如果返回非0结果，则表示测试失败，失败的时候就会给下面notifications设置的邮箱发送邮件。

```
#!/bin/sh

make test

# make test didn't return status code correctly
# use this to find whether the make test failed
cat tests/\*.diff

if [ $? -eq 0 ]; then
	exit 1;
fi
```
从上面的注释里也提到，因为PHP生成的[测试不能正常的返回错误码](https://bugs.php.net/bug.php?id=60285)，这样的结果是即使测试失败了，
travis也会忽略，所以采用了一个折中的方式来测试。因为测试失败会生产diff文件，
报告测试失败的具体原因，cat的好处是能把错误详细信息报告出来，也能方便调试。

每次提交代码到Github上时，Travis将自动对新提交的版本进行测试。

除了PHP，其实你可以在Travis上测试绝大部分的软件，因为travis提供的是一个完整的环境，
而你可以编写任何的脚本来进行你的测试工作。
