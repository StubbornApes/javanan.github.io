---
layout: blog
istop: true
jishu: true
category: jqueryAjax标准写法
title: jqueryAjax标准写法
tags:
- Ajax
- jquery
- 标准写法
date:   2017-10-23 09:16:54
---


```

$.ajax({
    url:"http://www.xxx",//请求的url地址
    dataType:"json",//返回的格式为json
    async:true,//请求是否异步，默认true异步，这是ajax的特性
    data:{"id":"value"},//参数值
    type:"GET",//请求的方式
    beforeSend:function(){},//请求前的处理
    success:function(req){},//请求成功的处理
    complete:function(){},//请求完成的处理
    error:function(){}//请求出错的处理
});

```