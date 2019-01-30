# SpringBoot基础系列-内置Servlet容器
## 概述
SpringBoot支持的内置Servlet容器类型：
- Tomcat
- Jetty
- Undertow

内置Servlet容器默认启用端口8080。

针对内置的Tomcat而言，如果部署在CentOS服务器上的话，需要注意：Tomcat会自动使用一个临时目录来存放编译的jsp文件，上传的文件等，这个目录却可能会被tmpWatch发现而被删除，导致应用出错。可以通过自定义tmpWatch的配置，让它放过tomcat.*目录，又或者配置server.tomcat.basedir来指定一个目录而使得内置Tomcat使用一个非临时目录。
## 关于Servlet、Filter、Listener
两种注册方式：
### 方式一：作为Spring Beans
Servlet、Filter、Listener都是被注册到嵌入式容器中的Spring Bean。这样便于获取application.properties或者application.yml中的配置信息。

默认情况下，如果上下文中只有一个Servlet，那么它会映射到`/`，如果有多个Servlet，那么将以对应Servlet的beanName为路径前缀进行映射；Filter则会映射到`/*`。

如果对上面的默认映射方式不满意，可以使用ServletRegistrationBean，FilterRegistrationBean，ServletListenerRegistrationBean来进行自定义控制。

### 方式二：使用组件扫描器
使用@WebServlet、@WebFilter、@WebListener这三个分别标注在自定义的Servlet、Filter、Listener类上用于标识，然后由@ServletComponentScan注解开启自动扫描来加载之前三个注解标注的类。
> 以上注解仅仅适用于嵌入式Servlet容器，对于独立的Servlet容器而言是不起作用的。

## 关于ServletContext初始化
内置的Servlet容器是不能直接执行Servlet3.0+下的javax.servlet.ServletContainerInitializer（Servlet容器初始化器）或者Spring中的org.springframework.web.WebApplicationInitializer（web应用初始化器）的，这是有意而为，为了避免那些站门为运行在war包中而设计的初始化器影响到SpringBoot的以jar包方式运行。

在SpringBoot应用中正确的使用ServletContext初始化器的方法是：提供一个实现了org.springframework.boot.web.servlet.ServletContextInitializer接口的Bean。这个接口是函数式接口只有一个抽象方法：
```java
// 这个接口用于编程式配置一个Servlet3.0+的ServletContext。
// 实现了该接口的类不会被SpringServletContainerInitializer检测到，因此也就不能被Servlet容器自动执行。
// 这个接口的主要目的是为了将Servlet容器的初始化配置交给Spring来管理，而不是由Servlet容器自身来进行。
@FunctionalInterface
public interface ServletContextInitializer {
	void onStartup(ServletContext servletContext) throws ServletException;
}
```
## 关于ServletWebServerApplicationContext
ServletWebServerApplicationContext是SpringBoot为内嵌的Servlet容器专门提供的应用上下文。它通过搜索一个ServletWebServerFactory来引导自身的加载。而ServletWebServerFactory一般就是：
- TomcatServletWebServerFactory
- JettyServletWebServerFactory
- UndertowServletWebServerFactory

ServletWebServerApplicationContext在创建web容器实例的时候会执行ServletContextInitializer初始化器。
## 定制Servlet容器
### 方式一：application.properties
在application.properties中配置server.前缀的各种属性来定制Servlet容器。

常见设置包括：
- 网络设置：
    - 监听请求端口：server.port
    - 要绑定的接口地址：server.address
- Session设置：
    - 是否持久化session：server.servlet.session.persistence
    - session超时时间：server.servlet.session.timeout
    - session数据的位置：server.servlet.session.store-dir
    - session-cookie配置：server.servlet.session.cookie.*
- 错误管理设置：
    - 错误页面配置：server.error.path
- SSL设置：
- HTTP compression设置：
### 方式二：编程定制
    实现WebServerFactoryCustomizer接口，WebServerFactoryCustomizer提供了针对ConfigurableServletWebServerFactory的访问，而在ConfigurableServletWebServerFactory中有诸多定制set方法。
```java
@Component
public class MyWebServerFactoryCustomzer implements WebServerFactoryCustomizer {
    @Override
    public void customize(WebServerFactory factory) {
        if(factory instanceof TomcatServletWebServerFactory){
            TomcatServletWebServerFactory factory1 = (TomcatServletWebServerFactory)factory;
            // 执行定制
        }
    }
}
```
还可以通过直接定制ConfigurableServletWebServerFactory来实现定制:
```java
@Configurtion
public class AppConfig {
    @Bean
    public ConfigurableServletWebServerFactory webServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setPort(9000);
        factory.setSessionTimeout(10, TimeUnit.MINUTES);
        factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
        return factory;
    }
}
```


