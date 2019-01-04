# SpringBoot基础系列-缓存体系
## 步骤
### 开启缓存
使用@EnableCaching注解开启缓存功能，该注解需要配置到@Configuration配置类上。
```java
@Configuration
@EnableCaching
public class CacheConfig {
}
```
> cache提示是Springboot内置功能，不需要进行其他配置
### 使用缓存
```java
@Service
@Log4j2
public class AnimalService {
    //查
    @Cacheable("animalById")
    public ResponseEntity<Animal> getAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.selectById(id));
    }
}
```
> 解析：@Cacheable注解标注于方法之上，其参数animalById为自定义的缓存名称，执行模式为：当我们企图调用这个方法查询动物的时候，会首先到缓存名称为animalById的缓存中寻找有没有缓存的结果，如果有直接获取返回，不再调用方法，如果没有，则调用方法，最后将结果缓存到animalById缓存中，并将结果返回。
## 简单原理
SpringBoot对缓存系统进行了集中抽象，具体的缓存执行还需要依靠具体的缓存系统
如果没有指定使用哪种缓存系统的话，SpringBoot会按照下面的排序依次查找使用：
- Generic
- JCache
- EhCache 2.x
- Hazelcast
- Couchbase
- Redis
- Caffeine
- Simple
### JCache
    JCache就是JSR-107，它也是一种缓存抽象体系，其实现有多种，Ehcache 3，Hazelcast和Infinispan等如果存在多个实现者，那么就需要手动明确指定了
```yaml
spring:
  cache:
    jcache:
      provider:  #明确指定使用的缓存提供者
      config: #指定缓存系统使用配置文件路径
```
### EhCache 2.x
    如果可以从类路径下找到名为ehcache.xml的配置文件，EhCacheCacheManager可以由自动配置完成获取。
### Infinispan
    只能手动指定配置文件位置：
```yaml
spring:
  cache:
    infinispan:
      config: #配置路径
```
### Redis
    如果Redis可用并已配置，RedisCacheManager可以被自动配置提供。
### Simple
    当我们没有指定具体的缓存系统的情况下，默认使用ConcurrentHashMap作为缓存。
### None
    当我们想要禁用缓存功能时，采用如下配置
```yaml
spring:
  cache:
    type: none #禁用缓存功能
```