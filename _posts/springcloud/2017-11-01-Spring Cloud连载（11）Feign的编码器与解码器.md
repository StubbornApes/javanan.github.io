---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509630308804&di=f7760a3abbdb1efb8837ddfc4dcc6453&imgtype=0&src=http%3A%2F%2Fwww.2cto.com%2Fuploadfile%2FCollfiles%2F20160820%2F201608200935191530.png
title: Spring Cloud连载（11）Feign的编码器与解码器
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
date:   2017-11-01 11:25:30
---

# **[本站小福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

原文作者：杨大仙的程序空间

本文要点

             Feign编码器与解码器

## 11.1 Feign的编码器与解码器

        本小节所有的案例都是单独使用Feign，Feign在Spring Cloud的使用将在后面章节讲述，请读者注意该细节。

### 5.2.1 编码器

        向服务发送请求的过程中，有些情况需要对请求的内容进行处理。例如服务端发布的服务接收的是JSON格式参数，而客户端使用的是对象，这种情况就可以使用编码器，将对象转换为JSON字符串。

        为服务端编写一个REST服务，处理POST请求，请见代码清单5-7。

        代码清单5-7：codes\05\5.1\rest-server\src\main\java\org\crazyit\cloud\MyController.java

```
	/**
	 * 参数为JSON
	 */
	@RequestMapping(value = "/person/create", method = RequestMethod.POST,
        consumes = MediaType.APPLICATION_JSON_VALUE)
	@ResponseBody
	public String createPerson(@RequestBody Person person) {
		System.out.println(person.getName() + "-" + person.getAge());
		return "Success, Person Id: " + person.getId();
	}
```

        控制器中，发布了一个“/person/create”的服务，需要传入JSON格式的请求参数。在客户端中，要调用该服务，先编写接口，再使用注解进行修饰，请见代码清单5-8。

        代码清单5-8：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\PersonClient.java

```
public interface PersonClient {

    @RequestLine("POST /person/create")
    @Headers("Content-Type: application/json")
    String createPerson(Person person);

    @Data
    class Person {
        Integer id;
        String name;
        Integer age;
        String message;
    }
}
```

        注意在客户端的服务接口中，使用了@Headers注解，声明请求的内容类型为JSON，接下来再编写运行类，如代码清单5-9。

        代码清单5-9：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\EncoderTest.java

```
/**
 * 运行主类
 * @author 杨恩雄
 *
 */
public class EncoderTest {

	public static void main(String[] args) {
		// 获取服务接口
		PersonClient personClient = Feign.builder()
				.encoder(new GsonEncoder())
				.target(PersonClient.class, "http://localhost:8080/");
		// 创建参数的实例
		Person person = new Person();
		person.id = 1;
		person.name = "Angus";
		person.age = 30;
		String response = personClient.createPerson(person);
		System.out.println(response);
	}
}
```

        运行类中，在创建服务接口实例时，使用了encoder方法来指定编码器，本案例使用了Feign提供的GsonEncoder类，该类会在发送请求过程中，将请求的对象转换为JSON字符串。Feign支持插件式的编码器，如果Feign提供的编码器无法满足要求，还可以使用自定义的编码器，这部分内容在后面章节讲述。启动服务，运行代码清单5-9，可看到服务已经调用成功，运行后输出如下：

```
Success, Person Id: 1
```

### 5.2.2 解码器

        编码器是对请求的内容进行处理，解码器则会对服务响应的内容进行处理，例如解析响应的JSON或者XML字符串，转换为我们所需要的对象，在代码中通过以下的代码片断设置解码器：

```
PersonClient personService = Feign.builder()
.decoder(new GsonDecoder())
.target(PersonClient.class, "http://localhost:8080/");
```

        在前面章节中，我们已经使用过GsonDecoder解码器，在此不再作赘述。

### 5.2.3 XML的编码与解码

        除了支持JSON的处理外，Feign还为XML的处理了提供编码器与解码器，可以使用JAXBEncoder与JAXBDecoder来进行编码与解码。为服务端添加发布XML的接口，请见代码清单5-10。

        代码清单5-10：codes\05\5.1\rest-server\src\main\java\org\crazyit\cloud\MyController.java

```
	/**
	 * 参数与返回值均为XML
	 */
	@RequestMapping(value = "/person/createXML", method = RequestMethod.POST,
			consumes = MediaType.APPLICATION_XML_VALUE,
			produces = MediaType.APPLICATION_XML_VALUE)
	@ResponseBody
	public String createXMLPerson(@RequestBody Person person) {
		System.out.println(person.getName() + "-" + person.getId());
		return "<result><message>success</message></result>";
	}
```

        在服务端发布的服务方法中，声明了传入的参数为XML。需要注意的是，服务端项目“rest-server”使用的是“spring-boot-starter-web”进行构建，默认情况下不支持XML接口，调用接口时会得到以下异常信息：

```
{"timestamp":1502705981406,"status":415,"error":"Unsupported Media Type","exception":"org.springframework.web.HttpMediaTypeNotSupportedException","message":"Content type 'application/xml;charset=UTF-8' not supported","path":"/person/createXML"}
```

        为服务端的pom.xml加入以下依赖即可解决该问题：

```
<dependency>
    <groupId>com.fasterxml.jackson.jaxrs</groupId>
    <artifactId>jackson-jaxrs-xml-provider</artifactId>
    <version>2.9.0</version>
</dependency>
```

        编写客户端时，先定义好服务接口以及对象，接口请见代码清单5-11。

        代码清单5-11：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\PersonClient.java

```
public interface PersonClient {

	@RequestLine("POST /person/createXML")
	@Headers("Content-Type: application/xml")
	Result createPersonXML(Person person);

	@Data
	@XmlRootElement
	class Person {
		@XmlElement
		Integer id;
		@XmlElement
		String name;
		@XmlElement
		Integer age;
		@XmlElement
		String message;
	}

	@Data
	@XmlRootElement
	class Result {
		@XmlElement
		String message;
	}
}

```

        在接口中，定义了“Content-Type”为XML，使用了JAXB的相关注解来修饰Person与Result。接下来，只需要调用createPersonXML方法即可请求服务，请见代码清单5-12。

        代码清单5-12：codes\05\5.2\feign-use\src\main\java\org\crazyit\feign\XMLTest.java

```
package org.crazyit.feign;

import org.crazyit.feign.PersonClient.Person;
import org.crazyit.feign.PersonClient.Result;

import feign.Feign;
import feign.jaxb.JAXBContextFactory;
import feign.jaxb.JAXBDecoder;
import feign.jaxb.JAXBEncoder;

public class XMLTest {

	public static void main(String[] args) {
		JAXBContextFactory jaxbFactory = new JAXBContextFactory.Builder().build();
		// 获取服务接口
		PersonClient personClient = Feign.builder()
				.encoder(new JAXBEncoder(jaxbFactory))
				.decoder(new JAXBDecoder(jaxbFactory))
				.target(PersonClient.class, "http://localhost:8080/");
		// 构建参数
		Person person = new Person();
		person.id = 1;
		person.name = "Angus";
		person.age = 30;
		// 调用接口并返回结果
		Result result = personClient.createPersonXML(person);
		System.out.println(result.message);
	}
}

```

        本小节的请求有一点特殊，请求服务时传入参数为XML、返回的结果也是XML，目的是为了能使编码与解码一起使用。开启服务，运行代码清单5-12，可以看到服务端与客户端的输出。

### 5.2.4 自定义编码器与解码器

        根据前面的两小节可知，Feign的插件式编码器与解码器，可以对请求以及结果进行处理，如果对于一些特殊的要求，可以使用自定义的编码器与解码器。实现自定义编码器，需要实现Encoder接口的encode方法，而对于解码器，则要实现Decoder接口的decode方法，例如以下的代码片断：

```
package org.crazyit.feign;

import java.lang.reflect.Type;

import feign.RequestTemplate;
import feign.codec.EncodeException;
import feign.codec.Encoder;

public class MyEncoder implements Encoder {

	public void encode(Object object, Type bodyType, RequestTemplate template)
			throws EncodeException {
		// 实现自己的Encode逻辑
	}
}

```

        在使用时，调用Feign的API来设置编码器或者解码器即可，实现较为简单，在此不再赘述。









我的官网
![我的博客](https://github.com/javanan/javanan.github.io/blob/master/style/images/slifelogo.png?raw=true)

我的官网<http://guan2ye.com>

我的CSDN地址<http://blog.csdn.net/chenjianandiyi>

我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>

我的github<https://github.com/javanan>

我的码云地址<https://gitee.com/jamen/>

阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>




