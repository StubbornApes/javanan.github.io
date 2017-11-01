---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image:
title: Spring Cloud连载（14）Spring Cloud整合Feign
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
date:   2017-11-01 11:25:53
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**

原文作者：杨大仙的程序空间


## 12 自定义Feign客户端

        Feign使用一个Client接口来发送请求，默认情况下，使用HttpURLConnection连接HTTP服务。与前面的编码器类似，客户端也采用了插件式设计，也就是说，我们可以实现自己的客户端。本小节将使用HttpClient来实现一个简单的Feign客户端。为pom.xml加入HttpClient的依赖：

```
Spring Cloud整合Feign

        前面讲解了Feign的使用，在了解如何单独使用Feign后，再学习Spring Cloud中使用Feign，将会有非常大的帮助。虽然Spring Cloud对Feign进行了封装，但万变不离其宗，只要了解其内在原理，使用起来就可以得心应手。

        在开始本小节前，先准备Spring Cloud的测试项目。测试案例主要有以下三个项目：

spring-feign-server：Eureka服务器端项目，端口为8761，代码目录为codes\05\5.3\spring-feign-server。
spring-feign-provider：服务提供者，代码目录为codes\05\5.3\spring-feign-provider，该项目可以在控制台中根据输入的端口号启动多个实例，本小节启动8080与8081这两个端口，该项目提供以下两个REST服务：
第一个地址为“/person/{personId}”，请求后返回Person实例，Person的message属性为HTTP请求的URL。
第二个地址为“/hello”的服务，返回“Hello World”字符串。
spring-feign-invoker：服务调用者项目，对外端口为9000，代码目录codes\05\5.3\spring-feign-invoker，本小节例子主要在该项目下使用Feign。
Spring Cloud整合Feign

        为服务调用者（spring-feign-invoker）的pom.xml文件加入以下依赖：

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
        在服务调用者的启动类中，打开Feign开关，请见代码清单5-18。

        代码清单5-18：

        codes\05\5.3\spring-feign-invoker\src\main\java\org\crazyit\cloud\InvokerApplication.java

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class InvokerApplication {

	public static void main(String[] args) {
		SpringApplication.run(InvokerApplication.class, args);
	}
}
        接下来，编写客户端接口，与直接使用Feign类似，代码清单5-19为服务端接口。

        代码清单5-19：

        codes\05\5.3\spring-feign-invoker\src\main\java\org\crazyit\cloud\PersonClient.java

@FeignClient("spring-feign-provider") //声明调用的服务名称
public interface PersonClient {

	@RequestMapping(method = RequestMethod.GET, value = "/hello")
	String hello();
}
        与单独使用Fiegn不同的是，接口使用了@FeignClient注解来修饰，并且声明了需要调用的服务名称，本例的服务提供者名称为“spring-feign-provider”。另外，接口方法使用了@RequestMapping来修饰，根据5.2.7章节可知，通过编写“翻译器（Contract）”，可以让Feign知道第三方注解的含义，Spring Cloud也提供翻译器，会将@RequestMapping注解的含义告知Feign，因此我们的服务接口就可以直接使用该注解。

        除了方法的@RequestMapping注解外，默认还支持@RequestParam、@RequestHeader、@PathVariable这3个参数注解，也就是说，在定义方法时，可以使用方式定义参数：

	@RequestMapping(method = RequestMethod.GET, value = "/hello/{name}")
	String hello(@PathVariable("name") String name);
        需要注意的是，使用了Spring Cloud的“翻译器”后，将不能再使用Feign的默认注解。接下来，在控制器中调用接口方法，请见代码清单是5-20。

        代码清单5-20：

        codes\05\5.3\spring-feign-invoker\src\main\java\org\crazyit\cloud\InvokerController.java

@RestController
@Configuration
public class InvokerController {

	@Autowired
	private PersonClient personClient;

	@RequestMapping(value = "/invokeHello", method = RequestMethod.GET)
	public String invokeHello() {
		return personClient.hello();
	}
}
        在控制器中，为其注入了PersonClient的bean，不难看出，客户端实例的创建及维护，Spring容器都帮我们实现了。查看本例的效果，请按以下步骤操作：

启动Eureka服务器（spring-feign-server）。
启动两个服务提供者（spring-feign-provider），控制台中分别输入8080与8081端口。
启动一个服务调用者（spring-feign-invoker），端口为9000。
在浏览器中输入：http://localhost:9000/invokerHello，可以看到服务提供者的“/hello”服务被调用。
Feign负载均衡

        在前面章节，我们尝试过编写自定义的Feign客户端，在Spring Cloud中，同样提供了自定义的Feign客户端。可能大家已经猜到，如果结合Ribbon使用，Spring Cloud所提供的客户端，会拥有负载均衡的功能。

        Spring Cloud实现的Feign客户端，类名为LoadBalancerFeignClient，在该类中，维护着与SpringClientFactory相关的实例，通过SpringClientFactory可以获取负载均衡器，负载均衡器会根据一定的规则来选取处理请求的服务器，最终实现负载均衡的功能。接下来，调用“服务提供者”的“/person/{personId}”服务来测试负载均衡。为客户端接口添加内容，请见代码清单5-21。

        代码清单5-21：

        codes\05\5.3\spring-feign-invoker\src\main\java\org\crazyit\cloud\PersonClient.java

	@RequestMapping(method = RequestMethod.GET, value = "/person/{personId}")
	Person getPerson(@PathVariable("personId") Integer personId);
        为“服务调用者”的控制器添加方法，请见代码清单5-22。

        代码清单5-22：

	@RequestMapping(value = "/router", method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public String router() {
		// 调用服务提供者的接口
		Person p = personClient.getPerson(2);
		return p.getMessage();
	}
        运行“服务调用者”，在浏览器中输入：http://localhost:9000/router，进行刷新，可以看到8080与8081端口被循环调用。

默认配置

        Spring Cloud为Feign的使用提供了各种默认属性，例如前面讲到的“注解翻译器（Contract）”、Feign客户端，默认情况下，Spring将会为Feign的属性提供了以下的Bean：

解码器（Decoder）：bean名称为feignDecoder，ResponseEntityDecoder类，
编码器（Encoder）：bean名称为feignEecoder，SpringEncoder类。
日志（Logger）： bean名称为feignLogger，Slf4jLogger类。
注解翻译器（Contract）： bean名称为feignContract，SpringMvcContract类。
Feign实例的创建者（Feign.Builder）：bean名称为feignBuilder，HystrixFeign.Builder类。Hystrix框架将在后面章节中讲述。
Feign客户端（Client）：bean名称为feignClient，LoadBalancerFeignClient类。
        一般情况下，Spring提供的这些Bean已经足够我们使用，如果有些更特殊的需求，可以实现自己的Bean，请见后面章节。
```

        新建feign.Client接口的实现类，具体实现请见代码清单5-13。

        代码清单5-13：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\MyFeignClient.java

```
public class MyFeignClient implements Client {

	public Response execute(Request request, Options options)
			throws IOException {
		System.out.println("====  这是自定义的Feign客户端");
		try {
			// 创建一个默认的客户端
			CloseableHttpClient httpclient = HttpClients.createDefault();
			// 获取调用的HTTP方法
			final String method = request.method();
			// 创建一个HttpClient的HttpRequest
			HttpRequestBase httpRequest = new HttpRequestBase() {
				public String getMethod() {
					return method;
				}
			};
			// 设置请求地址
			httpRequest.setURI(new URI(request.url()));
			// 执行请求，获取响应
			HttpResponse httpResponse = httpclient.execute(httpRequest);
			// 获取响应的主体内容
			byte[] body = EntityUtils.toByteArray(httpResponse.getEntity());
			// 将HttpClient的响应对象转换为Feign的Response
			Response response = Response.builder()
					.body(body)
					.headers(new HashMap<String, Collection<String>>())
					.status(httpResponse.getStatusLine().getStatusCode())
					.build();
			return response;
		} catch (Exception e) {
			throw new IOException(e);
		}
	}
}

```

        简单讲一下自定义Feign客户端的实现过程，在实现execute方法时，将Feign的Request实例，转换为HttpClient的HttpRequestBase，再使用CloseableHttpClient来执行请求，得到响应的HttpResponse实例后，再转换为Feign的Reponse实例返回。不仅我们实现的客户端，包括Feign自带的客户端以及其他扩展的客户端，实际上就是一个对象转换的过程。在运行类中直接使用我们的自定义客户端，请见代码清单5-14。

        代码清单5-14：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\MyClientTest.java

```
public class MyClientTest {

	public static void main(String[] args) {
		// 获取服务接口
		PersonClient personClient = Feign.builder()
				.encoder(new GsonEncoder())
				.client(new MyFeignClient())
				.target(PersonClient.class, "http://localhost:8080/");
		// 请求Hello World接口
		String result = personClient.sayHello();
		System.out.println("    接口响应内容：" + result);
	}
}

```

        运行代码清单5-14，输出如下：

```
====  这是自定义的Feign客户端
    接口响应内容：Hello World
```

        注意：在本例的实现中，笔者简化了实现，自定义的客户端中并没有转换请求头等信息，因此使用本例的客户端，无法请求其他格式的服务。

        虽然Feign也有HttpClient的实现，但本例的目的主要是向大家展示Feign客户的原理。举一反三，如果我们实现一个客户端，在实现中调用Ribbon的API，来实现负载均衡的功能，是完全可以实现的。幸运的是，Feign已经帮我们实现了RibbonClient，可以直接使用，更进一步，Spring Cloud也实现自己的Client，我们将在后面章节中讲述。

        本文节选自《疯狂Spring Cloud微






我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld>




