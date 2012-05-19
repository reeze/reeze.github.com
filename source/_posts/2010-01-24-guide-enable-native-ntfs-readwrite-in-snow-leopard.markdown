---
date: '2010-01-24 18:19:31'
layout: post
slug: guide-enable-native-ntfs-readwrite-in-snow-leopard
status: publish
title: 开启Mac OS X Snow Leopard的NTFS原生读写
wordpress_id: '165'
categories:
- Mac/iOS
tags:
- MacFUSE
- NTFS
- OS X
- Snow Leopard
---

在Mac OS X下一直使用MacFUSE和NTFS-3G来访问ntfs分区，这次装了Snow leopard 10.6.2之后出现fusefs.kext can't load的错误，从官方得知目前macfuse在snow leopard下有问题，在网上看到[这篇贴子](http://forums.macrumors.com/showthread.php?t=785376)提到Snow Leopard其实原生就支持NTFS分区的读写，真是个好消息。

开启的方法有两种：
一种是在/etc/fstab文件里增加挂载选项，基本步骤是：
1，卸载NTFS-3G或者其他读写ntfs分区的软件
2，打开终端$ diskutil info /Volumes/分区名 或者使用磁盘工具获得分区的UUID
3，编辑/etc/fstab文件，增加一行 UUID=分区的UUID none ntfs rw
4，如果还有其他的分区要挂载，则继续上面的步骤2－3增加其他分区

这种方式比较烦琐，第二种方法就比较简单。
大家应该都注意过系统已启动就会自动挂载所有的ntfs分区，这个命令在/sbin/mount_ntfs
这个方法所要做的就是给这个默认的挂载命令增加可读写参数，按如下步骤在终端命令行操作：
$ sudo mv /sbin/mount_ntfs /sbin/mount_ntfs.orig
$ sudo vim /sbin/mount_ntfs
编辑这个文件，加入如下内容：
#!/bin/sh
/sbin/mount_ntfs.orig -o rw "$@“ ＃这里调用默认的挂载命令，不过增加了 rw参数，开启读写

保存这个文件，然后：
$ sudo chown root:wheel /sbin/mount_ntfs
$ sudo chmod 755 /sbin/mount_ntfs

然后重启，启动后，你应该就能得到一个可以自由读写的ntfs分区了。不过声明一点，这个功能据说不稳定，不知道是因为版权原因还是真的很不稳定，苹果默认没有开启这个功能。不过我更相信苹果。

＝＝＝＝＝＝＝＝
后话：刚好公司配了一台Dell E6400,偶尔看到有人在上面[装了一个Leopard](http://bbs.pcbeta.com/archiver/tid-625920.html),于是手痒也在上面装了一个，如果有人也有这台笔记本也可以试试看，不过我装好的系统还是有问题：

1，触摸屏一碰就乱跑，我直接禁用就好了，反正也不喜欢用。
2，关机和重启不断电，每天关机的次数也有限，也就无所谓了。
3，声音控制不了，只能在具体的应用程序里控制。
其实问题还是挺多的，不过基本上的使用我觉得还是没什么问题的，触摸屏的问题比较烦人，还好我不喜欢用触摸屏。如果有人也想尝试可以试试看。我用的安装文件是 Snow_Leopard_10.6.1-10.6.2_SSE2_SSE3_Intel_AMD_by_Hazard.iso, 至于安装方法[PCbeta](http://bbs.pcbeta.com/forum-185-1.html)上有很多的教程可以参考，摸索一下大概都没有什么问题，安装的时候一定要做好备份，因为我到目前已经因为装Mac OS X丢失了不下10次数据了，大部分情况下都是分区被合并。所以一定要小心一点。
