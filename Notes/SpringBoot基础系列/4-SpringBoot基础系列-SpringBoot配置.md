# SpringBoot基础系列-SpringBoot配置
## 一、概述
属性配置方式：
- properties文件
- yml文件
- 环境变量
- 命令行参数

属性值的使用方式：
- @Value("${propertyKey}")注解获取
- 从Environment中获取
- 使用@ConfigurationProperties绑定到Bean
## 二、具体配置
### 2.1 随机值配置
适用类型：
- integers
- longs
- uuids
- strings

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```
> 说明：rendom.value结果是strings；random.int*结果是integers；random.long结果是longs；random.uuid结果是uuids

### 2.2 命令行属性
命令行属性会被转换成为property，而保存到Environment之中，而且优先级极高，一般是在最后进行保存，如果有相同的属性会进行覆盖。
### 2.3 `application.properties`配置文件
格式：
- *.properties
- *.yml

`application.properties`属性文件会被SpringBoot应用自动加载，而且有一个加载顺序：
- 当前目录的/config子目录下
- 当前目录下
- classpath目录的/config子目录下
- classpath目录下
> 上面的排列顺序从上到下是按照优先级从高到低排列，而实际上我们一般使用都在classpath目录下

通过Environment属性`spring.config.name`我们可以自定义`applicaiton.properties`文件的名称，通过Environment属性`spring.config.location`自定义`applicaiton.properties`文件的位置。这两个配置要在应用启用之前配置，所以需要将其配置到系统环境变量或者系统参数或者命令行参数中优先读取。
```youtrack
java -jar xxx.jar --spring.config.name=myAppConfig
java -jar xxx.jar --spring.config.location=classpath:custon-config/,file:./custon-config/
java -jar xxx.jar --spring.config.additional-location=classpath:custon-config/,file:./custon-config/
```
上面将其定义为命令行参数。其中后两个配置是不同的，spring.config.location会覆盖默认的搜索路径，spring.config.additional-location不会覆盖默认的搜索路径
### 2.4 `application-{profile}.properties`配置文件
我们可以在applicaiton.prperties所在目录定义applicaiton-{profile}.properties配置文件作为某个profile的专属配置文件，只有在该profile处于active状态时才会读取。
如果在application.properties和application-{profile}.properties中定义的相同名称的配置内容，后者会覆盖前者。
### 2.5 属性占位符
我们可以在属性配置时使用占位符，动态的使用其他属性的值：
```properties
name=weiyihaoge
desc=${name} is a good man.
```
### 2.6 使用YMAL文件替换properties
YMAL的依赖包SnakeYAML会被Spring-boot-starter自动加载。
无论是YMAL还是properties，只要被加载到内存，其实都会设置到environment之中，这时我们使用@Value("${propertyKey}")就能获取到属性的值，该注解其实是在从environment中获取值。
YMAL配置文件除了配置格式不同于properties之外，配置方式基本相同。下面主要看看几个不同之处：
#### 2.6.1 Multi-profile YMAL
使用properties配置文件时，不同的profile需要定义不同的配置文件，但是使用YMAL配置文件时，我们可以在一个YMAL文件中定义所有的profile配置。
```yaml
server:
  port: 8080
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
#### 2.6.2 @PropertySource
YMAL配置内容无法通过@PropertySource注解加载，如果要使用该注解加载配置内容，只能使用properties配置文件。
@PropertySource注解一般是用于加载自定义的属性配置文件的，因为如果是默认的配置文件application.properties或者application.yml都会被自动加载，根本用不到这个注解，也只有自定义的配置文件需要这个注解单独进行加载，而该注解只能用于properties配置文件，那么我们就有一个原则：不要自定义YMAL文件，凡是自定义的配置文件全部使用properties文件，而默认的配置完全可以采用application.yml，使用YMAL的优势。
### 2.7 类型安全的配置属性
所谓类型安全的配置属性即我们可以将自定义的配置内容直接对应到一个配置类中，在应用启动后生成一个配置Bean供程序使用。
这一般在配置属性比较多的情况下使用，因为这种情况下使用@Value有些过于麻烦。
使用方法：
#### 2.7.1 第一步：添加自定义配置数据
可以在默认的配置文件application.yml中添加，也可以在自定义的配置文件中添加（如果自定义配置文件，一定要定义成properties文件）
- 在application.yml中添加配置内容
```yaml
#属性映射测试
app:
  name: springdemo
  size: 100M
  user: weiyihaoge
  version: 0.0.1
```
- 在myConfig.properties中添加配置内容
```properties
app.name=springdemo2
app.size=50M
app.user=ahaha
app.version=1.0.0
```
#### 2.7.2 定义承接属性的Bean类
- 针对application.yml中定义的属性
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ConfigurationProperties("app")
@Configuration
public class AppProperty {
    private String name;
    private String size;
    private String user;
    private String version;
}
```
- 针对自定义myConfig.properties中定义的属性
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ConfigurationProperties("app")
@Configuration
@PropertySource("classpath:/config/myConfig.properties")
public class AppProperty {
    private String name;
    private String size;
    private String user;
    private String version;
}
```
#### 2.7.3 使用
```java
@Controller
@RequestMapping("base")
@Log4j2
@Api(hidden = true)
public class Base {
    @Autowired
    private AppProperty property;
    
    @RequestMapping(value = "/getProperties",method = RequestMethod.GET)
    @ResponseBody
    @ApiOperation(value = "获取配置属性", httpMethod = "GET")
    public String getProperty(){
        return property.toString();
    }
}
```
#### 2.7.4 执行结果
浏览器执行以下请求：
```txt
http:127.0.0.1:8080/base/getProperties
```
- 默认配置文件的结果
```txt
AppProperty(name=springdemo, size=100M, user=weiyihaoge, version=0.0.1)
```
- 自定义配置文件的结果
```txt
AppProperty(name=springdemo2, size=50M, user=ahaha, version=1.0.0)
```
> 注意：如果在默认的配置文件和自定义配置文件中配置了同样的内容，那么自定义的内容将不会被映射，默认的配置文件中配置的信息会优先被映射。