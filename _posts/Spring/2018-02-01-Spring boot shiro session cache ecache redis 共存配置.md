---
layout: blog
istop: true
jishu: true
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509503652&di=cf9ab469d47904d5a9ab910f3134110f&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.sz886.com%2Feditor%2Fimage%2F20170507%2F20170507175839_7806.png
category: Spring
title: Spring boot shiro session cache ecache redis 共存配置
tags:
- Spring boot
- shiro
- session
- cache
- redis
date: 2018-02-01 20:25:54
---

# SecurityManager

对应 shiro来说 SecurityManager  非常重要，这里配置了
Realm
CacheManager
RememberMeManager
sessionManager
可以说是shiro的核心

我们今天就是要 配置 sessionManager  和 CacheManager
让 ecache和redis来缓存 session和 AuthorizationInfo的数据信息
使得应用能集群部署，多个机器之间共享session和 认证授权数据信息。


## SecurityManager的配置
```
    //配置核心安全事务管理器
    @Bean
    public SecurityManager securityManager(@Qualifier("authRealm")AuthRealm authRealm,@Qualifier("redisCacheManager")CacheManager
            cacheManager) {

       logger.info("--------------shiro已经加载----------------");
        DefaultWebSecurityManager manager=new DefaultWebSecurityManager();
        // 设置realm.
        manager.setRealm(authRealm);

        //注入缓存管理器;
        //注意:开发时请先关闭，如不关闭热启动会报错
        manager.setCacheManager(cacheManager);//这个如果执行多次，也是同样的一个对象;
        //注入记住我管理器;
        manager.setRememberMeManager(rememberMeManager());


        return manager;
    }
```

这里配置了

```
 manager.setCacheManager(cacheManager)
```

#  CacheConfig 配置 ehCacheManager 和redisCacheManager

```
@Configuration
@EnableCaching
public class CacheConfig {
    private Logger logger = org.slf4j.LoggerFactory.getLogger(getClass());

    @Bean(name = "ehCacheManager")
    public EhCacheManager ehCacheManager() {
        logger.info("--------------ehCacheManager init---------------");
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:cache/ehcache-shiro.xml");
        logger.info("--------------ehCacheManager init---------------"+cacheManager);
        return cacheManager;
    }

    @Autowired
    private RedisTemplate redisTemplate;

    @Bean(name = "redisCacheManager")
    @Primary
    public RedisCacheManager redisCacheManager() {
        logger.info("--------------redis cache init---------------");
        RedisCacheManager cacheManager = new RedisCacheManager();
        cacheManager.setRedisTemplate(redisTemplate);
        logger.info("--------------redis cache ---------------"+cacheManager);
        return cacheManager;
    }
}

```


## 使用 ecache做缓存

1 、那么 加入依赖

```
   <dependency>
            <groupId>org.ehcache</groupId>
            <artifactId>ehcache</artifactId>
            <version>${ehcache.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache-core</artifactId>
            <version>${ehcache.core.version}</version>
        </dependency>
        <!-- shiro ehcache -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>${shiro.version}</version>
        </dependency>
```

2、配置 cacheManager

```
    @Bean(name = "ehCacheManager")
    public EhCacheManager ehCacheManager() {
        logger.info("--------------ehCacheManager init---------------");
        EhCacheManager cacheManager = new EhCacheManager();
        cacheManager.setCacheManagerConfigFile("classpath:cache/ehcache-shiro.xml");
        logger.info("--------------ehCacheManager init---------------"+cacheManager);
        return cacheManager;
    }
```

3、配置 ehcache-shiro.xml

```
<ehcache name="shiroCache">
    <!-- 设置缓存文件 .data 的创建路径。
    如果该路径是 Java 系统参数，当前虚拟机会重新赋值。
    下面的参数这样解释：
    user.home      – 用户主目录
    user.dir       – 用户当前工作目录
    java.io.tmpdir – 默认临时文件路径 -->
    <diskStore path="D:/test/ehcache" />
    <cacheManagerEventListenerFactory class="" properties="" />
    <!--缺省缓存配置。CacheManager 会把这些配置应用到程序中。
        下列属性是 defaultCache 必须的:
        maxElementsInMemory：设置基于内存的缓存可存放对象的最大数目。
        maxElementsOnDisk：设置基于硬盘的缓存可存放对象的最大数目。
        maxInMemory       - 缓存可以存储的总记录量
        eternal           - 缓存是否永远不销毁.如果是，超时设置将被忽略，对象从不过期
        timeToIdleSeconds - 当缓存闲置时间(秒)超过该值，则缓存自动销毁
        timeToLiveSeconds - 缓存创建之后，到达该时间(秒)缓存自动销毁
        overflowToDisk    - 当缓存中的数据达到最大值时，是否把缓存数据写入磁盘.
        -->
    <cache name="books"
           maxElementsInMemory="50000"
           timeToLiveSeconds="25">
    </cache>
    <defaultCache maxElementsInMemory="10000" eternal="false" timeToIdleSeconds="20" timeToLiveSeconds="25"
                  overflowToDisk="false" diskPersistent="false" diskExpiryThreadIntervalSeconds="20" />
</ehcache>
```

4、SecurityManager 注入  ehCacheManager
 @Qualifier("ehCacheManager")CacheManager


## 使用redis做缓存 auth数据
使用redis比较麻烦一些，没有用

```
    <!-- https://mvnrepository.com/artifact/org.crazycake/shiro-redis -->
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>2.4.6</version>
</dependency>


```
这样的 jar  只能自己来弄一个 cache 和 cacheManager
当然 也可以使用 shiro-redis  这样就不需要自己实现 cache 和 cacheManager

我们先看自己实现的方式

1、实现Cache<K, V>

```
public class RedisCache<K, V> implements Cache<K, V> {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    private static final String REDIS_SHIRO_CACHE = "slife-shiro-cache:";
    private String cacheKey;
    private RedisTemplate<K, V> redisTemplate;
    private long globExpire = 30;

    @SuppressWarnings("rawtypes")
    public RedisCache(String name, RedisTemplate client) {
        this.cacheKey = REDIS_SHIRO_CACHE + name + ":";
        this.redisTemplate = client;
    }

    @Override
    public V get(K key) throws CacheException {
        redisTemplate.boundValueOps(getCacheKey(key)).expire(globExpire, TimeUnit.MINUTES);
        return redisTemplate.boundValueOps(getCacheKey(key)).get();
    }

    @Override
    public V put(K key, V value) throws CacheException {
        V old = get(key);
        redisTemplate.boundValueOps(getCacheKey(key)).set(value);
        return old;
    }

    @Override
    public V remove(K key) throws CacheException {
        V old = get(key);
        redisTemplate.delete(getCacheKey(key));
        return old;
    }

    @Override
    public void clear() throws CacheException {
        redisTemplate.delete(keys());
    }

    @Override
    public int size() {
        return keys().size();
    }

    @Override
    public Set<K> keys() {
        return redisTemplate.keys(getCacheKey("*"));
    }

    @Override
    public Collection<V> values() {
        Set<K> set = keys();
        List<V> list = new ArrayList<>();
        for (K s : set) {
            list.add(get(s));
        }
        return list;
    }

    private K getCacheKey(Object k) {
        return (K) (this.cacheKey + k);
    }
}
```


2、实现 CacheManager

```
public class RedisCacheManager implements CacheManager {

    private static final Logger logger = LoggerFactory.getLogger(RedisCacheManager.class);


    private RedisTemplate redisTemplate;

    @Override
    public <K, V> Cache<K, V> getCache(String name) throws CacheException {
        return new RedisCache<K, V>(name, redisTemplate);
    }

    public RedisTemplate<String, Object> getRedisTemplate() {
        return redisTemplate;
    }

    public void setRedisTemplate(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }


}
```

3、注入RedisCacheManager  到容器

```
 @Autowired
    private RedisTemplate redisTemplate;

    @Bean(name = "redisCacheManager")
    @Primary
    public RedisCacheManager redisCacheManager() {
        logger.info("--------------redis cache init---------------");
        RedisCacheManager cacheManager = new RedisCacheManager();
        cacheManager.setRedisTemplate(redisTemplate);
        logger.info("--------------redis cache ---------------"+cacheManager);
        return cacheManager;
    }
```

4、SecurityManager 注入  redisCacheManager
 @Qualifier("redisCacheManager")CacheManager

ok 这样 就搞定了 redis和ecache 缓存shrio的认证授权数据。



接下来看看 怎么用使用redis进行基于shiro的session集群共享

# redis进行基于shiro的session集群共享


1、加入依赖

```
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>
```


2、 配置session

```
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30) //maxInactiveIntervalInSeconds: 设置Session失效时间，使用Redis Session之后，原Boot的server.session.timeout属性不再生效
public class SessionConfig {
}
```

这样 就可以了



#  shiro-redis jar 缓存 shiro数据

1、加入依赖

```
<!-- https://mvnrepository.com/artifact/org.crazycake/shiro-redis -->
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>2.4.6</version>
</dependency>
```

2、 配置  redisCacheManager1

```
    @Bean(name = "redisCacheManager1")
    @Primary
    public RedisCacheManager redisCacheManager1() {
        logger.info("--------------redis cache init---------------");
        RedisCacheManager redisCacheManager=  new RedisCacheManager();
        redisCacheManager.setRedisManager(redisManager());
        logger.info("--------------redis cache ---------------"+redisCacheManager);
        return redisCacheManager;
    }


    public RedisManager redisManager() {
        RedisManager redisManager= new RedisManager();
        redisManager.setHost("119.29.53.182");
        redisManager.setPassword("123456");
        redisManager.setPort(6379);
        return redisManager;
    }
```

3、SecurityManager 注入  redisCacheManager1
 @Qualifier("redisCacheManager1")CacheManager
 ok 这样就搞定了


# 自己实现一些 操作 ecahe和redis的工具类







我的官网
![我的博客](http://upload-images.jianshu.io/upload_images/2830896-69dc8891bfc3cd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld>
# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# **[我的开源项目spring boot 搭建的一个企业级快速开发脚手架](https://gitee.com/jamen/slife)**
![1.jpg](http://upload-images.jianshu.io/upload_images/2830896-66de965f818533c5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# qq群 421351927  大家可以相互学习。



