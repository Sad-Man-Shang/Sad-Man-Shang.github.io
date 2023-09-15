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

## A. 具体实现流程（授权码模式

### 1. 前端将用户引导到gitee认证页面上

```
https://gitee.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code
```

client_id是应用注册的时候创建的，redirect_uri是回调到后端的地址

### 2. 用户在gitee认证页面上进行授权后

用户浏览器会被重定向到redirect_uri(大概是controller层的某个接口)，并且带有code参数，后端可以获得code参数，这里，用户可以看到code，这也是为什么不直接返回access token的原因

获得的code可以配合grant_type、client_id、redirect_uri（**起到一个校验的作用和，看看和之前的授权也面试不是一个**）、client_secret来获取access_token

然后用access_token来获得用户的信息

### 3. 为什么不直接返回code？

这里没有直接返回access_token是因为code的返回会经过用户浏览器，可能被截取，而授权码模式需要client_secret+code来获取access_token，并且发生在授权服务器和应用服务器之间，不会涉及到用户浏览器，更为安全。

**Ref：**[gitee doc](https://gitee.com/api/v5/oauth_doc#/)｜[code demo](https://blog.csdn.net/m0_46845579/article/details/126834169?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-126834169-blog-131341706.235%5Ev38%5Epc_relevant_sort_base1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-126834169-blog-131341706.235%5Ev38%5Epc_relevant_sort_base1&utm_relevant_index=5)

















# 二、Canal

canal模拟slave读取master的binlog，并且声称具体的数据，通过配置文件，可以将canal同步到的结果发送到rabbitmq中，然后从springboot中使用相关注解获得消息并且消费（根据id删除缓存）





i



# 负一、问题

nacos

 gateway

**和openfeign的矛盾**



redis resttemplate的定制RedisConfig

redis为什么存储用户信息的时候需要使用虚拟persionId作为key而不是直接使用token？
