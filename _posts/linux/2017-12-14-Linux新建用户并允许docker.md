---
layout: blog
istop: true
jishu: true
background-image: http://upload-images.jianshu.io/upload_images/2830896-69dc8891bfc3cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700
category: docker
title: Linux新建用户并允许docker
tags:
- docker
date:   2017-12-14 09:16:54
---

# 创建用户

1.作用
useradd或adduser命令用来建立用户帐号和创建用户的起始目录，使用权限是超级用户。

　　2.格式

```
useradd [-d home] [-s shell] [-c comment] [-m [-k template]] [-f inactive] [-e expire ] [-p passwd] [-r] name

```

3.主要参数

　　-c：加上备注文字，备注文字保存在passwd的备注栏中。

　　-d：指定用户登入时的主目录，替换系统默认值/home/<用户名>

　　-D：变更预设值。

　　-e：指定账号的失效日期，日期格式为MM/DD/YY，例如06/30/12。缺省表示永久有效。

　　-f：指定在密码过期后多少天即关闭该账号。如果为0账号立即被停用；如果为-1则账号一直可用。默认值为-1.

　　-g：指定用户所属的群组。值可以使组名也可以是GID。用户组必须已经存在的，期默认值为100，即users。

　　-G：指定用户所属的附加群组。

　　-m：自动建立用户的登入目录。

　　-M：不要自动建立用户的登入目录。

　　-n：取消建立以用户名称为名的群组。

　　-r：建立系统账号。

　　-s：指定用户登入后所使用的shell。默认值为/bin/bash。

　　-u：指定用户ID号。该值在系统中必须是唯一的。0~499默认是保留给系统用户账号使用的，所以该值必须大于499。

4.说明

　　useradd可用来建立用户账号，它和adduser命令是相同的。账号建好之后，再用passwd设定账号的密码。使用useradd命令所建立的账号，实际上是保存在/etc/passwd文本文件中。

5.案例

```

　#useradd -u 544 -d /usr/testuser1  -g users -m  testuser1

```

加-m 如果主目录不存在则自动创建

6.设置用户的密码

```
passwd ${username}

# 输入密码
```


# 创建docker用户组并把用户加入组

1、 首先创建docker用户组，如果docker用户组存在可以忽略

```
sudo groupadd docker

```

2、把用户添加进docker组中

```
sudo gpasswd -a ${USER} docker
```

3、重启docker

```
sudo service docker restart

```

4、如果普通用户执行docker命令，如果提示get …… dial unix /var/run/docker.sock权限不够，则修改/var/run/docker.sock权限
使用root用户执行如下命令，即可

```
sudo chmod a+rw /var/run/docker.sock

```

我的官网
![我的博客](http://upload-images.jianshu.io/upload_images/2830896-69dc8891bfc3cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr>
# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# **[我的开源项目spring boot 搭建的一个企业级快速开发脚手架](https://gitee.com/jamen/slife)**
![1.jpg](http://upload-images.jianshu.io/upload_images/2830896-66de965f818533c5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



