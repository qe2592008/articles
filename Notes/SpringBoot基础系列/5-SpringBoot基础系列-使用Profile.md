# SpringBoot基础系列-使用Profile
## 一、概述
Profile主要用于区分不同的环境。
## 二、使用
### 2.1 @Profile
在某个类、或者方法上添加@Profile注解，指定具体的profile环境标签，那么只有在该profile处于active的情况下该类、方法才会被加载、执行。
```java
@Profile({"dev","test"})
public class Xxx{
    
    @Profile({"dev"})
    @Bean
    public Xxx xxx(){
        return new Xxx();
    }
}
```
### 2.2 多环境配置
#### 2.2.1 properties配置文件
使用properties配置文件实现多环境配置，只能通过添加多个application-{profile}.properties来实现。

比如：application-dev.properties,application-test.properties
#### 2.2.2 YAML配置文件
使用YAML实现多环境配置要简单的多，只需要一个文件即可，application.yml

在文件中使用---来区分多个环境，每个环境都需要配置spring.profile属性，不配置的属于默认环境
```yaml
server:
  port: 8080
#属性映射测试
app:
  name: springdemo
  size: 100M
  user: weiyihaoge
  version: 0.0.1
---
spring:
  profiles: dev
server:
  port: 8081
---
spring:
  profiles: test
server:
  port: 8082
---
spring:
  profiles: pro
server:
  port: 8083
```
### 2.3 激活profiles
可以在命令行参数、系统参数、application.properties等处进行配置
#### 2.3.1 命令行
```youtrack
--spring.profiles.active=dev
```
#### 2.3.2 application.properties
```properties
spring.profiles.active=dev
```
### 2.4 添加profiles
我们可以在不修改已启动的profiles的基础上添加新的profiles

使用spring.profiles.include属性进行配置

还可以使用编程的方式实现，使用如下的方式添加：
```java
SpringApplication.setAdditionalProfiles("development");
```