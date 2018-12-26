# SpringBoot整合Swagger2
## 步骤
### 第一步：添加必要的依赖
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```
### 第二步：添加必要的配置
    一般无配置项，必要时可以添加自定义配置项，在配置类中读取
### 第三步：添加配置类(重点)
```java
// swagger2的配置内容仅仅就是需要创建一个Docket实例
@Configuration
@EnableSwagger2 //启用swagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.springbootdemo"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("springboordemo")
                .description("Springboot整合Demo")
                .version("0.0.1")
                .build(); // 这部分信息其实可以自定义到配置文件中读取
    }
}
```
> 通过@Configuration注解，让Spring-boot来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2Configuration。
  再通过buildDocket函数创建Docket的Bean之后，
  buildApiInfo()用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。
  select() 函数返回一个 ApiSelectorBuilder 实例用来控制哪些接口暴露给Swagger2来展现。
  一般采用指定扫描的包路径来定义
  Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被@ApiIgnore指定的请求）
### 第四步：在Controller和Bean上添加Swagger注解
```java
@RestController
@RequestMapping("/user")
@Log4j2
@Api(description = "用户接口")
public class UserApi {

    @Autowired
    private UserService service;

    @ApiOperation(value = "添加用户", notes = "根据给定的用户信息添加一个新用户",response = ResponseEntity.class,httpMethod = "PATCH")
    @RequestMapping(value = "/addUser",method = RequestMethod.PATCH)
    public ResponseEntity<User> addUser(final User user) {
        log.info("执行添加用户操作");
        return service.addUser(user);
    }

    @ApiOperation(value = "更新用户", notes = "根据给定的用户信息修改用户",response = ResponseEntity.class,httpMethod = "POST")
    @RequestMapping(value = "/updateUser", method = RequestMethod.POST)
    public ResponseEntity<User> updateUser(final User user) {
        log.info("执行修改用户操作");
        return service.updateUser(user);
    }

    @ApiOperation(value = "删除用户", notes = "根据给定的用户ID删除一个用户",response = ResponseEntity.class,httpMethod = "DELETE")
    @RequestMapping(value = "/deleteUser", method = RequestMethod.DELETE)
    public ResponseEntity<User> deleteUser(final int useId) {
        log.info("执行删除用户操作");
        return service.deleteUser(useId);
    }

    @ApiOperation(value = "查询用户", notes = "根据给定的用户ID获取一个用户",response = ResponseEntity.class,httpMethod = "GET")
    @RequestMapping(value = "getUser", method = RequestMethod.GET)
    public ResponseEntity<User> getUser(final int useId) {
        log.info("执行查询单个用户操作");
        return service.getUser(useId);
    }

    @ApiOperation(value = "查询用户", notes = "根据给定的用户信息查询用户",response = ResponseEntity.class,httpMethod = "POST")
    @RequestMapping(value = "getUsers", method = RequestMethod.POST)
    public ResponseEntity<List<User>> getUsers(final User user) {
        log.info("根据条件查询用户");
        return service.getUsers(user);
    }
}
```
```java
@ApiModel(value = "用户模型")
public class User {
    @ApiModelProperty("用户ID")
    private int useId;
    @ApiModelProperty("用户姓名")
    private String useName;
    @ApiModelProperty("用户性别")
    private UseSex useSex;
    @ApiModelProperty("用户年龄")
    private int useAge;
    @ApiModelProperty("用户身份证号")
    private String useIdNo;
    @ApiModelProperty("用户手机号")
    private String usePhoneNum;
    @ApiModelProperty("用户邮箱")
    private String useEmail;
    @ApiModelProperty("创建时间")
    private LocalDateTime createTime;
    @ApiModelProperty("修改时间")
    private LocalDateTime modifyTime;
    @ApiModelProperty("用户状态")
    private UseState useState;
}
```
### 第五步：启动应用,浏览器请求
```txt
http://localhost:8080/swagger-ui.html
```
(结束)

