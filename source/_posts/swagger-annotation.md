---
title: Swagger2 常用注解
date: 2020-01-03 20:15:43
tags: Swagger
categories: Swagger
---

Spring Boot 开发restful接口时，往往会有很多RESTful API，一般会选择swagger对接口进行管理

## Spring Boot添加swagger支持

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.9.2</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.9.2</version>
</dependency>
```

## 配置类

```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Parameter;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Configuration
@EnableSwagger2
public class Swagger2Configuration {

    @Bean
    public Docket createRestApi() {
		// 构建一个header
        ParameterBuilder parameterBuilder = new ParameterBuilder()
                .parameterType("header")
                .name(BaseConst.AUTH_TOKEN)
                .defaultValue(null)
                .description("token")
                .modelRef(new ModelRef("string"))
                .required(false);

        List<Parameter> parameters = Stream.of(parameterBuilder.build()).collect(Collectors.toList());

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo()).globalOperationParameters(parameters).select()
                .apis(RequestHandlerSelectors.basePackage("com.iogogogo"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("RESTful APIs")
                .description("描述")
                .termsOfServiceUrl("").contact(new Contact("name", "url", "email"))
                .version("1.0")
                .build();
    }
}

```



## 注解详细说明

| 属性               | **取值**           | **作用**                         |
| :----------------- | :----------------- | -------------------------------- |
| 作用范围           | @API               | 使用位置                         |
| 对象属性           | @ApiModelProperty  | 用在出入参数对象的字段上         |
| 协议集描述         | @Api               | 用于controller类上               |
| 协议描述           | @ApiOperation      | 用在controller的方法上           |
| Response集         | @ApiResponses      | 用在controller的方法上           |
| Response           | @ApiResponse       | 用在 @ApiResponses里边           |
| 非对象参数集       | @ApiImplicitParams | 用在controller的方法上           |
| 非对象参数描述     | @ApiImplicitParam  | 用在@ApiImplicitParams的方法里边 |
| 描述返回对象的意义 | @ApiModel          | 用在返回对象类上                 |
| 标记@RequestBody   | @ApiParam          | 用在方法参数                     |

## @ApiImplicitParam

| **属性**     | **取值**     | **作用**                               |
| ------------ | ------------ | -------------------------------------- |
| paramType    | 查询参数类型 |                                        |
|              | path         | 以地址的形式提交数据                   |
|              | query        | 直接跟参数完成自动映射赋值             |
|              | body         | 以流的形式提交 仅支持POST              |
|              | header       | 参数在request headers 里边提交         |
|              | form         | 以form表单的形式提交 仅支持POST        |
| dataType     |              | 参数的数据类型 只作为标志说明 不会验证 |
|              | Long         |                                        |
|              | String       |                                        |
|              | Integer      |                                        |
| name         |              | 接收参数名                             |
| value        |              | 接收参数的意义描述                     |
| required     |              | 参数是否必填                           |
|              | true         | 必填                                   |
|              | false        | 非必填                                 |
| defaultValue |              | 默认值                                 |