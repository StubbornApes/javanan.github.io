---
layout: blog
istop: true
jishu: true
category: Spring Boot
background-image: http://img.blog.csdn.net/20171117001831707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast
title: Spring Boot 搭建的一个企业级快速开发脚手架
tags:
- 阿里云
- Security
- 阿里云优惠券
- 阿里云优惠
- ECS
- RDS
- OSS
- 建站
- 云服务器
- Spring Cloud
- Spring Boot
- OAuth2
- JWT
- 脚手架
date:   2017-11-22 11:24:54
---



#slife
spring boot 搭建的一个企业级快速开发脚手架。

这本来是我自己平时测试用的项目，没打算开源。
但今天放到 开源中国 和 GitHub 没想到会被 码云设置为推荐项目。并且还上了今日热门项目 第一名
![这里写图片描述](http://img.blog.csdn.net/20171117000240329?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# [ 項目地址https://gitee.com/jamen/slife](https://gitee.com/jamen/slife)

###技术栈
1. Spring Boot <p>
2. MySQL<p>
3. Freemark <p>
4. SiteMesh <p>
5. Shiro  <p>
6. Boostrapt <p>
7. mybatis、mybatisPlus <p>
8. redis <p>
9. Activiti <p>


#编码约定
系统分为controller、service、dao层。
controller主要负责转发、service主要负责业务逻辑、dao主要是数据库的操作。


###文件名称约定
在页面文件夹中，按照功能模块分别建立不同的文件夹存放页面，如用户的页面就放在user文件夹中，而角色的就放在role文件夹中。
1. 页面如果是列表类型的。页面的文件名用list.ftl命名。
2. 页面如果是详情类型的。页面的文件名用detail.ftl命名。

###controller、service、dao方法名称约定
1. 如果是增加数据操作用insert做前缀。
2. 如果是删除操作用delete做前缀
3. 如果是修改操作用update做前缀
6. 如果是查询操作用select做前缀


#数据库读写分离




#缓存ecache、redis




#新建模块
1. new Module <br>
2. GroupId --->com.slife  <br>
3. ArtifactId---> slife-模块名称   如  slife-activiti     <br>
4. Version --> 版本号   如 1.0SNAPSHOT <br>
5. Module-Name--> slife-模块名称   如  slife-activiti     <br>
6. 提交新建模块  <br>
7. pom 文件引入
```
    <name>slife-模块名称</name>

    <dependencies>
        <dependency>
            <groupId>com.slife</groupId>
            <artifactId>slife-common</artifactId>
        </dependency>

        .
        .
        .其他的依赖
        .
    </dependencies>
```


#JDK版本 1.8
```

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <compilerArgs>
                        <arg>-parameters</arg>
                    </compilerArgs>
                    <useIncrementalCompilation>false</useIncrementalCompilation>
                </configuration>
            </plugin>
        </plugins>
    </build>

```




#新建一个功能模块
1、创建数据库

2、创建entity类

3、创建service类

4、创建controller类

5、创建list界面
```

5.1 到其他list复制代码过


5.2 修改
 <script>
        var url = "${base}/sys/user/";
 </script>

 中的 url 为你刚刚创建的 controller的类
 @Controller
 @RequestMapping(value = "/sys/user")
 public class SysUserController extends BaseController {

 的  @RequestMapping(value = "/sys/user") 值



5.3 修改搜索条件
目前的搜索条件有
    /**
     * 等于
     */
    public static final String SEARCH_EQ="search_eq_";

    /**
     * 左模糊
     */
    public static final String SEARCH_LLIKE="search_llike_";

    /**
     * 右模糊
     */
    public static final String SEARCH_RLIKE="search_rlike_";

    /***
     * 全模糊
     */
    public static final String SEARCH_LIKE="search_like_";



     <input type="text" class="form-filter input-sm _search" name="search_eq_login_name">

     只要在  input中 的 name 加入 search_eq_ 前缀 再加数据库中的字段名称即可



5.4 修改表格的字段名称

```






# 项目截图介绍

## 系统用户管理
![这里写图片描述](http://img.blog.csdn.net/20171117001831707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171117001847740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 系统菜单管理
![这里写图片描述](http://img.blog.csdn.net/20171117001902072?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20171117001920829?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 系统角色管理

    RBAC权限管理模型
![这里写图片描述](http://img.blog.csdn.net/20171117001942150?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 日志监控

    系统自定义注解，结合AOP，监控用户操作行为

![这里写图片描述](http://img.blog.csdn.net/20171117002000218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## API文档

    swaggerUi接口文档展示

![这里写图片描述](http://img.blog.csdn.net/20171117002017884?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 数据库监控

    使用druid监控数据库健康。本来这里是三个数据源的，使用aop动态的书写切换。没上传到git，需要的同学可以私我

![这里写图片描述](http://img.blog.csdn.net/20171117002033093?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## maven构建 多模块开发

    根据不同的业务，不在不同的业务模块中开发，如果基本的用户、组织等的管理在 sys模块
    日志的业务逻辑在 log模块

![这里写图片描述](http://img.blog.csdn.net/20171117002057751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
**可插拔式部署**
    把不同的模块打包成jar，对应的freemark文件也打包在对应的模块jar中。实现了功能模块的可插拔式部署。

![这里写图片描述](http://img.blog.csdn.net/20171117002126619?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 联系方式

qq群421351927 <a target="_blank" href="//shang.qq.com/wpa/qunwpa?idkey=3d00028ad6bec03e99d0491e6fb055b3edbd5be3ef9ab5adbafb8a13851ba7eb"><img border="0" src="//pub.idqqimg.com/wpa/images/group.png" alt="SLife" title="SLife"></a>


<br>


# **[福利 点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**


<br>


# 个人资源
我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>

<br>


# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



















