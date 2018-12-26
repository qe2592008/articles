# SpringBoot高级系列-内置容器源码解析
## 概述
    SpringBoot采用了内置Servlet容器的方式来简化项目的部署。无需任何配置即可启动项目。
    要说容器的启动，还要追踪到javax.servlet包中。
## 基础
### Servlet 3.0+
    javax包是Java的基于web的扩展包，其中javax.servlet定义的是有关Servlet的相关接口和类，Servlet容器的实现都需要依赖于这个包，这也就为所有基于Servlet的web容器提供了一个统一的实现基础（这个基础就是Servlet）。Tomcat也不例外，我们可以在其目录lib下找到对应的jar包：servlet-api.jar。
    近年来我们已经逐步转移到Servlet 3.0+上来了，就以此为例。
    我们观察这个包，可以发现里面定义了许多我们经常使用的接口和类，包括Cookie,Servlet,Filter,HttpSession,ServletContext,ServletConfig等等。
    现在我们来深入了解下这个包里面的内容。
#### Servlet体系
    Servlet体系主要指的就是Servlet继承结构：Servlet <- GenericServlet <- HttpServlet
##### Servlet
    Servlet接口是一切java web应用的基础接口。这个接口中定义了所有Servlet都需要实现的方法：包括三个生命周期方法和两个辅助方法,具体如下源码注释所述。
```java
public interface Servlet {
    // 生命周期方法1-初始化方法，在构造一个新的Servlet时执行初始化操作
    public void init(ServletConfig config) throws ServletException;
    // 辅助方法1-获取ServletConfig配置信息，其中包括初始化值和一些启动参数
    public ServletConfig getServletConfig();
    // 生命周期方法2-请求处理方法，接收请求并进行处理并给出一个响应结果
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
    // 辅助方法2-获取有关Servlet的基本信息，例如：作者、版本、版权等信息
    public String getServletInfo();
    // 生命周期方法3-销毁方法，用于在该Servlet实例销毁之前清理资源之用
    public void destroy();
}
```
##### GenericServlet
    GenericServlet抽象类实现了Servlet接口，是其通用实现，提供了Servlet的通用功能，这是独立于协议的Srevlet实现，下面要介绍的HttpServlet就是在此基础上增加了对Http协议的支持，如果要实现基于其他别的协议的支持的Servlet,只要继承该抽象类即可。
    GenericServlet还实现了ServletConfig接口，同时还持有一个ServletConfig的引用。至于方法，基本上就是对Servlet和ServletConfig中方法的简单实现。
##### HttpServlet
    HttpServlet是javax提供的针对Http协议的Servlet抽象实现，他继承自GenericServlet，它针对Http的各种不同类型的请求分别定义了不同的处理方法：doXXX方法。
    Spring中的DispatchServlet就是继承自HttpServlet而来。
#### ServetContext体系
    
##### ServletConfig
    ServletConfig是Servlet配置，它和Servlet是一一对应的，主要被Servlet容器来在Servlet初始化时传递配置信息。
    ServletConfig接口内部定义的方法，都是一些与Servlet配置相关的方法。
```java
public interface ServletConfig {
    // 获取Servlet的名称，可能是由<servlet-name>配置的名称，或者是类名
    public String getServletName();
    // 获取当前ServletContext的引用
    public ServletContext getServletContext();
    // 获取给定name的初始化参数值
    public String getInitParameter(String name);
    // 获取配置中所有的初始化参数的name集
    public Enumeration<String> getInitParameterNames();
}
```
##### ServletContext
    
#### Filter体系
    
#### Request体系
    
#### Response体系
    
#### Async体系
    
#### Cookie和Session体系
    
    
    