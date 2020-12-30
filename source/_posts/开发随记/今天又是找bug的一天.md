---
title: 今天又是找bug的一天
date: 2066-01-27 21:04:20
tags:
categories: 开发随记
---

## Maven

## Maven依赖产生回路

时间：2020/9/27  耗时两个小时

在搭建环境时 dependencies(依赖版本管理模块)错误的将该模块的父类设置未最外层项目，导致最外层父类项目在引用dependencies时产生回路。

同时Idea的Maven默认会使用本地缓存的库来编译工程，对于上次下载失败的库，maven会在`~/.m2/repository////`目录下创建xxx.lastUpdated文件，一旦这个文件存在，那么在直到下一次nexus更新之前都不会更新这个依赖库。

可以找到这个文件删除之Maven将会从新加载依赖关系。

### Maven缓存的愿意找不到主类

mvn clean compile，将项目重新编译

mvn install，打包

mvn spring-boot:run，启动项目

mvn package，打成war包

## spring cloud 搭建

### 服务无法注册到Nacos

检测扫描路径

检测注解

maven重新打包

### Oauth2

```test
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'resourceHandlerMapping' defined in class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.web.servlet.HandlerMapping]: Factory method 'resourceHandlerMapping' threw exception; nested exception is java.lang.IllegalStateException: No ServletContext set

```

问题，所在添加事务引起，具体原因未知（）

后续    Oauth2项目路径下没对数据库操作。



### 注入feign没有找到Bean

+ 检查是否开启了EnableFeignClients
+ 检查feign接口注解是否错误
+ 在调用feign的启动类加上扫描，扫描这个包

__注意__

在多个模块的时候命名，包的命名千万不要一样，不然在扫描的时候会扫描不上



## Spring Boot 搭建

### Mybatis中的错误

#### 1 . 找不到映射文件UserMapper

错误原因：Mapper文件中为添加注解@Mapper

```java
Field sysUserMapper in com.xiayu.providerserver.service.TestService required a bean of type 'com.xiayu.providerserver.mapper.SysUserMapper' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type 'com.xiayu.providerserver.mapper.SysUserMapper' in your configuration.

```

#### 2 .没有找到映射的方法

status=500 ,mapper和mapper.xml映射关系没联系上

```java
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.xiayu.providerserver.mapper.UserMapper.findUserById

```

#### 3 . 没有找到URl

```java
Failed to configure a DataSource: 'url' attribute is not specified and no em
```

原因版本问题 设置spring.datasource.jdbc-url=jdbc:mysq



###没有找到mapper

原因是在文件开始前没有扫描mapper


```
@MapperScan(basePackages = "com.xiayu.spring.boot.mybatis.mapper")

org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'com.xiayu.spring.boot.mybatis.SpringBootMybatisApplicationTests': Unsatisfied dependency expressed through field 'userMapper'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.xiayu.spring.boot.mybatis.mapper.UserMapper' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
	
```

#### 总结

+ 在接口Mapper文件中,需要添加@mapper注解

+ 在application配置文件中需要添加mapper.xml文件扫描

+ 在使用逆向工程生成的方法时需要添加pojo别名扫描

+ 在连接数据库的时候注意版本问题，在使用连接插件的时候注意启动配置

### Bean生成失败

```
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.reborn.tutor.provider.api.UserAdminService' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

错误原因在 application.yml 配置中 dubbo扫描包的时候没扫描到service

```yaml
dubbo:
  scan:
    base-packages: com.reborn.tutor.provider.service
  protocol:
    name: dubbo
    port: -1
    serialization: kryo
  registry:
    address: nacos://139.155.87.157:8848
    provider:
      loadbalance: roundrobin  #负载均衡方式  轮循
      # random 随机，roundrobin 轮循，leastactive 最少活跃调用数，consistenthash一致性 Hash

```

### @Autowired注入为null的几种情况

1.在应用的Filter或Listener中使用了@Autowired ，

原因：因为Filter和Listener加载顺序优先于spring容器初始化实例，所以使用@Autowired肯定为null了~~

解决：用ApplicationContext根据bean名称（注意名称为实现类而不是接口）去获取bean，随便写个工具类即可

2.你写的代码有问题，没加@Service注解等 ，这一类低级错误自己检查即可

3.你写的@Service、@Componet、@Configuration、@Repository等Spring注解未被扫描到，

```java
@SpringBootApplication(scanBasePackageClasses = {ProviderAdminApplication.class, DubboSentinelConfiguration.class})
@MapperScan(basePackages = "com.xiayu.demoCloud.provider.mapper")
public class ProviderAdminApplication {

    public static void main(String[] args) {
            SpringApplication.run(ProviderAdminApplication.class, args);

    }
}
```

