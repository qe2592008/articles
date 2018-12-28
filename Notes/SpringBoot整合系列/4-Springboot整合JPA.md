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
public interface UserRepository extends JpaRepository<User, Serializable> {  
}
```
> 注意：
>  继承自JpaRepository的持久层可以直接使用其定义好的CRUD操作，其实只有增删查操作，关于修改的操作还是需要自定义的。
>
> 继承了JpaRepository之后，就不再需要添加@Repository注解了
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

#### 第一步：添加继承接口

```java
// 继承JpaSpecificationExecutor之后可以使用复杂的查询
public interface UsersRepository extends JpaSpecificationExecutor<User>, JpaRepository<User, Serializable> {
}
```

#### 第二步：使用Specification

> 用法解析：
>
> ​	首先看Specification接口，在其中只有一个抽象方法，所以它就是一个函数式接口，另外还定义了两个静态方法，两个默认方法：我们来看看源码：
>
> > Hibenate的Criteria查询解析
> >
> > - EntityManager：实体管理器，这是持久化的开端，通过它的getCriteriaBuilder方法可以得到条件构造器CriteriaBuilder，而其本身的实例是被自动创建到IOC容器的，可以直接注入使用。
> >
> > - CriteriaBuilder：条件构造器，用于构造查询条件，可以拥有多个，通过它的createQuery方法可以获得一个CriteriaQuery实例
> >
> > - CriteriaQuery：高级查询功能，涉及：
> >
> >   - select：用于指定返回对象，代表SQL中的select子句
> >   - multiselect：用于指定具体要返回的多个字段，代表SQL中的select子句
> >   - where：用于聚合所有的查询条件谓语，代表SQL中的where子句
> >   - groupBy：用于分组，代表SQL中的group By子句
> >   - having：代表SQL中的having子句
> >   - orderBy：代表SQL中的order by子句
> >   - distinct：代表distinct关键字（去重）
> >
> >   可以通过它的from方法得到Root根对象
> >
> > - Root：代表的是From子句，这里一般代表查询的根对象
> >
> > ```java
> > // EntityManager是一切的开始，直接注入即可使用
> > @Autowired
> > private EntityManager entityManager;
> > 
> > public ResponseEntity<Page<User>> getUserList(){
> >     // CriteriaBuilder是条件构造器
> >     CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
> >     // 创建查询用户分页数据的查询实例
> >     CriteriaQuery<User> criteriaQuery = criteriaBuilder.createQuery(User.class);
> >     // 创建查询用户数据总量的查询实例
> >     CriteriaQuery<Tuple> criteriaCountQuery = criteriaBuilder.createQuery(Tuple.class);
> >     // 设置两个查询的来源，即from哪个模型对应的表
> >     Root<User> root = criteriaQuery.from(User.class);
> >     Root<User> countRoot = criteriaCountQuery.from(User.class);
> >     // 使用CriteriaBuilder来构造查询条件p1和p2
> >     Predicate p1 = criteriaBuilder.equal(root.get("useState"), UseSex.MAN);
> >     Predicate p2 = criteriaBuilder.or(
> >         criteriaBuilder.equal(root.get("useState"),UseState.COMMON),
> >         criteriaBuilder.equal(root.get("useState"),UseState.CANCLE));
> >     // 将查询条件连同其他一些需求条件一起封装到CriteriaQuery中
> >     criteriaQuery
> >         .where(p1,p2)// 组合条件
> >         .distinct(true)// 去重
> >         .select(root)// 设置查询对象
> >         .orderBy(criteriaBuilder.asc(root.get("createTime")));// 排序
> >     criteriaCountQuery
> >                 .where(p1,p2)// 组合条件
> >                 .distinct(true)// 去重
> >                 .multiselect(criteriaBuilder
> >                         .count(countRoot.get("useId"))// 设置查询内容，这里为count
> >                         .alias("total"));// 为count设置别名，用于下面获取
> >     // 创建最终的查询对象Query，它和CriteriaQuery不同之处在于，
> >     // 前者是最终执行查询操作并处理结果的查询对象，后者是用来组装的查询功能的查询对象，
> >     // 简单的说，这个createQuery的作用就是将之前组装好的查询对象，
> >     // 编译成为正在的SQL字符串封装到Query中
> >     Query query = entityManager.createQuery(criteriaQuery);
> >     Query countQuery = entityManager.createQuery(criteriaCountQuery);
> >     query.setMaxResults(pageSize);
> >     query.setFirstResult(pageId * pageSize);
> >     // 执行查询操作，并返回结果
> >     List<User> users = query.getResultList();
> >     List<Tuple> tuples = countQuery.getResultList();
> >     Tuple tu = tuples.get(0);
> >     Long count = (Long)tu.get("total");
> >     Page<User> userPage = new PageImpl<User>(users, pageable,count);
> >     return ResponseEntity.ok(userPage);
> > }
> > ```
>
> ​	单独使用Hibernate就是这样，但是我们使用Spring Data JPA的时候，不必如此，因为在JPA中的Specification为我们作了封装，也就是准备工作，将CriteriaBuilder，CriteriaQuery，Root这三者都准备好了，可以直接使用，而且对最后的执行查询的操作也做了封装，而我们的工作就剩下组装查询实例CriteriaQuery了。
>
> ​	而且Specification接口是一个函数式接口，我们可以采用Lambda来编程，非常简单，就上面的代码可以简化为：
>
> ```java
> public ResponseEntity<Page<User>> getUserList(final int pageId, final int pageSize){
>     Pageable pageable = new PageRequest(pageId, pageSize);
>     return ResponseEntity.ok(usersRepository.findAll((root, query, cb) -> {
>         Predicate p1 = cb.equal(root.get("useSex"), UseSex.MAN);
>         Predicate p2 = cb.or(
>             cb.equal(root.get("useState"),UseState.COMMON),
>             cb.equal(root.get("useState"),UseState.FREEZE));
>         query.where(p1, p2).orderBy(cb.asc(root.get("createTime")));
>         return null;
>     }, pageable));
> }
> ```
>
> ​	如何，是不是大大的简化的代码量呢？
>
> ​	我们有必要讲解下如何使用Specification
>
> > Specification
> >
> > 

### 分页查询
    使用Pageable来实现分页，需要传递页码和页距两个参数
### 排序
    使用Sort来实现查询结果排序，可以单独作为参数，也可以组合到Pageable之中。
(暂时结束)