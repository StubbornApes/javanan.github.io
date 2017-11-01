---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://static.oschina.net/uploads/space/2017/0925/105429_Vygk_3665821.png
title: Spring Cloud连载（2）搭建开发环境
tags:
- 阿里云
- 双十一
- 阿里云优惠券
- 阿里云优惠
- ECS
- RDS
- OSS
- 建站
- 云服务器
- Spring Cloud
date:   2017-11-01 11:15:54
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**

原文作者：杨大仙的程序空间


# 2 开发环境搭建

        工欲善其事，必先利其器。在讲述本书的技术内容前，先将开发环境搭建好，本书所涉及基础环境将在本章准备，包括Eclipse、Maven等。如果读者对Maven、Eclipse、Spring Boot等项目较为熟悉，可以直接跳过本章的相关章节。

        笔者建议读者在查阅本书过程中，使用与本书相同的工具以及版本。本章使用的Java版本为1.8，图2-1为“java –version”命令的输出，Java安装与配置较为简单，本书不再赘述。

![](https://static.oschina.net/uploads/space/2017/0925/105429_Vygk_3665821.png)

图2-1 Java版本

        注：本书全部的案例均在Windows7下开发和运行。

## 2.1 安装与配置Maven

### 2.1.1 关于Maven

        Maven是Apache下的一个开源项目，用于项目的构建。使用Maven可以对项目的依赖包进行管理，支持构建脚本的继承，对于一些模块（子项目）较多的项目来说，Maven是更好的选择，子项目可以继承父项目的构建脚本，减少了构建脚本的冗余。

        除此之外，Maven本身的插件机制让其更加强大和灵活，使用者可以配置各种Maven插件来完成自己的事，如果感觉官方或者第三方提供的Maven插件不够用，还可以自行编写符合自己要求的Maven插件。Maven为使用者提供了一个统一的依赖仓库，各种开源项目的发布包可以在上面找到，在一间公司或者一个项目组内部，甚至可以搭建私有的Maven仓库，将自己项目的包放到私有仓库中，供其他项目组或者开发者使用。

        Maven的众多特性中，最为重要的是它对依赖包的管理，Maven将项目所使用的依赖包的信息放到pom.xml的dependencies节点。例如我们需要使用spring-core模块的jar包，只需在pom.xml配置该模块的依赖信息，Maven会自动将spring-beans等模块引入到我们项目的环境变量中。Spring Cloud项目基于Spring Boot搭建，正是由于依赖管理的特性，使得Maven与Spring Boot更加相得益彰，可以让我们更快速的搭建一个可用的开发环境。

### 2.1.2 下载与安装Maven

        本书所使用的Maven版本为3.5，可以到Maven官方网站下载：[http://maven.apache.org/](http://maven.apache.org/)。下载并解压后得到apache-maven-3.5.0目录，将主目录下的的bin目录加入到系统的环境变量中，如图2-2所示。

![](https://static.oschina.net/uploads/space/2017/0925/105520_v0kz_3665821.png)

图2-2 修改环境变量

        配置完后，打开cmd命令行，输入“mvn –v”，可以看到输出的Maven版本信息。Maven下载的依赖包会存放到本地仓库中，默认路径为：C:\Users\用户名\.m2\repository。

### 2.1.3 配置远程仓库

        如果不进行仓库配置，默认情况下，会到apache官方的仓库下载依赖包，由于Apache官方的仓库位于国外，下载速度较慢，会降低开发效率，笔者建议使用国内的Maven仓库或者搭建自己的私服，本书重点不是Maven，因此直接使用了由阿里云提供的Maven仓库。修改apache-maven-3.5.0/conf目录下的setting.xml，在mirrors节点下加入以下配置：

```
<mirror> 

    <id>alimaven</id> 

    <name>aliyun maven</name> 

    <url>http://maven.aliyun.com/nexus/content/groups/public/</url> 

    <mirrorOf>central</mirrorOf>         

</mirror>
```

        配置完后，以后在使用过程中，Maven会先到阿里云的仓库中下载依赖包。另外，需要注意的是，本书的大部分案例，都没有使用Maven的继承特性，每一个Maven项目都可以独立引入。

## 2.2 安装Eclipse

### 2.2.1 Eclipse版本

        本书使用Eclipse作为开发工具，使用版本为Luna（4.4），大家可以从以下的地址得到该版本的Eclipse：[http://www.eclipse.org/downloads/packages/eclipse-ide-java-developers/lunasr2](http://www.eclipse.org/downloads/packages/eclipse-ide-java-developers/lunasr2)，也可以在本书所附的soft目录下找到该版本的Eclipse。目前Eclipse已经发展到4.7版本，本书主要在Eclipse中使用Maven插件。

### 2.2.2 在Eclipse配置Maven

        Luna版本的Eclipse自带了Maven插件，默认使用的是Maven3.2，由于我们前面安装的是Maven3.5版本，因此需要在Eclipse中指定Maven版本以及配置文件。指定Maven的配置如图2-3所示，指定配置文件如图2-4所示。

![](https://static.oschina.net/uploads/space/2017/0925/105603_2M9p_3665821.png)

图2-3 Eclipse指定Maven版本

![](https://static.oschina.net/uploads/space/2017/0925/105614_1waO_3665821.png)

图2-4 指定Maven配置文件

        注意：本书的案例，如无特别说明均以Maven项目的形式导入。

        如读者已经安装Eclipse、Maven等工具，可直接跳过本文。






我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>




