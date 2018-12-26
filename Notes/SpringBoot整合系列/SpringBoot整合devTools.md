# SpringBoot整合devTools
## 步骤
### 第一步：添加必要的jar包
```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```
### 第二步：devTools的使用
#### 默认功能
```java
Map<String, Object> properties = new HashMap<>();
properties.put("spring.thymeleaf.cache", "false");// 禁用thymeleaf缓存
properties.put("spring.freemarker.cache", "false");// 禁用freemarker缓存
properties.put("spring.groovy.template.cache", "false");// 禁用groovy template缓存
properties.put("spring.mustache.cache", "false");// 禁用mustache缓存
properties.put("server.servlet.session.persistent", "true");// 开启session持久化功能
properties.put("spring.h2.console.enabled", "true");// 启用h2内存数据库控制台
properties.put("spring.resources.cache.period", "0");// 禁用Resource缓存
properties.put("spring.resources.chain.cache", "false");// 禁用Resouse链缓存功能
properties.put("spring.template.provider.cache", "false");// 禁用某个缓存
properties.put("spring.mvc.log-resolved-exception", "true");// 启用异常的警告日志
properties.put("server.error.include-stacktrace", "ALWAYS");// 启用栈跟踪功能
properties.put("server.servlet.jsp.init-parameters.development", "true");// 启用开发环境的jsp Servlet初始化参数
properties.put("spring.reactor.stacktrace-mode.enabled", "true");// 启用recator开发时栈跟踪功能
```
- 自动禁用缓存
    - 禁用thymeleaf缓存
    - 禁用freemarker缓存
    - 禁用groovy template缓存
    - 禁用mustache缓存
    - 禁用Resource缓存
    - 禁用Resouse链缓存
    - 禁用某个缓存
- 开启session持久化功能（应用重启前）
- 启用h2内存数据库控制台
- 启用异常的警告日志
- 启用栈跟踪功能
- 启用开发环境的jsp Servlet初始化参数
- 启用recator开发时栈跟踪功能
如何禁用默认功能呢？
```properties
#禁用devTools默认功能
spring.devtools.add-properties=false
#启用请求详情日志功能
spring.http.log-request-details=true
```
#### 自动重启
##### 重启报告日志
```properties
#禁用重启报告日志
spring.devtools.restart.log-condition-evaluation-delta=false
```
##### 设置不触发自动重启的目录
默认情况下，以下目录均属于不触发自动重启的目录：
/META-INF/maven
/META-INF/resources
/resources
/static
/public
/templates
自定义方式：
```properties
#设置不触发的目录，会覆盖默认目录设置
spring.devtools.restart.exclude=static/**,public/**
#设置除了默认的目录之外的不触发目录
spring.devtools.restart.additional-exclude=/huahua
```
##### 设置非类路径下文件变化触发重启
```properties
#设置触发重启的非classpath目录
spring.devtools.restart.additional-paths=/xxx
```
##### 禁用重启
```properties
#禁用重启功能
spring.devtools.restart.enabled=false
```
完全禁用的方式是需要在应用启动之前设置：
```java
// 应用启动的main方法在执行启动run方法之前设置禁用重启属性
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```
##### 触发器文件
为了避免任意修改都会导致重启的麻烦，使用触发器文件，只有该文件修改了，才会触发重启：
```properties
spring.devtools.restart.trigger-file=/triggerFile/triggerFile.txt
```
##### 定制重启类加载器
devtools的重启是由两个类加载器来实现的，一个是base ClassLoader，一个是restart ClassLoader，前者用于加载那些不会发生改变的类，后者用于加载那些开发者自定义的类。一般情况下，都没问题，但有时会发生类加载问题，这时需要我们定制restart ClassLoader。
这里所说的定制是指重新设置重启类加载需要加载的类。
方式：在META-INF下新建文件：spring-devtools.properties
```properties
#排除的文件
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
#包含的文件
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```
##### 一些限制
使用ObjectInputStream进行反序列化的对象无法正常使用自动重启功能，还包括一些第三方反序列化框架，
#### LiveReload
资源更新后自动刷新浏览器
```properties
#禁用自动刷新,默认开启
spring.devtools.livereload.enabled=false
```
#### 全局设置
作用于机器上所有使用了devtools的应用的设置:配置文件名称固定为：.spring-boot-devtools.properties
```properties
#配置公共触发器文件
spring.devtools.reload.trigger-file=.reloadtrigger
```
#### 远程应用
启用devtools的远程支持需要如下设置：
pom.xml
```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<excludeDevtools>false</excludeDevtools>
			</configuration>
		</plugin>
	</plugins>
</build>
```
然后设置：
```properties
spring.devtools.remote.secret=mysecret
```
如此设置之后，远程的接受连接的部件就已经开启了，本地的需要手动开启（以上设置均在远程项目上设置）
##### 运行远程应用
（略）