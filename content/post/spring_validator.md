---
title: "SpringBoot 中如何使用 Validator 检测含有泛型的对象"
date: 2022-03-19
tags: ["spring", "validator", "springboot"]
description : "SpringBoot 中如何使用 Validator 检测含有泛型的对象"
---

该文章主要是讲述了 SpringBoot 中如何使用 Validator 检测含有泛型的对象

## 版本对应关系
SpringBoot : 2.6.4， 需要 Java 11 版本

自带 Hibernate Validator:  6.2.2.Final； 最新版本是 7.0.4.Final， 由于 7.0 版本以上依赖的是 Jakarta Bean Validation 3.0。而是 6.0 版本依赖的是 `jakarta.validation:jakarta.validation-api:2.0.2`, 而 Spring 注册的 `LocalValidatorFactoryBean`  在运行时也与 Bean Validation 2.0 和 Hibernate Validator 6.0 兼容。

代码前提：
```java
// ApiRequest.java

public class ApiRequest<T> {
    private Long sid;

    private T request;

    public T getRequest() {
        return request;
    }

    public void setRequest(T request) {
        this.request = request;
    }

    public Long getSid() {
        return sid;
    }

    public void setSid(Long sid) {
        this.sid = sid;
    }
}


// HelloTest.java
@RestController
@Validated
@RequestMapping("/")
public class HelloTest {

    @RequestMapping("hello")
    public String sayHello(@RequestBody ApiRequest<@Valid @NotNull User> apiRequest) {
    // 说明：@Valid 是要透传到 User 里面，为了检测这个类也是需要非空的，那么需要注解，防止传入 "{}"
        return "Hello" + apiRequest.getRequest();
    }
}

// User.java
public class User {

    @NotBlank
    @Size(min = 1)
    private String name;

    @NotBlank
    @Min(10)
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

## 使用
### 实现 ValueExtractor

```java
// ApiRequestValueExtraction.java -- 实现了 ValueExtractor
public class ApiRequestValueExtraction implements ValueExtractor<ApiRequest<@ExtractedValue ?>> {
    @Override
    public void extractValues(ApiRequest<?> apiRequest, ValueReceiver valueReceiver) {
        valueReceiver.value(null, apiRequest.getRequest());
    }
}
```

### 注入
#### 使用 SPI 的方式注入
- 使用注解 `@ExtractedValue` 和在 `resource` 目录中新增文件 `META-INF/services/jakarta.validation.valueextraction.ValueExtractor`，使用 Java SPI 的方式进行加载, 文件内容如下:
```
com.kaitoshy.validator.config.ApiRequestValueExtraction
```
这个填入的就是之前 ValueExtraction 的实现类。

接下来使用 curl 或者 Postman 进行请求连接测试， 以下是 curl 的命令：
```shell
curl --location --request POST 'localhost:8080/hello' \

--header 'Content-Type: application/json' \

--data-raw '{

}'

```
返回显示为:
```log
{

"timestamp": "2022-03-19T15:14:36.861+00:00",

"status": 500,

"error": "Internal Server Error",

"path": "/hello"

}
```

这时查看终端发现
```log
javax.validation.ConstraintViolationException: sayHello.apiRequest: 不能为null
```
这里只是 `@NotNull User` 这里起了作用，并非 User 内部起作用了，所以将输入改成:
```json
{

"request": {}

}
```
这时 curl 返回和之前一样，但查看终端时查看报错信息，则变成了
```log
sayHello.apiRequest.age: 不能为null, sayHello.apiRequest.name: 不能为空
```

说明这个时候的 Validator 已经起作用了。继续将输输入修改成
```json
{
    "request": {
        "age":17,
        "name": "kuroba kaito"
    }
}
```
返回为:
```
HelloUser{name='kuroba kaito', age=17}
```
说明验证通过了。


#### 使用 LocalValidatorFactoryBean 注入

Spring 官方文档上有这么一句话：
>By default, if Bean Validation is present on the classpath (for example, Hibernate Validator), the LocalValidatorFactoryBean is registered as a global Validator for use with @Valid and Validated on controller method arguments.

简单翻译一下就是：如果 Bean Validation 在 classpath 中存在，例如使用了 Hibernate Validator，那么会默认注册一个全局的名为 ` LocalValidatorFactoryBean` 的全局 Validator 用于在 Controller 方法和参数的检验。增加以下代码：

```java
@Component
@Primary
public class ApiRequestValidatorBean extends LocalValidatorFactoryBean {
    @Override
    protected void postProcessConfiguration(Configuration<?> configuration) {
        configuration.addValueExtractor(new ApiRequestValueExtraction());
        super.postProcessConfiguration(configuration);
    }
}
```

用于注册 LocalValidatorFactoryBean 。
使用以上的测试命令得出和上述一样的结构，这里就不展开描述了。


## 总结
这里讲述了如何使用 Hibernate Validator 检测带有泛型的 DTO 的数据的合法性。首先需要实现 `ValueExtractor` 类中的 `extractValues` 方法，其次将该实例注入到 Spring 的容器中，这里是介绍了两种方式，Hibernate 官网介绍了 4 种。

本文章介绍了使用 Hibernate 提供的 SPI 方式，即新建文件 `resource/META-INF/services/jakarta.validation.valueextraction.ValueExtractor` , 然后将实现类写入即可，每行表示一个。
还有种方式就是使用 `LocalValidatorFactoryBean`  的方式注入。



> 参考资料
>
> spring 官方地址 ：
> https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation-spring
>
>https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-validation
>
>https://github.com/spring-projects/spring-framework/commit/6890e65d2cd98ad89e08e2afbbb1213241a791ac
>
> Hibernate 官方地址： https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-gettingstarted
