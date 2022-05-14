# Springboot集成Swagger



## 一、引入swagger依赖

使用版本为2.6.1

```java
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```

swagger提供了swagger-bootstrap-ui插件更好的显示Restful风格的接口界面

引入swagger-bootstrap-ui依赖

```java
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.9.2</version>
</dependency>
```



## 二、创建配置文件



使用JavaConfig的形式配置SwaggerConfig

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket buildDocket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(buildApiInf())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jl.app.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    
    private ApiInfo buildApiInf() {
        return new ApiInfoBuilder()
                // 界面显示的文档名称
                .title("嘉乐前后端RESTful API文档")
                // 作者相关信息
                .contact(new Contact("ymLike", "https://mrbird.cc", "1607689961@qq.com"))
                // 文档版本
                .version("1.0")
                .build();
    }

}
```

@EnableSwagger2 注解来启用swagger2，

api()定义了扫描的包路径，也就是你的接口controller所在的package

title()定义界面的文档名称

contact()定义了作者相关信息

version()定义了文档版本



## Swagger常用注解



![](..\picture\clipboard-1615427441923.png)

##### @Api：修饰整个类，描述Controller的作用；

```java
@Api(tags = "设备管理")
@Controller
@RequestMapping("/device")
@Slf4j
@CrossOrigin
public class DeviceController {
    ...
}
```

##### @ApiOperation：描述一个类的一个方法，或者说一个接口；

```java
@ApiOperation(value = "查询添加/分享/所有设备列表", notes = "根据序列号数组查询对应设备列表")
```

##### @ApiImplicitParam：一个请求参数；

```java
@ApiImplicitParam(name = "sn", value = "设备序列号", required = true, dataType = "String", paramType = "query")
```

> 说明
> name ：参数名。 
> value ： 参数的具体意义，作用。 
> required ： 参数是否必填。 
> dataType ：参数的数据类型。 
> paramType ：查询参数类型，这里有几种形式：
>     path 以地址的形式提交数据
>     query 直接跟参数完成自动映射赋值
>     body 以流的形式提交，仅支持POST
>     header 参数在request headers里面提交
>     form 以form表单的形式提交，仅支持POST

##### @ApiImplicitParams：多个请求参数。

```java
@ApiImplicitParams({
        @ApiImplicitParam(name = "username", value = "用户手机号", required = true, dataType = "String"),
        @ApiImplicitParam(name = "type", value = "0.添加/1.分享/为空查询所有的设备", required = false, dataType = "Integer"),
        @ApiImplicitParam(name = "pageSize", value = "当前页", required = false, dataType = "Integer"),
        @ApiImplicitParam(name = "pageIndex", value = "每页最大条数", required = false, dataType = "Integer")
})
```

##### @ApiIgnore：使用该注解忽略这个API；

```java
@ApiIgnore
@GetMapping("/getToken")
@ResponseBody
public String getAccessToken() throws IOException {
    ...
}
```

```java
public Result<?> shareDevice(@ApiIgnore QueryRequest queryRequest){...}
```



##### @ApiModel：用对象来接收参数；

```java
@ApiModel(value = "user_device", description = "用户设备对应关系")
@TableName("user_device")
@EqualsAndHashCode(callSuper = true)
@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class UserDevice extends Model<UserDevice> {
    
}
```

##### @ApiModelProperty：用对象接收参数时，描述对象的一个字段；

```java
private String id;
@ApiModelProperty(value = "用户手机号", name = "username", required = true)
private String username;
@ApiModelProperty(value = "设备序列号", name = "sn", required = true)
private String sn;
```



##### @ApiParam：单个参数描述；

```java
@RestController
public class UserController {
     @ApiOperation(value="获取用户信息",tags={"获取用户信息copy"},notes="注意问题点")
     @GetMapping("/getUserInfo")
     public User getUserInfo(@ApiParam(name="id",value="用户id",required=true) Long id,@ApiParam(name="username",value="用户名") String username) {
     // userService可忽略，是业务逻辑
      User user = userService.getUserInfo();
      return user;
  }
}
```



##### @ApiResponse：HTTP响应其中1个描述；

##### @ApiResponses：HTTP响应整体描述；

##### @ApiError ：发生错误返回的信息；









## 配置全局code返回状态码



swagger默认显示的状态码可能不是我们项目设定的，那么可以根据自己的项目需求设定所有接口的返回码，如下例

```java
 @Bean
    public Docket createRestApi() {
    	//设定全局code为0表示成功，200表示失败
        List<ResponseMessage> responseMessageList = new ArrayList<>();
        responseMessageList.add(new ResponseMessageBuilder().code(0).message("成功").build());
        responseMessageList.add(new ResponseMessageBuilder().code(200).message("失败").build());

        return new Docket(DocumentationType.SWAGGER_2)
                .globalResponseMessage(RequestMethod.GET, responseMessageList)
                .globalResponseMessage(RequestMethod.POST, responseMessageList)
                .enable(swaggerShow)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }


```



## 访问测试



![](..\picture\clipboard.png)



导出为html

https://cloud.tencent.com/developer/article/1332445

导出为word

https://www.cnblogs.com/jmcui/p/8298823.html

https://github.com/JMCuixy/SwaggerToWord

导出为pdf

https://blog.csdn.net/qq_29534483/article/details/81227308

https://blog.csdn.net/qq_29534483/article/details/81235081



## swagger-ui-layer主题风格

swagger开源了一种更为好看的主题风格：swagger-ui-layer

需要引入依赖

```java
<dependency>
    <groupId>com.github.caspar-chen</groupId>
    <artifactId>swagger-ui-layer</artifactId>
    <version>1.1.3</version>
</dependency>
```

访问地址：/docs.html

访问如下所示：

![image-20210326163508837](..\picture\image-20210326163508837.png)