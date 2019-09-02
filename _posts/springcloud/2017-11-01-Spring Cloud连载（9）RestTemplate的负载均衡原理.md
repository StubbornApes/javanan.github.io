---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509630308802&di=f0523fd8eadfe2d5cd6316499ce13e0c&imgtype=0&src=http%3A%2F%2Fwww.uml.org.cn%2Fzjjs%2Fimages%2F201703290502.jpg
title: Spring Cloud连载（9）RestTemplate的负载均衡原理
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
- RestTemplate
- 负载均衡
date:   2017-11-01 11:25:10
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

原文作者：杨大仙的程序空间


## 9 RestTemplate负载均衡原理

        本文要点

             RestTemplate的负载均衡原理

### 9.1 @LoadBalanced注解概述

        RestTemplate本是spring-web项目中的一个REST客户端，它遵循REST的设计原则，提供简单的API让我们可以调用HTTP服务。RestTemplate本身不具有负载均衡的功能，该类也与Spring Cloud没有关系，但为何加入@LoadBalanced注解后，一个RestTemplate实例就具有负载均衡的功能呢？实际上这要得益于RestTemplate的拦截器功能。

        在Spring Cloud中，使用@LoadBalanced修饰的RestTemplate，在Spring容器启动时，会为这些被修饰过的RestTemplate添加拦截器，拦截器中使用了LoadBalancerClient来处理请求，LoadBalancerClient本来就是Spring封装的负载均衡客户端，通过这样间接处理，使得RestTemplate就拥有了负载均衡的功能。

        本小节将模仿拦截器机制，带领大家实现一个简单的RestTemplate，以便让大家更了解@LoadBalanced以及RestTemplate的原理。本小节的案例只依赖了spring-boot-starter-web模块：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.5.4.RELEASE</version>
</dependency>
```

### 9.2 编写自定义注解以及拦截器

        先模仿@LoadBalanced注解，编写一个自定义注解，请见代码清单4-16。

        代码清单4-1：

        codes\04\4.5\rest-template-test\src\main\java\org\crazyit\cloud\MyLoadBalanced.java

```
package org.crazyit.cloud;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.beans.factory.annotation.Qualifier;

@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MyLoadBalanced {

}

```

        注意MyLoadBalanced注解中使用了@Qualifier限定注解，接下来编写自定义的拦截器，请见代码清单4-17。

        代码清单4-17：

        codes\04\4.5\rest-template-test\src\main\java\org\crazyit\cloud\MyInterceptor.java

```
package org.crazyit.cloud;

import java.io.IOException;

import org.springframework.http.HttpRequest;
import org.springframework.http.client.ClientHttpRequestExecution;
import org.springframework.http.client.ClientHttpRequestInterceptor;
import org.springframework.http.client.ClientHttpResponse;

/**
 * 自定义拦截器
 *
 * @author 杨恩雄
 *
 */
public class MyInterceptor implements ClientHttpRequestInterceptor {

	public ClientHttpResponse intercept(HttpRequest request, byte[] body,
			ClientHttpRequestExecution execution) throws IOException {
		System.out.println("=============  这是自定义拦截器实现");
		System.out.println("               原来的URI：" + request.getURI());
		// 换成新的请求对象（更换URI）
		MyHttpRequest newRequest = new MyHttpRequest(request);
		System.out.println("               拦截后新的URI：" + newRequest.getURI());
		return execution.execute(newRequest, body);
	}
}
```

        在自定义拦截器MyInterceptor中，实现了intercept方法，该方法会将原来的HttpRequest对象，转换为我们自定义的MyHttpRequest，MyHttpRequest是一个自定义的请求类，实现如下请见代码清单4-18。

        代码清单4-18：

        codes\04\4.5\rest-template-test\src\main\java\org\crazyit\cloud\MyHttpRequest.java

```
package org.crazyit.cloud;

import java.net.URI;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpRequest;

/**
 * 自定义的请求类，用于转换URI
 * @author 杨恩雄
 *
 */
public class MyHttpRequest implements HttpRequest {

	private HttpRequest sourceRequest;

	public MyHttpRequest(HttpRequest sourceRequest) {
		this.sourceRequest = sourceRequest;
	}

	public HttpHeaders getHeaders() {
		return sourceRequest.getHeaders();
	}

	public HttpMethod getMethod() {
		return sourceRequest.getMethod();
	}

	/**
	 * 将URI转换
	 */
	public URI getURI() {
		try {
			URI newUri = new URI("http://localhost:8080/hello");
			return newUri;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return sourceRequest.getURI();
	}
}

```

        MyHttpRequest类中，会将原来请求的URI进行改写，只要使用了这个对象，所有的请求都会被转发到[http://localhost:8080/hello](http://localhost:8080/hello)这个地址。**Spring Cloud****在对****RestTemplate****进行拦截的时候，也做了同样的事情，只不过并没有像我们这样****固定了****UR****I****，而是对“源请求”进行了更加灵活的处理。**接下来使用自定义注解以及拦截器。

### 9.3 使用自定义拦截器以及注解

        编写一个Spring的配置类，在初始化的bean中为容器中的RestTemplate实例设置自定义拦截器，本例的Spring自动配置类请见代码清单4-19。

        代码清单4-19：

        codes\04\4.5\rest-template-test\src\main\java\org\crazyit\cloud\MyAutoConfiguration.java

```
package org.crazyit.cloud;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestInterceptor;
import org.springframework.web.client.RestTemplate;


@Configuration
public class MyAutoConfiguration {

	@Autowired(required=false)
	@MyLoadBalanced
	private List<RestTemplate> myTemplates = Collections.emptyList();

	@Bean
	public SmartInitializingSingleton myLoadBalancedRestTemplateInitializer() {
		System.out.println("====  这个Bean将在容器初始化时创建    =====");
		return new SmartInitializingSingleton() {

			public void afterSingletonsInstantiated() {
				for(RestTemplate tpl : myTemplates) {
					// 创建一个自定义的拦截器实例
					MyInterceptor mi = new MyInterceptor();
					// 获取RestTemplate原来的拦截器
					List list = new ArrayList(tpl.getInterceptors());
					// 添加到拦截器集合
					list.add(mi);
					// 将新的拦截器集合设置到RestTemplate实例
					tpl.setInterceptors(list);
				}
			}
		};
	}
}

```

        配置类中，定义了RestTemplate实例的集合，并且使用了@MyLoadBalanced以及@Autowired注解进行修饰，@MyLoadBalanced中含有@Qualifier注解，简单来说，就是Spring容器中，使用了@MyLoadBalanced修饰的RestTemplate实例，将会被加入到配置类的RestTemplate集合中。

        在容器初始化时，会调用myLoadBalancedRestTemplateInitializer方法来创建Bean，该Bean在初始化完成后，会遍历RestTemplate集合并为它们设置“自定义拦截器”，请见代码清单4-19中的粗体代码。下面在控制器中使用@MyLoadBalanced来修饰调用者的RestTemplate。

### 9.4 控制器中使用RestTemplate

        控制器代码请见代码清单4-20。

        代码清单4-20：

        codes\04\4.5\rest-template-test\src\main\java\org\crazyit\cloud\InvokerController.java

```
package org.crazyit.cloud;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
@Configuration
public class InvokerController {

	@Bean
	@MyLoadBalanced
	public RestTemplate getMyRestTemplate() {
		return new RestTemplate();
	}

	/**
	 * 浏览器访问的请求
	 */
	@RequestMapping(value = "/router", method = RequestMethod.GET,
			produces = MediaType.APPLICATION_JSON_VALUE)
	public String router() {
		RestTemplate restTpl = getMyRestTemplate();
		// 根据名称来调用服务，这个URI会被拦截器所置换
		String json = restTpl.getForObject("http://my-server/hello", String.class);
		return json;
	}

	/**
	 * 最终的请求都会转到这个服务
	 */
	@RequestMapping(value = "/hello", method = RequestMethod.GET)
	public String hello() {
		return "Hello World";
	}
}

```

        注意控制器的hello方法，前面实现的拦截器，会将全部请求都转到这个服务中。控制器的RestTemplate，使用了@MyLoadBalanced注解进行修饰，熟悉前面使用RestTemplate的读者可发现，我们实现的注解，与Spring提供的@LoadBalanced注解使用方法一致。控制器的router方法中，使用这个被拦截过的RestTemplate发送请求。

        打开浏览器，访问[http://localhost:8080/router](http://localhost:8080/router)，可以看到实际上调用了hello服务，在访问该地址时，控制台输出如下：

```
=============  这是自定义拦截器实现
               原来的URI：http://my-server/hello
               拦截后新的URI：http://localhost:8080/hello
```

        Spring Cloud对RestTemplate的拦截实现更加复杂，并且在拦截器中，使用LoadBalancerClient来实现请求的负载均衡功能，我们在实际环境中，并不需要实现自定义注解以及拦截器，用Spring提供的现成API即可，本小节目的是展示RestTemplate的原理。

 






我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>




