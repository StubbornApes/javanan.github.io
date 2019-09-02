---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509630308803&di=66e49512345ecca2a3b7007f702a63bb&imgtype=0&src=http%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F4758%2F201607%2F4758-20160701164510718-73468435.png
title: Spring Cloud连载（10）客户端Feign介绍
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
date:   2017-11-01 11:25:20
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

原文作者：杨大仙的程序空间

**本文要点**

             REST客户端

        Spring Cloud集群中，各个角色的通信基于REST服务，因此在调用服务时，就不可避免的需要使用REST服务的请求客户端。前面的章节中使用了Spring自带的RestTemplate，RestTemplate使用的是HttpClient发送请求。本章中，将介绍另一个REST客户端：Feign。

## 10 REST客户端Feign介绍

        在学习Feign前，先了解REST客户端，本小节将简单地讲述Apache CXF与Restlet这两款Web Service框架，并使用这两个框架来编写REST客户端，最后再编写一个Feign的Hello World例子。通过此过程，让大家可以对Feign有一个初步的印象。如已经掌握这两个REST框架，可直接到后面章节学习Feign。

        本章的各个客户端，将会访问8080端口的“/person/{personId}”和“/hello”这两个服务中的一个，服务端项目使用“spring-boot-starter-web”进行搭建，本小节对应的服务端项目目录为：codes\05\5.1\rest-server。

### 10.1 使用CXF调用REST服务

        CXF是目前一个较为流行的Web Service框架，是Apache下的一个开源项目。使用CXF可以发布和调用各种协议的服务，包括SOAP协议、XML/HTTP等，当前CXF已经对REST风格的Web Service提供支持，可以发布或调用REST风格的Web Service。由于CXF可以与Spring进行整合使用并且配置简单，因此得到许多开发者的青睐，而笔者以往所在公司的大部分项目，均使用CXF来发布和调用Web Service，本章所使用的CXF版本为3.1.10，Maven中加入以下依赖：

```
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-core</artifactId>
			<version>3.1.10</version>
		</dependency>
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-rt-rs-client</artifactId>
			<version>3.1.10</version>
		</dependency>
```

        编写代码请求/person/{personId}服务，请见代码清单5-1。

        代码清单5-1：codes\05\5.1\rest-client\src\main\java\org\crazyit\cloud\CxfClient.java

```
package org.crazyit.cloud;

import java.io.InputStream;

import javax.ws.rs.core.Response;

import org.apache.cxf.helpers.IOUtils;
import org.apache.cxf.jaxrs.client.WebClient;

/**
 * CXF访问客户端
 * @author 杨恩雄
 *
 */
public class CxfClient {

	public static void main(String[] args) throws Exception {
		// 创建WebClient
		WebClient client = WebClient.create("http://localhost:8080/person/1");
		// 获取响应
		Response response = client.get();
		// 获取响应内容
		InputStream ent = (InputStream) response.getEntity();
		String content = IOUtils.readStringFromStream(ent);
		// 输出字符串
		System.out.println(content);
	}
}

```

        客户端中，使用了WebClient类发送请求，获取响应后读取输入流，获取服务返回的JSON字符串。，运行代码清单5-1，可看到返回的信息。

### 10.2 使用Restlet调用REST服务

        Restlet是一个轻量级的REST框架，使用它可以发布和调用REST风格的Web Service。本小节例子所使用的版本为2.3.10，Maven依赖如下：

```
		<dependency>
			<groupId>org.restlet.jee</groupId>
			<artifactId>org.restlet</artifactId>
			<version>2.3.10</version>
		</dependency>
		<dependency>
			<groupId>org.restlet.jee</groupId>
			<artifactId>org.restlet.ext.jackson</artifactId>
			<version>2.3.10</version>
		</dependency>
```

        客户端实现请见代码清单5-2。

        代码清单5-2：codes\05\5.1\rest-client\src\main\java\org\crazyit\cloud\RestletClient.java

```
package org.crazyit.cloud;

import java.util.HashMap;
import java.util.Map;

import org.restlet.data.MediaType;
import org.restlet.ext.jackson.JacksonRepresentation;
import org.restlet.representation.Representation;
import org.restlet.resource.ClientResource;

/**
 * Restlet客户端
 * @author 杨恩雄
 *
 */
public class RestletClient {

	public static void main(String[] args) throws Exception {
		ClientResource client = new ClientResource(
				"http://localhost:8080/person/1");
		// 调用get方法，服务器发布的是GET
		Representation response = client.get(MediaType.APPLICATION_JSON);
		// 创建JacksonRepresentation实例，将响应转换为Map
		JacksonRepresentation jr = new JacksonRepresentation(response,
				HashMap.class);
		// 获取转换后的Map对象
		Map result = (HashMap) jr.getObject();
		// 输出结果
		System.out.println(result.get("id") + "-" + result.get("name") + "-"
				+ result.get("age") + "-" + result.get("message"));
	}
}

```

        代码清单5-2中使用Restlet的API较为简单，在此不过多赘述，但需要注意的是，在Maven中使用Restlet，要额外配置仓库地址，笔者成书时Apache官方仓库中，并没有Restlet的包。在项目的pom.xml文件中增加以下配置：

```
	<repositories>
		<repository>
			<id>maven-restlet</id>
			<name>Restlet repository</name>
			<url>http://maven.restlet.org</url>
		</repository>
	</repositories>
```

### 10.3 Feign框架介绍

        Feign是一个Github上一个开源项目，目的是为了简化Web Service客户端的开发。在使用Feign时，可以使用注解来修饰接口，被注解修饰的接口具有访问Web Service的能力，这些注解中既包括了Feign自带的注解，也支持使用第三方的注解。除此之外，Feign还支持插件式的编码器和解码器，使用者可以通过该特性，对请求和响应进行不同的封装与解析。

        Spring Cloud将Feign集成到netflix项目中，当与Eureka、Ribbon集成时，Feign就具有负载均衡的功能。Feign本身在使用上的简便性，加上与Spring Cloud的高度整合，使用该框架在Spring Cloud中调用集群服务，将会大大降低开发的工作量。

### 10.4 第一个Feign程序

        先使用Feign编写一个Hello World的客户端，访问服务端的“/hello”服务，得到返回的字符串。当前Spring Cloud所依赖的Feign版本为9.5.0，本章案例中的Feign也使用该版本。建立名称为“feign-client”的Maven项目，加入以下依赖：

```
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-core</artifactId>
			<version>9.5.0</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-gson</artifactId>
			<version>9.5.0</version>
		</dependency>
```

        新建接口HelloClient，请见代码清单5-3。

        代码清单5-3：codes\05\5.1\feign-client\src\main\java\org\crazyit\cloud\HelloClient.java

```
package org.crazyit.cloud;

import feign.RequestLine;

/**
 * 客户端调用的服务接口
 * @author 杨恩雄
 *
 */
public interface HelloClient {

	  @RequestLine("GET /hello")
	  String sayHello();
}

```

        HelloClient表示一个服务接口，接口的“sayHello”方法中，使用了@RequestLine注解，表示使用GET方法，向“/hello”发送请求。接下来编写客户端的运行类，请见代码清单5-4。

代码清单5-4：codes\05\5.1\feign-client\src\main\java\org\crazyit\cloud\HelloMain.java

```
package org.crazyit.cloud;

import feign.Feign;
import feign.gson.GsonDecoder;

public class HelloMain {

	public static void main(String[] args) {
		// 调用Hello接口
		HelloClient hello = Feign.builder().target(HelloClient.class,
				"http://localhost:8080/");
		System.out.println(hello.getClass().getName());
		System.out.println(hello.sayHello());
	}
}

```

        运行类中，使用Feign创建HelloClient接口的实例，最后调用接口定义的方法。运行代码清单5-4，可以看到返回的“Hello World”字符串，可见接口已经被调用。熟悉AOP的朋友大概已经猜到，Feign实际上会帮我们动态生成代理类。Feign使用的是JDK的动态代理，生成的代理类，会将请求的信息封装，交给feign.Client接口发送请求，而该接口的默认实现类，最终会使用java.net.HttpURLConnection来发送HTTP请求。

### 10.5 请求参数与返回对象

        本案例中有两个服务，另外一个地址为“/person/{personId}”，需要传入参数并且返回JSON字符串，编写第二个Feign客户端，调用该服务。新建PersonClient服务类，定义调用接口并添加注解，请见代码清单5-5。

        代码清单5-5：codes\05\5.1\feign-client\src\main\java\org\crazyit\cloud\PersonClient.java

```
package org.crazyit.cloud;

import lombok.Data;
import lombok.NoArgsConstructor;
import feign.Param;
import feign.RequestLine;

/**
 * Person客户端服务类
 * @author 杨恩雄
 *
 */
public interface PersonClient {

	@RequestLine("GET /person/{personId}")
	Person findById(@Param("personId") Integer personId);

	@Data // 为所有属性加上setter和getter等方法
	class Person {
		Integer id;
		String name;
		Integer age;
		String message;
	}
}

```

        定义的接口名称为“findById”，参数为“personId”。需要注意的是，由于会返回Person实例，我们在接口中定义了一个Person的类，为了减少代码量，使用了Lombok项目，使用了该项目的@Data注解。要使用Lombok，需要添加以下Maven依赖：

```
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.16.18</version>
		</dependency>
```

        准备好提供服务的客户端类后，再编写运行类。运行类基本上与前面的HelloWorld类似，请见代码清单5-6。

        代码清单5-6：codes\05\5.1\feign-client\src\main\java\org\crazyit\cloud\PersonMain.java

```
package org.crazyit.cloud;

import org.crazyit.cloud.PersonClient.Person;

import feign.Feign;
import feign.gson.GsonDecoder;

/**
 * Person服务的运行主类
 * @author 杨恩雄
 *
 */
public class PersonMain {

	public static void main(String[] args) {
		PersonClient personService = Feign.builder()
				.decoder(new GsonDecoder())
				.target(PersonClient.class, "http://localhost:8080/");
		Person person = personService.findById(2);
		System.out.println(person.id);
		System.out.println(person.name);
		System.out.println(person.age);
		System.out.println(person.message);
	}
}

```

        调用Person服务的运行类中，添加了解码器的配置，GsonDecoder会将返回的JSON字符串，转换为接口方法返回的对象，关于解码器等内容，将在后面章节中讲述。运行代码清单5-6，可以看到最终的输出。

        本小节使用了CXF、Restlet、Feign来编写REST客户端，在编写客户端的过程中，可以看到Feign的代码更加“面向对象”，至于是否更加简洁，则见仁见智。下面的章节，将深入了解Feign的各项功能。

 







我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>




