# SpringBoot整合JPA进行数据库开发

## 步骤

### 第一步：添加必要的jar包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>${querydsl.version}</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>com.querydsl</groupId>
  <artifactId>querydsl-jpa</artifactId>
  <version>${querydsl.version}</version>
</dependency>
```
### 第二步：添加必要的配置
```properties
spring.datasource.url = jdbc\:h2\:file\:D\:\\testdb;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username = sa
spring.datasource.password = sa
spring.datasource.driverClassName =org.h2.Driver
```
### 第三步：添加实体，并添加注解
```java
@Entity  
@Table  
public class User {  
     @Id 
     @GeneratedValue(strategy = GenerationType.IDENTITY) 
     private int useId;
     @Column 
     private String useName; 
     @Column 
     private UseSex useSex; 
     @Column 
     private int useAge; 
     @Column 
     private String useIdNo; 
     @Column 
     private String usePhoneNum; 
     @Column 
     private String useEmail; 
     @Column 
     private LocalDateTime createTime; 
     @Column 
     private LocalDateTime modifyTime;
     @Column 
     private UseState useState;
 }
```
### 第四步：添加持久层
```java
@Repository  
public interface UserRepository extends JpaRepository {  
}
```
> 注意：
>  继承自JpaRepository的持久层可以直接使用其定义好的CRUD操作，其实只有增删查操作，关于修改的操作还是需要自定义的。
### 第五步：持久层的使用
```java
@Service  
public class UserService {  
     @Autowired 
     private UserRepository repository; 
     public ResponseEntity addUser(final User user) { 
         return new ResponseEntity<>(repository.save(user),HttpStatus.OK); 
     }
}
```
> 注意：其实在JpaRepository中已经定义了许多方法用于执行实体的增删查操作。
## JPA高级功能
### 方法名匹配
在UserRepository中定义按照规则命名的方法，JPA可以将其自动转换为SQL，而免去手动编写的烦恼，比如定义如下方法：
```java
User getUserByUseIdNo(String useIdNo);
```
JPA会自动将其转换为如下的SQL：
```sql
select * from USER where use_id_no = ?
```
下面简单罗列方法命名规则：
| 关键字 | 例子 | sql |
| --- | --- | --- |
| And | findByNameAndAge | ...where x.name=?1 and x.age=?2 |
| Or | findByNameOrAge | ...where x.name=?1 or x.age=?2 |
| Between | findByCreateTimeBetween | ...where x.create_time between ?1 and ?2 |
| LessThan | findByAgeLessThan | ...where x.age < ?1 |
| GreaterThan | findByAgeGreaterThan | ...where x.age > ?1 |
| IsNull | findByAgeIsNull | ...where x.age is null |
| IsNotNull,NotNull | findByAgeIsNotNull | ...where x.age not null  |
| Like | findByNameLike | ...where x.name like ?1 |
| NotLike | findByNameNotLike | ...where x.name not like ?1 |
| OrderBy | findByAgeOrderByNameDesc | ...where x.age =?1 order by x.name desc |
| Not | findByNameNot | ...where x.name <>?1 |
| In | findByAgeIn | ...where x.age in ?1 |
| NotIn | findByAgeNotIn | ...where x.age not in ?1 |
### @Query注解
    使用@Query注解在接口方法之上自定义执行SQL。
```java
@Modifying
@Query(value = "update USER set USE_PHONE_NUM = :num WHERE USE_ID= :useId", nativeQuery = true)
void updateUsePhoneNum(@Param(value = "num") String num, @Param(value = "useId") int useId);
```
    上面的更新语句必须加上@Modifying注解，其实在JpaRepository中并备有定义任何更新的方法，所有的更新操作均需要我们自己来定义，一般采用上面的方式来完成。
```java
/**
 * 表示一个查询方法是修改查询，这会改变执行的方式。只在@Query注解标注的方法或者派生的方法上添加这个注解，而不能
 * 用于默认实现的方法，因为这个注解会修改执行方式，而默认的方法已经绑定了底层的APi。
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })
@Documented
public @interface Modifying {
	boolean flushAutomatically() default false;
	boolean clearAutomatically() default false;
}
```
### JPQL(SQL拼接)
    使用JPQL需要在持久层接口的实现列中完成，即UserRepositoryImpl，这个类是UserRepository的实现类，我们在其中定义JPQL来完成复杂的SQL查询，包括动态查询，连表查询等高级功能。
### QBE(QueryByExampleExecutor)
    使用QBE来进行持久层开发，需要用到两个接口类，Example和ExampleMatcher，开发方式如下：
```java
List users = repository.findAll(Example.of(user));
```
    或者配合ExampleMarcher使用：
```java
ExampleMatcher matcher = ExampleMatcher.matching().withIgnoreCase();
List users = repository.findAll(Example.of(user, matcher));
```
    以上逻辑一般位于service之中。其中user模型中保存着查询的条件值，null的字段不是条件，只有设置了值的字段才是条件。ExampleMatcher是用来自定义字段匹配模式的。
### 复杂查询
    在Spring Data JPA中我们可以使用Specification来构建复杂多变的动态查询条件，甚至联表查询，几乎可以以编码方式实现sql条件编写。
    
    
### 分页查询
    使用Pageable来实现分页，需要传递页码和页距两个参数
### 排序
    使用Sort来实现查询结果排序，可以单独作为参数，也可以组合到Pageable之中。
(暂时结束)