# SpringBoot整合hssqldb
## 步骤
### 第一步：添加必要的依赖
```xml
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
### 第二步：添加必要的配置（未验证）
```properties
#hssqldb配置
spring.jpa.show-sql = true #启用SQL语句的日志记录
spring.jpa.hibernate.ddl-auto = update  #设置ddl模式
##数据源设置
spring.datasource.url = jdbc:hsqldb:file:D:\\test\\testdb2
spring.datasource.username = sa  #配置数据库用户名
spring.datasource.password = sa  #配置数据库密码
spring.datasource.driverClassName =org.hssqldb.jdbcDriver  #配置JDBC Driver
##数据初始化设置
spring.datasource.schema=classpath:db/schema.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
spring.datasource.data=classpath:db/data.sql 
```