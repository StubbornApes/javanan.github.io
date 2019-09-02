---
layout: blog
istop: true
springcloud: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509630308801&di=5e84d7097f60d983314a08c27357e730&imgtype=0&src=http%3A%2F%2Fimg.educity.cn%2Fimg_10%2F263%2F2014010504%2F17804043904.jpg
title: Spring Cloud连载（7）Ribbon负载均衡器
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
- 负载均衡
date:   2017-11-01 11:24:40
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

原文作者：杨大仙的程序空间

# 7 Ribbon负载均衡器

        本文要点

             Ribbon负载均衡器

## 7.1 Ribbon负载均衡器

        Ribbon提供了几个负载均衡的组件，其目的就是为了让请求转给合适的服务器处理，因此，如何选择合适的服务器，便成为负载均衡机制的核心。本小节将围绕Ribbon负载均衡器的组件，向大家展示Ribbon负载均衡的实现机制。

### 7.1.1 负载均衡器

        Ribbon的负载均衡器接口，定义了服务器的操作，主要是用于进行服务器选择。前面的例子中，客户端使用了RestClient类，在发送请求时，会使用负载均衡器（ILoadBalancer）接口，根据特定的逻辑来选择服务器，服务器列表可使用listOfServers进行配置，也可以使用动态更新机制。代码清单4-4使用负载均衡器来选择服务器。

        代码清单4-4：

        codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\ChoseServerTest.java

```
public class ChoseServerTest {

	public static void main(String[] args) {
		// 创建负载均衡器
		BaseLoadBalancer lb = new BaseLoadBalancer();
		// 添加服务器
		List<Server> servers = new ArrayList<Server>();
		servers.add(new Server("localhost", 8080));
		servers.add(new Server("localhost", 8081));
		lb.addServers(servers);
		// 进行6次服务器选择
		for(int i = 0; i < 6; i++) {
			Server s = lb.chooseServer(null);
			System.out.println(s);
		}
	}

}
```

        代码中使用了BaseLoadBalancer这个负载均衡器，将两个服务器对象加入到负载均衡器中，再调用6次chooseServer方法，可以看到输出如下：

```
localhost:8081
localhost:8080
localhost:8081
localhost:8080
localhost:8081
localhost:8080
```

        根据结果可知，最终选择的服务器与前面章节一致，可以判定本例第一个Ribbon例子，选择服务器的逻辑是一致的，在默认情况下，会使用RoundRobinRule的规则逻辑。

### 7.1.2 自定义负载规则

        根据前一小节可知，选择哪个服务器进行请求处理，由ILoadBalancer接口的chooserServer方法决定，而在BaseLoadBalancer类中，则使用IRule接口的choose方法来决定选择哪一个服务器对象。如果想自定义负载均衡规定，可以编写一个IRule接口的实现类。代码清单4-5实现了自己的负载规定。

        代码清单4-5：codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\MyRule.java

```
public class MyRule implements IRule {

	ILoadBalancer lb;

	public MyRule() {
	}

	public MyRule(ILoadBalancer lb) {
		this.lb = lb;
	}

	public Server choose(Object key) {
		// 获取全部的服务器
		List<Server> servers = lb.getAllServers();
		// 只返回第一个Server对象
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

        在自定义规则类中，实现的choose方法，调用了ILoadBalancer的getAllServers方法返回全部的服务器，为了简单起见，本例只返回第一个服务器。为了能在负载均衡器中使用自定义的规则，需要修改选择服务器的代码，请见代码清单4-6。

        代码清单4-6：

        codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\TestMyRule.java

```
public class TestMyRule {

	public static void main(String[] args) {
		// 创建负载均衡器
		BaseLoadBalancer lb = new BaseLoadBalancer();
		// 设置自定义的负载规则
		lb.setRule(new MyRule(lb));
		// 添加服务器
		List<Server> servers = new ArrayList<Server>();
		servers.add(new Server("localhost", 8080));
		servers.add(new Server("localhost", 8081));
		lb.addServers(servers);
		// 进行6次服务器选择
		for(int i = 0; i < 6; i++) {
			Server s = lb.chooseServer(null);
			System.out.println(s);
		}
	}

}
```

        运行代码清单4-6，可以看到，请求6次所得到的服务器均为“localhost:8080”。以上是直接使用编码方式来设置负载规则，可以使用配置的方式来完成这些工作。修改Ribbon的配置，让请求的客户端，使用我们定义的负载规则，请见代码清单4-7。

        代码清单4-7：

        codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\TestMyRuleConfig.java

```
public class TestMyRuleConfig {

	public static void main(String[] args) throws Exception {
		// 设置请求的服务器
		ConfigurationManager.getConfigInstance().setProperty(
				"my-client.ribbon.listOfServers",
				"localhost:8080,localhost:8081");
		// 配置规则处理类
		ConfigurationManager.getConfigInstance().setProperty(
				"my-client.ribbon.NFLoadBalancerRuleClassName",
				MyRule.class.getName());
		// 获取REST请求客户端
		RestClient client = (RestClient) ClientFactory
				.getNamedClient("my-client");
		// 创建请求实例
		HttpRequest request = HttpRequest.newBuilder().uri("/person/1").build();
		// 发 送10次请求到服务器中
		for (int i = 0; i < 6; i++) {
			HttpResponse response = client.executeWithLoadBalancer(request);
			String result = response.getEntity(String.class);
			System.out.println(result);
		}
	}

}
```

        请求客户端中，与前面章节的客户端基本一致，只是加入了“my-clent.ribbon.NFLoadBalancerRuleClassName”属性，设置了自定义规则处理类为MyRule，这个配置项同样可以在配置文件中使用，包括Spring Cloud的配置文件（application.yml等）。

        启动前面章节的服务器端两次，分别设置8080与8081端口，再运行代码清单4-7，可以看到输出了6次{"id":1,"name":"Crazyit","age":30,"message":"http://localhost:8080/person/1"}，根据结果可知，我们的自定义规则生效，请求只让8080端口处理。

        在实际环境中，如果要实现自定义的负载规则，可能还需要结合各种因素，例如考虑具体业务的发生时间、服务器性能等，实现中还可能还涉及使用计算器、数据库等技术，具体情形会更为复杂，本例的负载规则较为简单，目的是让读者了解负载均衡的原理。

### 7.1.3 Ribbon自带的负载规则

        Ribbon提供了若干个内置的负载规则，使用者完全可以直接使用，主要有以下内置的负载规则：

              RoundRobinRule：系统默认的规则，通过简单的轮询服务列表来选择服务器，其他的规则在很多情况下，仍然使用RoundRobinRule。

              AvailabilityFilteringRule：该规则会忽略以下服务器：

                一、 无法连接的服务器：在默认情况下，如果3次连接失败，该服务器将会被置为“短路”的状态，该状态将持续30秒，如果再次连接失败，“短路”状态的持续时间将会以几何级增加。可以通过修改niws.loadbalancer.<clientName>.connectionFailureCountThreshold属性，来配置连接失败的次数。

                二、并发数过高的服务器：如果连接到该服务器的并发数过高，也会被这个规则忽略，可以通过修改<clientName>.ribbon.ActiveConnectionsLimit属性来设定最高并发数。

              WeightedResponseTimeRule：为每个服务器赋予一个权重值，服务器的响应时间越长，该权重值就是越少，这个规则会随机选择服务器，这个权重值有可能会决定服务器的选择。

              ZoneAvoidanceRule：该规则以区域、可用服务器为基础，进行服务器选择。使用Zone对服务器进行分类，可以理解为机架或者机房。

              BestAvailableRule：忽略“短路”的服务器，并选择并发数较低的服务器。

              RandomRule：顾名思义，随机选择可用的服务器。

              RetryRule：含有重试的选择逻辑，如果使用RoundRobinRule选择服务器无法连接，那么将会重新选择服务器。

        以上提供的负载规则，基本可以满足大部分的需求，如果有更为复杂的要求，建议实现自定义负载规则。

### 7.1.4 Ping机制

        IPing接口的实现类负责，如果单独使用Ribbon，在默认情况下，不会激活Ping机制，默认的实现类为DummyPing。代码清单4-8，使用另外一个IPing实现类PingUrl。

        代码清单4-8：

        codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\TestPingUrl.java

```
public class TestPingUrl {

	public static void main(String[] args) throws Exception {
		// 创建负载均衡器
		BaseLoadBalancer lb = new BaseLoadBalancer();
		// 添加服务器
		List<Server> servers = new ArrayList<Server>();
		// 8080端口连接正常
		servers.add(new Server("localhost", 8080));
		// 一个不存在的端口
		servers.add(new Server("localhost", 8888));
		lb.addServers(servers);
		// 设置IPing实现类
		lb.setPing(new PingUrl());
		// 设置Ping时间间隔为2秒
		lb.setPingInterval(2);
		Thread.sleep(6000);
		for(Server s : lb.getAllServers()) {
			System.out.println(s.getHostPort() + " 状态：" + s.isAlive());
		}
	}

}
```

        代码清单4-8，使用了代码的方法来设置负载均衡器使用PingUrl，设置了每隔2秒，就向两个服务器请求，PingUrl实际是使用的是HttpClient，以上例子中，实际上会请求“http://localhost:8080”与“http://localhost:8888”这两个地址，在运行前先以8080端口启动前面章节的服务器，最终效果为8080的服务器状态正常，而8888的服务器则无法连接，运行代码清单4-8，可以看到输出如下：

```
localhost:8080 状态：true
localhost:8888 状态：false
```

        除了在代码中配置使用IPing类外，还可以在配置中设置IPing实现类，请见代码清单4-9。

        代码清单4-9：

        codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\TestPingUrlConfig.java

```
public class TestPingUrlConfig {

	public static void main(String[] args) throws Exception {
		// 设置请求的服务器
		ConfigurationManager.getConfigInstance().setProperty(
				"my-client.ribbon.listOfServers",
				"localhost:8080,localhost:8888");
		// 配置Ping处理类
		ConfigurationManager.getConfigInstance().setProperty(
				"my-client.ribbon.NFLoadBalancerPingClassName",
				PingUrl.class.getName());
		// 配置Ping时间间隔
		ConfigurationManager.getConfigInstance().setProperty(
				"my-client.ribbon.NFLoadBalancerPingInterval",
				2);
		// 获取REST请求客户端
		RestClient client = (RestClient) ClientFactory
				.getNamedClient("my-client");
		Thread.sleep(6000);
		// 获取全部服务器
		List<Server> servers = client.getLoadBalancer().getAllServers();
		System.out.println(servers.size());
		// 输出状态
		for(Server s : servers) {
			System.out.println(s.getHostPort() + " 状态：" + s.isAlive());
		}
	}
}
```

        注意代码中的以下两个配置：

              my-client.ribbon.NFLoadBalancerPingClassName：配置IPing的实现类。

              my-client.ribbon.NFLoadBalancerPingInterval：配置Ping操作的时间间隔。

        以上两个配置，同样可以使用在配置文件中。

### 7.1.5 自定义Ping

        通过前面章节的案例可知，实现自定义Ping较为简单，先实现IPing接口，然后再通过配置来设定具体的Ping实现类，代码清单4-10为自定义的Ping类。

        代码清单4-10：codes\04\4.2\first-ribbon-client\src\main\java\org\crazyit\cloud\MyPing.java

```
public class MyPing implements IPing {

	public boolean isAlive(Server server) {
		System.out.println("这是自定义Ping实现类：" + server.getHostPort());
		return true;
	}
}

```

        要使用自定义的Ping类，通过修改<client>.<nameSpace>.NFLoadBalancerPingClassName配置即可，在此不再赘述。

### 7.1.6 其他配置

        本小节主要介绍了Ribbon负载均衡器的负载规则以及Ping，这两部分可以通过配置来实现逻辑的改变，除了这两部分外，还可以使用以下的配置，来改变负载均衡器的其他行为：

              NFLoadBalancerClassName：指定负载均衡器的实现类，可利用该配置，实现自己的负载均衡器。

              NIWSServerListClassName：服务器列表处理类，用来维护服务器列表，Ribbon已经实现动态服务器列表。

              NIWSServerListFilterClassName：用于处理服务器列表拦截。







我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>




