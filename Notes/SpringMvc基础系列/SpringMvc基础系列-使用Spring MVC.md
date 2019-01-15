# SpringMvc基础系列-使用Spring MVC
## 一、概述
关于使用Spring MVC，有如下几点需要了解：
- 配置DispatchServlet
- 创建Controller
- 使用数据绑定
- 使用类型转换器
- 使用参数校验器
- 使用文件上传下载
- 测试
- 注解总结
## 配置DispatchServlet
关于DispatchServlet，请查看[SpringMvc基础系列-DispatchServlet]()

DispatchServlet是一个Servlet，要想在应用中使用，必须经过配置：
### web.xml
```xml
<web-app>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```
### WebApplicationInitializer
```java
public class MyWebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        
        ServletRegistration.Dynamic dispatcher =
                container.addServlet("dispatcher", new DispatcherServlet(appContext));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");
    }
}
```
WebApplicationInitializer接口用来实现以编程方式配置ServletContext，代替web.xml。

详情查看[SpringMvc基础系列-WebApplicationInitializer]()
### Starter
在于SpringBoot整合使用的时候，就很简单了，因为一切都被自动配置完成了。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
详情可见[SpringBoot整合Spring MVC](https://github.com/qe2592008/articles/blob/develop/Notes/SpringBoot%E6%95%B4%E5%90%88%E7%B3%BB%E5%88%97/9-SpringBoot%E6%95%B4%E5%90%88SpringMvc.md)
和[SpringBoot基础系列-web开发](https://github.com/qe2592008/articles/blob/develop/Notes/SpringBoot%E5%9F%BA%E7%A1%80%E7%B3%BB%E5%88%97/7-SpringBoot%E5%9F%BA%E7%A1%80%E7%B3%BB%E5%88%97-web%E5%BC%80%E5%8F%91.md)
## 控制器
使用@Controller标注于类，在一个类中就可以定义多个处理器，每个处理器使用注解@RequestMapping分配一个唯一的映射请求即可。

注意，标注了@Controller和@Component、@Service、@Repository的类或接口，都需要配置注解扫描器来扫描发现。当然这在SpringBoot里面是自动配置完成了，其他时候就需要手动配置了
- xml方式：
```xml
<context:component-scan base-package-"com.example.controller"/>
```
- 注解方式

使用@ComponentScan
### 重定向

## 数据绑定

## 类型转换

## 参数校验

## 文件上传

## 文件下载

## 注解总结
### @Controller
一般标注于控制器类上方，作用和@Component一样，只是有一层逻辑意义，用于标识控制器类。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	// 指定控制器Bean的名称
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```
### @RestController
其实就是@Controller和@ResponseBody注解的组合，当二者一起出现时，就可以使用该注解替换。

一般用于开发Rest API的控制器类上。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {

	// 指定控制器Bean名称
	@AliasFor(annotation = Controller.class)
	String value() default "";
}
```
### @RequestMapping
主要用于定义请求映射路径URI。
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
	// 指定映射的名称，如果在类和方法均使用了该方法，则最后的名称为二者的组合，中间用#隔开
	String name() default "";
	
	// 指定映射路径，如果来类和方法均使用了该方法，则最后的路径为二者的组合，中间用/隔开
	// 作用和path方法的作用一样
	@AliasFor("path")
	String[] value() default {};// 常用
	
	// 指定映射路径，可以使用ANT格式匹配模式，与value方法一致
	@AliasFor("value")
	String[] path() default {};
	
	// 指定要映射的请求方法，值包括GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE.
	// 可定义多个
	RequestMethod[] method() default {};// 常用
	
	// 定义参数匹配条件，可定义多个
	// 支持：
	//      myParam=myValue（某个参数等于某个值）、
	//      myParam!=myValue（某个参数不等于某个值）、
	//      !myParam（不存在某个参数）
	String[] params() default {};
	
	// 定义请求头匹配条件，可定义多个
	// 支持：
	//      My-Header=myValue（某个请求头等于某个值）
	//      My-Header!=myValue（投个请求头不等于某值）
	//      !My-Header（不存在某个请求头）
	String[] headers() default {};
	
	// 指定请求媒体类型匹配条件，可以定义多个
	// 支持："text/plain"，{"text/plain", "application/*"}，"!text/plain"
	String[] consumes() default {};
	
	// 定义响应媒体类型匹配条件，可定义多个，支持同上
	String[] produces() default {};
}
```
由该注解引导出的注解包括：
- @GetMapping
- @DeleteMapping
- @PatchMapping
- @PostMapping
- @PutMapping

这几个注解中的内容和RequestMapping基本一致，只是去掉了method方法，毕竟请求方法已被限定。
### @RequestBody
标注于Controller方法参数之上，用于指定，请求body中的数据需要绑定到指定的模型参数中。
这里绑定的过程需要有HttpMessageConverter来完成，并且在绑定时，还可以由@Valid来完成参数校验工作
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {
	// 指定请求body是否是必须的，默认为true，如果请求body为null会抛出异常，
	// 指定位false之后，如果为请求body为null,则绑定到方法参数的值为null
	boolean required() default true;
}
```
### RequestHeader
标注于控制器方法参数之上，用于将请求头信息绑定到指定参数之上。

如果方法参数是Map<String,String>、MultiValueMap<String,String>、HttpHeaders，则会将所有的请求头信息都绑定到其中。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestHeader {

	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 要绑定的请求头名称，只在绑定一个请求头信息时指定
	@AliasFor("value")
	String name() default "";

	// 指定要绑定的请求头是否必需，默认为true，若不存在则抛异常，置为false,若不存在则为null
	boolean required() default true;

	// 指定请求头的默认值，暗中将required置为false
	String defaultValue() default ValueConstants.DEFAULT_NONE;
}
```
### @RequestParam
标注于控制器方法参数之上，用于将请求参数绑定到指定参数之上。

如果参数类型为Map，并且制定了请求参数的名称，则需要采用合适的转换策略将请求餐宿转换为一个Map键值对

如果参数类型为Map<String,String>，或者MultiValueMap<String,String>，并且没有指定要绑定的请求参数名，那么将会将所有的请求参数的键值对保存到指定的Map参数中。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestParam {
	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 指定要绑定的请求参数名称
	@AliasFor("value")
	String name() default "";

	// 指定所有绑定的请求参数是否必需，默认为true，如果不存在则抛异常，置为false，如果不存在则为null
	boolean required() default true;

	// 指定请求参数的默认值，暗中将required置为false
	String defaultValue() default ValueConstants.DEFAULT_NONE;
}
```
### @RequestAttribute
标注于控制器方法参数之上，用于将请求属性绑定到指定参数之上。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestAttribute {

	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 指定要绑定的请求属性的名称，默认的名称是从绑定到的方法参数名称推导出来的
	@AliasFor("value")
	String name() default "";

	// 指定请求属性是否是必需的，默认为true，没有则抛异常，
	// 置为false，若请求属性不存在，则参数为null，或者Optional
	boolean required() default true;
}
```
### @CookieValue
标注于控制器方法参数之上，用于注定将某个cookie的值绑定到这个控制器方法参数上。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CookieValue {
	// name方法的别名
	@AliasFor("name")
	String value() default "";
	
	// 指定要执行绑定的cookie名称
	@AliasFor("value")
	String name() default "";

	// 执行要绑定的cookie是否是必需的，默认为true，如果不存在将会抛出异常，
	// 若设为false，如果不存在，则当前参数为null
	boolean required() default true;
	
	// 指定一个默认值，同时暗中指定了required为false。
	String defaultValue() default ValueConstants.DEFAULT_NONE;
}
```
### @PathVariable
该注解用于控制器方法参数之上，用于将路径中的变量绑定到参数中。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PathVariable {

	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 要绑定的路径变量名，即路径中用‘/{v}’指定的v
	@AliasFor("value")
	String name() default "";

	// 指定是否路径变量是必需的，默认为true，如果不存在则抛出异常，置为null，则为null，或者Optional
	boolean required() default true;

}
```
### @RequestPart
标注于控制器方法的参数之上，专门针对请求参数中的multipart/form-data参数进行绑定。

可以绑定的参数的类型包括：MultipartFile配合MultipartResolver、javax.servlet.http.Part配合Servlet 3.0的Multipart请求。

要做文件上传功能时就会用到这个注解了。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestPart {
	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 指定要绑定的请求参数名称，这个请求必然是multipart/form-data请求
	@AliasFor("value")
	String name() default "";

	// 指定的part是否必需，默认为true，如果不存在则抛异常，置为false，如果不存在则为null
	boolean required() default true;
}
```
### @MatrixVariable
该注解用的并不多，它标注于控制器方法参数之上，能够将请求路径中指定的键值对绑定到指定的参数中

该功能需要单独的开启
```xml
<mvc:annotation-driven enable-matrix-variables="true"/>  
```
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MatrixVariable {

	// name方法的别名
	@AliasFor("name")
	String value() default "";

	// 指定矩阵变量的名称
	@AliasFor("value")
	String name() default "";

	// 矩阵变量的所在路径段的名称
	String pathVar() default ValueConstants.DEFAULT_NONE;

	// 矩阵变量是否必需，默认为true，如果不存在则抛出异常，置为false，不存在则为null
	boolean required() default true;

	// 指定矩阵变量的默认值，暗中将required置为false了
	String defaultValue() default ValueConstants.DEFAULT_NONE;
}
```
### @ResponseBody
可标注于方法和类之上，是一个标志注解，内部无方法，表示被标注方法的放回结果需要写入响应body中。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseBody {
}
```
### @ResponseStatus
可用于标注方法和异常类，指明需要放回的code和reason。

应用到控制器方法时，将会在方法被调用之后将指定的code应用到响应中去，这个code将会覆盖其他方式产生的code，比如ResponseEntity或者redirect:。

当在一个异常类上使用该注解，或者设置reason属性的时候，将会触发HttpServletResponse.sendError方法。

使用HttpServletResponse.sendError方法之后，响应就结束了，不会再往里面写入任何东西了。并且Servlet容器会将reason写入一个Html错误页面，这个对于Rest API明显不合适，对于这种情况，不推荐使用ResponseStatus注解，而是使用ResponseEntity。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ResponseStatus {

	// code的别名方法
	@AliasFor("code")
	HttpStatus value() default HttpStatus.INTERNAL_SERVER_ERROR;
	
	// 指定返回的状态编码，默认为500
	@AliasFor("value")
	HttpStatus code() default HttpStatus.INTERNAL_SERVER_ERROR;

	// 指定响应的原因描述
	String reason() default "";
}
```
### CrossOrigin

### SessionAttribute
标注于控制器方法参数上，用于将session属性绑定到指定的方法参数上。
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface SessionAttribute {

	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 指定要绑定到方法参数上的session属性的名称
	@AliasFor("value")
	String name() default "";

	// 指定session属性是否必需，默认为true，如果不存在则抛出异常，
	// 置为false,如果不存在则参数值为null
	boolean required() default true;
}
```
### SessionAttributes
标注于控制器类之上，用于指定将当前控制器内的各个处理器方法中的某些返回值的范围为session范围。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface SessionAttributes {

	// names的别名方法
	@AliasFor("names")
	String[] value() default {};

	// 指定需要保存到session中的model内部的属性名称，那么model中指定名称的属性会保存到session中
	@AliasFor("value")
	String[] names() default {};

	// 指定需要保存到session的model内部的属性的类型，那么model中所有这种类型的属性都会保存到session中
	Class<?>[] types() default {};
}
```
### @ControllerAdvice
这个注解可解析为控制器通知，它可以对所有被@Controller标注的类起作用，也就是对所有的控制器起作用。

它主要用于将@ExceptionHandler、@InitBinder、@ModelAttribute标注的方法内容作用到所有的控制器。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {

	// basePackages的别名方法
	@AliasFor("basePackages")
	String[] value() default {};

	// 指定作用到的控制器的包的范围
	@AliasFor("value")
	String[] basePackages() default {};

	// 通过指定具体的类来限定作用范围，作用于指定类所在的包范围
	Class<?>[] basePackageClasses() default {};

	// 指定一些控制器类型，表示只有这些类型的控制器能够被作用到
	Class<?>[] assignableTypes() default {};
	
	// 指定一些注解，表示被这些注解的任意个标注的控制器类都会被作用到
	Class<? extends Annotation>[] annotations() default {};
}
```
### @RestControllerAdvice
这个注解就是@ControllerAdvice和@ResponseBody的组合，当两个注解一起出现的时候，就可以用这个注解替换。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {

	// 同上
	@AliasFor("basePackages")
	String[] value() default {};

	// 同上
	@AliasFor("value")
	String[] basePackages() default {};

	// 同上
	Class<?>[] basePackageClasses() default {};

	// 同上
	Class<?>[] assignableTypes() default {};

	// 同上
	Class<? extends Annotation>[] annotations() default {};
}
```
### @ExceptionHandler
### @InitBinder
### @ModelAttribute
该注解标注于控制器方法参数或者控制器方法之上，分别表示将指定的参数或者方法返回值保存到Model中，暴露给view。
```java
@Target({ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ModelAttribute {

	// name的别名方法
	@AliasFor("name")
	String value() default "";

	// 指定参数或者返回值绑定到Model时的名称（键名）,
	// 如果未指定，将会从参数会返回值类型上推导出来一个名称
	@AliasFor("value")
	String name() default "";
	
	// 是否启用数据绑定，默认为true,置为false，则会禁用数据绑定。
	boolean binding() default true;
}
```

