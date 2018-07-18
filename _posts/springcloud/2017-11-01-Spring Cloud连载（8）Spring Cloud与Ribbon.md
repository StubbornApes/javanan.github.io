---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://static.oschina.net/uploads/space/2017/1014/155334_asbK_3665821.png
title: Spring Cloud连载（8）Spring Cloud与Ribbon
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
- Ribbon
date:   2017-11-01 11:24:54
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr)**

原文作者：杨大仙的程序空间


## 8 Spring Cloud与RibbonRibbon

        本章要点

             Spring Cloud中使用Ribbon

        Spring Cloud集成了Ribbon，结合Eureka，可实现客户端的负载均衡。我们前面章节所使用的RestTemplate（被@LoadBalanced修饰）、还有后面章节的Feign，都已经拥有负载均衡功能。本小节将以RestTemplate为基础，讲述及测试Eureka中的Ribbon配置。

### 8.1 准备工作

        为了本小节的测试做准备，按顺序进行以下工作：

         新建Eureka服务器端项目，命名为“cloud-server”，端口8761，

            代码目录codes\04\4.4\cloud-server。

         新建Eureka服务提供者项目，命名为“cloud-provider”，

            代码目录codes\04\4.4\cloud-provider，该项目主要进行以下工作：

            1. 在控制器里面，发布一个REST服务，地址为“/person/{personId}”，

                请求后返回Person实例，其中Person的message为HTTP请求的URL。

            2. 服务提供者需要启动两次，因此在控制台中需要输入启动端口。

         新建Eureka服务调用者项目，命名为“cloud-invoker”，对外端口为9000，

            代码目录codes\04\4.4\cloud-invoker。本例的负载均衡配置主要针对服务调用者。

        以上项目准备完成并启动后，结构如图4-2所示。

![](https://static.oschina.net/uploads/space/2017/1014/155334_asbK_3665821.png)

图4-2 准备的项目结构图

        注意：Eureka相关项目的建立，可参见前面章节。

### 8.2 使用代码配置Ribbon

        在前面章节讲述了负载规则以及Ping，在Spring Cloud中，可将自定义的负载规则以及Ping类，放到服务调用者中，查看效果。新建自定义的IRule与IPing，两个实现类请见代码清单4-11。

        代码清单4-11：

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\MyRule.java

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\MyPing.java

```
package org.crazyit.cloud;

import java.util.List;

import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.Server;

public class MyRule implements IRule {

	private ILoadBalancer lb;

	public Server choose(Object key) {
		List<Server> servers = lb.getAllServers();
		System.out.println("这是自定义服务器定规则类，输出服务器信息：");
		for(Server s : servers) {
			System.out.println("        " + s.getHostPort());
		}
		return servers.get(0);
	}

	public void setLoadBalancer(ILoadBalancer lb) {
		this.lb = lb;
	}

	public ILoadBalancer getLoadBalancer() {
		return this.lb;
	}
}
```

 

```
package org.crazyit.cloud;

import com.netflix.loadbalancer.IPing;
import com.netflix.loadbalancer.Server;

public class MyPing implements IPing {

	public boolean isAlive(Server server) {
		System.out.println("自定义Ping类，服务器信息：" + server.getHostPort());
		return true;
	}
}

```

        根据两个自定义的IRule和IPing类可知，实际上跟4.3章节中的自定义实现类似，服务器选择规则中只返回集合中的第一个实例，IPing实现仅仅是控制输入服务器信息。接下来，新建配置类，返回规则与Ping的Bean，请见代码清单4-12。

        代码清单4-12：

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\config\MyConfig.java

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\config\CloudProviderConfig.java

```
package org.crazyit.cloud.config;

import org.crazyit.cloud.MyPing;
import org.crazyit.cloud.MyRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.netflix.loadbalancer.IPing;
import com.netflix.loadbalancer.IRule;

public class MyConfig {
	@Bean
	public IRule getRule() {
		return new MyRule();
	}
	@Bean
	public IPing getPing() {
		return new MyPing();
	}
}

```

```
package org.crazyit.cloud.config;

import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.ribbon.RibbonClient;
import org.springframework.context.annotation.Configuration;

@RibbonClient(name="cloud-provider", configuration=MyConfig.class)
public class CloudProviderConfig {

}

```

        代码清单4-12中，CloudProviderConfig配置类，使用了@RibbonClient注解，配置了RibbonClient的名称为“cloud-provider”，对应的配置类为“MyConfig”，也就是名称为“cloud-provider”的客户端，将使用MyRule与MyPing两个类。在服务调用者的控制器中，加入对外服务，服务中调用RestTemplate，如代码清单4-13。

        代码清单4-13：

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\InvokerController.java

```
@RestController
@Configuration
public class InvokerController {

	@LoadBalanced
	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	@RequestMapping(value = "/router", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
	public String router() {
		RestTemplate restTpl = getRestTemplate();
		// 根据名称调用服务
		String json = restTpl.getForObject("http://cloud-provider/person/1",
				String.class);
		return json;
	}
}
```

        以上的控制器中，为RestTemplate加入了@LoadBalanced修饰，与前面章节类似，在此不再赘述，关于RestTemplate的原理，将在本章后面章节讲述。进行以下操作，查看本例效果：

             启动一个Eureka服务器（cloud-server）。

             启动两次Eureka服务提供者（cloud-provider），分别输入8080与8081端口。

             启动一个Eureka服务调用者（cloud-invoker）。

             打开浏览器访问[http://localhost:9000/router](http://localhost:9000/router)，可以看到调用服务后返回的JSON字符串，不管刷新多少次，最终都只会访问其中一个端口。

### 8.3 使用配置文件设置Ribbon

        在前面使用Ribbon时，可以通过配置来定义各个属性，在使用Spring Cloud时，这些属性同样可以配置到application.yml中，以下的配置同样生效：

```
cloud-provider:
  ribbon:
    NFLoadBalancerRuleClassName: org.crazyit.cloud.MyRule
    NFLoadBalancerPingClassName: org.crazyit.cloud.MyPing
    listOfServers: http://localhost:8080/,http://localhost:8081/
```

        为cloud-provider这个客户端，配置了规则处理类、Ping类以及服务器列表，以同样的方式运行本小节例子，可看到同样的效果，在此不再赘述。

        代码与配置文件的方式进行配置，两种方式的效果一致，但对比起来，明显是配置文件的方式更加简便。

        注意：本案例的cloud-invoker模块中，默认使用了代码的方式来配置Ribbon，配置文件中的配置已被注释。

### 8.4 Spring使用Ribbon的API

        Spring Cloud对Ribbon进行封装，例如像负载客户端、负载均衡器等，我们可以直接使用Spring的LoadBalancerClient来处理请求以及服务选择。代码清单4-14，在服务器调用者的控制器中使用LoadBalancerClient。

        代码清单4-14：

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\InvokerController.java

```
	@Autowired
	private LoadBalancerClient loadBalancer;

	@RequestMapping(value = "/uselb", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
	public ServiceInstance uselb() {
		// 查找服务器实例
		ServiceInstance si = loadBalancer.choose("cloud-provider");
		return si;
	}
```

        除了使用Spring封装的负载客户端外，还可以直接使用Ribbon的API，代码4-15，直接获取Spring Cloud默认环境中，各个Ribbon的实现类。

        代码清单4-15：

        codes\04\4.4\cloud-invoker\src\main\java\org\crazyit\cloud\InvokerController.java

```
	@Autowired
	private SpringClientFactory factory;

	@RequestMapping(value = "/defaultValue", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
	public String defaultValue() {
		System.out.println("==== 输出默认配置：");
		// 获取默认的配置
		ZoneAwareLoadBalancer alb = (ZoneAwareLoadBalancer) factory
				.getLoadBalancer("default");
		System.out.println("    IClientConfig: "
				+ factory.getLoadBalancer("default").getClass().getName());
		System.out.println("    IRule: " + alb.getRule().getClass().getName());
		System.out.println("    IPing: " + alb.getPing().getClass().getName());
		System.out.println("    ServerList: "
				+ alb.getServerListImpl().getClass().getName());
		System.out.println("    ServerListFilter: "
				+ alb.getFilter().getClass().getName());
		System.out.println("    ILoadBalancer: " + alb.getClass().getName());
		System.out.println("    PingInterval: " + alb.getPingInterval());
		System.out.println("==== 输出 cloud-provider 配置：");
		// 获取 cloud-provider 的配置
		ZoneAwareLoadBalancer alb2 = (ZoneAwareLoadBalancer) factory
				.getLoadBalancer("cloud-provider");
		System.out.println("    IClientConfig: "
				+ factory.getLoadBalancer("cloud-provider").getClass()
						.getName());
		System.out.println("    IRule: " + alb2.getRule().getClass().getName());
		System.out.println("    IPing: " + alb2.getPing().getClass().getName());
		System.out.println("    ServerList: "
				+ alb2.getServerListImpl().getClass().getName());
		System.out.println("    ServerListFilter: "
				+ alb2.getFilter().getClass().getName());
		System.out.println("    ILoadBalancer: " + alb2.getClass().getName());
		System.out.println("    PingInterval: " + alb2.getPingInterval());
		return "";
	}
```

        代码中使用了SpringClientFactory，通过该实例，可获取各个默认的实现类以及配置，分别输出了默认配置以及“cloud-provider”配置。运行代码清单4-15，浏览器中访问地址[http://localhost:8080/defaultValue](http://localhost:8080/defaultValue)，可看到控制台输出如下：

```
==== 输出默认配置：
    IClientConfig: com.netflix.loadbalancer.ZoneAwareLoadBalancer
    IRule: com.netflix.loadbalancer.ZoneAvoidanceRule
    IPing: com.netflix.niws.loadbalancer.NIWSDiscoveryPing
    ServerList: org.springframework.cloud.netflix.ribbon.eureka.DomainExtractingServerList
    ServerListFilter: org.springframework.cloud.netflix.ribbon.ZonePreferenceServerListFilter
    ILoadBalancer: com.netflix.loadbalancer.ZoneAwareLoadBalancer
    PingInterval: 30
==== 输出 cloud-provider 配置：
    IClientConfig: com.netflix.loadbalancer.ZoneAwareLoadBalancer
    IRule: org.crazyit.cloud.MyRule
    IPing: org.crazyit.cloud.MyPing
    ServerList: org.springframework.cloud.netflix.ribbon.eureka.DomainExtractingServerList
    ServerListFilter: org.springframework.cloud.netflix.ribbon.ZonePreferenceServerListFilter
    ILoadBalancer: com.netflix.loadbalancer.ZoneAwareLoadBalancer
    PingInterval: 30
```

        根据输出可知，cloud-provider客户端使用的负载规则类以及Ping类，是我们自定义的实现类。

        一般情况下，Spring已经帮我们封装好了Ribbon，我们只需要直接调用RestTemplate等API来访问服务即可。

本文节选自《疯狂Spring Cloud微服务架构实战》






我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr>




