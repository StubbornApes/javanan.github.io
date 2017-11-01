---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://static.oschina.net/uploads/space/2017/0925/211656_jMKn_3665821.png
title: Spring Cloud连载（4）第一个Eureka程序
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
date:   2017-11-01 11:25:54
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**

原文作者：杨大仙的程序空间



# 4 微服务发布与调用

      要点

             认识Eureka框架

             运行Eureka服务器

             发布微服务

             调用微服务

        本章将讲述Spring Cloud中Eureka的使用，包括在Eureka服务器上发布、调用微服务，Eureka的配置以及Eureka集群等内容。

## 4.1 Eureka介绍

        Spring Cloud集成了Netflix OSS的多个项目，形成了spring-cloud-netflix项目，该项目包含了多个子模块，这些子模块对集成的Netflix旗下框架进行了封装，本小节将讲述其中一个较为重要的服务管理框架：Eureka。

### 4.1.1 关于Eureka

        Eureka提供基于REST的服务，在集群中主要用于服务管理。Eureka提供了基于Java语言的客户端组件，客户端组件实现了负载均衡的功能，为业务组件的集群部署创造了条件。使用该框架，可以将业务组件注册到Eureka容器中，这些业务组件可进行集群部署，Eureka主要维护这些服务的列表并自动检查它们的状态。

### 4.1.2 Eureka架构

        一个简单的Eureka集群，需要有一个Eureka服务器、若干个服务提供者。我们可以将业务组件注册到Eureka服务器中，其他客户端组件可以向服务器获取服务并且进行远程调用。图3-1为Eureka的架构图。

![](https://static.oschina.net/uploads/space/2017/0927/114342_7m9L_3665821.png)

图3-1 Eureka架构图

        图3-1中有两个服务器，服务器支持集群部署，每个服务器也可以作为对方服务器的客户端进行相互注册与复制。图3-1的三个Eureka客户端，两个用于发布服务，其中一个用于调用服务。不管服务器还是客户端，都可以部署多个实例，如此一来，就很容易构建高可用的服务集群。

### 4.1.3服务器端

        对于注册到服务器端的服务组件，Eureka服务器并没有提供后台的存储，这些注册的服务实例被保存在内存的注册中心，它们通过心跳来保持其最新状态，这些操作都可以在内存中完成。客户端存在着相同的机制，同样在内存中保存了注册表信息，这样的机制提升了Eureka组件的性能，每次服务的请求都不必经过服务器端的注册中心。

### 4.1.4服务提供者

        作为Eureka客户端存在的服务提供者，主要进行以下工作：第一、向服务器注册服务；第二、发送心跳给服务器；第三、向服务器端获取注册列表。当客户端注册到服务器时，它将会提供一些关于它自己的信息给服务器端，例如自己的主机、端口、健康检测连接等。

### 4.1.5服务调用者

        对于发布到Eureka服务器的服务，使用调用者可对其进行服务查找与调用，服务调用者也是作为客户端存在，但其职责主要是发现与调用服务。在实际情况中，有可能出现本身既是服务提供者，也是服务调用者的情况，例如传统的企业应用三层架构中，服务层会调用数据访问层的接口进行数据操作，它本身也会提供服务给控制层使用。

        本小节对Eureka作了介绍，读者大概了解Eureka架构及各个角色的作用即可，说一千道一万，还不如来个案例实际，下一小节将以一个案例来展示Eureka的作用。

## 4.2 第一个Eureka应用

        本小节将编写一个Hello Wrold小程序，来演示Eureka作用，程序中将会包含服务器、服务提供者以及服务调用者。

### 4.2.1 构建服务器

        先创建一个名称为first-ek-server的Maven项目作为服务器，在pom.xml文件中加入Spring Cloud的依赖，如代码清单3-1所示。

        代码清单3-1：codes\03\3.2\first-ek-server\pom.xml

```
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
	</dependencies>
```

        加入的spring-cloud-starter-eureka-server，会自动引入spring-boot-starter-web，因此只需要加入该依赖，我们的项目就具有Web容器的功能。接下来，编写一个最简单的启动类，启动我们的Eureka服务器，启动类如代码清单3-2所示。

        代码清单3-2：codes\03\3.2\first-ek-server\src\main\java\org\crazyit\cloud\FirstServer.java

```
package org.crazyit.cloud;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class FirstServer {

	public static void main(String[] args) {
		new SpringApplicationBuilder(FirstServer.class).run(args);
	}

}
```

        启动类几乎与前面章节的Spring Boot项目一致，只是加入了@EnableEurekaServer，声明这是一个Eureka服务器。直接运行FirstServer即可启动Eureka服务器，需要注意的是，本例中并没有配置服务器端口，因此默认端口为8080，我们将端口配置为8761，在src/main/resources目录下创建application.yml配置文件，内容如下：

```
server:
  port: 8761
```

        本书的大部分案例使用yml文件进行配置，以上的配置片断声明服务器的HTTP端口为8761，该配置是Spring Boot的公共配置，Sprin Boot有近千个公共配置，如想了解这些配置，可参看Spring Boot的文档。运行FirstServer的main方法后，可以看到控制台输出如下：

```
2017-08-04 15:35:58.900  INFO 4028 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8761 (http)

2017-08-04 15:35:58.901  INFO 4028 --- [           main] .s.c.n.e.s.EurekaAutoServiceRegistration : Updating port to 8761

2017-08-04 15:35:58.906  INFO 4028 --- [           main] org.crazyit.cloud.FirstServer            : Started FirstServer in 12.361 seconds (JVM running for 12.891)

2017-08-04 15:35:59.488  INFO 4028 --- [nio-8761-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
```

        在启动过程中会出现部分异常信息，暂时不需要进行处理，将在后面章节讲解如何避免出现这些异常。成功启动后，打开浏览器，输入：[http://localhost:8761](http://localhost:8761/)，可以看到Eureka服务器控制台，如图3-2所示。

![](https://static.oschina.net/uploads/space/2017/1010/193331_XNPm_3665821.png)

图3-2 Eureka控制台

        在图3-2的下方，可以看到服务的实例列表，目前我们并没有注册服务，因此列表为空。

### 4.2.2 服务器注册开关

        在启动Eureka服务器时，会在控制台看到以下两个异常信息：

```
java.net.ConnectException: Connection refused: connect  com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
```

        这是由于在服务器启动时，服务器会把自己当作一个客户端，注册去Eureka服务器，并且会到Eureka服务器抓取注册信息，它自己本身只是一个服务器，而不是服务的提供者（客户端），因此可以修改application.yml文件，修改以下两个配置：

```
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
```

        以上配置中的eureka.client.registerWithEureka属性，声明是否将自己的信息注册到Eureka服务器，默认值为true。属性eureka.client.fetchRegistry则表示，是否到Eureka服务器中抓取注册信息。将这两个属性设置为false，则启动时不会出现异常信息。

### 4.2.3 编写服务提供者

        在前面搭建环境章节，我们使用了Spring Boot来建立了一个简单的Web工程，并且在里面编写了一个REST服务，本例中的服务提供者，与该案例类似。建立名称为“first-ek-service-invoker”的项目，pom.xml中加入依赖，如代码清单3-2所示。

代码清单3-2：codes\03\3.2\first-ek-service-provider\pom.xml

```
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
	</dependencies>
```

        在src/main/resources目录中建立application.yml配置文件，文件内容如代码清单3-3所示。

        代码清单3-3：codes\03\3.2\first-ek-service-provider\src\main\resources\application.yml

```
spring:
  application:
    name: first-service-provider
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

        以上配置中，将应用名称配置为“first-service-provider”，该服务将会被注册到端口为8761的Ereka服务器，也就是本小节前面所构建的服务器。另外，还使用了eureka.instance.hostname来配置该服务实例的主机名称。编写一个Controller类，并提供一个最简单的REST服务，如代码清单3-4。

        代码清单3-4：

        codes\03\3.2\first-ek-service-provider\src\main\java\org\crazyit\cloud\FirstController.java

```
package org.crazyit.cloud;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FirstController {

	@RequestMapping(value = "/person/{personId}", method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public Person findPerson(@PathVariable("personId") Integer personId) {
		Person person = new Person(personId, "Crazyit", 30);
		return person;
	}
}
```

        编写启动类，请见代码清单3-5。

        代码清单3-5：

codes\03\3.2\first-ek-service-provider\src\main\java\org\crazyit\cloud\FirstServiceProvider.java

```
package org.crazyit.cloud;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class FirstServiceProvider {

	public static void main(String[] args) {
		new SpringApplicationBuilder(FirstServiceProvider.class).run(args);
	}
}
```

        在启动类中，使用了@EnableEurekaClient注解，声明该应用是一个Eureka客户端。配置完成后，运行服务器项目“first-ek-server”的启动类FirstServer，再运行代码清单3-5的FirstServiceProvider，的浏览器中访问Eureka：[http://localhost:8761/](http://localhost:8761/)，可以看到服务列表如图3-3所示。

![](https://static.oschina.net/uploads/space/2017/0925/211640_AS4J_3665821.png)

图3-3 查看发布的服务

        如图3-3，可以看到当前注册的服务列表，只有我们编写的“first-service-provider”。服务注册成功后，接下来编写服务调用者。

### 4.2.4 编写服务调用者

        服务被注册、发布到Eureka服务器后，就需要有程序去发现它，并且进行调用。此处所说的调用者，是指同样注册到Eureka的客户端，来调用其他客户端发布的服务，简单的说，就是Eureak内部调用。由于同一个服务，可能会部署多个实例，调用过程可能涉及负载均衡、服务器查找等问题，Netflix的项目已经帮我们解决，并且Spring Cloud已经封装了一次，我们仅需编写少量代码，就可以实现服务调用。

    新建名称为“first-ek-service-invoker”的项目，在pom.xml文件中加入依赖，如代码清单3-6所示。

    代码清单3-6：codes\03\3.2\first-ek-service-invoker\pom.xml

```
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>
	</dependencies>
```

        建立配置文件application.yml，内容如代码清单3-7。

        代码清单3-7：codes\03\3.2\first-ek-service-invoker\src\main\resources\application.yml

```
server:
  port: 9000
spring:
  application:
    name: first-service-invoker
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

        在配置文件中，配置了应用名称为“first-service-invoker”，这个调用者的访问端口为9000，需要注意的，这个调用本身也可以对外提供服务。与提供者一样，使用eureka的配置，将调用者注册到“first-ek-server”上面。下面编写一个控制器，让调用者对外提供一个测试的服务，代码清单3-8为控制器的代码实现。

        代码清单3-8：

        codes\03\3.2\first-ek-service-invoker\src\main\java\org\crazyit\cloud\InvokerController.java

```
package org.crazyit.cloud;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@Configuration
public class InvokerController {

	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	@RequestMapping(value = "/router", method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public String router() {
		RestTemplate restTpl = getRestTemplate();
		// 根据应用名称调用服务
		String json = restTpl.getForObject(
				"http://first-service-provider/person/1", String.class);
		return json;
	}
}
```

        在控制器中，配置了RestTemplate的bean，RestTemplate本来是spring-web模块下面的类，主要用来调用REST服务，本身并不具备调用分布式服务的能力，但是RestTemplate的bean被@LoadBalanced注解修饰后，这个RestTemplate实例就具有访问分布式服务的能力，关于该类的一些机制，我们将放到负载均衡一章中讲解。

        在控制器中，新建了一个router的测试方法，用来对外发布REST服务，该方法只是一个路由作用，实际上使用RestTemplate来调用“first-ek-service-provider”（服务提供者）的服务。需要注意的是，调用服务时，仅仅是通过服务名称来进行调用。接下来编写启动类，如代码清单3-9所示。

        代码清单3-9：

        codes\03\3.2\first-ek-service-invoker\src\main\java\org\crazyit\cloud\FirstInvoker.java

```
package org.crazyit.cloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class FirstInvoker {

	public static void main(String[] args) {
		SpringApplication.run(FirstInvoker.class, args);
	}
}
```

        在启动类中，使用了@EnableDiscoveryClient注解来修改启动类，该注解使得服务调用者，有能力去Eureka中发现服务，需要注意的是@EnableEurekaClient注解已经包含了@EnableDiscoveryClient的功能，也就是说，一个Eureka客户端，本身就具有发现服务的能力。配置完成后，依次执行以下操作：

             启动服务器（first-ek-server）

             启动服务提供者（first-ek-service-provider）

             启动服务调用者（first-ek-service-invoker）

        使用浏览器访问Eureka，可看到注册的客户信息，如图3-4。

![](https://static.oschina.net/uploads/space/2017/0925/211656_jMKn_3665821.png)

图3-4 服务列表

        全部成功启动后，在浏览器中访问服务调用者发布的“router”服务：

[        http://localhost:9000/router](http://localhost:9000/router)

[        ](http://localhost:9000/router)可以看到在浏览器输出：{"id":1,"name":"Crazyit","age":30}

        根据输出可知，实际上调用了服务提供者的/person/1服务，第一个Eureka应用到此结束，下面对这个应用程序的结构作一个简单描述。

### 4.2.5 程序结构

        本案例新建了三个项目，如果读者对程序的结构不太清晰，可以参看图3-5。

![](https://static.oschina.net/uploads/space/2017/0925/211708_EwJr_3665821.png)

图3-5 本案例的结构

        如图3-5，Eureka服务为本例的“first-ek-server”，服务提供者为“first-ek-service-provider”，而调用者为“first-ek-service-invoker”，用户通过浏览器访问调用者的9000端口的router服务，router服务中查找服务提供者的服务并进行调用。在本例中，服务调用有点像路由器的角色。





我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>




