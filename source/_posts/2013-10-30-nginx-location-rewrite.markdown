---
layout: post
title: "nginx中location的匹配和rewrite"
date: 2013-10-30 23:59
comments: true
categories: 
---

最近在线上进行nginx规则的调整的时候遇到一个问题，发现在location匹配时候可能会踩到的一个坑。

location在匹配规则的时候匹配的是归一化之后的URL，比如多个斜杠或者URL中带".", ".."的都会被
归一化。

而在内部rewrite的时候新的URL地址是不会再次被归一化的。这种不一致如果没有留意可能会遇到问题。

比如：

	````
	if ($request_uri ~ "/api") {
		rewrite (.*) /newapi/$1;	# 斜杠多余了
	}

	location /newapi/api {
		set $testapi 1;
	}

	location /newapi {
		# ...	
	}
	````

对于上面的配置中，rewrite的时候不小心多写了个斜杠，对于这个配置，
如果用地址：/api访问的话  /newapi/api 这个location是不能被匹配的。
而用地址`/newapi//api`直接访问是可以匹配到/newapi/api这个location的。

**本质上是因为用户直接访问的URL会先归一化处理，而rewrite之后是不会处理的。**

具体见文档： <http://nginx.org/en/docs/http/ngx_http_core_module.html>
