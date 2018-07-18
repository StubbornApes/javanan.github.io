---
layout: blog
istop: true
jishu: true
category: Spring Boot Admin
background-image: http://upload-images.jianshu.io/upload_images/44770-415b947617b5b89f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700
title: Spring Boot Admin 全方位监控你的微服务
tags:
- Security
- 阿里云优惠券
- 建站
- 云服务器
- Spring Cloud
- Spring Boot
- OAuth2
- 脚手架
date:   2017-12-07 11:24:54
---


spring-boot-admin，简称SBA，是一个针对spring-boot的actuator接口进行UI美化封装的监控工具。他可以：在列表中浏览所有被监控spring-boot项目的基本信息，详细的Health信息、内存信息、JVM信息、垃圾回收信息、各种配置信息（比如数据源、缓存列表和命中率）等，还可以直接修改logger的level。
官网：https://github.com/codecentric/spring-boot-admin
使用指南：http://codecentric.github.io/spring-boot-admin/1.5.0/

只需简单几步，就可以配置和使用SBA（分为监控端和被监控端）：
监控端：
1、创建项目（略）
2、引入依赖：

```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-server</artifactId>
    <version>1.5.0</version>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-server-ui</artifactId>
    <version>1.5.0</version>
</dependency>

```

3、配置文件（application.yml）配置（可选）：

```

spring:
  application:
    name: svc-monitor
  boot:
    admin:
      context-path: /sba    # 配置访问路径为：http://localhost:64000/svc-monitor/sba
server:
  port: 64000
  context-path: /svc-monitor/ #统一为访问的url加上一个前缀


```


以上配置是为了指定一个特别的访问路径。如果不这样配置，则访问路径为：http://localhost:64000

4、使用@EnableAdminServer注解激活SBA：

```
@SpringBootApplication
@EnableScheduling
@EnableAdminServer
public class SvcMonitorApplication {
    public static void main(String[] args) {
        SpringApplication.run(SvcMonitorApplication.class, args);
    }
}

```

被监控端（spring-boot项目）向监控端注册自己：
1、添加依赖：

```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>1.5.0</version>
</dependency>
```

2、配置文件（application.yml）配置:

```
spring:
  boot:
    admin:
      client:
        prefer-ip: true # 解决windows下运行时无法识别主机名的问题
      url: http://localhost:64000/svc-monitor # 向服务端注册的地址
management:
  port: 64001
  security:
    enabled: false # spring-boot 1.5.2之后严格执行安全策略，所以需要配置这个为false
info: #定义各种额外的详情给服务端显示
  app:
    name: "@project.name@" #从pom.xml中获取
    description: "@project.description@"
    version: "@project.version@"
    spring-boot-version: "@project.parent.version@"
```

3、其他配置：
如果需要显示项目版本号，需要在pom.xml中添加这个（build-info）：

```

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```

4、问题解决：
如果发现被监控端启动的时候出现InetAddress.getLocalHost() throws UnknownHostException错误，是因为没配置本机机器名和ip的对应关系。
解决方法：
编辑hosts文件：
vi /etc/hosts
添加ip和机器名的关联：192.168.0.31 host31 myhost-31

监控端和被监控端都启动后，访问：http://localhost:64000/svc-monitor/sba，就可以看到被监控服务的各种详情了。

以上是被监控端主动注册法。

还有另外一种方法是：如果被监控端已经使用了Spring Cloud向Eureka注册了服务，则可以由监控端直接去Euraka中发现并监控这个服务。此方法调试起来比较复杂，这里先不介绍了。


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

