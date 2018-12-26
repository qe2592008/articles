# SpringBoot整合Spring MVC
## 步骤
### 第一步：添加必要依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### 第二步：添加必要的配置
无
### 第三步：添加必要的配置类
> SpringBoot整合SpringMVC没有必需的配置类，只有在想要自定义的时候添加一些实现了WebMvcConfigurer接口的配置类
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    // 添加针对swagger的处理，避免swagger404
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
    }
    //...自定义实现WebMvcConfigurer中的若干默认方法
}
```
### 第四步：整合模板引擎
#### 整合Freemarker
##### 第一步：添加必要的依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
##### 第二步：添加必要的设置(重点)
```properties
#Freemarker-config
# 设置模板前后缀名
#spring.freemarker.prefix=
spring.freemarker.suffix=.ftl
spring.freemarker.enabled=true
# 设置文档类型
spring.freemarker.content-type=text/html
spring.freemarker.request-context-attribute=request
# 设置ftl文件路径
spring.freemarker.template-loader-path=classpath:/templates/
# 设置页面编码格式
spring.freemarker.charset=UTF-8
# 设置页面缓存
spring.freemarker.cache=false
```
##### 第三步：添加必要的配置类
无
##### 第四步：添加控制器和动态页面
```java
@Controller
@RequestMapping("base")
@Log4j2
@Api(hidden = true)
public class Base {
    @RequestMapping("/book")
    @ApiOperation(value = "测试",hidden = true)
    public String toBookIndexPage(ModelMap model){
        log.info("进来啦！！！");
        model.put("name","浩哥");
        return "/book/index";
    }
}
```
resources/book/index.ftl
```ftl
<#assign base = request.contextPath/>
<!DOCTYPE HTML>
<HTML>
<HEAD>
    <TITLE>测试首页</TITLE>
    <base id="base" href="${base}">
    <link rel="stylesheet" href="/webjars/bootstrap/3.3.7-1/css/bootstrap.min.css" />
    <script src="/webjars/jquery/3.1.1/jquery.min.js"></script>
    <script src="/webjars/bootstrap/3.3.7-1/js/bootstrap.min.js"></script>
</HEAD>
<BODY>
${name}
<a class="getBook" onclick="dianji()">点击</a><br/>
<button onclick="dianji()">点击</button>
</BODY>
<SCRIPT>
    function dianji() {
        $.ajax({
            url: "/account/g/1",
            type: "GET",
            success: function (data) {
                alert(data);
            }
        })
    }
    var base = document.getElementById("base").href;
    // 与后台交互
    _send = function(async,url, value, success, error) {
        $.ajax({
            async : async,
            url : base + '/' + url,
            contentType : "application/x-www-form-urlencoded; charset=utf-8",
            data : value,
            dataType : 'json',
            type : 'post',
            success : function(data) {
                success(data);
            },
            error : function(data) {
                error(data);
            }
        });
    };

</SCRIPT>
</HTML>
```
#### 整合Thymeleaf
##### 第一步：添加必要的jar包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
##### 第二步：添加必要的配置
```properties
spring.thymeleaf.cache=false
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.enabled=true
spring.thymeleaf.mode=HTML
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.servlet.content-type=text/html
```
以上配置中除了第一个之外，其余皆可不配置，上面的值也是默认值，需要修改的时候再进行配置
##### 第三步：添加必要配置类
无
##### 第四步：添加控制器和动态页面
```java
@Controller
public class BaseController {
    @RequestMapping("index")
    public String toIndex(ModelMap model){
        model.put("name","首页啊");
        return "index";
    }
}
```
resources/index.html
```html
<!Doctype html>
<html>
<head>
    <title>下一页</title>
</head>
<body>
<h1 th:text="${name}">Hello World</h1>
</body>
</html>
```
#### 整合WebJar
##### 第一步：添加必要的jar包
```xml
<!--导入bootstrap-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>3.3.7-1</version>
</dependency>
<!--导入Jquery-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.1.1</version>
</dependency>
```
##### 第二步：使用WebJar开发前端页面
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dalaoyang</title>
    <link rel="stylesheet" href="/webjars/bootstrap/3.3.7-1/css/bootstrap.min.css" />
    <script src="/webjars/jquery/3.1.1/jquery.min.js"></script>
    <script src="/webjars/bootstrap/3.3.7-1/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container"><br/>
    <div class="alert alert-success">
        <a href="#" class="close" data-dismiss="alert" aria-label="close">×</a>
        Hello, <strong>Dalaoyang!</strong>
    </div>
</div>
</body>
</html>
```
(结束)