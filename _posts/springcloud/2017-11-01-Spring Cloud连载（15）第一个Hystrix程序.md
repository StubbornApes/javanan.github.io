---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510224935&di=9f3a08de5dadc9c282de3f02a7a003f6&imgtype=jpg&er=1&src=http%3A%2F%2Fs2.51cto.com%2Fwyfs02%2FM02%2F8E%2F8D%2FwKiom1jFACCx2lbgAAEWcb8v6-w59.jpeg-wh_651x-s_2509118232.jpeg
title: Spring Cloud连载（15）第一个Hystrix程序
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
- Hystrix
date:   2017-11-01 11:25:54
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**

原文作者：杨大仙的程序空间


## 第一个Hystrix程序

        先编写一个简单的Hello World程序，展示Hystrix的基本作用。前面的章节会单独讲解Hystrix框架，后面章节才会整合Spring Cloud一起使用。

### 准备工作

        使用Spring Boot的spring-boot-starter-web项目，建立一个普通的Web项目，发布两个测试服务用于测试，控制器的代码请见代码清单6-1。

        代码清单6-1：

        codes\06\6.2\first-hystrix-server\src\main\java\org\crazyit\cloud\MyController.java

```
@RestController
public class MyController {

	@GetMapping("/normalHello")
	public String normalHello(HttpServletRequest request) {
		return "Hello World";
	}

	@GetMapping("/errorHello")
	public String errorHello(HttpServletRequest request) throws Exception {
		// 模拟需要处理10秒
		Thread.sleep(10000);
		return "Error Hello World";
	}
}

```

        一个正常的服务，另外一个服务则需要等待10秒才有返回。本例的Web项目对应的代码目录为codes\06\6.2\first-hystrix-server，启动类是ServerApplication。

### 客户端使用Hystrix

        结合Hystrix来请求Web服务，可能与原来的方式不太一样。新建项目“first-hystrix-client”，在pom.xml中加入以下依赖：

```
		<dependency>
			<groupId>com.netflix.hystrix</groupId>
			<artifactId>hystrix-core</artifactId>
			<version>1.5.12</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<version>1.7.25</version>
			<artifactId>slf4j-log4j12</artifactId>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>

```

        本书Spring Cloud所使用的Hystrix版本为1.5.12，我们也使用与其一致的版本。客户端项目除了要使用Hystrix外，还会使用HttpClient模块访问Web服务，因此要加入相应的依赖。新建一个命令类，实现请见代码清单6-2。

        代码清单6-2：

        codes\06\6.2\first-hnystrix-client\src\main\java\org\crazyit\cloud\HelloCommand.java

```
public class HelloCommand extends HystrixCommand<String> {

	private String url;

	CloseableHttpClient httpclient;

	public HelloCommand(String url) {
		// 调用父类的构造器，设置命令组的key，默认用来作为线程池的key
		super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
		// 创建HttpClient客户端
		this.httpclient = HttpClients.createDefault();
		this.url = url;
	}

	protected String run() throws Exception {
		try {
			// 调用 GET 方法请求服务
			HttpGet httpget = new HttpGet(url);
			// 得到服务响应
			HttpResponse response = httpclient.execute(httpget);
			// 解析并返回命令执行结果
			return EntityUtils.toString(response.getEntity());
		} catch (Exception e) {
			e.printStackTrace();
		}
		return "";
	}
}

```

        新建运行类，执行HelloCommand，如代码清单6-3所示。

        代码清单6-3：

        codes\06\6.2\first-hnystrix-client\src\main\java\org\crazyit\cloud\HelloMain.java

```
public class HelloMain {

	public static void main(String[] args) {
		// 请求正常的服务
		String normalUrl = "http://localhost:8080/normalHello";
		HelloCommand command = new HelloCommand(normalUrl);
		String result = command.execute();
		System.out.println("请求正常的服务，结果：" + result);
	}
}

```

        正常情况下，直接调用HttpClient的API来请求Web服务，而前面的命令类与运行类，则通过命令来执行调用的工作。在命令类HelloCommand中，实现了父类的run方法，使用HttpClient调用服务的过程，都放到了该方法中。运行HelloMain类，可以看到，结果与平常调用Web服务无异。接下来，测试使用Hystrix的情况下调用有问题的服务。

### 调用错误服务

        假设我们所调用的Hello服务发生故障，导致无法正常访问，那么对于客户端来说，如何自保呢？本例将调用延时的服务，为客户端设置回退方法。修改HelloCommand类，加入回退方法，请见代码清单6-4。

        代码清单6-4：

        codes\06\6.2\first-hnystrix-client\src\main\java\org\crazyit\cloud\HelloCommand.java

```
	protected String getFallback() {
		System.out.println("执行 HelloCommand 的回退方法");
		return "error";
	}

```

        在运行类中，调用发生故障的服务，请见代码清单6-5。

        代码清单6-5：

        codes\06\6.2\first-hnystrix-client\src\main\java\org\crazyit\cloud\HelloErrorMain.java

```
public class HelloErrorMain {

	public static void main(String[] args) {
		// 请求异常的服务
		String normalUrl = "http://localhost:8080/errorHello";
		HelloCommand command = new HelloCommand(normalUrl);
		String result = command.execute();
		System.out.println("请求异常的服务，结果：" + result);
	}
}

```

        运行HelloErrorMain类，输出如下：

```
执行 HelloCommand 的回退方法
请求异常的服务，结果：error

```

        根据结果可知，回退方法被执行。本例中调用的“errorHello”服务，会阻塞10秒才有返回。默认情况下，如果调用的Web服务无法在1秒内完成，那么将会触发回退。

        回退更像是一个备胎，当请求的服务无法正常返回时，就调用该“备胎”的实现。这样做，可以很好的保护客户端，服务端所提供的服务受网络等条件的制约，如果有服务真的需要10秒才能返回结果，而客户端又没有容错机制，后果就是，客户端将一直等待返回，直到网络超时或者服务有响应，而外界会一直不停地发送请求给客户端，最终导致的结果就是，客户端因请求过多而瘫痪。






我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>




