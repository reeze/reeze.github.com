---
date: '2010-11-28 22:10:23'
layout: post
slug: doubanshow-for-wordpress
status: publish
title: '[Plugin]豆瓣秀 For WordPress'
wordpress_id: '1419'
categories:
- PHP
tags:
- plugin
- widget
- Wordpress
- 插件
- 豆瓣秀
---

我喜欢在博客中显示我在豆瓣上的一些信息，比如想看哪些书哪些电影神马的。刚开始用的是 Robin的 [http://www.robb.com.cn/plugins/](http://www.robb.com.cn/plugins) 的 WP-DoubanShow插件，他用的是豆瓣API。这个插件需要手动修改主题模板文件。刚开始用的挺好，修改就修改吧。不过后来换过几次主题后发现每次修改模板文件还挺“脏”的。遂问robin能否修改成widget的方式，他说官方推出了一个豆瓣秀功能[http://www.douban.com/service/badgemaker](http://www.douban.com/service/badgemaker)， 所以不继续维护了。看过官方的说明。发现要在Wordpress中用也只能手动修改模板，不过官方提供了一个生成js的设置项，可以根据需要生成相应的脚本，选项也还算简单。 在网上搜了一番，没有给wordpress用的插件，所以自己写了一个，设置项和官方的一样。下面简单的说明一下：







  1. 第一步：下载插件文件： [DoubanShow.zip](/uploads/2010/11/DoubanShow.zip), 或者在管理界面中添加插件, 搜索douban即可看到"豆瓣秀For Wordpress" 选择安装, 如果这样的话,下面的上传步骤就不需要了


  2. 第二步：上传安装。 后台管理的  插件 -> 添加插件 -> 上传中上传下载的文件。[![](/uploads/2010/11/upload.png)](/uploads/2010/11/upload.png)

[![](/uploads/2010/11/active2.png)](/uploads/2010/11/active.png)
上传完后记得“启用插件"

  3.


第三步：在后台管理的 外观 -> 小工具 中选择”豆瓣秀“ 拖到右侧你想放置的位置。 然后点击拖过去的豆瓣秀箭头。出现如下设置：


[![](/uploads/2010/11/settings.png)](/uploads/2010/11/settings.png)


标题默认为空，就是不显示标题。也可以设置成你想要的标题。然后要设置好你的豆瓣ID，记住不是豆瓣的登录用户名。 设置好以后。去你的页面看看效果吧，也可以看我博客页面右下角。
