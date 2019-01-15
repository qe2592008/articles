# SpringMvc基础系列-DispatchServlet
## 一、概述
SpringMvc实现web框架的基础就是DispatchServlet，又称为前端控制器。

DispatchServlet，从命名就能看出它的作用，分发，分发的是什么呢？当然就是请求了。从此可以得出，DispatchServlet主要用于直接面对各种请求，将其分发给具体的请求处理组件来执行。

下面看看如何在应用中配置DispatchServlet：
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
### JavaConfig
```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletCxt) {
        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();
        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```
### SpringBoot Starter
在pom中添加如下依赖，会自动完成配置
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
详情可见[SpringBoot整合Spring MVC](https://github.com/qe2592008/articles/blob/develop/Notes/SpringBoot%E6%95%B4%E5%90%88%E7%B3%BB%E5%88%97/9-SpringBoot%E6%95%B4%E5%90%88SpringMvc.md)
## 