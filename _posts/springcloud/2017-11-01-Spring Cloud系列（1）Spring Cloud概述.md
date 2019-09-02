---
layout: blog
istop: true
jishu: true
background-image: https://static.oschina.net/uploads/space/2017/0924/095718_XMPj_3665821.png
category: Spring Cloud
title: Spring Cloud系列（1）Spring Cloud概述
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
date:   2017-11-01 11:11:54
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

原文作者：杨大仙的程序空间

# **1 Spring Cloud概述**

        本文要点

             传统应用的问题

             微服务与Spring Cloud

             本书介绍

        本章将会简述Spring Cloud的功能，描述什么是Spring Cloud，它能为我们带来什么，为后面学习该框架的知识打下理论的基础。

## 1.1 传统的应用

### 1.1.1 单体应用

        在此之前，笔者所在公司开发Java程序，大都使用Struts、Spring、Hibernate（MyBatis）等技术框架，每一个项目都会发布一个单体应用。例如开发一个进销存系统，将会开发一个war包部署到Tomcat中，每一次需要开发新的模块或添加新功能时，都会在原来的基础上不断的添加。若干年后，这个war包不断的膨胀，程序员在进行调试时，服务器也可能需要启动半天，维护这个系统的效率极为低下。这样一个war包，涵盖了库存、销售、会员、报表等模块，如图1-1。

![](https://static.oschina.net/uploads/space/2017/0924/095718_XMPj_3665821.png)

图1-1 单体应用

        这样的单体应用隐患非常多，任何的一个bug，都有可能导致整个系统宕机。笔者印象最深刻的是，曾经有一客户在高峰期，导出一张销售明细报表（数据量较大），最终造成整个系统瘫痪，前台的销售人员无法售卖。维护这样一个系统，不仅效率极低，而且充满风险，项目组的各个成员惶惶不可终日，我们需要本质上的改变。

### 1.1.2 架构演进

        针对以上的单体应用的问题，我们参考SOA架构，将各个模块划分独立的服务模块（war），并且使用了数据库的读写分离，架构如图1-2。

![](https://static.oschina.net/uploads/space/2017/0924/095730_JNIO_3665821.png)

图1-2 架构演进

        各个模块之间会存在相互调用的依赖关系，例如销售模块会调用会员模块的接口，为了减少各个模块之间的耦合，我们加入了企业服务总线（ESB），各模块与ESB之间的架构如图1-3所示。

![](https://static.oschina.net/uploads/space/2017/0924/095740_2lFc_3665821.png)

图1-3 ESB

        加入ESB后，各个模块将服务发布到ESB中，它们与ESB之间使用SOAP协议进行通信。图1-2与图1-3的架构实现后，整个系统的性能有了明显的提升，各个模块的耦合度也降低了。运行了一段日子后，又出现了新的问题，由于销售终端数量的增多，销售模块明显超过其承受能力，为了保证销售前端的正常运行，我们使用了Nginx做负载均衡，请见图1-4。

![](https://static.oschina.net/uploads/space/2017/0924/095748_jOP0_3665821.png)

图1-4 使用Nginx

        随着销售模块的增多，带来了许多问题，例如管理这些模块，对于运维工程师来说，是一项艰巨的任务，一旦销售模块有所修改，他们将通宵达旦进行升级。另外，企业服务总线也有可能成为性能的瓶颈，虽然目前仍未出现该问题，但我们需要未雨绸缪。

### 1.1.3 架构要求

        从前面的架构演进可知，应用中的每一个点，都有可能成为系统的问题点。随着互联网应用的普及，在大数据、高并发的环境下，我们的系统架构需要面对更为严苛的挑战，我们需要一套新的架构，它起码能满足以下要求：

              高性能：这是应用程序的基本要求。

              独立性：其中一个模块出现bug或者其他问题，不可以影响其他模块或者整个应用。

              容易扩展：应用中的每一个节点，都可以根据实际需要进行扩展。

              便于管理：对于各个模块的资源，可以轻松进行管理、升级，减少维护成本。

              状态监控与警报：对整个应用程序进行监控，当某一个节点出现问题时，能及时发出警报。

        为了能解决遇到的问题、达到以上的架构要求，我们开始研究Spring Cloud。

## 1.2 微服务与Spring Cloud

### 1.2.1 什么是微服务

        微服务一词来源Martin Fowler的“Microservices”一文，微服务是一种架构风格，将单体应用划分为小型的服务单元，微服务之间使用HTTP的API进行资源访问与操作。

        在对单体应用的划分上，微服务与前面的SOA架构有点类似，但是SOA架构侧重于将每个单体应用的服务集成到ESB上，而微服务做得更加彻底，强调将整个模块变成服务组件，微服务对模块的划分粒度可能会更细。以我们前面的销售、会员模块为例，在SOA架构中，只需要将相应的服务发布到ESB容器就可以了，而在微服务架构中，这两个模块本身，将会变为一个或多个的服务组件。SOA架构与微服务架构，请见图1-5与图1-6。

![](https://static.oschina.net/uploads/space/2017/0924/095758_YXOu_3665821.png)

图1-5 SOA架构

![](https://static.oschina.net/uploads/space/2017/0924/095810_5ep4_3665821.png)

图1-6 微服务架构

        在微服务的架构上，Martin Fowler的文章肯定了Netflix的贡献，接下来，我们了解一下Netflix OSS。

### 1.2.2 关于Netflix OSS

        Netflix是一个互联网影片提供商，在几年前，Netflix公司成立了自己的开源中心，名称为Netflix Open Source Software Center，简称Netflix OSS。这个开源组织专注于大数据、云计算方面的技术，提供了多个开源框架，这些框架包括大数据工具、构建工具、基于云平台的服务工具等。Netflix所提供的这些框架，很好的遵循微服务所推崇的理念，实现了去中心化的服务管理、服务容错等机制。

### 1.2.3 Spring Cloud与Netflix

        Spring Cloud并不是一个具体的框架，大家可以把它理解为一个工具箱，它提供的各类工具，可以帮助我们快速的构建分布式系统。

        Spring Cloud的各个项目基于Spring Boot，将Netflix的多个框架进行封装，并且通过自动配置的方式将这些框架绑定到Spring的环境中，从而简化了这些框架的使用。由于Spring Boot的简便，使得我们在使用Spring Cloud时，很容易的将Netflix各个框架整合进我们的项目中。Spring Cloud下的“Spring Cloud Netflix”模块，主要封装了Netflix的以下项目：

              Eureka：基于REST服务的分布式中间件，主要用于服务管理。

              Hystrix：容错框架，通过添加延迟阀值以及容错的逻辑，来帮助我们控制分布式系统间组件的交互。

              Feign：一个REST客户端，目的是为了简化Web Service客户端的开发

              Ribbon：负载均衡框架，在微服务集群中为各个客户端的通信提供支持，它主要实现中间层应用程序的负载均衡

              Zuul：为微服务集群提供过代理、过滤、路由等功能。

### 1.2.4 Spring Cloud的主要模块

        除了Spring Cloud Netflix模块外，Spring Cloud还包括以下几个重要的模块：

              Spring Cloud Config：为分布式系统提供了配置服务器和配置客户端，通过对它们的配置，可以很好的管理集群中的配置文件。

              Spring Cloud Sleuth：服务跟踪框架，可以与Zipkin、Apache HTrace和ELK等数据分析、服务跟踪系统进行整合，为服务跟踪、解决问题提供了便利。

              Spring Cloud Stream：用于构建消息驱动微服务的框架，该框架在Spring Boot的基础上，整合了“Spring Integration”来连接消息代理中间件。

              Spring Cloud Bus：连接RabbitMQ、Kafka等消息代理的集群消息总线。

## 1.3 本章小结

        本章的1.1小节，对传统的单体应用、SOA架构做了一个简单的总结，在此过程中分析我们所遇到的问题。在1.2小节，简单介绍了微服务与Spring Cloud。接下来，我们正式开始讲述本书的知识点。







我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>




