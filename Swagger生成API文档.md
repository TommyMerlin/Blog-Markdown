---
title: Swagger生成API文档
date: 2020-05-26 13:38:11
tags:
- Java
- Swagger
- API
categories:
- Java
- Spring
- Spring Boot
---

### 1.引入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

<!-- more -->

### 2.Swagger配置

`config/SwaggerConfig.java`

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    private static final String VERSION = "1.0";
    private static final String SWAGGER_SCAN_PACKAGE = "com.coderhuye.mybatis.controller";
    private static final Contact CONTACT = new Contact("HuYe", "https://www.coderhuye.com", "coderhuye@coderhuye.com");

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage(SWAGGER_SCAN_PACKAGE))
                // 可以根据url路径设置哪些请求加入文档，忽略哪些请求
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //设置文档的标题
                .title("个人网站后台服务")
                // 设置文档的描述
                .description("个人网站后台服务 API 接口文档")
                // 设置文档的版本信息
                .version(VERSION)
                // 设置联系方式
                .contact(CONTACT)
                // 设置文档的License信息
                //.termsOfServiceUrl("https://www.coderhuye.com")
                .build();
    }
}
```

### 3.使用注解

`controller/UserController.java`

```java
@Api("用户操作 API 接口")
@RestController
@CrossOrigin
public class UserController {
    private final UserServiceImpl userService;
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);

    public UserController(UserServiceImpl userService, EmailService emailService) {
        this.userService = userService;
    }

    @ApiOperation("获取用户列表")
    @GetMapping({"/users", "/"})
    public AjaxResult getAllUsers() {
        List<User> users = userService.findAllUsers();
        return new AjaxResult(users);
    }

    @ApiOperation("根据username获取用户信息")
    @ApiImplicitParam(name = "username", value = "用户名", required = true)
    @GetMapping("/user/{username}")
    public AjaxResult getUserByUsername(@PathVariable(required = true) String username) {
        User user = userService.findUserByUsername(username);
        List<User> userList = new ArrayList<>();
        userList.add(user);
        return new AjaxResult(userList);
    }
}
```

### 4.打开 Swagger API 文档

http://localhost:8080/swagger-ui.html