# SpringBoot整合SpringValidation
## 步骤
### 第一步：添加必要的依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
当我们仅仅使用Spring Validation时可以添加以上依赖来实现，其实Spring的Validation模块位于Spring-context包，属于核心包，在web依赖添加的时候就会一并添加进来其依赖的包也会添加进来，所以如果要开发Springboot项目，并不用单独添加该依赖。
### 第二步：使用SpringValidation
SpringValidation可以算是Spring核心功能，被Spring所包容，无需任何配置即可开始使用，具体使用参考文章[Spring基础系列-参数校验](https://www.cnblogs.com/V1haoge/p/9953744.html)
(结束)