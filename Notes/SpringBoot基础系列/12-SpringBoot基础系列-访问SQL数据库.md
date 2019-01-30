# SpringBoot基础系列-访问SQL数据库
## 一、概述
SpringBoot提供了针对访问SQL数据库的广泛支持。可以直接使用针对JDBC进行简单封装的JdbcTemplate来直接访问数据库，也可以使用像Hibernate一样的ORM框架来访问数据库。Spring Data还可以支持直接创建按照制定规则定义的方法名来创建查询。
## 二、配置数据源
java中的`javax.sql.DataSource`接口定义了基本的数据库连接创建规则。
### 2.1 嵌入式数据库
嵌入式数据库就是内存数据库，这种数据库不会持久化保存数据，你需要在应用启东时创建数据，并在应用关闭时清除数据，数据都保存在内存中，这对开发来说是极为方便的。

SpringBoot可以自动配置针对内存数据库H2、HSQL、Derby等连接参数，只要你添加了对应的pom依赖，例如：
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<scope>runtime</scope>
</dependency>
```
`spring-boot-starter-data-jpa`依赖引入了spring-jdbc，来构建数据库连接，`hsqldb`引入了HSQL数据库的依赖。

如果你要对内存数据库自定义URL连接，那么一定要记得将内存数据库的自动关闭功能置为失效。如果是H2数据库，你需要设置`DB_CLOSE_ON_EXIT=FALSE`，如果是HSQL数据库，你需要设置`shutdown=false`。
### 2.2 生产数据库
生产数据库可以通过一个数据源池来获取数据库连接。SpringBoot采用如下规则创建数据源池：
- 只要HikariCP可用，优先使用HikariCP，它具有优秀的性能和并发性。
- 然后如果Tomcat数据源池可用，则使用它。
- 如果上面二者皆不可用，而DBCP2可用，则使用DBCP2。
> 使用`spring-boot-starter-jdbc`或者`spring-boot-starter-data-jpa`依赖的话，会自动添加HikariCP数据源池。

> 可用使用`spring.datasource.type`来手动指定要使用的数据源池类型。

数据源连接配置一般包括以下四点：
```properties
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
> `spring.datasource.url`必须指定，不然的话，SpringBoot会自动配置一个内存数据库。

> `spring.datasource.driver-class-name`通常无需设置，因为SpringBoot可以从`spring.datasource.url`中解析出来。

> 要保证驱动器类存在，即对应的数据库依赖包被正确的引入，否则无法创建连接。
### 2.3 JNDI数据源
JNDI数据源貌似很少使用，只有在使用JNDI来访问数据库时才使用：
```properties
spring.datasource.jndi-name=java:jboss/datasources/customers
```
## 三、访问数据库
### 3.1 使用JdbcTemplate
JdbcTemplate是JDBC的简单封装，简化的JDBC访问数据库的流程，出现ORM框架之后，甚少使用，一般只在极小型应用中使用。

使用时会将SQL脚本和代码耦合到一起，维护不易，不推荐使用。
### 3.2 使用JPA
使用JPA需要引入`spring-boot-starter-data-jpa`依赖。
#### 3.1 Entity
习惯上，Entity类需要在persistence.xml文件中定义，SpringBoot采用自动扫描来实现相同的功能，需要配合注解使用。

相关注解包括：@Entity、@Embeddable、@MappedSuperclass等，被其标注的类可以被注解@EnableAutoConfiguration或者@SpringBootApplication扫描到。
#### 3.2 Repositories
- 创建Repository类，注入EntityManager来手动创建SQL，填充参数、处理结果
- 创建Repository接口继承JpaRepository<T, ID>
    - 可以使用指定规则创建方法，以方法名称来创建脚本。
    - 复杂查询可以使用@Query注解来实现
    - 通过Example来编程实现简单的查询功能
- 创建Repository接口继承JpaSpecificationExecutor<T>
    - 可以使用Specification来编程实现复杂的查询功能
具体查看[SpringBoot整合JPA]()
### 3.3 使用JOOQ
具体参见[SpringBoot整合JOOQ]()
