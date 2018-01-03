---
layout: blog
istop: true
jishu: true
background-image: https://upload-images.jianshu.io/upload_images/2830896-69dc8891bfc3cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700
background: purple
title: "使用Docker快速安装部署Redis"
date: 2018-01-03 09:25
category: redis
tags:
- redis
- Docker
---


# 什么是redis？

感觉没必要介绍了，可以看我另外两篇博客
[redis系列（1）之安装和集群部署](http://blog.csdn.net/chenjianandiyi/article/details/60575492)

[ redis系列（2）基础知识](http://blog.csdn.net/chenjianandiyi/article/details/50516109)

# Docker快速安装部署

一般先 pull 镜像

```
docker pull redis
```

然后是运行镜像

```
$ docker run --name some-redis -d redis

```
到这里 一个能提高服务的redis已经部署成功。这里默认暴露了6379 端口。


# 配置持久化方式启动

```
$ docker run --name some-redis -d redis redis-server --appendonly yes

```
当然也可以把持久化的数据存到物理机
-v <宿主机目录>:<容器目录>

最后的命令为

```
$ docker run --name some-redis -v /docker/host/dir:/data -d redis redis-server --appendonly yes

```


# --link关联容器
我们在使用Docker的时候，经常可能需要连接到其他的容器，比如：web服务需要连接数据库。按照往常的做法，需要先启动数据库的容器，映射出端口来，然后配置好客户端的容器，再去访问。其实针对这种场景，Docker提供了--link 参数来满足。
--link=container_name or id:name

比如你的应用服务需要使用redis 可以这么启动。

```
$ docker run --name some-app --link some-redis:redis -d application-that-uses-redis

```
或者  or via redis-cli

```
$ docker run -it --link some-redis:redis --rm redis redis-cli -h redis -p 6379

```

不过我不喜欢用这样方式连接容器，应为如果容器多 了  能把你 link成 懵逼

我喜欢用创建一个 内网的方式

# 创建一个网段来连接容器

创建一个网络

```
docker network create -d bridge --subnet 172.25.0.0/16 hydra_work
```

其他容器加入改网络

```

docker build -t hydra/eureka:1.0 .

docker run -d --network=hydra_work --name h-eureka  -p 7000:7000 hydra/eureka:1.0
```


# 自定义 redis.conf

```
$ docker run -v /myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf --name myredis redis redis-server /usr/local/etc/redis/redis.conf

```


# 最快的安装方式

直接使用 阿里云或者腾讯云的 云redis就好了，功能齐全并且强大稳定。我们现在的项目也是使用了他们的。
两者的 优惠券
[阿里云Redis https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)


[腾讯云的优惠购买地址
https://cloud.tencent.com/redirect.php?redirect=1005&cps_key=1cdaea7b77fe67188b187bce55796594](https://cloud.tencent.com/redirect.php?redirect=1005&cps_key=1cdaea7b77fe67188b187bce55796594)


我的官网
![我的博客](http://upload-images.jianshu.io/upload_images/2830896-69dc8891bfc3cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>
# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# **[我的开源项目spring boot 搭建的一个企业级快速开发脚手架](https://gitee.com/jamen/slife)**
![1.jpg](http://upload-images.jianshu.io/upload_images/2830896-66de965f818533c5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



