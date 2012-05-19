---
date: '2009-06-19 20:21:18'
layout: post
slug: start-sshd-service-in-mac-os-x
status: publish
title: Mac OS 启动sshd服务
wordpress_id: '14'
categories:
- Mac/iOS
tags:
- Mac
- ssh
- sshd
---

**[UPDATE]  系统偏号设置  -> 共享 -> 远程登陆 ,开启即可.  下面的方法不推荐使用.**

想push自己写的一些代码到本地的版本库中去，看了很多的协议，都挺麻烦的，至今没有配置好一个git server，遂放弃搭建server，直接使用ssh来提交到本地

无奈Mac OS X 似乎默认不启动SSHD服务。所以我尝试启动

Leopard:etc reeze$ sshd
sshd re-exec requires execution with an absolute path
看来不行。
不过这个提示错误似乎不是很明白。上网baidu了一把。发现
只能用绝对路径启动。不知道为什么程序非得用绝对路劲启动。有时间研究下为何是这样的，或者有知道的直接告诉我：）
$ sudo /usr/sbin/sshd
服务就启动了
不能每次都这么运行一下先啊。放在 ~/.bashrc 似乎可以，不过这个得需要管理员权限
得sudo，每次输入密码很烦人。干脆放到 /etc/rc.common,

#####
/usr/sbin/sshd

网上有人说apple不推荐这么干。但是我又不知道怎么让他自动启动。先就这么放着吧。找到好方法再改。
