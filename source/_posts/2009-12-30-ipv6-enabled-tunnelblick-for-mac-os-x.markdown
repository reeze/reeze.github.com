---
date: '2009-12-30 13:20:15'
layout: post
slug: ipv6-enabled-tunnelblick-for-mac-os-x
status: publish
title: 支持IPv6的Tunnelblick For Mac OS X OpenVPN客户端
wordpress_id: '147'
categories:
- Mac/iOS
tags:
- MacOSX
- Tools
- VPN
---

在[yegle](http://yegle.net)那里买了OpenVPN服务 ，用着还挺不错，我也在教育网，所以只能使用支持IPv6版的客户端，openvpn默认不支持IPv6,不过yegle提供了相应的[IPv6补丁](http://github.com/jjo/openvpn-ipv6)，在Linux下以及Windows下使用的都挺好，最近又在折腾电脑，装了个Snow Leopard，基本没什么问题了，除了我的ATI 2600 XT硬件加速暂时无解外其他的都挺爽，使用了yegle推荐的[Tunnelblick](http://code.google.com/p/tunnelblick/downloads/list), 总是连接不上，它提示让我查看日志，但是根本就找不到地方看日志，直接cd 到 Tunnelblick的包里面直接执行openvpn命令，提示不支持udp6 ，又是不支持 Ipv6，本想直接自己编译一个openvpn，但是想想那样就太不clean了，编译成app的话可以通用，还能共享给有需要的人多好啊，去google code checkout了一份代码，直接编译，错误百出，我是在Snow Leopard上编译的，仔细查看原来需要MacOSX10.4u的SDK 重新安装10.4的SDK还是未果，后来发现SDK的安装目录居然不一样。。自己手动拷贝过去也不行。。 切换到傲Leopard下安装，折腾了好久终于编译成功，废话太多了。呵呵，共享出来给需要的人吧：）

猛击这个连接 下载[http://code.google.com/p/tunnelblick-ipv6/downloads/list](http://code.google.com/p/tunnelblick-ipv6/downloads/list)
可惜的是yegle不再提供IPv6用户的续费了，不过我到期之后差不多也要从学校滚蛋了。
我提供的这个版本的tunnelblick的配置文件位置是  ~/Library/openvpn  最新版的配置放在 ~/Library/Application Support/Tunnelblick/Configuration目录里面。

马上2009年就要过去了。最近很久都没有更新日志了，其实之间也想写一些东西，但是都丢在草稿箱里没写完。论文还有很多没有写完，马上就要交了，要抵制住诱惑乖乖写论文。
