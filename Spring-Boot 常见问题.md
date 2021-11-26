---
title: Spring Boot 常见问题
date: 2021-06-10 10:38:08
tags:
- Java
- Spring Boot
- 常见问题
categories:
- Java
- Spring
- Spring Boot
---

# @RequestBody 和 @RequestParam 的区别

## @RequestParam

注解 `@RequestParam` 接收的参数是来自 HTTP 请求体或请求 url 的 QueryString 中。

RequestParam **接收数据**来自于：请求行 URL 的 param 与 请求体中的 param。

 `@RequestParam` 有三个配置参数：

-  **required** 表示是否必须，默认为 true。
- **defaultValue** 可设置请求参数的默认值。
- **value** 为接收 url 的参数名（相当于 key 值）。

 `@RequestParam` 用来处理 Content-Type 为 application/x-www-form-urlencoded 编码的内容，Content-Type默认为该属性。 `@RequestParam` 也可用于其它类型的请求，例如：POST、DELETE等请求。

<!-- more-->

## @RequestBody

注解 `@RequestBody` 接收的参数是来自 requestBody 中，即**请求体**。一般用于处理非 Content-Type: application/x-www-form-urlencoded编码格式的数据，比如：**application/json**、application/xml 等类型的数据。

就 application/json 类型的数据而言，使用注解 `@RequestBody` 可以将 body 里面所有的 json 数据传到后端，后端再进行解析。

GET 请求中，因为没有 HttpEntity，所以**@RequestBody**并不适用。

POST 请求中，通过 HttpEntity 传递的参数，必须要在请求头中声明数据的类型 **Content-Type**，SpringMVC通过使用 HandlerAdapter 配置的 HttpMessageConverters 来解析HttpEntity中的数据，然后绑定到相应的 bean 上。

**注意：**前端使用$.ajax的话，一定要指定 contentType: "application/json;charset=utf-8;"，默认为 `application/x-www-form-urlencoded`。

AJAX POST **请求示例**（使用 JSON.stringify() 格式化）：

```javascript
$.post({
    url: "http://localhost:8080/user/save",
    data: JSON.stringify({
        "id": 1,
        "username": "Zhangsan",
        "password": "123456"
    }),
    contentType: "application/json;charset=utf-8;",
    success: (data) => {
        console.log(data);
    }
});
```

## POST 请求

① form-data、x-www-form-urlencoded：不可以用 `@RequestBody`；可以用 `@RequestParam`。

② application/json：json 字符串部分可以用 `@RequestBody`；url 中的 ? 后面参数可以用 `@RequestParam`。

## GET 请求

- @RequestBody

GET 请求中不可以使用 `@RequestBody`。

- @RequestParam

```java
(@RequestParam Map map)
在 url 中的 ? 后面添加参数即可使用
```

```java
(@RequestParam String waterEleId,@RequestParam String enterpriseName)
在 url 中的 ? 后面添加参数即可使用
```

```java
(@RequestParam Object object)
GET 请求中不可以使用
```

# @NotBlank、@NotEmpty 和 @NotBlank 的区别

- @NotBlank 应用在 **String** 类型
- @NotEmpty 应用在**集合**类上面
- @NotNull 应用在**基本类型**上面

## @NotBlank

不能为 null 且 trim() 之后 size>0，说明此注解只能加在有实际字符的串上。

## @NotEmpty 

加了 @NotEmpty 的 String 类、Collection、Map、数组，是不能为 null 或者长度为 0 的（String、Collection、Map的 isEmpty() 方法）。

## @NotNull 

不能为 null，但可以为 empty，没有 Size 的约束。

# 自定义验证规则

自带的验证规则（@NotBlank、@NotEmpty、@NotBlank、@Email 等）可能无法满足要求，可自定义实现注解。

**实例**：校验特定字段的值是否在可选范围。

比如我们现在多了这样一个需求：User 类多了一个 gender 字段，gender 字段只能是 `male`、`female` 这两个中的一个。

第一步你需要创建一个注解：

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = GenderValidator.class)
@Documented
public @interface Gender {
    String message() default "性别不在可选之内";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

第二步你需要实现 `ConstraintValidator`接口，并重写 `isValid` 方法：

```java
public class GenderValidator implements ConstraintValidator<Gender,String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        HashSet<String> genders = new HashSet<>();
        genders.add("male");
        genders.add("female");
        return genders.contains(value);
    }
}
```

现在就可以使用这个注解：

```java
@Gender
private String gender;
```

**注**：在需要验证的参数上加上了`@Valid`注解，如果验证失败，它将抛出 `MethodArgumentNotValidException`。

