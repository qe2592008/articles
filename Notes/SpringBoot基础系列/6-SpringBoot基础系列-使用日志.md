# SpringBoot基础系列-使用日志
## 概述
SpringBoot使用Common Logging进行日志操作，Common Logging是一个日志功能框架，没有具体的实现，具体的日志操作需要具体的日志框架来实现。
常用的日志框架包括：JUL(Java Util Logging)、Log4J2、Logback。
默认情况下，使用的是Logback作为底层实现。
## 日志格式
SpringBoot的默认的日志格式如下：
```txt
2018-11-21 10:23:34.966  INFO 12588 --- [  restartedMain] c.e.s.SpringbootdemoApplication          : Starting SpringbootdemoApplication on PC-20170621WOWM with PID 12588 (F:\Code\etongdai\etongdai-reactor\springbootdemo\target\classes started by Administrator in F:\Code\etongdai\etongdai-reactor\springbootdemo)
2018-11-21 10:23:34.968  INFO 12588 --- [  restartedMain] c.e.s.SpringbootdemoApplication          : No active profile set, falling back to default profiles: default
2018-11-21 10:23:34.968 DEBUG 12588 --- [  restartedMain] o.s.boot.SpringApplication               : Loading source class com.example.springbootdemo.SpringbootdemoApplication
```
格式为：(date) (time) (log level) (process Id) --- ([thread name]) (logger name) : (log message)
### 日志级别
- ERROR(FATAL也属此类)
- WARN
- INFO
- DEBUG
- TRACE
## 日志输出
### 控制台输出
默认情况下，SpringBoot的日志就是输出控制台，而且默认是INFO级别，也就是ERROR、WARN、INFO这三个级别的日志会被输出。
#### 设置日志级别
##### 命令行参数
```youtrack
java -jar xxx.jar --debug
```
##### application.properties
```properties
debug=true
```
#### 彩色输出（无甚用处）
### 文件输出
#### 设置日志输出文件
application.properties
```properties
logging.file=xxx.log
logging.path=/log/
```
前者用于指定输出日志的文件，后者用于指定日志输出文件的位置，其名称为默认的spring.log。
#### 设置日志文件大小
默认情况下当日志文件达到10M大小的时候就会轮转（重新开始），旧的日志内容默认会自动存档，而且自动存档默认是无限期的,可以使用如下配置：
```properties
#设置日志文件的最大尺寸，大于该尺寸，日志开始轮转
logging.file.max-size=20MB
#设置存档日志文件的最大容量
logging.file.max-history=100
```
### 日志级别
SprngBoot中集成了多个模块，我们可以对其分别进行日志级别设置：
```properties
#设置root级日志级别
logging.level.root=WARN
#设置spring web框架的日志级别
logging.level.org.springframework.web=DEBUG
#设置spring中集成的hibernate的日志级别
logging.level.org.hibernate=ERROR
```
## 日志组
为避免针对各个系统进行日志设置，提供了日志组，将相同日志级别的系统模块设置成一组，统一设置一致的日志级别，SpringBoot提供了默认的日志组，我们也能自定义日志组：
### 自定义日志组
```properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```
通过如下配置统一设置日志级别：
```properties
logging.level.tomcat=TRACE
```
### SpringBoot内置日志组
|序号|Name|Loggers|
|1|web|org.springframework.core.codec, org.springframework.http, org.springframework.web|
|2|sql|org.springframework.jdbc.core, org.hibernate.SQL|
通过Name值即可统一设置其中包含的Loggers的日志级别
```properties
logging.level.web=DEBUG
```
## 定制Log配置
SpringBoot底层支持多种日志实现，可以通过添加某种日志系统的jar包的方式来使其自动激活可用（SpringBoot的自动配置功能的作用），然后可以通过在classpath根路径下或者是logging.config配置属性(在application.properties中配置)指定的目录下自定义日志配置文件来进行深度定制。
针对不同的日志底层实现，需要自定义不同名称的日志配置文件
|序号|Logging System|fileName|
|1|Logback|logback-spring.xml, logback-spring.groovy, logback.xml, or logback.groovy|
|2|Log4j2|log4j2-spring.xml or log4j2.xml|
|3|JDK (Java Util Logging)|logging.properties|
推荐使用*-spring.xml格式命名的配置文件作为自定义日志配置文件名
## 扩展Logback
Spring Boot包含了许多可以帮助进行高级配置的Logback扩展。可以在logback-spring.xml配置文件中使用这些扩展。
### 基于profile的日志配置
可以配置在某个profile处于激活时使用的日志配置
```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>
<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>
<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```
### Environment属性
通过<springProperty>标签可以在日志配置文件中使用来自application.properties中配置的属性，因为application.properties中配置的属性会被加载到Environment中，所以也就是获取环境中的属性了。
```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
		defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
	<remoteHost>${fluentHost}</remoteHost>
	...
</appender>
```
相关：[Spring Boot 日志配置(超详细)](https://www.jianshu.com/p/f67c721eea1b)
