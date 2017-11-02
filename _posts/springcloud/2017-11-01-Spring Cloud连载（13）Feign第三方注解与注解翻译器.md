---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509630251328&di=c1a6a987c6b0f91b91b213c07a300667&imgtype=0&src=http%3A%2F%2Fwww.51wendang.com%2Fpic%2Fbaf2561926f3c9faf69c2b4d%2F2-355-png_6_0_0_0_0_0_0_892.979_1262.879-893-0-0-893.jpg
title: Spring Cloud连载（13）Feign第三方注解与注解翻译器
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
- Feign
date:   2017-11-01 11:25:52
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**

原文作者：杨大仙的程序空间


## 12 自定义Feign客户端

        Feign使用一个Client接口来发送请求，默认情况下，使用HttpURLConnection连接HTTP服务。与前面的编码器类似，客户端也采用了插件式设计，也就是说，我们可以实现自己的客户端。本小节将使用HttpClient来实现一个简单的Feign客户端。为pom.xml加入HttpClient的依赖：

```
Feign第三方注解与注解翻译器

使用第三方注解

        根据前面章节可知，通过注解修改的接口方法，可以让接口方法获得访问服务的能力。除了Feign自带的方法外，还可以使用第三方的注解。如果想使用JAXRS规范的注解，可以使用Feign的“feign-jaxrs”模块，在pom.xml中加入以下依赖即可：

		<!-- Feign对JAXRS的支持 -->
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-jaxrs</artifactId>
			<version>9.5.0</version>
		</dependency>
		<!-- JAXRS -->
		<dependency>
			<groupId>javax.ws.rs</groupId>
			<artifactId>jsr311-api</artifactId>
			<version>1.1.1</version>
		</dependency>
        在使用注解修饰接口时，可以直接使用@GET、@Path等注解，例如想要使用GET方法调用“/hello”服务，可以定义以下接口：

	@GET @Path("/hello")
	String rsHello();
        以上修饰接口的，实际上就等价于“@RequestLine("GET /hello")”。为了让Feign知道这些注解的作用，需要在创建服务客户端时，调用contract方法来设置JAXRS注解的解析类，请见以下代码：

		RSClient rsClient = Feign.builder()
				.contract(new JAXRSContract())
				.target(RSClient.class, "http://localhost:8080/");
        设置了JAXRSContract类后，Feign就知道如何处理JAXRS的相关注解，下一小节，将讲解Feign是如何处理第三方注解的。

Feign解析第三方注解

        根据前一小节可知，设置了JAXRSContract后，Feign就知道如何处理接口中的JAXRS注解。JAXRSContract继承了BaseContract类，BaseContract类实现了Contract接口，简单的说，一个Contract就相当于一个翻译器，Feign本身并不知道这些第三方注解的含义，而通过实现一个翻译器（Contract）来告诉Feign，这些注解是做什么的。

        为了让读者能够了解其中的原理，本小节将使用一个自定义注解，并且翻译给Feign，让其去使用。代码清单5-15为自定义注解以及客户端接口的代码。

        代码清单5-15：

        codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\contract\MyUrl.java

        codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\contract\HelloClient.java

@Target(METHOD)
@Retention(RUNTIME)
public @interface MyUrl {

	// 定义url与method属性
	String url();
	String method();
}

public interface HelloClient {

	@MyUrl(method = "GET", url = "/hello")
	String myHello();
}
        接下来，就要将MyRrl注解的作用告诉Feign，新建Contract继承BaseContract类，实现请见代码清单5-16。

        代码清单5-16：

        codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\contract\MyContract.java

public class MyContract extends Contract.BaseContract {

	@Override
	protected void processAnnotationOnClass(MethodMetadata data, Class<?> clz) {

	}

	/**
	 * 用于处理方法级的注解
	 */
	protected void processAnnotationOnMethod(MethodMetadata data,
			Annotation annotation, Method method) {
		// 是MyUrl注解才进行处理
		if(MyUrl.class.isInstance(annotation)) {
			// 获取注解的实例
			MyUrl myUrlAnn = method.getAnnotation(MyUrl.class);
			// 获取配置的HTTP方法
			String httpMethod = myUrlAnn.method();
			// 获取服务的url
			String url = myUrlAnn.url();
			// 将值设置到模板中
			data.template().method(httpMethod);
			data.template().append(url);
		}
	}

	@Override
	protected boolean processAnnotationsOnParameter(MethodMetadata data,
			Annotation[] annotations, int paramIndex) {
		return false;
	}
}
        在MyContract类中，需要实现三个方法，分别是处理类注解、处理方法注解、处理参数注解的方法，由于我们只定了一个方法注解@MyRul，因此实现processAnnotationOnMethod即可。

        在processAnnotationOnMethod方法中，通过Method的getAnnotation获取MyUrl的实例，将MyUrl的url、method属性分别设置到Feign的模板中。在创建客户端时，再调用contract方法即可，请见代码清单5-17。

        代码清单5-17：

        codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\contract\ContractTest.java

public class ContractTest {

	public static void main(String[] args) {
		// 获取服务接口
		HelloClient helloClient = Feign.builder()
				.contract(new MyContract())
				.target(HelloClient.class, "http://localhost:8080/");
		// 请求Hello World接口
		String result = helloClient.myHello();
		System.out.println("    接口响应内容：" + result);
	}
}
        运行代码清单5-18，可看到控制台输出如下：

    接口响应内容：Hello World
        由本例可知，一个Contract实际上承担的是一个翻译的作用，将第三方（或者自定义）注解的作用告诉Feign。在Spring Cloud中，也实现了Spring的Contract，可以在接口中使用@RequestMapping注解，读者在学习Spring Cloud整合Feign时，见到使用@RequestMapping修饰的接口，就可以明白其中的原理。
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




