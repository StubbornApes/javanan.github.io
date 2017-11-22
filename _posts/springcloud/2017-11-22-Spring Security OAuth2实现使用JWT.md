---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1510224935&di=9f3a08de5dadc9c282de3f02a7a003f6&imgtype=jpg&er=1&src=http%3A%2F%2Fs2.51cto.com%2Fwyfs02%2FM02%2F8E%2F8D%2FwKiom1jFACCx2lbgAAEWcb8v6-w59.jpeg-wh_651x-s_2509118232.jpeg
title: Spring Security OAuth2实现使用JWT
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
- Hystrix
- OAuth2
- JWT
date:   2017-11-22 11:25:54
---


# 1、概括

在博客中，我们将讨论如何让Spring Security OAuth2实现使用JSON Web Tokens。


# 2、Maven 配置

首先，我们需要在我们的pom.xml中添加spring-security-jwt依赖项。
```
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
</dependency>
```

我们需要为Authorization Server和Resource Server添加spring-security-jwt依赖项。


# 3、授权服务器

接下来，我们将配置我们的授权服务器使用JwtTokenStore - 如下所示

```
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore())
                 .accessTokenConverter(accessTokenConverter())
                 .authenticationManager(authenticationManager);
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        return defaultTokenServices;
    }
}
```

请注意，在JwtAccessTokenConverter中使用了一个对称密钥来签署我们的令牌 - 这意味着我们需要为资源服务器使用同样的确切密钥。


# 4、资源服务器

现在，我们来看看我们的资源服务器配置 - 这与授权服务器的配置非常相似：

```
@Configuration
@EnableResourceServer
public class OAuth2ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(ResourceServerSecurityConfigurer config) {
        config.tokenServices(tokenServices());
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("123");
        return converter;
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        return defaultTokenServices;
    }
}
```
请记住，我们将这两个服务器定义为完全独立且可独立部署的服务器。这就是我们需要在新配置中再次声明一些相同的bean的原因。

# 5、令牌中的自定义声明

现在让我们设置一些基础设施，以便能够在访问令牌中添加一些自定义声明。框架提供的标准声明都很好，但大多数情况下我们需要在令牌中使用一些额外的信息来在客户端使用。 我们将定义一个TokenEnhancer来定制我们的Access Token与这些额外的声明。 在下面的例子中，我们将添加一个额外的字段“组织”到我们的访问令牌 - 与此CustomTokenEnhancer：

```
public class CustomTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(
     OAuth2AccessToken accessToken,
     OAuth2Authentication authentication) {
        Map<String, Object> additionalInfo = new HashMap<>();
        additionalInfo.put("organization", authentication.getName() + randomAlphabetic(4));
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
        return accessToken;
    }
}
```

然后，我们将把它连接到我们的授权服务器配置 - 如下所示：

```
@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
    tokenEnhancerChain.setTokenEnhancers(
      Arrays.asList(tokenEnhancer(), accessTokenConverter()));

    endpoints.tokenStore(tokenStore())
             .tokenEnhancer(tokenEnhancerChain)
             .authenticationManager(authenticationManager);
}

@Bean
public TokenEnhancer tokenEnhancer() {
    return new CustomTokenEnhancer();
}
```

有了这个新的配置启动和运行 - 这是一个令牌令牌有效载荷看起来像：

```
{
    "user_name": "john",
    "scope": [
        "foo",
        "read",
        "write"
    ],
    "organization": "johnIiCh",
    "exp": 1458126622,
    "authorities": [
        "ROLE_USER"
    ],
    "jti": "e0ad1ef3-a8a5-4eef-998d-00b26bc2c53f",
    "client_id": "fooClientIdPassword"
}
```

## 5.1、在JS客户端使用访问令牌

最后，我们要在AngualrJS客户端应用程序中使用令牌信息。我们将使用angular-jwt库。 所以我们要做的就是在index.html中使用“组织”声明：

```
<p class="navbar-text navbar-right">{{organization}}</p>

<script type="text/javascript"
  src="https://cdn.rawgit.com/auth0/angular-jwt/master/dist/angular-jwt.js">
</script>

<script>
var app = angular.module('myApp', ["ngResource","ngRoute", "ngCookies", "angular-jwt"]);

app.controller('mainCtrl', function($scope, $cookies, jwtHelper,...) {
    $scope.organiztion = "";

    function getOrganization(){
        var token = $cookies.get("access_token");
        var payload = jwtHelper.decodeToken(token);
        $scope.organization = payload.organization;
    }
    ...
});
```

# 6、不对称的KeyPair

在我们以前的配置中，我们使用对称密钥来签署我们的令牌：

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("123");
    return converter;
}
```
我们还可以使用非对称密钥（公钥和私钥）来执行签名过程。


## 6.1、生成JKS Java KeyStore文件
我们首先使用命令行工具keytool生成密钥 - 更具体地说.jks文件：

```
keytool -genkeypair -alias mytest
                    -keyalg RSA
                    -keypass mypass
                    -keystore mytest.jks
                    -storepass mypass
```

该命令将生成一个名为mytest.jks的文件，其中包含我们的密钥 - 公钥和私钥。 还要确保keypass和storepass是一样的。

## 6.2、导出公钥

接下来，我们需要从生成的JKS中导出我们的公钥，我们可以使用下面的命令来实现：

```
keytool -list -rfc --keystore mytest.jks | openssl x509 -inform pem -pubkey
```

示例回应如下所示：

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
MIIDCzCCAfOgAwIBAgIEGtZIUzANBgkqhkiG9w0BAQsFADA2MQswCQYDVQQGEwJ1
czELMAkGA1UECBMCY2ExCzAJBgNVBAcTAmxhMQ0wCwYDVQQDEwR0ZXN0MB4XDTE2
MDMxNTA4MTAzMFoXDTE2MDYxMzA4MTAzMFowNjELMAkGA1UEBhMCdXMxCzAJBgNV
BAgTAmNhMQswCQYDVQQHEwJsYTENMAsGA1UEAxMEdGVzdDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAICCtlreMdhLQ5eNQu736TrDKrmTMjsrXjtkbFXj
Cxf4VyHmL4nCq9EkM1ZKHRxAQjIhl0A8+aa4o06t0Rz8tv+ViQQmKu8h4Ey77KTM
urIr1zezXWBOyOaV6Pyh5OJ8/hWuj9y/Pi/dBP96sH+o9wylpwICRUWPAG0mF7dX
eRC4iBtf4BKswtH2ZjYYX6wbccFl65aVA09Cn739EFZj0ccQi10/rRHtbHlhhKnj
iy+b10S6ps2XAXtUWfZEEJuN/mvUJ+YnEkZw30wHrENwq5QFiSpdpHFlNR8CasPn
WUUmdV+JBFzTMsz3TwWxplOjB3YacsCO0imU+5l+AQ51CnkCAwEAAaMhMB8wHQYD
VR0OBBYEFOGefUBGquEX9Ujak34PyRskHk+WMA0GCSqGSIb3DQEBCwUAA4IBAQB3
1eLfNeq45yO1cXNl0C1IQLknP2WXg89AHEbKkUOA1ZKTOizNYJIHW5MYJU/zScu0
yBobhTDe5hDTsATMa9sN5CPOaLJwzpWV/ZC6WyhAWTfljzZC6d2rL3QYrSIRxmsp
/J1Vq9WkesQdShnEGy7GgRgJn4A8CKecHSzqyzXulQ7Zah6GoEUD+vjb+BheP4aN
hiYY1OuXD+HsdKeQqS+7eM5U7WW6dz2Q8mtFJ5qAxjY75T0pPrHwZMlJUhUZ+Q2V
FfweJEaoNB9w9McPe1cAiE+oeejZ0jq0el3/dJsx3rlVqZN+lMhRJJeVHFyeb3XF
lLFCUGhA7hxn2xf3x1JW
-----END CERTIFICATE-----
```

我们只取得我们的公钥，并将其复制到我们的资源服务器src / main / resources / public.txt中

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAgIK2Wt4x2EtDl41C7vfp
OsMquZMyOyteO2RsVeMLF/hXIeYvicKr0SQzVkodHEBCMiGXQDz5prijTq3RHPy2
/5WJBCYq7yHgTLvspMy6sivXN7NdYE7I5pXo/KHk4nz+Fa6P3L8+L90E/3qwf6j3
DKWnAgJFRY8AbSYXt1d5ELiIG1/gEqzC0fZmNhhfrBtxwWXrlpUDT0Kfvf0QVmPR
xxCLXT+tEe1seWGEqeOLL5vXRLqmzZcBe1RZ9kQQm43+a9Qn5icSRnDfTAesQ3Cr
lAWJKl2kcWU1HwJqw+dZRSZ1X4kEXNMyzPdPBbGmU6MHdhpywI7SKZT7mX4BDnUK
eQIDAQAB
-----END PUBLIC KEY-----
```

## 6.3、Maven 配置

接下来，我们不希望JMS文件被maven过滤进程拾取 - 所以我们将确保将其排除在pom.xml中：

```
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>*.jks</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

如果我们使用Spring Boot，我们需要确保我们的JKS文件通过Spring Boot Maven插件添加到应用程序classpath - addResources：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## 6.4、授权服务器

现在，我们将配置JwtAccessTokenConverter使用mytest.jks中的KeyPair，如下所示：

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    KeyStoreKeyFactory keyStoreKeyFactory =
      new KeyStoreKeyFactory(new ClassPathResource("mytest.jks"), "mypass".toCharArray());
    converter.setKeyPair(keyStoreKeyFactory.getKeyPair("mytest"));
    return converter;
}
```

## 6.5、资源服务器

最后，我们需要配置我们的资源服务器使用公钥 - 如下所示：

```
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    Resource resource = new ClassPathResource("public.txt");
    String publicKey = null;
    try {
        publicKey = IOUtils.toString(resource.getInputStream());
    } catch (final IOException e) {
        throw new RuntimeException(e);
    }
    converter.setVerifierKey(publicKey);
    return converter;
}
```

# 7、结论

放弃吧~~~~~~


# **[博客福利：点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=vf2b5zld&utm_source=vf2b5zld)**


# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
