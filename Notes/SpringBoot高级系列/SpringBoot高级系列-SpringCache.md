# SpringBoot高级系列-SpringCache使用
## 概述
SpringCache本身是一个缓存体系的抽象实现，并没有具体的缓存能力，要使用SpringCache还需要配合具体的缓存实现来完成。
虽然如此，但是SpringCache是所有Spring支持的缓存结构的基础，而且所有的缓存的使用最后都要归结于SpringCache，那么一来，要想使用SpringCache，还是要仔细研究一下的。
## 缓存注解
SpringCache缓存功能的实现是依靠下面的这几个注解完成的。
- @EnableCaching：开启缓存功能
- @Cacheable：定义缓存，用于触发缓存
- @CachePut：定义更新缓存，触发缓存更新
- @CacheEvict：定义清除缓存，触发缓存清除
- @Caching：组合定义多种缓存功能
- @CacheConfig：定义公共设置，位于class之上
### @EnableCaching
该注解主要用于开启基于注解的缓存功能,使用方式为：
```java
@EnableCaching
@Configuration
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(new ConcurrentMapCache("default")));
        return cacheManager;
    }
}
```
> 注意：在SpringBoot中使用SpringCache可以由自动配置功能来完成CacheManager的注册，SpringBoot会自动发现项目中拥有的缓存系统，而注册对应的缓存管理器，当然我们也可以手动指定。

使用该注解和如下XML配置具有一样的效果：
```xml
<beans>
    <cache:annotation-driven/>
    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager>
        <property name="caches">
            <set>
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean>
                    <property name="name" value="default"/>
                </bean>
            </set>
        </property>
    </bean>
</beans>
```
下面来看看@EnableCaching的源码：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(CachingConfigurationSelector.class)
public @interface EnableCaching {
    // 用于设置使用哪种代理方式，默认为基于接口的JDK动态代理（false），
    // 设置为true，则使用基于继承的CGLIB动态代理
	boolean proxyTargetClass() default false;
	// 用于设置切面织入方式(设置面向切面编程的实现方式)，
	// 默认为使用动态代理的方式织入，当然也可以设置为ASPECTJ的方式来实现AOP
	AdviceMode mode() default AdviceMode.PROXY;
	// 用于设置在一个切点存在多个通知的时候各个通知的执行顺序，默认为最低优先级，
	// 其中数字却大优先级越低，这里默认为最低优先级，int LOWEST_PRECEDENCE =
	// Integer.MAX_VALUE;，却是整数的最大值
	int order() default Ordered.LOWEST_PRECEDENCE;
}
public enum AdviceMode {
	PROXY,
	ASPECTJ
}
public interface Ordered {
	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;
	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
	int getOrder();
}
```
由上面的源码可以看出，缓存功能是依靠AOP来实现的。
### @Cacheable
该注解用于标注于方法之上用于标识该方法的返回结果需要被缓存起来，标注于类之上标识该类中所有方法均需要将结果缓存起来。
该注解标注的方法每次被调用前都会触发缓存校验，校验指定参数的缓存是否已存在（已发生过相同参数的调用），若存在，直接返回缓存结果，否则执行方法内容，最后将方法执行结果保存到缓存中。
#### 使用
```java
@Service
@Log4j2
public class AnimalService {
    @Autowired
    private AnimalRepository animalRepository;
    //...
//    @Cacheable("animalById")
    @Cacheable(value = "animalById", key = "#id")
    public ResponseEntity<Animal> getAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.selectById(id));
    }
    //...
}
```
上面的实例中两个@Cacheable配置效果其实是一样的，其中value指定的缓存的名称，它和另一个方法cacheName效果一样，一般来说这个缓存名称必须要有，因为这个是区别于其他方法的缓存的唯一方法。
这里我们介绍一下缓存的简单结构，在缓存中，每个这样的缓存名称的名下都会存在着多个缓存条目，这些缓存条目对应在使用不同的参数调用当前方法时生成的缓存，所有一个缓存名称并不是一个缓存，而是一系列缓存。
另一个key用于指定当前方法的缓存保存时的键的组合方式，默认的情况下使用所有的参数组合而成，这样可以有效区分不同参数的缓存。当然我们也可以手动指定，指定的方法是使用SPEL表达式。
这里我么来简单看看其源码，了解下其他几个方法的作用：
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
    // 用于指定缓存名称，与cacheNames()方法效果一致
	@AliasFor("cacheNames")
	String[] value() default {};
	// 用于指定缓存名称，与value()方法效果一致
	@AliasFor("value")
	String[] cacheNames() default {};
	// 用于使用SPEL手动指定缓存键的组合方式，默认情况使用所有的参数来组合成键，除非自定义了keyGenerator。
	// 使用SPEL表达式可以根据上下文环境来获取到指定的数据：
	// #root.method：用于获取当前方法的Method实例
	// #root.target：用于获取当前方法的target实例
	// #root.caches：用于获取当前方法关联的缓存
	// #root.methodName：用于获取当前方法的名称
	// #root.targetClass：用于获取目标类类型
	// #root.args[1]：获取当前方法的第二个参数，等同于：#p1和#a1和#argumentName
	String key() default "";
	// 自定义键生成器，定义了该方法之后，上面的key方法自动失效，这个键生成器是：
	// org.springframework.cache.interceptor.KeyGenerator，这是一个函数式接口，
	// 只有一个generate方法，我们可以通过自定义的逻辑来实现自定义的key生成策略。
	String keyGenerator() default "";
	// 用于设置自定义的cacheManager(缓存管理器),可以自动生成一个cacheResolver
	// （缓存解析器），这一下面的cacheResolver()方法设置互斥
	String cacheManager() default "";
	// 用于设置一个自定义的缓存解析器
	String cacheResolver() default "";
	// 用于设置执行缓存的条件，如果条件不满足，方法返回的结果就不会被缓存，默认无条件全部缓存。
	// 同样使用SPEL来定义条件，可以使用的获取方式同key方法。
	String condition() default "";
	// 这个用于禁止缓存功能，如果设置的条件满足，就不执行缓存结果，与上面的condition不同之处在于，
	// 该方法执行在当前方法调用结束，结果出来之后，因此，它除了可以使用上面condition所能使用的SPEL
	// 表达式之外，还可以使用#result来获取方法的执行结果，亦即可以根据结果的不同来决定是否缓存。
	String unless() default "";
	// 设置是否对多个针对同一key执行缓存加载的操作的线程进行同步，默认不同步。这个功能需要明确确定所
	// 使用的缓存工具支持该功能，否则不要滥用。
	boolean sync() default false;
}
```
如何自定义一个KeyGenerator呢？
```java
public class AnimalKeyGenerator implements KeyGenerator {
    @Override
    public Object generate(Object target, Method method, Object... params) {
        StringBuilder sb = new StringBuilder("animal-");
        sb.append(target.getClass().getSimpleName()).append("-").append(method.getName()).append("-");
        for (Object o : params) {
            String s = o.toString();
            sb.append(s).append("-");
        }
        return sb.deleteCharAt(sb.lastIndexOf("-")).toString();
    }
}
```
### @CachePut
该注解用于更新缓存，无论结果是否已经缓存，都会在方法执行结束插入缓存，相当于更新缓存。一般用于更新方法之上。
```java
@Service
@Log4j2
public class AnimalService {
    @Autowired
    private AnimalRepository animalRepository;
    //...
    @CachePut(value = "animalById", key = "#animal.id")
    public ResponseEntity<Animal> updateAnimal(final Animal animal){
        Wrapper<Animal> animalWrapper = new UpdateWrapper<>();
        ((UpdateWrapper<Animal>) animalWrapper).eq("id",animal.getId());
        animalRepository.update(animal, animalWrapper);
        return ResponseEntity.ok(this.getAnimalById(animal.getId()));
    }
    //...
}
```
这里指定更新缓存，value同样还是缓存名称，这里更新的是上面查询操作的同一缓存，而且key设置为id也与上面的key设置对应。
如此设置之后，每次执行update方法时都会直接执行方法内容，然后将返回的结果保存到缓存中，如果存在相同的key,直接替换缓存内容执行缓存更新。
下面来看看源码：
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CachePut {
    // 同上
	@AliasFor("cacheNames")
	String[] value() default {};
	// 同上
	@AliasFor("value")
	String[] cacheNames() default {};
	// 同上
	String key() default "";
	// 同上
	String keyGenerator() default "";
	// 同上
	String cacheManager() default "";
	// 同上
	String cacheResolver() default "";
	// 同上
	String condition() default "";
	// 同上
	String unless() default "";
}
```
只有一点要注意：这里的设置一定要和执行缓存保存的方法的@Cacheable的设置一致，否则无法准确更新。
### @CacheEvict
该注解主要用于删除缓存操作。
```java
@Service
@Log4j2
public class AnimalService {
    @Autowired
    private AnimalRepository animalRepository;
    //...
    @CacheEvict(value = "animalById", key = "#id")
    public ResponseEntity<Integer> deleteAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.deleteById(id));
    }
    //...
}
```
简单明了。
看看源码：
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {
    // 同上
	@AliasFor("cacheNames")
	String[] value() default {};
	// 同上
	@AliasFor("value")
	String[] cacheNames() default {};
	// 同上
	String key() default "";
	// 同上
	String keyGenerator() default "";
	// 同上
	String cacheManager() default "";
	// 同上
	String cacheResolver() default "";
	// 同上
	String condition() default "";
	// 这个设置用于指定当前缓存名称名下的所有缓存是否全部删除，默认false。
	boolean allEntries() default false;
	// 这个用于指定删除缓存的操作是否在方法调用之前完成，默认为false，表示先调用方法，在执行缓存删除。
	boolean beforeInvocation() default false;
}
```
### @Caching
这个注解用于组个多个缓存操作，包括针对不用缓存名称的相同操作等。
源码：
```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {
    // 用于指定多个缓存设置操作
	Cacheable[] cacheable() default {};
    // 用于指定多个缓存更新操作
	CachePut[] put() default {};
	// 用于指定多个缓存失效操作
	CacheEvict[] evict() default {};
}
```
简单用法：
```java
@Service
@Log4j2
public class AnimalService {
    @Autowired
    private AnimalRepository animalRepository;
    //...
    @Caching(
        evict = {
            @CacheEvict(value = "animalById", key = "#id"),
            @CacheEvict(value = "animals", allEntries = true, beforeInvocation = true)
        }
    )
    public ResponseEntity<Integer> deleteAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.deleteById(id));
    }
    @Cacheable("animals")
    public ResponseEntity<Page<Animal>> getAnimalPage(final Animal animal, final int pageId, final int pageSize){
        Page<Animal> page = new Page<>();
        page.setCurrent(pageId);
        page.setSize(pageSize);
        return ResponseEntity.ok((Page<Animal>) animalRepository.selectPage(page,packWrapper(animal, WrapperType.QUERY)));
    }
    //...
}
```
### @CacheConfig
该注解标注于类之上，用于进行一些公共的缓存相关配置。
源码为：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CacheConfig {
    // 设置统一的缓存名，适用于整个类中的方法全部是针对同一缓存名操作的情况
	String[] cacheNames() default {};
	// 设置统一个键生成器，免去了每个缓存设置中单独设置
	String keyGenerator() default "";
	// 设置统一个自定义缓存管理器
	String cacheManager() default "";
	// 设置统一个自定义缓存解析器
	String cacheResolver() default "";
}
```