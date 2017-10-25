---
layout: blog
banana: true
category: OncePerRequestFilter
title: OncePerRequestFilter
tags:
- OncePerRequestFilter
date: 2017-10-25 10:25:54
---


OncePerRequestFilter  顾名思义---》一次请求仅仅经过一个的filter，而不需要重复执行。

Filter不都是仅仅经过一次的吗？
不是的！不然就不会有这个类了。。
此方式是为了兼容不同的web container，特意而为之（jsr168），也就是说并不是所有的container都像我们期望的只过滤一次，servlet版本不同，表现也不同。
如，servlet2.3与servlet2.4也有一定差异
 写道
在servlet-2.3中，Filter会过滤一切请求，包括服务器内部使用forward转发请求和<%@ include file="/index.jsp"%>的情况。
到了servlet-2.4中Filter默认下只拦截外部提交的请求，forward和include这些内部转发都不会被过滤，但是有时候我们需要 forward的时候也用到Filter。

 因此，为了兼容各种不同的运行环境和版本，默认filter继承OncePerRequestFilter是一个比较稳妥的选择。
