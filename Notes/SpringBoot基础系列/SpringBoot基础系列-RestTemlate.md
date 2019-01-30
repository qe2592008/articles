# SpringBoot基础系列-RestTemplate
## 一、概述
RestTemplate是Spring提供的用于访问Rest服务的类，内部封装了访问的细节，可以实现流数据到对象的自动转换，很是方便。

SpringBoot出现之后，微服务兴起，RestTemplate变成了SpringBoot开发的微服务之间相互远程调用的必要手段。
## 二、使用
### 2.1 配置
由于RestTemplate在使用之前均需要经过手动的配置，所以SpringBoot并未对其实现自动配置。

有两种配置方案，一种是使用SpringBoot提供的RestTemplateBuilder来构建RestTemplate实例，SpringBoot虽然未自动配置RestTemplate，但是却配置了RestTemplateBuilder，我们可以注入它来用它构建RestTemplate实例；另一种就是直接使用RestTemplate的构造器来创建实例。

如果是在SpringBoot应用中，推荐使用第一种方式，毕竟这是SpringBoot推荐的方式，而且这种方式还可以进行许多定制功能。如果实在一般的Spring Mvc项目中，那就必须使用第二种了，因为第一种的RestTemplateBuilder是由SpringBoot专门提供的。

下面罗列下简单的实现：

**第一种**
```java
@Configuration
public class RestTemplateConfig {
    @Autowired
    private RestTemplateBuilder restTemplateBuilder;
    @Bean
    public RestTemplate restTemplate(){
        return restTemplateBuilder.build();
    }
}
```
**第二种**
```java
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate(factory);
    }
    @Bean
    public SimpleClientHttpRequestFactory factory(){
        return new SimpleClientHttpRequestFactory();
    }
}
```
上面的两种实现中，都没有进行任何的定制，其实这是不可能的，在实际的应用中需要根据实际的情况进行各种定制，下面看看可以定制的内容。
### 2.2 定制
使用上面的第一种方式可以进行很多功能定制：
#### 2.2.1 数据转换器：HttpMessageConverter
在RestTemplate中默认有一套检测加载数据转换器的流程的，它会检测当前项目中有无指定的类，通过这个来决定是否加载指定的数据转换器。

在restTemplateBuilder中定义了三种方法用于配置数据转换器：
- defaultMessageConverters()：表示使用默认的数据转换器，这里不做设置
- messageConverters(Collection<? extends HttpMessageConverter<?>> messageConverters)：表示使用指定的数据转换器，默认的数据转换器将会全部失效
- additionalMessageConverters(Collection<? extends HttpMessageConverter<?>> messageConverters)：表示将指定的数据转换器添加到可用的转换器列表中

当然如果要添加新的数据转换器，需要手动实现HttpMessageConverter来编写转换逻辑，重写方法即可。
```java
public class MyMessageConverter<T> implements HttpMessageConverter<T> {
    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }
    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return false;
    }
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return null;
    }
    @Override
    public T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }
    @Override
    public void write(T t, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
    }
    public static MyMessageConverter create() {
        return new MyMessageConverter();
    }
}
```
```java
@Configuration
public class RestTemplateConfig {
    @Autowired
    private RestTemplateBuilder restTemplateBuilder;
    @Bean
    public RestTemplate restTemplate() {
        return restTemplateBuilder
//                .defaultMessageConverters()
//                .messageConverters(MyMessageConverter.create())
                .additionalMessageConverters(MyMessageConverter.create())
                .build();
    }

}
```
#### 2.2.2 请求拦截器
请求拦截器的功能是可以在请求到达服务器之前，拦截住进行一些处理，比如统一添加一些header信息等，操作完成之后，需要恢复请求流程。

数据转换器的作用是对请求响应中的数据与Java对象之间进行转换。

我们可以自定义拦截器，通过实现ClientHttpRequestInterceptor的唯一方法intercept来实现。
> ClientHttpRequestInterceptor是一个函数式接口，我们可以采用Lambda表达式来实现，避免新增一个类。
```java
@Configuration
public class RestTemplateConfig {
    @Autowired
    private RestTemplateBuilder restTemplateBuilder;
    @Bean
    public RestTemplate restTemplate() {
        return restTemplateBuilder
//                .interceptors(MyHttpRequestInterceptor.create())
                .additionalInterceptors(((request, body, execution) -> {
                    request.getHeaders().add("newHeaderKey", "newHeadervalue");
                    return execution.execute(request,body);
                }))// Lambda表达式
                .build();
    }
}
```
```java
public class MyHttpRequestInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        HttpHeaders headers = request.getHeaders();
        headers.add("newHeaderKey","newHeaderValue");
        return execution.execute(request,body);
    }
    public static MyHttpRequestInterceptor create(){
        return new MyHttpRequestInterceptor();
    }
}
```
推荐使用Lambda表达式方式。
#### 2.2.3 请求工厂
请求工厂就是ClientHttpRequestFactory，主要用于生成ClientHttpRequest请求实例。

ClientHttpRequest代表的是一个客户端请求，其中有一个方法execute，用于执行请求，并返回一个ClientHttpResponse响应实例。
```java
public interface ClientHttpRequest extends HttpRequest, HttpOutputMessage {
	ClientHttpResponse execute() throws IOException;
}
```
ClientHttpRequestFactory主要有两个实现类，分别采用不同的底层技术来实现Http请求：
- SimpleClientHttpRequestFactory：基于JDK的URLConnection来实现Http连接
- HttpComponentsClientHttpRequestFactory：基于Apache的HttpClient来实现Http连接

在这里设置请求工厂的目的主要也就是设置采用哪种底层机制来创建Http连接。
#### 2.2.4 定制器

#### 2.2.5 错误处理器

#### 2.2.6 超时时间

#### 2.2.7 
### 2.3 使用

## 三、原理解析

































RestTemplate用于访问远程Rest应用，一般RestTemplate均需要定制，因此SpringBoot并没有自动配置RestTemplate实例，但是却配置了RestTemplateBuilder实例，必要时可以从中构建一个RestTemplate实例出来。

RestTemplate是访问远程应用的一种方式，除此之外还有：
- JDK的URLConnection
- Apache的Http Client
- Netty的异步HTTP Client
- Feign
## 二、配置使用
正如上面所述，应用中RestTemplate均需要手动配置，配置方式也有两种，一种是使用SpringBoot官方推荐的RestTemplateBuilder来构建RestTemplate实例，另一种是直接使用RestTemplate的构造器来创建实例，实际上第一种方式是对第二种的再封装。











## 二、定制
有三种定制RestTemplate的方法：
### 2.1 使用RestTemplateBuilder

### 2.2 使用RestTemplateCustomizer

### 2.3 自定义RestTemplateBuilder


参考：
- [RestTemplate实践](https://www.cnblogs.com/duanxz/p/3510622.html)