---
layout: blog
istop: true
jishu: true
category: Spring Cloud
background-image: http://blog.springcloud.cn/images/sc-lx/slueth.png
title: spring security oauth2框架整合的例子
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
date:   2017-12-04 11:24:54
---





## 一、本文简介

本文主要讲解Java编程中spring boot框架+spring security框架+spring security oauth2框架整合的例子,并且oauth2整合使用jwt方式存储

## 二、学习前提

首先是讲解oauth2的基本说明:

推荐查看:

http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html

上面的地址可以基本了解下oauth2的原理

JWT的认知和基本使用:

推荐查看:

1.

什么是JWT?

2.

Java编程中java-jwt框架使用

## 三、代码编辑

开始代码:

认证服务:

几个核心的配置类:

AuthorizationServerConfiguration.java

```
package com.leftso.config.security;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

/**
 * 认证授权服务端
 *
 * @author leftso
 *
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

	@Value("${resource.id:spring-boot-application}") // 默认值spring-boot-application
	private String resourceId;

	@Value("${access_token.validity_period:3600}") // 默认值3600
	int accessTokenValiditySeconds = 3600;

	@Autowired
	private AuthenticationManager authenticationManager;

	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		endpoints.authenticationManager(this.authenticationManager);
		endpoints.accessTokenConverter(accessTokenConverter());
		endpoints.tokenStore(tokenStore());
	}

	@Override
	public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
		oauthServer.tokenKeyAccess("isAnonymous() || hasAuthority('ROLE_TRUSTED_CLIENT')");
		oauthServer.checkTokenAccess("hasAuthority('ROLE_TRUSTED_CLIENT')");
	}

	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
		clients.inMemory().withClient("normal-app").authorizedGrantTypes("authorization_code", "implicit")
				.authorities("ROLE_CLIENT").scopes("read", "write").resourceIds(resourceId)
				.accessTokenValiditySeconds(accessTokenValiditySeconds).and().withClient("trusted-app")
				.authorizedGrantTypes("client_credentials", "password").authorities("ROLE_TRUSTED_CLIENT")
				.scopes("read", "write").resourceIds(resourceId).accessTokenValiditySeconds(accessTokenValiditySeconds)
				.secret("secret");
	}

	/**
	 * token converter
	 *
	 * @return
	 */
	@Bean
	public JwtAccessTokenConverter accessTokenConverter() {
		JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter() {
			/***
			 * 重写增强token方法,用于自定义一些token返回的信息
			 */
			@Override
			public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
				String userName = authentication.getUserAuthentication().getName();
				User user = (User) authentication.getUserAuthentication().getPrincipal();// 与登录时候放进去的UserDetail实现类一直查看link{SecurityConfiguration}
				/** 自定义一些token属性 ***/
				final Map<String, Object> additionalInformation = new HashMap<>();
				additionalInformation.put("userName", userName);
				additionalInformation.put("roles", user.getAuthorities());
				((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInformation);
				OAuth2AccessToken enhancedToken = super.enhance(accessToken, authentication);
				return enhancedToken;
			}

		};
		accessTokenConverter.setSigningKey("123");// 测试用,资源服务使用相同的字符达到一个对称加密的效果,生产时候使用RSA非对称加密方式
		return accessTokenConverter;
	}

	/**
	 * token store
	 *
	 * @param accessTokenConverter
	 * @return
	 */
	@Bean
	public TokenStore tokenStore() {
		TokenStore tokenStore = new JwtTokenStore(accessTokenConverter());
		return tokenStore;
	}
}
```

SecurityConfiguration.java

```
package com.leftso.config.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import com.leftso.entity.Account;
import com.leftso.repository.AccountRepository;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
	// 查询用户使用
	@Autowired
	AccountRepository accountRepository;

	@Autowired
	public void globalUserDetails(AuthenticationManagerBuilder auth) throws Exception {
		// auth.inMemoryAuthentication()
		// .withUser("user").password("password").roles("USER")
		// .and()
		// .withUser("app_client").password("nopass").roles("USER")
		// .and()
		// .withUser("admin").password("password").roles("ADMIN");
		//配置用户来源于数据库
		auth.userDetailsService(userDetailsService());
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().antMatchers(HttpMethod.OPTIONS).permitAll().anyRequest().authenticated().and()
				.httpBasic().and().csrf().disable();
	}

	@Override
	@Bean
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}

	@Bean
	public UserDetailsService userDetailsService() {
		return new UserDetailsService() {
			@Override
			public UserDetails loadUserByUsername(String name) throws UsernameNotFoundException {
				// 通过用户名获取用户信息
				Account account = accountRepository.findByName(name);
				if (account != null) {
					// 创建spring security安全用户
					User user = new User(account.getName(), account.getPassword(),
							AuthorityUtils.createAuthorityList(account.getRoles()));
					return user;
				} else {
					throw new UsernameNotFoundException("用户[" + name + "]不存在");
				}
			}
		};

	}
}
```

受保护的资源服务:

核心配置:

SecurityConfiguration.java

```
package com.leftso.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers(HttpMethod.OPTIONS).permitAll()
                .anyRequest().authenticated()
                .and().httpBasic()
                .and().csrf().disable();
    }

}

```

ResourceServerConfiguration.java

```
package com.leftso.config;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.DefaultTokenServices;
import org.springframework.security.oauth2.provider.token.ResourceServerTokenServices;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;
import org.springframework.security.web.util.matcher.RequestMatcher;

/**
 * 资源服务端
 *
 * @author leftso
 *
 */
@Configuration
@EnableResourceServer
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {

	@Value("${resource.id:spring-boot-application}")
	private String resourceId;

	@Override
	public void configure(ResourceServerSecurityConfigurer resources) {
		// @formatter:off
		resources.resourceId(resourceId);
		resources.tokenServices(defaultTokenServices());
		// @formatter:on
	}

	@Override
	public void configure(HttpSecurity http) throws Exception {
		// @formatter:off
		http.requestMatcher(new OAuthRequestedMatcher()).authorizeRequests().antMatchers(HttpMethod.OPTIONS).permitAll()
				.anyRequest().authenticated();
		// @formatter:on
	}

	private static class OAuthRequestedMatcher implements RequestMatcher {
		public boolean matches(HttpServletRequest request) {
			String auth = request.getHeader("Authorization");
			// Determine if the client request contained an OAuth Authorization
			boolean haveOauth2Token = (auth != null) && auth.startsWith("Bearer");
			boolean haveAccessToken = request.getParameter("access_token") != null;
			return haveOauth2Token || haveAccessToken;
		}
	}

	// ===================================================以下代码与认证服务器一致=========================================
	/**
	 * token存储,这里使用jwt方式存储
	 *
	 * @param accessTokenConverter
	 * @return
	 */
	@Bean
	public TokenStore tokenStore() {
		TokenStore tokenStore = new JwtTokenStore(accessTokenConverter());
		return tokenStore;
	}

	/**
	 * Token转换器必须与认证服务一致
	 *
	 * @return
	 */
	@Bean
	public JwtAccessTokenConverter accessTokenConverter() {
		JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter() {
//			/***
//			 * 重写增强token方法,用于自定义一些token返回的信息
//			 */
//			@Override
//			public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
//				String userName = authentication.getUserAuthentication().getName();
//				User user = (User) authentication.getUserAuthentication().getPrincipal();// 与登录时候放进去的UserDetail实现类一直查看link{SecurityConfiguration}
//				/** 自定义一些token属性 ***/
//				final Map<String, Object> additionalInformation = new HashMap<>();
//				additionalInformation.put("userName", userName);
//				additionalInformation.put("roles", user.getAuthorities());
//				((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInformation);
//				OAuth2AccessToken enhancedToken = super.enhance(accessToken, authentication);
//				return enhancedToken;
//			}

		};
		accessTokenConverter.setSigningKey("123");// 测试用,授权服务使用相同的字符达到一个对称加密的效果,生产时候使用RSA非对称加密方式
		return accessTokenConverter;
	}

	/**
	 * 创建一个默认的资源服务token
	 *
	 * @return
	 */
	@Bean
	public ResourceServerTokenServices defaultTokenServices() {
		final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
		defaultTokenServices.setTokenEnhancer(accessTokenConverter());
		defaultTokenServices.setTokenStore(tokenStore());
		return defaultTokenServices;
	}
	// ===================================================以上代码与认证服务器一致=========================================
}

```

项目源码下载:

https://github.com/leftso/demo-spring-boot-security-oauth2

## 四、测试上面的编码

测试过程

步骤一:打开浏览器，输入地址

```
http://localhost:8080/oauth/authorize?client_id=normal-app&response_type=code&scope=read&redirect_uri=/resources/user
```

会提示输入用户名密码,这时候输入用户名leftso,密码111aaa将会出现以下界面

点击Authorize将获取一个随机的code,如图:

 打开工具postmain,输入以下地址获取授权token

```
localhost:8080/oauth/token?code=r8YBUL&grant_type=authorization_code&client_id=normal-app&redirect_uri=/resources/user
```

注意:url中的code就是刚才浏览器获取的code值

注意:url中的code就是刚才浏览器获取的code值
获取的token信息如下图:

注意:url中的code就是刚才浏览器获取的code值
获取的token信息如下图:
![token](http://www.leftso.com/resources/assist/images/blog/7c03ded7d6524068a137ba060654f094.png)

这时候拿到token就可以访问受保护的资源信息了，如下

```
localhost:8081//resources/user
```

首先,直接访问资源，会报错401如图:

我们加上前面获取的access token再试:

```
localhost:8081//resources/user?access_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsic3ByaW5nLWJvb3QtYXBwbGljYXRpb24iXSwidXNlcl9uYW1lIjoibGVmdHNvIiwic2NvcGUiOlsicmVhZCJdLCJyb2xlcyI6W3siYXV0aG9yaXR5IjoiUk9MRV9VU0VSIn1dLCJleHAiOjE0OTEzNTkyMjksInVzZXJOYW1lIjoibGVmdHNvIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6IjgxNjI5NzQwLTRhZWQtNDM1Yy05MmM3LWZhOWIyODk5NmYzMiIsImNsaWVudF9pZCI6Im5vcm1hbC1hcHAifQ.YhDJkMSlyIN6uPfSFPbfRuufndvylRmuGkrdprUSJIM
```

这时候我们就能成功获取受保护的资源信息了:

到这里spring boot整合security oauth2 的基本使用已经讲解完毕.

## 五、扩展思维

留下一些扩展

1.认证服务的客户端信息是存放内存的,实际应用肯定是不会放内存的，考虑数据库,默认有个DataSource的方式,还有一个自己实现clientDetail接口方式

2.jwt这里测试用的最简单的对称加密，实际应用中使用的一般都是RSA非对称加密方式

文章版权：本文为[左搜技术博客](http://www.leftso.com/)原创文章，未经博主允许不得转载。欢迎分享文章。

分类标签：[编程技术](http://www.leftso.com/blog/coding) [spring boot](javascript:;) [oauth2](javascript:;) [java编程](javascript:;)

发布时间：`2017-04-04 23:00:56` 最后编辑：`2017-11-12 20:59:01`



# **[点我获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr)**



我的官网<http://guan2ye.com>
我的CSDN地址<http://blog.csdn.net/chenjianandiyi>
我的简书地址<http://www.jianshu.com/u/9b5d1921ce34>
我的github<https://github.com/javanan>
我的码云地址<https://gitee.com/jamen/>
阿里云优惠券<https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=9ytvzpwr>
# **[阿里云教程系列网站http://aliyun.guan2ye.com](http://aliyun.guan2ye.com)**
![1.png](http://upload-images.jianshu.io/upload_images/2830896-5b23cf095c19945d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# **[我的开源项目spring boot 搭建的一个企业级快速开发脚手架](https://gitee.com/jamen/slife)**
![1.jpg](http://upload-images.jianshu.io/upload_images/2830896-66de965f818533c5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
