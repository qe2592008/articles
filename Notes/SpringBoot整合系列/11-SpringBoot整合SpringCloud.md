# SpringBoot整合SpringCloud
## 整合Eureka（服务注册发现）
### 简介
### 单机Eureka Server
#### 第一步：添加必要的依赖POM
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
#### 第二步：添加必要的配置
application.yml
```yuml
# 单机配置
spring:
  application:
    name: eureka-server
server:
  port: 8000
eureka:
  instance:
    hostname: eureka-server
  client:
    # 单机不注册自身，默认为true，集群部署设置为true
    registerWithEureka: false
    # 单机不用拉取注册信息，默认为true，集群部署设置为true
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```
#### 第三步：添加必要的注解
启动类添加注解：@EnableEurekaServer
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```
（完成）
### 集群Eureka Server
#### 第一步：添加必要的依赖POM
同上
#### 第二步：添加必要的配置
application.yml
```yuml
#集群配置
spring:
  application:
    name: eureka-server
---
spring:
  profiles: server01
server:
  port: 8001
eureka:
  instance:
    hostname: server01
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://server03:8003/eureka/
---
spring:
  profiles: server03
server:
  port: 8003
eureka:
  instance:
    hostname: server03
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://server01:8001/eureka/
```
#### 第三步：添加系统配置
修改host文件，做好域名映射(windows环境)
```txt
127.0.0.1 server01
127.0.0.1 server03
```
#### 第四步：添加必要的注解
同上
#### 第五步：使用IDEA根据不同的profiles启动两个Eureka Server服务，形成集群
![]()
### Eureka Client
#### 第一步：添加必要的依赖POM
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
#### 第二步：添加必要的配置
```yuml
eureka:
  client:
    service-url:
      defaultZone: http://server01:8001/eureka/
  instance:
    prefer-ip-address: true
spring:
  application:
    name: eureka-invoke
server:
  port: 8081
```
#### 第三步，添加必要的注解
启动类添加注解：@EnableEurekaClient
```java
@EnableEurekaClient
@SpringBootApplication
public class EurekaInvokeApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaInvokeApplication.class, args);
    }
}
```
（完成）
## 整合Zookeeper（服务发现）
### 简介
### 
## 整合Consul（服务发现）
### 简介

## 整合Ribbon（负载均衡）
### 简介

### 基础整合
#### 第一步：添加必要的依赖POM
同Eureka Client
> Ribbon的依赖被Eureka Client所包含，如果单独使用Ribbon的话，可以单独添加Ribbon的依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```
#### 第二步：添加必要的配置
（无）
#### 第三步：添加必要的注解
与RestTemplate配合使用：@LoadBalanced
```java
@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
> Feign自动内置了Ribbon，无需设置，自动支持
### 自定义规则
> Ribbon默认采用的是轮询方式进行负载均衡，我们也可以自定义策略，甚至针对某个服务自定义,方法很简单，添加配置即可
```yuml
# 针对eureka-service服务进行负载均衡规则自定义设置，取代Java配置方式：RibbonConfig和RibbonRuleConfig
eureka-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```
## 整合Feign（声明式rest调用）
### 基础整合
#### 第一步：添加必要的依赖POM
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
> openfeign是新版的Feign组件
#### 第二步：
### 
## 整合Hystrix
### 通用整合
#### 第一步：添加必要的依赖POM
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<!--由于SpringBoot2.0版本没有自动导入javanica，所以需要手动添加下面的依赖，才能使用@HystrixCommond注解-->
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-javanica</artifactId>
    <version>RELEASE</version>
</dependency>
```
#### 第二步：添加必要的注解
启动类添加注解：@EnableCircuitBreaker或者@EnableHystrix
```java
@EnableEurekaClient
@SpringBootApplication
@EnableFeignClients
@EnableHystrix
public class EurekaInvokeApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaInvokeApplication.class, args);
    }
}
```
#### 第三步：使用Hystrix
InvokeApi.java
```java
@RestController
@RequestMapping("/invoke")
public class InvokeApi {
    @RequestMapping("/do")
    @HystrixCommand(fallbackMethod = "invokeFallback")
    public void invoke(){
        restTemplate.getForEntity(serviceUrl,null);
    }
    // 定义的出错调用的方法
    public void invokeFallback(){
        System.out.println("容错");
    }
}
```
#### 第四步：如果eureka-service不要启动，那么就是微服务不可用，那么就会执行invokeFallback方法
### 配置@HystrixCommond
#### 配置方法
```java
@RestController
@RequestMapping("/invoke")
public class InvokeApi {
    @RequestMapping("/do")
    @HystrixCommand(fallbackMethod = "invokeFallback", commandProperties = {
            @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500")
    },threadPoolProperties = {
            @HystrixProperty(name = "coreSize", value = "10"),
            @HystrixProperty(name = "maximumSize", value = "20")
    })
    public void invoke(){
        restTemplate.getForEntity(serviceUrl,null);
    }
    // 定义的出错调用的方法
    public void invokeFallback(){
        System.out.println("容错");
    }
}
```
#### commandProperties配置项
- execution.isolation.strategy：设置隔离策略
  - THREAD：线程隔离策略-它在单独的线程上执行，并发请求受到线程池中线程数量的限制（默认）
  - SEMAPHORE：信号隔离策略-它在调用线程上执行，并发请求受到信号量计数的限制
- execution.isolation.thread.timeoutInMilliseconds：设置调用超时时间，毫秒，超过时间直接执行回退方法
- execution.timeout.enabled：指定是否使用超时
- execution.isolation.thread.interruptOnTimeout：设置超时时是否中断执行
- execution.isolation.thread.interruptOnCancel：设置取消发生时是否中断执行
- execution.isolation.semaphore.maxConcurrentRequests：采用信号隔离策略时，用于设置最大并发请求量，超过设置拒绝执行
- fallback.isolation.semaphore.maxConcurrentRequests：设置调用回退方法的最大并发数，超过设置的线程直接抛出异常
- fallback.enabled：设置是否在发生异常后调用回退方法
- circuitBreaker.enabled：此属性确定断路器是否用于跟踪健康状况，以及当断路器跳闸时是否用于短路请求。
- circuitBreaker.requestVolumeThreshold：此属性设置将使电路跳闸的滚动窗口中的最小请求数，只有达到该值才能出发断路
- circuitBreaker.sleepWindowInMilliseconds：此属性设置断路后拒绝请求的时间量，然后允许再次尝试确定是否应该再次关闭电路。
- circuitBreaker.errorThresholdPercentage：此属性将错误率设置为或高于此错误率，电路应跳闸并开始短路请求回退逻辑。
- circuitBreaker.forceOpen：是否强制断路，优先于forceClosed
- circuitBreaker.forceClosed：是否强制开启通路
- metrics.rollingStats.timeInMilliseconds：设置统计滚动窗口的持续时间，毫秒
- metrics.rollingStats.numBuckets：设置滚动统计窗口划分的桶数
- metrics.rollingPercentile.enabled：是否应跟踪执行延迟并将其计算为百分比。如果禁用它们，所有汇总统计信息(平均值、百分位数)将返回为-1
- metrics.rollingPercentile.timeInMilliseconds：设置滚动窗口的持续时间，其中保存执行时间，以允许以毫秒为单位进行百分位数计算。
- metrics.rollingPercentile.numBuckets：设置滚动百分比窗口将被分成的桶数。
- metrics.rollingPercentile.bucketSize：设置每个bucket保存的最大执行次数
- metrics.healthSnapshot.intervalInMilliseconds：设置在允许获取计算成功和错误百分比并影响断路器状态的健康快照之间等待的时间(以毫秒为单位)
- requestCache.enabled：是否应该将HystrixCommand.getCacheKey()与HystrixRequestCache一起使用，以通过请求范围的缓存提供重复数据删除功能
- requestLog.enabled：是否应该将HystrixCommand执行和事件记录到HystrixRequestLog
#### threadPoolProperties配置项
- coreSize：设置核心线程池大小
- maximumSize：设置最大线程池大小
- maxQueueSize：设置BlockingQueue实现的最大队列大小
- queueSizeRejectionThreshold：设置队列大小拒绝阈值
- keepAliveTimeMinutes：设置保持活动时间(以分钟为单位)
- allowMaximumSizeToDivergeFromCoreSize：此属性允许最大尺寸配置生效。然后，该值可以等于或高于coreSize。设置coreSize < maximumSize将创建一个线程池，该线程池可以维持maximumSize并发性，但是在相对不活动期间将返回线程到系统
- metrics.rollingStats.timeInMilliseconds：此属性设置统计滚动窗口的持续时间(以毫秒为单位)。这是线程池保存度量的时间
- metrics.rollingStats.numBuckets：此属性设置滚动统计窗口划分的桶数
### 配置@HystrixCollapser
HystrixCollapser表示请求合并器，将多个相同接口的请求合并为一个请求一起发起，可以有效减少请求的数量以缓解依赖服务线程池的资源
#### 配置方法
```java
@RestController
@RequestMapping("/invoke")
public class InvokeApi {
    @HystrixCollapser(batchMethod = "invoke3s", collapserProperties = {
            @HystrixProperty(name = "maxRequestsInBatch", value = "100")
    })
    @RequestMapping("/do3")
    public void invoke3(String s){
        restTemplate.getForEntity(serviceUrl,null, s);
    }
    public void invoke3s(List<String> ss){
        restTemplate.getForObject(serviceUrl,null, StringUtils.join(ss, ","));
    }
}
```
> 注意：上面的例子只是伪代码，只为了讲述配置方式，执行不了，后期再修改    //TODO
#### collapserProperties配置项
- maxRequestsInBatch：此属性设置在触发批处理执行之前批处理中允许的最大请求数量。
- timerDelayInMilliseconds：此属性设置触发其执行的批处理创建后的毫秒数
- requestCache.enabled：此属性指示是否为调用hystrix折叠器.execute()和hystrixCollapser.queue()启用请求缓存
### 有关隔离策略的选择
- Hystrix的隔离策略有两种：THREAD和SEMAPHORE，默认是THREAD
- 一般情况保持默认即可
- 如果发生找不到上下文的运行时异常，可考虑隔离策略设置为SEMAPHORE
### Feign整合Hystrix
#### 第一步：添加必要的依赖
Feign的包中自动导入了Hystrix
#### 第二步：创建FeignClient
同上
#### 第三步：改造方式一：添加回调方法
```java
@FeignClient(name = "eureka-service", fallback = FeignClientFallback.class)
public interface InvokeFeign {
    @RequestMapping(value = "/servcie/do/{ss}", method = RequestMethod.GET,produces = "application/json;charset=UTF-8")
    void toInvoke(@PathVariable("ss") String ss);
}

@Component
class FeignClientFallback implements InvokeFeign{
    @Override
    public void toInvoke(String ss) {
        System.out.println("断路回调方法-"+ss);
    }
}
```
#### 第四步：改造方式二：打印回退日志
```java
@FeignClient(name = "eureka-service",
        fallbackFactory = FeignClientFallbackFactory.class
)
public interface InvokeFeign {
    @RequestMapping(value = "/servcie/do/{ss}", method = RequestMethod.GET,produces = "application/json;charset=UTF-8")
    void toInvoke(@PathVariable("ss") String ss);
}
@Component
class FeignClientFallbackFactory implements FallbackFactory<InvokeFeign> {
    private static final Logger LOG = LoggerFactory.getLogger(FeignClientFallbackFactory.class);
    @Override
    public InvokeFeign create(Throwable cause) {
        return new InvokeFeign() {
            @Override
            public void toInvoke(String ss) {
                FeignClientFallbackFactory.LOG.info("fallback reason was "+ cause);
                System.out.println("断路回调方法-"+ss);
            }
        };
    }
}
```
> 为指定FeignClient禁用Hystrix：
>
> ```java
> @Configuration
> public class FeignDisableHystrixConfig {
>     @Bean
>     @Scope("prototype")
>     public Feign.Builder feignBuilder(){
>         return Feign.builder();
>     }
> }
> ```
> ```java
> @FeignClient(name = "eureka-service",
>         configuration = FeignDisableHystrixConfig.class
> )
> public interface InvokeFeign {
>     @RequestMapping(value = "/servcie/do/{ss}", method = RequestMethod.GET,produces = "application/json; charset=UTF-8")
>     void toInvoke(@PathVariable("ss") String ss);
> }
>```

> 全局禁用Hystrix
>
> ```yaml
> feign.hystrix.enabled=fasle
> ```
## 整合Zuul（微服务网关）
### 简介
Zuul是微服务网关，Java代码开发。
Zuul的功能均是由过滤器来实现的，Zuul内部的过滤器分为四种：
- PRE（前置过滤器）：请求被路由之前进行调用，可实现身份验证、在集群中选择微服务、记录调试信息等功能
- ROUTE（路由过滤器）：将请求路由到微服务
- POST（后置过滤器）：请求被微服务执行完毕，返回响应后进行调用，可实现为响应添加Header、收集统计信息与指标、将响应从微服务发到客户端等功能
- ERROR（错误过滤器）：在上述阶段出现异常直接调用错误过滤器处理异常
### 基础整合
#### 第一步：创建SpringBoot工程，添加必要的依赖POM
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```
#### 第二步：添加必要的配置
```yaml
server:
  port: 8090
spring:
  application:
    name: eureka-gateway
eureka:
  client:
    service-url:
      defaultZone: http://server01:8001/eureka/
```
#### 第三步：添加必要的注解
启动类添加注解：@EnableZuulProxy
```java
@SpringBootApplication
@EnableZuulProxy
public class EurekaGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaGatewayApplication.class, args);
    }
}
```
### 路由设置
#### 第一步：补充必要的配置
```yaml
zuul:
  routes:
    # 单独使用、设置时需要手写IP端口：将api-b映射为127.0.0.1:8081
    invoke-api:
      path: /api-b/**
      url: http://127.0.0.1:8081
    # 与Eureka整合使用时可以直接使用应用名eureka-invoke：将api-a映射为eureka-invoke服务
    route-a:
      path: /api-a/**
      serviceId: eureka-invoke
```
#### 第二步：测试
浏览器访问：
```text
http://localhost:8090/api-a/invoke/do
http://localhost:8090/api-b/invoke/do
```
效果与直接访问：
```text
http://localhost:8081/invoke/do
```
效果一致
### 自定义Zuul过滤器
方式：继承抽象类ZuulFilter
```java
public class RequestLogFilter extends ZuulFilter {
    private static final Logger log = LoggerFactory.getLogger(RequestLogFilter.class);

    /**
     * 定义该过滤器的类型：pre、route、post、error
     * @return
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 定义过滤器执行顺序
     * @return
     */
    @Override
    public int filterOrder() {
        return 1;
    }

    /**
     * 定义是否执行该过滤器，返回true表示执行，false表示不执行
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 定义过滤器具体逻辑
     * @return
     */
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(request.getMethod()+"_"+request.getRequestURL().toString());
        return null;
    }
}
```
然后将该类加载到环境中
```java
@Configuration
public class CustomFilterConfig {
    @Bean
    public RequestLogFilter requestLogFilter(){
        return new RequestLogFilter();
    }
}
```
### 禁用某个过滤器
#### 添加必要的配置即可
```yaml
zuul:
  # 禁用RequestLogFilter过滤器
  RequestLogFilter:
    pre:
      disable: true
```
### Zuul的容错与回退
Zuul中默认已经内置了Hystrix，自动拥有容错能力，但是要实现回退还需要添加回退方法
#### 为Zuul添加回退
新版本需要实现FallbackProvider接口，老版本是ZuulFallbackProvider接口
```java
@Component
public class EurekaInvokeFallbackProvider implements FallbackProvider {
    /**
     * 定义为那个微服务提供回退
     * @return
     */
    @Override
    public String getRoute() {
        return "eureka-invoke";
    }

    /**
     *
     * @param route
     * @param cause
     * @return
     */
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            /**
             * 定义回退的状态码
             * @return
             * @throws IOException
             */
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            /**
             * 定义回退的状态码（数值类型）
             * @return
             * @throws IOException
             */
            @Override
            public int getRawStatusCode() throws IOException {
                return this.getStatusCode().value();
            }

            /**
             * 定义回退的状态内容（文本）
             * @return
             * @throws IOException
             */
            @Override
            public String getStatusText() throws IOException {
                return this.getStatusCode().getReasonPhrase();
            }

            @Override
            public void close() {
            }

            /**
             * 定义回退的响应体
             * @return
             * @throws IOException
             */
            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("eureka-invoke服务暂不可用，请稍后重试！".getBytes());
            }

            /**
             * 定义回退的headers
             * @return
             */
            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                MediaType type = new MediaType("application","json", Charset.forName("UTF-8"));
                headers.setContentType(type);
                return headers;
            }
        };
    }
}
```
### 在zuul服务器整合所有微服务的swagger
#### 第一步：为每个微服务和zuul服务器整合swagger2
参照[《SpringBoot整合Swagger2》](/Notes/SpringBoot整合系列/1-Springboot整合Swagger2.md)
#### 第二步：修改zuul服务器的Swagger2Config配置类
```java
@Configuration
@EnableSwagger2 //启用swagger2
@Primary// 多个Bean时优先使用
public class Swagger2Config implements SwaggerResourcesProvider {
    @Autowired
    RouteLocator routeLocator;
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .pathMapping("/")
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.eurekagateway"))// 这里基本每个应用都不同
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("eureka-gateway")// 这里基本每个应用都不同
                .description("Springboot整合Zuul")// 这里基本每个应用都不同
                .version("0.0.1")
                .build(); // 这部分信息其实可以自定义到配置文件中读取
    }

    @Override
    public List<SwaggerResource> get() {
        List<SwaggerResource> resources = new ArrayList<>();
        // 将当前应用添加到resources中
        resources.add(swaggerResource("zuul-gateway","/v2/api-docs","1.0"));
        // 循环将Eureka中获取的服务全部添加到resources中
        routeLocator.getRoutes().forEach(route -> {
            resources.add(swaggerResource(route.getId(),route.getFullPath().replace("**", "v2/api-docs"),"1.0"));
        });
        return resources;
    }

    private SwaggerResource swaggerResource(String name, String location, String version) {
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation(location);
        swaggerResource.setSwaggerVersion(version);
        return swaggerResource;
    }
}
```
## 整合Config
### 简介

### 基础整合
#### 第一步：新建SpringBoot工程，添加必要的依赖POM
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
#### 第二步：添加必要的配置
```yaml

```
#### 第三步：添加必要的注解
启动类添加注解@EnableConfigServer
```java
@EnableConfigServer
@SpringBootApplication
public class EurekaConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaConfigApplication.class, args);
    }
}
```