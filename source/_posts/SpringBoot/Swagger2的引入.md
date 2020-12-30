---
title:  Swagger2的引入
date: 2019-12-2 16:01:20
categories: SpringBoot 
---

## Swagger引入

### pom引入

```xml
<swagger2.version>2.8.0</swagger2.version>

  <!--swagger2 - Begin-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>${swagger2.version}</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>${swagger2.version}</version>
    </dependency>

    <!--处理版本兼容-->
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>20.0</version>
        </dependency>
 <!--swagger2 - End-->
```

### 配置文件

Swagger2Configuration

```java
package com.reborn.tutor.busines.configure;

import java.util.ArrayList;
import java.util.List;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.ApiKey;
import springfox.documentation.service.Parameter;
import springfox.documentation.service.SecurityScheme;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;


/**
 * Description:
 *
 * @version v1.0.0
 * @Author xiayu
 * @Date 2019/12/26 17:45
 */
@Configuration
@EnableSwagger2
public class Swagger2Configuration {


    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.reborn.tutor.busines.controller"))
                .paths(PathSelectors.any())
                .build()
                .globalOperationParameters(defaultHeader())
                ;
    }

    @Bean
    public SecurityScheme apiKey() {
        return new ApiKey("access_token", "accessToken", "header");
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title("reborn-tutor项目api文档").description("user服务模块")
                .termsOfServiceUrl("user").version("1.0.0").build();
    }

    /**
     * 设置默认参数 添加
     * @return
     */
    private static List<Parameter> defaultHeader() {
        // ParameterBuilder appType = new ParameterBuilder();
        // appType.name("app-type").description("应用类型").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
        // pars.add(appType.build());
        ParameterBuilder appToken = new ParameterBuilder();
        appToken.name("Authorization").description("令牌").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
        List<Parameter> pars = new ArrayList<>();
        pars.add(appToken.build());
        return pars;
    }
}

```

UserResourceServerConfiguration 资源服务器配置，不拦截swagger地址

```java
package com.reborn.tutor.busines.configure;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
/**
 * Description:
 *资源服务器
 * @version v1.0.0
 * @Author xiayu
 * @Date 2019/12/26 17:28
 */
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class UserResourceServerConfiguration extends ResourceServerConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .exceptionHandling()
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                //下边的路径放行
                .antMatchers("/v2/api-docs", "/swagger-resources/configuration/ui",
                "/swagger-resources","/swagger-resources/configuration/security",
                "/swagger-ui.html","/course/coursebase/**").permitAll()
                //拦截所有请求    并设置访问该资源的权限，这里设置为USER
                .antMatchers("/**").hasAuthority("USER");
    }
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        // 配置资源 ID
        resources.resourceId("backend-resources");
    }
}

```

### 对接口进 行描述

```java
  
@Api(value = "远程调用管理", description = "远程调用管理管理类")
@RestController
@RequestMapping(value = "/client")
public class ClientController {
/**
     * 在url中使用正则表达式
     * @param id
     * @return
     */
    @GetMapping("/{id:\\d+}")
    @JsonView(User.UserDetailView.class)
    @ApiOperation(value = "获取用户详情")
    public User get(@ApiParam(value = "用户id") @PathVariable String id){
        //throw new UserNotFoundException(id);
        System.out.println(id);
        User user = new User();
        user.setUsername("tom");
        return user;
    }
```

+ 使用`@ApiOperation(value = "获取用户详情")`对接口进行描述

```java
@ApiImplicitParams({
		@ApiImplicitParam(name = "subjectCode", value = "学科", required = true, dataType = "String", paramType = "query"),
		@ApiImplicitParam(name = "type", value = "1.区域 2.学校", required = true, dataType = "int", paramType = "query")
	})
@RequestMapping(value = "/classExam", method = RequestMethod.GET)
	@ResponseBody
	public ResultData classExam(  String classId, String subjectCode, int type) {
	
	}
```



### 对字段描述

```java
public class UserQueryCondition {

    @ApiModelProperty(value = "用户名",required = true),example="string")
    private String username;

    @ApiModelProperty(value = "年龄起始值")
    private int age;

    @ApiModelProperty(value = "年龄终止值")
    private int ageTo;
```

### 访问地址

访问端点`/swagger-ui.html`
例如:`http://localhost:8080/swagger-ui.html#/`