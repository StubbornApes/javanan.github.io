---
layout: blog
istop: true
jishu: true
background-image: http://upload-images.jianshu.io/upload_images/2830896-4e04eba5d828ae64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700
category: shadowsocks
title: 手把手教你阿里云上安装shadowsocks server
tags:
- shadowsocks
date:   2017-11-23 09:16:54
---

偶然发现阿里云全民云计算优惠力度挺大，1核/1G/1M带宽的ecs3年只需800块。而且自从19 big之后，北京这边用蓝灯变得很不流畅，正好阿里云有美国和香港节点，于是果断买之。然后搭建shadowsocks。

### 下载安装shadowsocks-libev

笔者选择的是centOS(LAMP)镜像，配置好并运行实例后用root远程登录，使用如下命令下载[shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev#install-from-repository-1)的repo到yum目录。其他系统请到[copr](https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/)下载相应repo。

```
cd /etc/yum.repos.d/
curl -O https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
```

下载完成后就可以用yum直接安装了

```
yum install shadowsocks-libev
```

### 配置server

修改/etc/shadowsocks-libev/config.json，将内容改为

```
{
    "server":"0.0.0.0",
    "server_port":18888,
    "password":"你自己的密码",
    "timeout":60,
    "method":"aes-256-cfb"
}
```

配置项及含义

| 配置项         | 含义           |
| ----------- | ------------ |
| server      | 改为全0表示接受外网访问 |
| server_port | 服务端监听端口      |
| password    | 连接口令         |
| method      | 口令加密方式       |

### 启动

使用命令`systemctl enable shadowsocks-libev`将ss加入开机启动，之后执行`systemctl start shadowsocks-libev`启动。


命令`systemctl status shadowsocks-libev`可以查看ss服务的状态。

使用命令`systemctl enable shadowsocks-libev`将ss加入开机启动，之后执行`systemctl start shadowsocks-libev`启动。
命令`systemctl status shadowsocks-libev`可以查看ss服务的状态。
[![ss状\](http://upload-images.jianshu.io/upload_images/2830896-066347ae56b20bfe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://www.wisedream.net/res/img/tricks/ss_running.png)ss状态

### 配置防火墙

由于有防火墙限制，ss启动后仍无法连接，需要在控制台开放相应的端口。

登录阿里云控制台，在”网络安全-安全组”中点击配置规则
 ![image](http://upload-images.jianshu.io/upload_images/2830896-fad84aacddbe8f90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![image](http://upload-images.jianshu.io/upload_images/2830896-4e04eba5d828ae64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image](http://upload-images.jianshu.io/upload_images/2830896-84ee933210e900d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后点击”添加安全组规则“,设置相应的参数
![image](http://upload-images.jianshu.io/upload_images/2830896-bee34d551631bac0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](http://upload-images.jianshu.io/upload_images/2830896-ed8ffacacbe4b3fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中端口范围设置为config.json中的server_port值。
之后通过telnet测试连接成功。

### 客户端连接


![image](http://upload-images.jianshu.io/upload_images/2830896-a71b4d7c7cbeaacb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
为浏览器设置socks5代理即可



如果有想买阿里云产品的同学可以先领抽奖券，[点击领券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld), 然后在下单的时候使用幸运券，支付成功后可以在[这个页面](https://m.aliyun.com/markets/aliyun/lucky1?spm=5176.doc50572.2.4.j5izTi)抽奖。选择节点的时候记得选择美国或香港节点。

原文作者：[圣僧]


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


