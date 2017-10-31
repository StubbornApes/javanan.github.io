---
layout: blog
background-image: http://upload-images.jianshu.io/upload_images/2830896-2fc331bb3ffff6ef?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
category: 插件
istop: true
jishu: true
title: 使用Docker搭建SonarQube检测代码质量
tags:
- Docker
- SonarQube
- 检测代码质量
date:   2017-10-30 18:25:54
---

# SonarQube 简介

> Sonar是一个用于代码质量管理的开源平台，用于管理源代码的质量，可以从七个维度检测代码质量
> 可以通过插件形式，支持包括java,C#,C/C++,PL/SQL,Cobol,JavaScrip,Groovy等等二十几种编程语言的代码质量管理与检测。

本文记录怎么使用docker安装SonarQube和使用SonarQube检测自己的代码。



## 1、获取 postgresql 的镜像

```
$ docker pull postgres
```

## 2、启动 postgresql

```
$ docker run --name postgresqldb -e POSTGRES_USER=root -e POSTGRES_PASSWORD=root -d postgres

# 其中 postgresqldb  为容器名称  POSTGRES_USER POSTGRES_PASSWORD 指定postgresql的用户名密码
```


## 3、获取 sonarqube 的镜像

```
$ docker pull sonarqube
```

## 4、启动 sonarqube

```
$ docker run --name sq --link postgresqldb -e SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonar -p 9000:9000 -d sonarqube

# 其中--link postgresqldb 是指和 postgresqldb 容器连接通讯， 用网关的方式也可以(可以看我另一篇docker多容器设置为一个网络的文章）
```

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-d1ce1370bd4d046f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，平台搭建完毕，接下来就是检测直接的代码了。

# 5、访问SonarQube

浏览器直接输入 服务器地址和9000端口即可

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-c9be830ad6326ef8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
账户密码都是 admin admin
```

# 6、安装jdk和maven
为了编译，这里需要安装jdk和maven
安装方式可以看我的另一篇博客
centos7安装JDK8<http://blog.csdn.net/chenjianandiyi/article/details/78069992>
centos7 安装maven<http://www.jianshu.com/p/ea2554ac6ad4>


# 7、检测maven项目

执行
上图中 copy的代码
或者

```
mvn sonar:sonar
```

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-d42c5cd989c0a6cb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-b32ce070e2576fc0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 8、查看检测结果
上图表示 编译成功，接下来就可以在浏览器上看检测的结果了

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-2fc331bb3ffff6ef?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-3a8c8ce8be7b3ea4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2830896-fa9700d31ae45fb1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 10、本文检测的项目叫 SLife 是一个使用spring boot、shiro、SiteMesh、Mysql、Redis、Boostrap（inspinia）、Mybatis（Mybatis plus）搭建的一个企业级快速开发项目。可以到我的github或者码云中获取源代码，

#  **[小福利：点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**


我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>