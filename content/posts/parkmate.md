---
title: "Parkmate"
date: 2023-08-10T17:39:54+08:00
lastmod: 2023-08-10T17:39:54+08:00
draft: false
authors: [Sad_man]
tags: [面试]
categories: [项目]
series: [学习笔记]
series_weight: 1
---



# 零、Reading list

## OAuth2.0

http://www.ruanyifeng.com/blog/2019/04/github-oauth.html

http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html

http://www.ruanyifeng.com/blog/2019/04/github-oauth.html





# 一、OAuth2.0



OAuth 2.0 在终端用户、第三方应用、授权方（通常是资源服务器的所有者）之间建立了一种安全的授权机制。在这个过程中，这三方之间的可见和透明的内容如下：

### 终端用户（资源所有者）

- **可见**：
  - 登录页面：用户在授权方的登录页面上输入凭据。
  - 授权请求：用户看到第三方应用请求访问特定资源的提示，并可以同意或拒绝。
- **透明**：
  - 授权码和访问令牌：这些通常对用户是不可见的。

### 第三方应用（客户端）

- **可见**：
  - 授权码：在授权码流程中，客户端可以接收授权码。
  - 访问令牌：客户端接收并使用访问令牌来访问受保护的资源。
  - 错误信息：例如，如果用户拒绝授权，客户端会接收到错误信息。
- **透明**：
  - 用户凭据：在非密码式流程中，客户端不直接处理用户的用户名和密码。
  - 用户的其他个人信息：除非用户授权，否则客户端无法访问。

### 授权方（资源服务器的所有者）

- **可见**：
  - 用户凭据：用户在授权方的页面上输入，用于身份验证。
  - 授权请求和响应：授权方管理整个授权流程，包括接收请求、发送授权码、验证授权码以及发放访问令牌。
  - 访问令牌的使用：授权方可以看到访问令牌被用于访问哪些资源。
- **透明**：
  - 用户与第三方应用的具体互动：例如，用户在第三方应用中的具体操作通常对授权方是不可见的。

总的来说，OAuth 2.0 的设计目的之一就是确保敏感信息（如用户凭据）仅在需要的方之间可见，并在不需要的方之间透明。这有助于增强安全性，同时允许第三方应用在用户授权的前提下访问特定资源。



在 OAuth 2.0 的授权码流程中，授权码（Authorization Code）和访问令牌（Access Token）确实有重要的区别和不同的用途：

### 授权码（Authorization Code）

- **用途**：授权码是一次性的临时代码，用于证明用户已经授权第三方应用访问特定资源。
- **获取**：用户被重定向到授权方的页面并完成登录和授权后，授权方会将授权码发送给第三方应用。
- **使用**：第三方应用收到授权码后，会将其发送到授权方，请求访问令牌。
- **生命周期**：授权码的有效期非常短，通常只在几分钟内有效。
- **安全性**：授权码本身不允许访问受保护的资源，因此即使被截获，也不会直接泄露用户的数据。

### 访问令牌（Access Token）

- **用途**：访问令牌是用来访问受保护资源的凭证。第三方应用可以使用访问令牌访问用户授权的资源。
- **获取**：第三方应用使用授权码向授权方请求访问令牌，授权方验证授权码后发放访问令牌。
- **使用**：第三方应用在所有后续对受保护资源的请求中都将携带访问令牌。
- **生命周期**：访问令牌有限的有效期，但通常比授权码长。一些实现还提供了刷新令牌，用于在访问令牌过期后获取新的访问令牌。
- **安全性**：访问令牌的泄露可能导致未授权的访问，因此必须在传输和存储中进行安全保护。

总结来说，授权码是一种中间步骤，用于在用户授权后获取访问令牌，而访问令牌则是用于实际访问受保护资源的凭证。这种分离的设计有助于增强安全性，因为即使授权码被截获，攻击者也无法直接使用它访问受保护的资源。





oauth2

具体配置类

四种认证方式

流程



nacos

 gateway

**和openfeign的矛盾**



redis resttemplate的定制RedisConfig

redis为什么存储用户信息的时候需要使用虚拟persionId作为key而不是直接使用token？











```
我想请伙伴们思索下，目前如果让你做用户的权限认证，你会怎么做? 你有什么选择？
1. 原始的方式，用户名+密码登录，后台通过 uuid 生成一个token，返回给前端，这个token我们可以
存储到mysql或者redis服务器中，并且设置它的一个过期时间。
2. spring security + jwt进行权限验证。目前这种形式，中小型的企业使用的比较多（一些内用的或者非
电商，非公开性的网站使用这种方式比较多）。
3. oauth2.0（需要spring security包的支持） 进行权限的认证。
   (1) oauth2.0 可以通过 用户名 + 密码的形式进行验证，他跟原始方式有区别，oauth2还需要一个 client id
   和client secret. 如果我们的 用户名 + 密码 有了，个人用户无需额外申请 client id
   和client secret，我们后台根据一定的规则，为这个用户名和密码生成一套client_id 和client secret
   (2) 客户端验证。我们通过 手机+验证码的形式，可以登录一个网站，后台会为当前用户随机生一个用户名，且当
   前用户没有密码。因此，我们采用客户端验证形式，为当前的手机用户创建 client id 和 client secret。以此
   来保证用户能够顺利的登录。
   (3) 授权码验证模式。我们开发的网站叫 x宝，我们如果想用x宝去登录另外一个网站，那么就需要使用到授权码模式。
   我们可以用微信登录 淘宝；微信有自己的oauth2.0的服务器； 淘宝需要在微信的 oauth服务中注册一个client id
   和 client secret，当我们用微信登录淘宝的那一刻，淘宝会向微信的oauth服务发送登录请求进行验证，验证通过后，微信会
   回调淘宝的接口，发送当前登陆者的微信非敏感信息给淘宝，然后淘宝将该微信用户的信息进行存储。
   (4) 隐藏模式、 简单模式（弃用），这个东西太简陋了。

===============================
对于我们的 oauth2.0的验证，会获取一个token，这个token可以通过check_token接口进行一个可用性的校验。
我们在 gateway里边，如果使用 check_token接口进行校验，通过的就可以放行，不同过的就拦截，可以做到全局的
token校验和拦截。
问：spring cloud gateway里，你要用这种check_token接口校验方式吗？会用，我会介绍两种gateway的token校验
方式，check_token接口这个是命令式编程（调用接口，等待返回结果，rest形式发送请求，等待结果，阻塞式的）；还会
使用另外一种方式，响应式编程（webflux， Mono<Object>, netty 的refactor模型就是响应式编程的基础）
我们完成了oauth2.0的校验以后，我们就会通过gateway集成 oauth2.0（2种方式。）
问：那种方式校长觉得更好呢？ 响应式编程，异步非阻塞，这个东西会提高我们的访问效率，但是只针对io密集型的，并且
IO等待时间长的这种场景下会有奇效。那么对于目前和check_token相比较，速度上不好评估，尤其是，我们将来将我们的
token的存放形式存放到redis中，会更加没办法评论这个速度问题
问：你这个用户名和密码还有clientid和secret都是假数据啊，是你给我们的sql里边insert进去的，不行啊，坑人。
答：我们搞完spring cloud gateway集成oauth2后，会直接进入user-service搞：用户名+密码注册登录；手机号+验证
码的注册登录；第三方账号（GITEE）的注册+登录。你说，到时候是不是就有真实的数据了啊？
问：校长啊，你这个 www.baidu.com 太坑啦，我就不明白什么叫注册回调接口。咋办吧
答：用户名+密码注册登录； password
   手机号+验证码的注册登录； client_credentials
   第三方账号（GITEE）的注册+登录。client_credentials

  我会额外为大家演示 code授权码的获取方式，并且注册回调接口(另外一个xxxapp想用咱们的商城账号登录自己的
  xxxapp，才需要注册回调接口。)

==============================
1. 我们去重写oauth的UserDetailsService，为了从我们的user表中提取到当前用户的信息。
    覆写 方法 ： public UserDetails loadUserByUsername
2. 去实现我们的验证服务配置类（抽象）
    public class Oauth2Config extends AuthorizationServerConfigurerAdapter
3. 去实现我们的 web security类（抽象）
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter



==================================
POST MAN 验证
1. http://localhost:8500/oauth/authorize?client_id=hebwiwangxz&response_type=code 这个东西虽然
能让我们访问成功，但是我们遗漏了一个非常中的内容，就是 redirect_url
我们看下 gitee 平台： https://gitee.com/oauth/authorize/?
client_id=b748350128876988dd470edf32fd71d5c2974a0d2b7b81a4d86b0b73d649b38f
&redirect_uri=http://localhost:9001/user/register/gitee&response_type=code&state=GITEE

我们就需要调整我们的代码，要进行 /oauth/authorize 接口的 redirect_uri 参数的校验
1. 我们不能够覆盖 /oauth/authorize
2. 我们能够拦截它
```









Oauth2Config

看一眼加密

````
Oauth2的使用过程中，需要继承AuthorizationServerConfigurerAdapter的配置类具做了哪些方面的配置？


在使用OAuth2进行授权服务配置时，继承`AuthorizationServerConfigurerAdapter`的配置类允许你自定义授权服务器的相关配置。这个配置类通常会覆盖以下方面的配置：

### 1. **客户端配置** (`configure(ClientDetailsServiceConfigurer clients)`)：
   你可以在此方法中定义客户端的详细信息，例如客户端ID、密钥、授权类型、重定向URI等。例如，使用内存或数据库存储客户端的详细信息。

### 2. **授权服务器端点配置** (`configure(AuthorizationServerEndpointsConfigurer endpoints)`)：
   此方法用于配置授权服务器端点的一些非安全特性，例如令牌存储、令牌自定义、授权类型等。你可以指定一个令牌存储方式（如JWT、内存或数据库存储）、用户批准和令牌增强。

### 3. **授权服务器安全配置** (`configure(AuthorizationServerSecurityConfigurer security)`)：
   这个方法允许你设置授权服务器的安全约束，例如令牌端点的安全约束、客户端身份验证、是否允许表单身份验证等。

下面是一个示例，说明了如何使用`AuthorizationServerConfigurerAdapter`进行基本配置：

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    // 配置客户端详情服务
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
               .withClient("client-id")
               .secret("client-secret")
               .authorizedGrantTypes("authorization_code")
               .scopes("read")
               .redirectUris("http://example.com/callback");
    }

    // 配置授权服务器端点
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore())
                 .authenticationManager(authenticationManager);
    }

    // 配置令牌端点的安全约束
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.tokenKeyAccess("permitAll()")
                .checkTokenAccess("isAuthenticated()");
    }

    // 其他Bean定义，例如TokenStore等
}
```

这个示例演示了如何配置客户端详情服务、设置令牌存储、管理身份验证以及设置令牌端点的安全约束。

通过继承`AuthorizationServerConfigurerAdapter`，你可以完全自定义OAuth2授权服务器的行为，满足各种复杂的安全和业务需求。
````



# Canal

