---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image:
title: Spring Cloud连载（12）自定义Feign客户端
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

## 12 自定义Feign客户端

        Feign使用一个Client接口来发送请求，默认情况下，使用HttpURLConnection连接HTTP服务。与前面的编码器类似，客户端也采用了插件式设计，也就是说，我们可以实现自己的客户端。本小节将使用HttpClient来实现一个简单的Feign客户端。为pom.xml加入HttpClient的依赖：

```
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>
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




