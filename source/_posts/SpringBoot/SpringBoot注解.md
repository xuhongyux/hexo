---
title: Spring Boot 注解
date: 2019-12-30 12:58:25
tags: 
	- final 和 Static
categories: SpringBoot 
---

## Spring Boot 注解

### **annotations**

+ @SpringBootApplication：申明让spring boot自动给程序进行必要的配置，这个配置等同于：@Configuration ，@EnableAutoConfiguration 和 @ComponentScan 三个配置。
+ @ResponseBody：表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，用于构建RESTful的api。在使用@RequestMapping后，返回值通常解析为跳转路径，加上@esponsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上@Responsebody后，会直接返回json数据。该注解一般会配合@RequestMapping一起使用。示例代码：

```java
@RequestMapping(“/test”)
@ResponseBody
public String test(){
    return”ok”;
}
```

+ @Controller：用于定义控制器类，在spring项目中由控制器负责将用户发来的URL请求转发到对应的服务接口（service层），一般这个注解在类中，通常方法需要配合注解@RequestMapping。示例代码：

```java
@Controller
@RequestMapping(“/demoInfo”)
public class DemoController {
@Autowired
private DemoInfoService demoInfoService;

@RequestMapping("/hello")
public String hello(Map<String,Object> map){
   System.out.println("DemoController.hello()");
   map.put("hello","from TemplateController.helloHtml");
   //会使用hello.html或者hello.ftl模板进行渲染显示.
   return"/hello";
}
}
```

+ @RestController：用于标注控制层组件(如struts中的action)，@ResponseBody和@Controller的合集。示例代码：

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(“/demoInfo2”)
publicclass DemoController2 {

@RequestMapping("/test")
public String test(){
   return "ok";
}
}
```

+ @RequestMapping：提供路由信息，负责URL到Controller中的具体函数的映射。

+ @EnableAutoConfiguration：SpringBoot自动配置（auto-configuration）：尝试根据你添加的jar依赖自动配置你的Spring应用。例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库”。你可以将@EnableAutoConfiguration或者@SpringBootApplication注解添加到一个@Configuration类上来选择自动配置。如果发现应用了你不想要的特定自动配置类，你可以使用@EnableAutoConfiguration注解的排除属性来禁用它们。
+ @ComponentScan：其实很简单，@ComponentScan主要就是定义**扫描的路径**从中找出标识了**需要装配**的类自动装配到spring的bean容器中,你一定都有用过@Controller，@Service，@Repository注解，查看其源码你会发现，他们中有一个**共同的注解@Component**，没错@ComponentScan注解默认就会装配标识了
+ @Controller，@Service，@Repository，@Component注解的类到spring容器中。当然，这个的前提就是你需要在所扫描包下的类上引入注解。
+ @Configuration：相当于传统的xml配置文件，如果有些第三方库需要用到xml文件，建议仍然通过
+ @Configuration类作为项目的配置主类——可以使用@ImportResource注解加载xml配置文件。
+ @Import：用来导入其他配置类。
+ @ImportResource：用来加载xml配置文件。
+ @Autowired：自动导入依赖的bean
+ @Service：一般用于修饰service层的组件
+ @Repository：使用@Repository注解可以确保DAO或者repositories提供异常转译，这个注解修饰的DAO或者repositories类会被ComponetScan发现并配置，同时也不需要为它们提供XML配置项。
+ @Bean：用@Bean标注方法等价于XML中配置的bean。
+ @Value：注入Spring boot application.properties配置的属性的值。示例代码：

```java
@Value("${provider.data.version}")
    public String  userVersion;
```

```yaml
###################################   外部配置   #########################################
#数据库版本
provider:
  data:
    version: 0.0.1
```

+ @Bean:相当于XML中的,放在方法的上面，而不是类，意思是产生一个bean,并交给spring管理。

@AutoWired：自动导入依赖的bean。byType方式。把配置好的Bean拿来用，完成属性、方法的组装，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。当加上（required=false）时，就算找不到bean也不报错。

@Qualifier：当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者，具体使用方式如下：

```java
  @Autowired
  @Qualifier(value = “demoInfoService”)
  private DemoInfoService demoInfoService;
```

@Resource(name=”name”,type=”type”)：没有括号内内容的话，默认byName。与@Autowired干类似的事。

### **JPA注解**

@Entity：@Table(name=”“)：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@MappedSuperClass:用在确定是父类的entity上。父类的属性子类可以继承。

@NoRepositoryBean:一般用作父类的repository，有这个注解，spring不会去实例化该repository。

@Column：如果字段名与列名相同，则可以省略。

@Id：表示该属性为主键。

@GeneratedValue(strategy = GenerationType.SEQUENCE,generator = “repair_seq”)：表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。

@SequenceGeneretor(name = “repair_seq”, sequenceName = “seq_repair”, allocationSize = 1)：name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

@Transient：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式

@JsonIgnore：作用是json序列化时将[Java ](http://lib.csdn.net/base/java)bean中的一些属性忽略掉,序列化和反序列化都受影响。

@JoinColumn（name=”loginId”）:一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

@OneToOne、@OneToMany、@ManyToOne：对应[hibernate](http://lib.csdn.net/base/javaee)配置文件中的一对一，一对多，多对一。

__注意__

时间格式化

```java
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
private Date createTime;
```

### **springMVC相关注解**

@RequestMapping：@RequestMapping(“/path”)表示该控制器处理所有“/path”的UR L请求。RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。该注解有六个属性：
params:指定request中必须包含某些参数值是，才让该方法处理。
headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
value:指定请求的实际地址，指定的地址可以是URI Template 模式
method:指定请求的method类型， GET、POST、PUT、DELETE等
consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回

@RequestParam：用在方法的参数前面。
@RequestParam
String a =request.getParameter(“a”)。

@PathVariable:路径变量。如

```java
   RequestMapping(“user/get/mac/{macAddress}”)
   public String getByMacAddress(@PathVariable String macAddress){
      //do something; 
   }
```

+ **@Configuration & @bean**[1.@Configuration](mailto:1.@Configuration)标注在类上，相当于把该类作为spring的xml配置文件中的``，作用为：配置spring容器(应用上下文)
+ [@Bean](mailto:2.@Bean)标注在方法上(返回某个实例的方法)，等价于spring的xml配置文件中的``，作用为：注册bean对象

```java
package com.test.spring.support.configuration;
@Configuration
public class TestConfiguration {
        public TestConfiguration(){
            System.out.println("spring容器启动初始化。。。");
        }

    //@Bean注解注册bean,同时可以指定初始化和销毁方法
    //@Bean(name="testNean",initMethod="start",destroyMethod="cleanUp")
    @Bean
    @Scope("prototype")
    public TestBean testBean() {
        return new TestBean();
    }
}
```

**注：
(1)、@Bean注解在返回实例的方法上，如果未通过@Bean指定bean的名称，则默认与标注的方法名相同；
(2)、@Bean注解默认作用域为单例singleton作用域，可通过@Scope(“prototype”)设置为原型作用域；
(3)、既然@Bean的作用是注册bean对象，那么完全可以使用@Component、@Controller、@Service、@Ripository等注解注册bean，当然需要配置@ComponentScan注解进行自动扫描。**

## Springboot的几个注解区别

###  @Bean  @Service注解区别

使用方式

1. @Configuration和@Bean的组合来创建Bean
2. 直接使用 @Service等注解放在类上的方式

@Service是告诉spring，这个类是一个服务
@Component，它的意思就是一个组件，相对来说比较中立，仅仅作为某种功能放在那里。老实说，其实@Service和@Component才是基本没什么差别，两者相互代替也没什么毛病。

@Configuration告诉spring，这个类是一个配置类

+ 这几个注解几乎可以说是一样的：因为被这些注解修饰的类就会被Spring扫描到并注入到Spring的bean容器中。

### @Resource @Reference@autowirte

+ @Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入。

+ @Reference是dubbo的注解，@Resource是spring的

+ @Reference也是注入，他一般注入的是分布式的远程服务的对象，需要dubbo配置使用

__总的来说他们的区别：@Reference注入的是分布式中的远程服务对象，@Resource和@Autowired注入的是本地spring容器中的对象。__

实战中

```java
    @RequestMapping(value = "/get_point", method = RequestMethod.POST)
    @ApiOperation(value = "获取知识点", httpMethod = "POST", response = ResultData.class)
    public ResultData getPoint(@RequestParam("subjectCode") String subjectCode) throws Exception {
        return this.pointInfoService.getPointInfo(subjectCode);
    }
    @RequestMapping(value = "/get_que_po", method = RequestMethod.POST)
    @ApiOperation(value = "获取单个题目知识点", httpMethod = "POST", response = ResultData.class)
    public ResultData getQuestion_point(@RequestBody(required = false) QuestionAndPointParam param){
        List<PointVo> pointInfoVos =  questionAndPointService.getPointInfo(param.getId());
        return new ResultData(200, null, pointInfoVos);
    }
```

