# SpringBoot整合SpringCloud
## 整合Eureka（服务注册发现）
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
## 整合Consul（服务发现）
## 整合Ribbon（负载均衡）
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

## 整合Zuul

## 整合Config
