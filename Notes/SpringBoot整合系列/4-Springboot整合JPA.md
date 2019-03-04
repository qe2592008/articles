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
@Transactional
public interface UserRepository extends JpaRepository<User, Serializable> {
    //...
    User getUserByUseIdNo(String useIdNo);
    //...
}
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
@Transactional
public interface UserRepository extends JpaRepository<User, Serializable> {
    //...
    @Modifying
    @Query(value = "update USER set USE_PHONE_NUM = :num WHERE USE_ID= :useId", nativeQuery = true)
    void updateUsePhoneNum(@Param(value = "num") String num, @Param(value = "useId") int useId);
    //...
}
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
##### 用法解析
首先看Specification接口，在其中只有一个抽象方法，所以它就是一个函数式接口，另外还定义了两个静态方法，两个默认方法：我们来看看源码：

 Hibenate的Criteria查询解析

- EntityManager：实体管理器，这是持久化的开端，通过它的getCriteriaBuilder方法可以得到条件构造器CriteriaBuilder，而其本身的实例是被自动创建到IOC容器的，可以直接注入使用。
- CriteriaBuilder：条件构造器，用于构造查询条件，可以拥有多个，通过它的createQuery方法可以获得一个CriteriaQuery实例
- CriteriaQuery：高级查询功能，涉及：
  - select：用于指定返回对象，代表SQL中的select子句
  - multiselect：用于指定具体要返回的多个字段，代表SQL中的select子句
  - where：用于聚合所有的查询条件谓语，代表SQL中的where子句
  - groupBy：用于分组，代表SQL中的group By子句
  - having：代表SQL中的having子句
  - orderBy：代表SQL中的order by子句
  - distinct：代表distinct关键字（去重）

   可以通过它的from方法得到Root根对象
- Root：代表的是From子句，这里一般代表查询的根对象

```java
// EntityManager是一切的开始，直接注入即可使用
@Repository
public class UserRepository {
    @Autowired
    private EntityManager entityManager;
    public ResponseEntity<Page<User>> getUserList(){
         // CriteriaBuilder是条件构造器
         CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
         // 创建查询用户分页数据的查询实例
         CriteriaQuery<User> criteriaQuery = criteriaBuilder.createQuery(User.class);
         // 创建查询用户数据总量的查询实例
         CriteriaQuery<Tuple> criteriaCountQuery = criteriaBuilder.createQuery(Tuple.class);
         // 设置两个查询的来源，即from哪个模型对应的表
         Root<User> root = criteriaQuery.from(User.class);
         Root<User> countRoot = criteriaCountQuery.from(User.class);
         // 使用CriteriaBuilder来构造查询条件p1和p2
         Predicate p1 = criteriaBuilder.equal(root.get("useState"), UseSex.MAN);
         Predicate p2 = criteriaBuilder.or(
             criteriaBuilder.equal(root.get("useState"),UseState.COMMON),
             criteriaBuilder.equal(root.get("useState"),UseState.CANCLE));
         // 将查询条件连同其他一些需求条件一起封装到CriteriaQuery中
         criteriaQuery
             .where(p1,p2)// 组合条件
             .distinct(true)// 去重
             .select(root)// 设置查询对象
             .orderBy(criteriaBuilder.asc(root.get("createTime")));// 排序
         criteriaCountQuery
                     .where(p1,p2)// 组合条件
                     .distinct(true)// 去重
                     .multiselect(criteriaBuilder
                             .count(countRoot.get("useId"))// 设置查询内容，这里为count
                             .alias("total"));// 为count设置别名，用于下面获取
         // 创建最终的查询对象Query，它和CriteriaQuery不同之处在于，
         // 前者是最终执行查询操作并处理结果的查询对象，后者是用来组装的查询功能的查询对象，
         // 简单的说，这个createQuery的作用就是将之前组装好的查询对象，
         // 编译成为正在的SQL字符串封装到Query中
         Query query = entityManager.createQuery(criteriaQuery);
         Query countQuery = entityManager.createQuery(criteriaCountQuery);
         query.setMaxResults(pageSize);
         query.setFirstResult(pageId * pageSize);
         // 执行查询操作，并返回结果
         List<User> users = query.getResultList();
         List<Tuple> tuples = countQuery.getResultList();
         Tuple tu = tuples.get(0);
         Long count = (Long)tu.get("total");
         Page<User> userPage = new PageImpl<User>(users, pageable,count);
         return ResponseEntity.ok(userPage);
    }
}
```
单独使用Hibernate就是这样，但是我们使用Spring Data JPA的时候，不必如此，因为在JPA中的Specification为我们作了封装，也就是准备工作，将CriteriaBuilder，CriteriaQuery，Root这三者都准备好了，可以直接使用，而且对最后的执行查询的操作也做了封装，而我们的工作就剩下组装查询实例CriteriaQuery了。
而且Specification接口是一个函数式接口，我们可以采用Lambda来编程，非常简单，就上面的代码可以简化为：
```java
@Repository
public class UserRepository {
    public ResponseEntity<Page<User>> getUserList(final int pageId, final int pageSize){
        Pageable pageable = new PageRequest(pageId, pageSize);
        return ResponseEntity.ok(usersRepository.findAll((root, query, cb) -> {
            Predicate p1 = cb.equal(root.get("useSex"), UseSex.MAN);
            Predicate p2 = cb.or(
                cb.equal(root.get("useState"),UseState.COMMON),
                cb.equal(root.get("useState"),UseState.FREEZE));
            query.where(p1, p2).orderBy(cb.asc(root.get("createTime")));
            return null;
        }, pageable));
    }
}
```
如何，是不是大大的简化的代码量呢？

我们有必要讲解下Specification的原理
##### Specification原理
Specification是一个函数式接口，源码如下：
```java
public interface Specification<T> extends Serializable {
    long serialVersionUID = 1L;
    // 这个是非的意思
    static <T> Specification<T> not(Specification<T> spec) {
        return Specifications.negated(spec);
    }
    // 这个是
    static <T> Specification<T> where(Specification<T> spec) {
        return Specifications.where(spec);
    }
    // 这个是与的意思
    default Specification<T> and(Specification<T> other) {
        return Specifications.composed(this, other, AND);
    }
    // 这个是或的意思
    default Specification<T> or(Specification<T> other) {
        return Specifications.composed(this, other, OR);
    }
    // 创建where子句 重点，上面的Lambda表达式用的就是这个方法
     // Root是From子句的根类型
     // CriteriaQuery封装高级查询功能，包括：select(指定返回的字段)
    @Nullable
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
}
```
既然Specification是函数式接口，那么其唯一的抽象方法就是开放的入口，在这个入口的前后，都已经被封装好，我们只需要通过这个入口提供该方法的实现即可，这个方法中需要的是条件组装、查询对象组装等内容。
可以简单的看看它在哪里被调用：
```java
// JPA中提供的公共方法，用于将Specification解封为Criteria
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    //...
    private <S, U extends T> Root<U> applySpecificationToCriteria(
        @Nullable Specification<U> spec, Class<U> domainClass,
        CriteriaQuery<S> query) {
        Assert.notNull(domainClass, "Domain class must not be null!");
        Assert.notNull(query, "CriteriaQuery must not be null!");
        Root<U> root = query.from(domainClass);
        if (spec == null) {
         return root;
        }
        CriteriaBuilder builder = em.getCriteriaBuilder();
        Predicate predicate = spec.toPredicate(root, query, builder);
        if (predicate != null) {
         query.where(predicate);
        }
        return root;
    }
    //...
}
```
上面的公共方法被如下两个方法调用，第一个是getQuery方法，主要用于获取查询记录
```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    //...
    protected <S extends T> TypedQuery<S> getQuery(@Nullable Specification<S> spec, Class<S> domainClass, Sort sort) {
         CriteriaBuilder builder = em.getCriteriaBuilder();
         CriteriaQuery<S> query = builder.createQuery(domainClass);
         Root<S> root = applySpecificationToCriteria(spec, domainClass, query);
         query.select(root);
         if (sort.isSorted()) {
             query.orderBy(toOrders(sort, root, builder));
         }
         return applyRepositoryMethodMetadata(em.createQuery(query));
    }
    //...
}
```
第二个是getCountQuery方法，主要用户获取查询记录的个数
```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    //...
    protected <S extends T> TypedQuery<Long> getCountQuery(@Nullable Specification<S> spec, Class<S> domainClass) {
        CriteriaBuilder builder = em.getCriteriaBuilder();
        CriteriaQuery<Long> query = builder.createQuery(Long.class);
        Root<S> root = applySpecificationToCriteria(spec, domainClass, query);
        if (query.isDistinct()) {
            query.select(builder.countDistinct(root));
        } else {
            query.select(builder.count(root));
        }
        // Remove all Orders the Specifications might have applied
        query.orderBy(Collections.<Order> emptyList());
        return em.createQuery(query);
    }
    //...
}
```
上面的方法最后被如下所示的方法所调用，而如下的方法却是有JPA封装好的持久化接口中定义的持久化方法的具体实现。这个正是我们在具体的项目中调用的方法，使用这样的方法，就免去了像之前那种纯粹由Hibernate来实现复杂查询的大部分代码了，很简单，这里做了封装，只将Specification交给我们来自定义。
 ```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    @Override
    public <S extends T> long count(Example<S> example) {
        return executeCountQuery(getCountQuery(new ExampleSpecification<S>(example), example.getProbeType()));
    }
    @Override
    public <S extends T> Optional<S> findOne(Example<S> example) {
        try {
            return Optional.of(
                getQuery(new ExampleSpecification<S>(example), example.getProbeType(), Sort.unsorted()).getSingleResult());
        } catch (NoResultException e) {
            return Optional.empty();
        }
    }
}
 ```
##### Specification复杂应用
###### 多表联表查询
###### In查询
根据给定的用户ID串，来获取对应的用户列表
```java
@Repository
public class UserRepository {
    public ResponseEntity<List<User>> getUserList(final String ids){
        return ResponseEntity.ok(usersRepository.findAll((root, query, cb) -> {
            Predicate p = root.get("id").in(Arrays.asList(ids.split(",")));
            query.where(p1).orderBy(cb.asc(root.get("createTime")));
            return null;
        }));
    }
}
```
### Auditing
Auditing表示审计功能，主要目的是自动插入创建时间创建人员，修改时间和修改人员。

涉及注解：
- @CreatedDate：标识创建时间字段，insert操作时插入
- @CreatedBy：标识创建人员字段，insert操作时插入
- @LastModifiedDate：标识修改时间字段，update更自动插入
- @LastModifiedBy：标识修改人员字段，update更自动插入
- @EnableJpaAuditing：启动审计功能，标注于启动类或者配置类
- @EntityListeners(AuditingEntityListener.class)：标注于目标类上，表示
- @MappedSuperclass：标注于公共类上，表示这是超类
实现方式：
1. 创建一个公共超类来统一为所有的实体类定义创建和修改的时间与人员字段，实体类均应该继承该超类。
2. 在超类上添加注解@MappedSuperclass和@EntityListeners(AuditingEntityListener.class)。
3. 在超类中定义四个字段和各自的get和set方法，并添加对应的注解。
4. 在启动类上添加@EnableJpaAuditing注解用于启动审计功能
### 分页查询
    使用Pageable来实现分页，需要传递页码和页距两个参数，分页查询的方式在之前的复杂查询里面已经罗列过了，这里不再赘述。
### 排序
    使用Sort来实现查询结果排序，可以单独作为参数，也可以组合到Pageable之中。
整合源码：[样例代码](https://github.com/qe2592008/springboot-integration/tree/develop/src/main/java/com/dh/springbootintegration/jpa)

(暂时结束)