# SpringBoot基础系列-web开发
## 一、整合Spring MVC
### 1.1 Spring MVC自动配置
当我们在POM中添加spring-boot-starter-web之后，SpringBoot就会自动进行SpringMVC整合配置，这些配置内容包括：
- 自动创建ContentNegotiatingViewResolver和BeanNameViewResolver的实例Bean
- 提供对持静态资源，包括WebJar的支持
- 自动创建Converter、GenericConverter和Formatter的实例Bean
- 提供对HttpMessageConverters的支持
- 自动创建MessageCodesResolver实例Bean
- 提供对静态欢迎页面index.html的支持
- 定制Favicon的支持
- 自动使用ConfigurableWebBindingInitializer实例Bean
### 1.2 定制Spring MVC
#### 1.2.1 定制方式一
保留默认的自动配置，然后在其基础上新增一些配置：
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
#### 1.2.2 定制方式二
完全控制Spring MVC，手动定制其各种功能：
```java
@EnableWebMvc
@Configuration
public class WebMvcConfig {
    //...自定义实现WebMvcConfigurer中的若干默认方法
}
```
#### 1.2.3 定制方式三
定制RequestMappingHandlerMapping、RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver实例：

通过声明一个WebMvcRegistrationsAdapter实例来提供这些组件。
### 1.3 HttpMessageConverters
即Http消息转换器，主要用于转换Http请求和响应，比如Objects会被自动转换成为JSON格式或者XML格式。编码类型默认为UTF-8。

可以定制该转换器，方式为：
```java
@Configuration
public class MyConfiguration {
	// 定制HttpMessageConverters
    @Bean
	public HttpMessageConverters customConverters() {
		HttpMessageConverter<?> additional = ...
		HttpMessageConverter<?> another = ...
		return new HttpMessageConverters(additional, another);
	}
}
```
### 1.4 定制JSON序列化与反序列化
SpringBoot默认使用Jackson进行Json操作。

可以定制序列化与反序列化操作，方式为：
```java
@JsonComponent
public class Example {
	public static class Serializer extends JsonSerializer<SomeObject> {
		// 定制json序列化逻辑...
	}
	public static class Deserializer extends JsonDeserializer<SomeObject> {
		// 定制json反序列化逻辑...
	}
}
```
或者
```java
@JsonComponent
public static class Serializer extends JsonSerializer<SomeObject> {
    // 定制json序列化逻辑...
}
```
#### 1.4.1 关于注解@JsonComponent
看看源码：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface JsonComponent {
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```
可以看到该注解是一个@Component，那么他的作用就类似与@Component,主要用于注册Bean。
### 1.5 MessageCodesResolver
即消息编码解析器，是Spring MVC内部用来生成错误编码来表示错误信息的。
### 1.6 静态内容
[待补充]
### 1.7 欢迎页面
SpringBoot首先会查找index.html静态欢迎页面，如果找不到再查找index.ftl之类的模板欢迎页面。
### 1.8 定制应用图标
SpringBoot会在配置的静态资源路径和类路径中（先后顺序）查找favicon.ico图标，将其用作应用图标。
### 1.9 ConfigurableWebBindingInitializer
SpringMVC通过一个WebBindingInitializer来为特定的请求提供一个WebDataBinder。如果自定义了ConfigurableWebBindingInitializer，那么SpringBoot将自动配置使SpringMVC使用它。
### 1.10 模板引擎
SpringBoot提供对以下模板引擎的自动支持：
- Freemarker
- Groovy
- Thymeleaf
- Mustache
### 1.11 错误处理
默认情况下，Spring Boot提供了一个/error映射，以合理的方式处理所有错误，并在servlet容器中注册为“全局”错误页面。
即在SpringBoot内部提供了这么一个控制器类BasicErrorController，接收/error请求，然后针对浏览器请求和客户端请求两种情况作了映射，分别返回不同的内容。浏览器请求返回一个公共的错误页面，而客户端请求则返回一个ResponseEntity实例。
#### 1.11.1 定制错误处理功能
- 方式一：定制错误页面
定制错误页面就是针对不同的code定义页面
在resources目录下的static目录(或者templates目录)下定义error目录，在error目录中定义401.html,404.html,500.html等错误页面，一旦SpringBoot应用发生了401、404、500错误就会跳转到自定义的错误页面中，而对于未自定义编码的错误还会跳转到公共错误页面
/static/error/404.html
/static/error/500.html
/templates/error/404.ftl
/templates/error/500.ftl
> 注意：必须定义到上面所说的目录中，而且名称必须为:错误编码.html格式，如果不按照以上规则，则定制不成功，其实如果想要自定义错误页面地址和名称也是可以的，只不过需要多加一个步骤：
添加EmbeddedServletContainerCustomizer的Bean实例用于手动设置错误页面的映射关系：
假如将500.html错误页面创建到resources目录下，也就是类路径根目录下，那么就需要使用如下自定义的ErrorViewResolver来处理了：
> /500.html
内容为：
> ```html
> <p>根目录的500错误文件</p>
>```
> MyErrorVivwResolver.java
> ```java
> @Component
> public class MyErrorVivwResolver implements ErrorViewResolver,ApplicationContextAware {
>    @Override
>    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
>        Resource resource = this.applicationContext.getResource("classpath:/");
>        try {
>            resource = resource.createRelative(status.value() + ".html");
>        } catch (IOException e) {
>            e.printStackTrace();
>        }
>        ModelAndView modelAndView = new ModelAndView(new HtmlResourceView(resource), model);
>        return modelAndView;
>    }
>    ApplicationContext applicationContext;
>    @Override
>    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
>        this.applicationContext = applicationContext;
>    }
>    private static class HtmlResourceView implements View {
>        private Resource resource;
>        HtmlResourceView(Resource resource) {
>            this.resource = resource;
>        }
>        @Override
>        public String getContentType() {
>            return MediaType.TEXT_HTML_VALUE;
>        }
>        @Override
>        public void render(Map<String, ?> model, HttpServletRequest request,
>                           HttpServletResponse response) throws Exception {
>            response.setContentType(getContentType());
>            FileCopyUtils.copy(this.resource.getInputStream(),
>                    response.getOutputStream());
>        }
>    }
> }
> ```
代码中不少内容是抄自SpringBoot内置的DefaultErrorViewResolver。
页面请求：
```text
http://localhost:8080/error
```
页面跳转到500错误页面，页面显示：
```text
根目录的500错误文件
```
- 方式二：无SpringMVC的错误页面映射（一般不涉及）
在不使用SpringMVC的情况下进行错误页面映射，需要使用ErrorPageRegistrar(错误页面注册器)来直接注册ErrorPages(错误页面)。
这个注册器直接与底层嵌入式servlet容器一起工作，即使没有Spring MVC的DispatcherServlet也可以工作。

### 1.12 跨域请求
跨源资源共享(Cross-origin resource sharing, CORS)是由大多数浏览器实现的W3C规范，它允许您以灵活的方式指定哪种跨域请求被授权，而不是使用一些不太安全、功能不太强大的方法，比如IFRAME或JSONP。
有两种配置方式：
#### 1.12.1 全局配置
全局配置针对的是应用的所有控制器接口
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    // 跨域请求全局配置
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/book/**");
    }
    //...自定义实现WebMvcConfigurer中的若干默认方法
}
```
#### 1.12.2 细粒度配置
细粒度指的是针对单个控制器中的方法，甚至是单个方法进行配置，使用@CrossOrigin注解
```java
@RestController
@RequestMapping("/book")
@Api(description = "书籍接口")
@Log4j2
@CrossOrigin(maxAge = 3600)
public class BookApi {
    
    @Autowired
    private BookService bookService;
    
    @CrossOrigin("http://localhost:8081")
    @RequestMapping(value = "/getBook", method = RequestMethod.GET)
    @ApiOperation(value = "获取一本书籍", notes = "根据ID获取书籍", httpMethod = "GET")
    public ResponseEntity<Book> getBook(final int bookId){
        return bookService.getBook(bookId);
    }
}
```
## 整合Spring WebFlux(暂不涉及)
Spring WebFlux是Spring响应式web框架。它不需要Servlet,并且所有请求都是异步非阻塞的。
该框架有两种开发风格，一种是基于注解的风格，一种是以功能为主的风格
基于注解的风格开发类似于SpringMVC
(暂不涉及)