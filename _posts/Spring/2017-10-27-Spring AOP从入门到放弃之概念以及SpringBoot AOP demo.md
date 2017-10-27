---
layout: blog
istop: true
jishu: true
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509503652&di=cf9ab469d47904d5a9ab910f3134110f&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.sz886.com%2Feditor%2Fimage%2F20170507%2F20170507175839_7806.png
category: Spring
title: Spring AOP从入门到放弃之概念以及SpringBoot AOP demo
tags:
- AOP
- Spring AOP
- SpringBoot
- AOP概念
date: 2017-10-27 20:25:53
---


# AOP核心概念

## 1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

## 2、切面（aspect）-》（通知+切点）

类是对物体特征的抽象，切面就是对横切关注点的抽象。
通知+切点
意思就是所有要被应用到增强（advice）代码的地方。(包括方法的方位信息)


## 3、连接点（joinpoint）-》（被拦截的方法）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是**被拦截的方法**，实际上连接点还可以是字段或者构造器


## 4、切入点（pointcut）-》（描述拦截那些方法的部分）

对连接点进行拦截的定义


## 5、通知（advice）-》（拦截后执行自己业务逻辑的那些部分）

**所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类**
这玩意也叫  **增强**
在逻辑层次上包括了我们抽取的公共逻辑和方位信息。因为Spring只能方法级别的应用AOP,也就是我们常见的before,after,after-returning,after-throwing,around五种，意思就是在方法调用前后，异常时候执行我这段公共逻辑呗。


## 6、目标对象

代理的目标对象

## 7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程。
比如根据Advice中的方位信息在指定切点的方法前后，执行增强。这个过程Spring 替我们做好了。利用的是**CGLIB动态代理技术**。


## 8、引入（introduction）

在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段


# 图解
上面那一堆看不懂对吗？ 我也不太懂。
来看张图
![这里写图片描述](http://img.blog.csdn.net/20171027152055213?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



# 通知（Advice）类型

切面一共有五种通知

## Before 某方法调用之前发出通知。

前置通知（Before advice） ：在某连接点（JoinPoint）之前执行的通知， 但这个通知不能阻止连接点前的执行。**在方法调用之前发出通**

```
    @Before("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void beforeTest() {
        System.out.println("执行 方法 之前 调用----");
    }
```

## After 某方法完成之后发出通知

后通知（After advice） ：当某连接点退出的时候执行的通知（不论是正常 返回还是异常退出）。
不考虑方法运行的结果 。**在方法调用之后发出通**

```
    @After("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterTest() {
        System.out.println();
        System.out.println("执行 方法 之后 调用----");
    }
```


## After-returning 将通知放置在被通知的方法成功执行之后。

方法正常返回后，调用通知。**在方法调用后，正常退出发出通**

```
    @AfterReturning("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterReturningTest() {
        System.out.println();
        System.out.println("执行 方法 AfterReturning 调用----");
    }
```

## After-throwing 将通知放置在被通知的方法抛出异常之后。

抛出异常后通知（After throwing advice） ： 在方法抛出异常退出时执行 的通知。**在方法调用时，异常退出发出通**

```
    @AfterThrowing("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterThrowingTest() {
        System.out.println();
        System.out.println("执行 方法 AfterThrowing 调用----");
    }
```

## Around 通知包裹在被通知的方法的周围知。
环绕通知（Around advice） ：包围一个连接点的通知，类似Web中Servlet 规范中的Filter的doFilter方法。可以在方法的调用前后完成自定义的行为，也可以选择不执行。**在方法调用之前和之后发出通**

```
    @Around("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void aroundTest() {
        System.out.println();
        System.out.println("执行 方法 前后 调用----");
    }
```


## 执行结果

```
2017-10-27 19:51:51.605 DEBUG 15592 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Last-Modified value for [/aspecttest] is: -1
执行 方法 之前 调用----
JoinpointTest++++执行我正常流水线的业务逻辑

执行 方法 之后 调用----

执行 方法 AfterReturning 调用----
2017-10-27 19:51:51.622 DEBUG 15592 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Null ModelAndView returned to DispatcherServlet with name 'dispatcherServlet': assuming HandlerAdapter completed request handling
2017-10-27 19:51:51.622 DEBUG 15592 --- [io-8081-exec-10] o.s.web.servlet.DispatcherServlet        : Successfully completed request
```




# 切入点表达式
  **切入点指示符**用来指示切入点表达式目的，在Spring AOP中目前**只有执行方法这一个连接点**，Spring AOP支持的AspectJ切入点指示符如下：


```
args()
定制join-point去匹配那些参数为指定类型的方法的执行动作。

@args()
定制join-point去匹配那些参数被指定类型注解的方法的执行动作

execution()
开始匹配在其内部编写的定制

this()
定制join-pont去匹配由AOP代理的Bean引用的指定类型的类。

target()
定制join-point去匹配特定的对象，这些对象一定是指定类型的类。

@target()
定制join-point去匹配特定的对象，这些对象要具有的指定类型的注解。

within()
定制join-point在必须哪一个包中。

@within()
定制join-point在必须由指定注解标注的类中。

@annotation
定制连接点具有指定的注解。

只有execution用来执行匹配，其他标志符都只是为了限制/定制他们所要匹配的连接点的位置。
```


## 命名及匿名切入点

![这里写图片描述](http://img.blog.csdn.net/20171027202343548?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 类型匹配语法

`*`：匹配任何数量字符。

 `..`：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。

 `+`：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。

### 例子

```
java.lang.String    匹配String类型；

java.*.String       匹配java包下的任何“一级子包”下的String类型；
					如匹配java.lang.String，但不匹配java.lang.ss.String

java..*            匹配java包及任何子包下的任何类型;
                   如匹配java.lang.String、java.lang.annotation.Annotation

java.lang.*ing     匹配任何java.lang包下的以ing结尾的类型；

java.lang.Number+  匹配java.lang包下的任何Number的自类型；
                   如匹配java.lang.Integer，也匹配java.math.BigInteger
```
### 详细语法

```
注解？ 修饰符? 返回值类型 类型声明?方法名(参数列表) 异常列表？

注解：可选，方法上持有的注解，如@Deprecated；

修饰符：可选，如public、protected；

返回值类型：必填，可以是任何类型模式；“*”表示所有类型；

类型声明：可选，可以是任何类型模式；

方法名：必填，可以使用“*”进行模式匹配；

参数列表：“()”表示方法没有任何参数；“(..)”表示匹配接受任意个参数的方法，“(..,java.lang.String)”表示匹配接受java.lang.String类型的参数结束，且其前边可以接受有任意个参数的方法；“(java.lang.String,..)” 表示匹配接受java.lang.String类型的参数开始，且其后边可以接受任意个参数的方法；“(*,java.lang.String)” 表示匹配接受java.lang.String类型的参数结束，且其前边接受有一个任意类型参数的方法；

异常列表：可选，以“throws 异常全限定名列表”声明，异常全限定名列表如有多个以“，”分割，如throws java.lang.IllegalArgumentException, java.lang.ArrayIndexOutOfBoundsException。

匹配Bean名称：可以使用Bean的id或name进行匹配，并且可使用通配符“*”；
```

## 组合切入点表达式

 AspectJ使用 且（&&）、或（||）、非（！）来组合切入点表达式。在Schema风格下，由于在XML中使用“&&”需要使用转义字符“&amp;&amp;”来代替之，所以很不方便，因此Spring ASP 提供了and、or、not来代替&&、||、！。





## 通知参数

使用**JoinPoint**获取：Spring AOP提供使用org.aspectj.lang.JoinPoint类型获取连接点数据，任何通知方法的第一个参数都可以是JoinPoint(环绕通知是ProceedingJoinPoint，JoinPoint子类)，当然第一个参数位置也可以是JoinPoint.StaticPart类型，这个只返回连接点的静态部分。

![这里写图片描述](http://img.blog.csdn.net/20171027203826726?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# 运用场景

AOP 、IOC 做为Spring 的支柱，使用场景非常广泛。

## 1、日志记录

**[我的另一篇博客](http://www.guan2ye.com/2017/10/27/Spring-AOP%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83%E4%B9%8B%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B3%A8%E8%A7%A3%E6%94%B6%E9%9B%86%E7%B3%BB%E7%BB%9F%E6%97%A5%E5%BF%97.html)**

![这里写图片描述](http://img.blog.csdn.net/20171027200638922?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2、权限控制


##3、事务


## 4、多数据源读写切换

**[我的另一篇博客](http://www.guan2ye.com/2017/10/27/Spring-AOP%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83%E4%B9%8B%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E8%AF%BB%E5%86%99%E5%8A%A8%E6%80%81%E5%88%87%E6%8D%A2.html)**

![这里写图片描述](http://img.blog.csdn.net/20171027200712890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



# 原理

动态代理
Spring中AOP代理由Spring的IOC容器负责生成、管理，其依赖关系也由IOC容器负责管理。因此，AOP代理可以直接使用容器中的其它bean实例作为目标，这种关系可由IOC容器的依赖注入提供。Spring创建代理的规则为：

1、默认使用Java动态代理来创建AOP代理，这样就可以为任何接口实例创建代理了

2、当需要代理的类不是代理接口的时候，Spring会切换为使用CGLIB代理，也可强制使用CGLIB



# spring boot 项目中定义使用自己的aop

## 1、引入jar

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

## 2、编写切面

```
@Aspect // FOR AOP
@Order(-99) // 控制多个Aspect的执行顺序，越小越先执行
@Component
public class AdviceTest {

@Pointcut(value="execution(* com.slife.java8.aspect.AspectTest.test())")
    public void poincut(){

    }

    @Before("poincut()")
    public void beforeTest() {
        System.out.println("执行 方法 之前 调用----");
    }
}
```

这样就完成了在spring boot 项目中定义使用自己的aop





# 注意代理模式
CGLIB动态代理技术
有时候你会发现 你的配置和我一样，但是aop没有生效，这很有可能是SpringMVC的配置的代理模式不对。

## 问题描述
方法里的xxxService对象如果使用autowared注入，无法启动aspect，
但是
xxxService = ctx.getBean("xxxxx")获取，是可以启用aspect的

## 原因
这个时候 xxxService 并不是注入进来的，即使有 @Autowired 注解，这时的注解没有任何作用。
只有 Spring 生成的对象才有 AOP 功能，因为 Spring 生成的代理对象才有 AOP 功能。


## 解决方法

```
配置spring.aop.proxy-target-class=true
```

[不错的一个问题描述已经解决方法](http://blog.csdn.net/mmm333zzz/article/details/16858209)



# 文章代码

```
/**
 * Created by chen on 2017/10/27.
 * <p>
 * Email 122741482@qq.com
 * <p>
 * Describe:
 */
@Service
public class JoinpointTest {

    public void JoinpointTest(){
        System.out.println("**********JoinpointTest*****************");
    }

    public void test(){
        System.out.println("JoinpointTest++++执行我正常流水线的业务逻辑");
    }
}

```


```
@Aspect // FOR AOP
@Order(-99) // 控制多个Aspect的执行顺序，越小越先执行
@Component
public class AdviceTest {

@Pointcut(value="execution(* com.slife.java8.aspect.AspectTest.test())")
    public void poincut(){

    }

    @Before("poincut()")
    public void beforeTest() {
        System.out.println("执行 方法 之前 调用----");
    }

    @After("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterTest() {
        System.out.println();
        System.out.println("执行 方法 之后 调用----");
    }

    @Around("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void aroundTest() {
        System.out.println();
        System.out.println("执行 方法 前后 调用----");
    }



    @AfterReturning("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterReturningTest() {
        System.out.println();
        System.out.println("执行 方法 AfterReturning 调用----");
    }



    @AfterThrowing("execution(* com.slife.java8.aspect.AspectTest.test())")
    public void afterThrowingTest() {
        System.out.println();
        System.out.println("执行 方法 AfterThrowing 调用----");
    }


    @Before("execution(* com.slife.java8..*test*(..))")
    public void aspecttest1() {
        System.out.println();
        System.out.println("执行 方法aspecttest1  Before 调用----");
    }


}

```


# **[点击获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**


