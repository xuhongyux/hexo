---
title: Springcloud简单环境搭建
date: 2019-07-19 15:04:20
tags: 
	- Springcloud
	- 环境搭建
categories: Springcloud 
---

## 使用技术

+ pring Boot + Spring cloud+ ssm

## 搭建流程

### 1. 创建项目

(1)在创建的时候先创建 Empty project用来装几个分开的组件，方便调试。

(2)创建Eureka,创建provider,创建consumenr

(3)在在provider中链接数据库，为consunmenr提供服务。

### 2.创建模块

Eureka在创建的时候需要勾选 Eureka server，用来发现服务

provider和consumenr在创建的时候需要勾选，Eureka Discovery Client 成为服务

(1)Eureka基础配置
application.yml

``` java
servlet:
application.nema: xiayu
server:
port: 8761
servlet:
context-path: /
eureka:
instance:
hostname: eureka-server # eureka实例的主机名
client:
register-with-eureka: false #不把自己注册到eureka上
fetch-registry: false #不从eureka上来获取服务的注册信息
service-url:
defaultZone: http://localhost:8761/eureka/

```

在项目启动前声明自己是server

```xml
/**
* 注册中心
* 1、配置Eureka信息
* 2、@EnableEurekaServer
*/
@EnableEurekaServer
```
(2).provider配置
```
application.yml

server:
port: 8001
spring:
application:
name: provider-server

eureka:
instance:
prefer-ip-address: true # 注册服务的时候使用服务的ip地址
client:
service-url:
defaultZone: http://localhost:8761/eureka/

注意在连接数据库的时候扫描XML，mapper文件

#加载Mybatis映射文件
mybatis.mapper-locations=classpath:mapper/*Mapper.xml

#数据库连接信息
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
#spring.datasource.url=jdbc:mysql://139.155.87.157:3306/private?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
spring.datasource.url=jdbc:mysql://localhost:3306/private?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123
```
和正常的配置区别不大，主要是加入Eureka 的信息，让Eureka能够发现这provider

(3). consumer配置

applicaty.yml
```java
spring:
application:
name: consumer-user1
server:
port: 8081

eureka:
instance:
prefer-ip-address: true # 注册服务的时候使用服务的ip地址
client:
service-url:
defaultZone: http://localhost:8761/eureka/
```
能够被eureka发现

在模块运行前开启服务发现，能够发现服务提供者provider
```java
@EnableDiscoveryClient//开启服务发现

@LoadBalanced //开启负载均衡机制
@Bean
public RestTemplate restTemplate(){
return new RestTemplate();
}

controller

@Autowired
RestTemplate restTemplate;

@GetMapping("/test1")
public String getTest(){
String s= restTemplate.getForObject("http://provider-server/test",String.class);

//返回 collertion类型

@GetMapping("/user1")
    private Collection getUser(){
        Collection forObject = restTemplate.getForObject("http://provider-server/user", Collection.class);
        return forObject;
    }
return s;
}

```

创建rest模板去provider调去方法
## 遇到的问题

  + 在创建模块的时候不能正确的创建模块，需要先创建一个 Empty projectt,在空的项目中添加模块。
  + 在调用数据空的时候没有扫描mapper.xml文件报找不到映射文件错误。
  + 在consumer调用的服务的时候需要注意传回来的参数的类型，类型可以转化成String。