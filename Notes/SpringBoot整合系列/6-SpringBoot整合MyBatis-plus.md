# SpringBoot整合MyBatis-plus
## 步骤
### 第一步：添加必要的依赖
第一种是在已存在MyBatis的情况下，直接添加mybatis-plus包即可。
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>2.1.8</version>
</dependency>
```
第二种是直接添加mybatis-plus的starter，它会自动导入mybatis的依赖包及其他相关依赖包
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.0.1</version>
</dependency>
```
### 第二步：添加必要的配置
> 注意：Mybatis-plus是MyBatis的再封装，添加MyBatis-plus之后我们的设置针对的应该是MyBatis-plus，而不是MyBatis。
```yaml
mybatis-plus:
  mapper-locations: classpath*:/mapper/*.xml
  type-aliases-package: com.example.springbootdemo.entity
  type-aliases-super-type: java.lang.Object
  type-handlers-package: com.example.springbootdemo.typeHandler
  type-enums-package: com.example.springbootdemo.enums
```
### 第三步：添加必要的配置类
```java
@EnableTransactionManagement
@Configuration
@MapperScan("com.example.springbootdemo.plusmapper")
public class MyBatisPlusConfig {
    
    // mybatis-plus分页插件
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
    
}
```
### 第四步：定义实体
```java
@Data
@Builder
@ToString
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
@TableName(value = "ANIMAL")
public class Animal {
    @TableId(value = "ID",type = IdType.AUTO)
    private Integer id;
    @TableField(value = "NAME",exist = true)
    private String name;
    @TableField(value = "TYPE",exist = true)
    private AnimalType type;
    @TableField(value = "SEX",exist = true)
    private AnimalSex sex;
    @TableField(value = "MASTER",exist = true)
    private String master;
}
```
```java
public enum AnimalType implements IEnum {
    CAT("1","猫"),DOG("2","狗"),TIGER("3","虎"),MOUSE("4","鼠"),MONKEY("5","猴"),LOAN("6","狮"),OTHER("7","其他");
    private final String value;
    private final String desc;
    AnimalType(final String value,final String desc){
        this.value=value;
        this.desc = desc;
    }
    @Override
    public Serializable getValue() {
        return value;
    }
    public String getDesc() {
        return desc;
    }
}
```
```java
public enum AnimalSex implements IEnum {
    MALE("1","公"),FEMALE("2","母");
    private final String value;
    private final String desc;
    AnimalSex(final String value,final String desc){
        this.value = value;
        this.desc = desc;
    }
    @Override
    public Serializable getValue() {
        return value;
    }
    public String getDesc() {
        return desc;
    }
}
```
### 第五步：定义mapper接口
```java
public interface AnimalRepository extends BaseMapper<Animal> {
}
```
> 解说：使用MyBatis Plus后Mapper只要继承BaseMapper接口即可，即使不添加XML映射文件也可以实现该接口提供的增删改查功能，还可以配合Wrapper进行条件操作，当然这些操作都仅仅限于单表操作，一旦涉及多表联查，那么还是乖乖添加**Mapper.xml来自定义SQL吧！！！
### 第六步：定义service（重点）
```java
@Service
@Log4j2
public class AnimalService {

    @Autowired
    private AnimalRepository animalRepository;

    //增
    public ResponseEntity<Animal> addAnimal(final Animal animal) {
        animalRepository.insert(animal);
        return ResponseEntity.ok(animal);
    }

    //删
    public ResponseEntity<Integer> deleteAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.deleteById(id));
    }

    public ResponseEntity<Integer> deleteAnimals(final Animal animal){
        return ResponseEntity.ok(animalRepository.delete(packWrapper(animal, WrapperType.QUERY)));
    }

    public ResponseEntity<Integer> deleteAnimalsByIds(List<Integer> ids){
        return ResponseEntity.ok(animalRepository.deleteBatchIds(ids));
    }

    public ResponseEntity<Integer> deleteAnimalsByMap(final Animal animal){
        Map<String, Object> params = new HashMap<>();
        if(Objects.nonNull(animal.getId())){
            params.put("ID",animal.getId());
        }
        if(StringUtils.isNotEmpty(animal.getName())){
            params.put("NAME", animal.getName());
        }
        if(Objects.nonNull(animal.getType())){
            params.put("TYPE", animal.getType());
        }
        if(Objects.nonNull(animal.getSex())){
            params.put("SEX", animal.getSex());
        }
        if (StringUtils.isNotEmpty(animal.getMaster())){
            params.put("MASTER", animal.getMaster());
        }
        return ResponseEntity.ok(animalRepository.deleteByMap(params));
    }

    //改
    public ResponseEntity<Integer> updateAnimals(final Animal animal, final Animal condition){
        return ResponseEntity.ok(animalRepository.update(animal, packWrapper(condition, WrapperType.UPDATE)));
    }

    public ResponseEntity<Integer> updateAnimal(final Animal animal){
        Wrapper<Animal> animalWrapper = new UpdateWrapper<>();
        ((UpdateWrapper<Animal>) animalWrapper).eq("id",animal.getId());
        return ResponseEntity.ok(animalRepository.update(animal, animalWrapper));
    }

    //查
    public ResponseEntity<Animal> getAnimalById(final int id){
        return ResponseEntity.ok(animalRepository.selectById(id));
    }

    public ResponseEntity<Animal> getOneAnimal(final Animal animal){
        return ResponseEntity.ok(animalRepository.selectOne(packWrapper(animal, WrapperType.QUERY)));
    }

    public ResponseEntity<List<Animal>> getAnimals(final Animal animal){
        return ResponseEntity.ok(animalRepository.selectList(packWrapper(animal, WrapperType.QUERY)));
    }

    public ResponseEntity<List<Animal>> getAnimalsByIds(List<Integer> ids){
        return ResponseEntity.ok(animalRepository.selectBatchIds(ids));
    }

    public ResponseEntity<List<Animal>> getAnimalsByMap(final Animal animal){
        Map<String, Object> params = new HashMap<>();
        if(Objects.nonNull(animal.getId())){
            params.put("ID",animal.getId());
        }
        if(StringUtils.isNotEmpty(animal.getName())){
            params.put("NAME", animal.getName());
        }
        if(Objects.nonNull(animal.getType())){
            params.put("TYPE", animal.getType());
        }
        if(Objects.nonNull(animal.getSex())){
            params.put("SEX", animal.getSex());
        }
        if (StringUtils.isNotEmpty(animal.getMaster())){
            params.put("MASTER", animal.getMaster());
        }
        return ResponseEntity.ok(animalRepository.selectByMap(params));
    }

    public ResponseEntity<List<Map<String, Object>>> getAnimalMaps(final Animal animal){
        return ResponseEntity.ok(animalRepository.selectMaps(packWrapper(animal, WrapperType.QUERY)));
    }

    //查个数
    public ResponseEntity<Integer> getCount(final Animal animal){
        return ResponseEntity.ok(animalRepository.selectCount(packWrapper(animal, WrapperType.QUERY)));
    }

    //分页查询
    public ResponseEntity<Page<Animal>> getAnimalPage(final Animal animal,final int pageId,final int pageSize){
        Page<Animal> page = new Page<>();
        page.setCurrent(pageId);
        page.setSize(pageSize);
        return ResponseEntity.ok((Page<Animal>) animalRepository.selectPage(page,packWrapper(animal, WrapperType.QUERY)));
    }

    private Wrapper<Animal> packWrapper(final Animal animal, WrapperType wrapperType){
        switch (wrapperType){
            case QUERY:
                QueryWrapper<Animal> wrapper = new QueryWrapper<>();
                if (Objects.nonNull(animal.getId()))
                    wrapper.eq("ID", animal.getId());
                if (StringUtils.isNotEmpty(animal.getName()))
                    wrapper.eq("name", animal.getName());
                if (Objects.nonNull(animal.getType()))
                    wrapper.eq("type", animal.getType());
                if (Objects.nonNull(animal.getSex()))
                    wrapper.eq("sex", animal.getSex());
                if (StringUtils.isNotEmpty(animal.getMaster()))
                    wrapper.eq("master", animal.getMaster());
                return wrapper;
            case UPDATE:
                UpdateWrapper<Animal> wrapper2 = new UpdateWrapper<>();
                if (Objects.nonNull(animal.getId()))
                    wrapper2.eq("ID", animal.getId());
                if (StringUtils.isNotEmpty(animal.getName()))
                    wrapper2.eq("name", animal.getName());
                if (Objects.nonNull(animal.getType()))
                    wrapper2.eq("type", animal.getType());
                if (Objects.nonNull(animal.getSex()))
                    wrapper2.eq("sex", animal.getSex());
                if (StringUtils.isNotEmpty(animal.getMaster()))
                    wrapper2.eq("master", animal.getMaster());
                return wrapper2;
            case QUERYLAMBDA:
                LambdaQueryWrapper<Animal> wrapper3 = new QueryWrapper<Animal>().lambda();
                if (Objects.nonNull(animal.getId()))
                    wrapper3.eq(Animal::getId, animal.getId());
                if (StringUtils.isNotEmpty(animal.getName()))
                    wrapper3.eq(Animal::getName, animal.getName());
                if (Objects.nonNull(animal.getType()))
                    wrapper3.eq(Animal::getType, animal.getType());
                if (Objects.nonNull(animal.getSex()))
                    wrapper3.eq(Animal::getSex, animal.getSex());
                if (StringUtils.isNotEmpty(animal.getMaster()))
                    wrapper3.eq(Animal::getMaster, animal.getMaster());
                return wrapper3;
            case UPDATELAMBDA:
                LambdaUpdateWrapper<Animal> wrapper4 = new UpdateWrapper<Animal>().lambda();
                if (Objects.nonNull(animal.getId()))
                    wrapper4.eq(Animal::getId, animal.getId());
                if (StringUtils.isNotEmpty(animal.getName()))
                    wrapper4.eq(Animal::getName, animal.getName());
                if (Objects.nonNull(animal.getType()))
                    wrapper4.eq(Animal::getType, animal.getType());
                if (Objects.nonNull(animal.getSex()))
                    wrapper4.eq(Animal::getSex, animal.getSex());
                if (StringUtils.isNotEmpty(animal.getMaster()))
                    wrapper4.eq(Animal::getMaster, animal.getMaster());
                return wrapper4;
            default:return null;
        }
    }
}
enum WrapperType{
    UPDATE,UPDATELAMBDA,QUERY,QUERYLAMBDA;
}
```
### 第七步：定义controller
```java
@RestController
@RequestMapping("animal")
@Api(description = "动物接口")
@Log4j2
public class AnimalApi {
    @Autowired
    private AnimalService animalService;
    @RequestMapping(value = "addAnimal",method = RequestMethod.PUT)
    @ApiOperation(value = "添加动物",notes = "添加动物",httpMethod = "PUT")
    public ResponseEntity<Animal> addAnimal(final Animal animal){
        return animalService.addAnimal(animal);
    }
    @RequestMapping(value = "deleteAnimalById", method = RequestMethod.DELETE)
    @ApiOperation(value = "删除一个动物",notes = "根据ID删除动物",httpMethod = "DELETE")
    public ResponseEntity<Integer> deleteAnimalById(final int id){
        return animalService.deleteAnimalById(id);
    }
    @RequestMapping(value = "deleteAnimalsByIds",method = RequestMethod.DELETE)
    @ApiOperation(value = "删除多个动物",notes = "根据Id删除多个动物",httpMethod = "DELETE")
    public ResponseEntity<Integer> deleteAnimalsByIds(Integer[] ids){
        return animalService.deleteAnimalsByIds(Arrays.asList(ids));
    }
    @RequestMapping(value = "deleteAnimals", method = RequestMethod.DELETE)
    @ApiOperation(value = "删除动物",notes = "根据条件删除动物",httpMethod = "DELETE")
    public ResponseEntity<Integer> deleteAnimalsByMaps(final Animal animal){
        return animalService.deleteAnimalsByMap(animal);
    }
    @RequestMapping(value = "deleteAnimals2", method = RequestMethod.DELETE)
    @ApiOperation(value = "删除动物",notes = "根据条件删除动物",httpMethod = "DELETE")
    public ResponseEntity<Integer> deleteAnimals(final Animal animal){
        return animalService.deleteAnimals(animal);
    }
    @RequestMapping(value = "getAnimalById",method = RequestMethod.GET)
    @ApiOperation(value = "获取一个动物",notes = "根据ID获取一个动物",httpMethod = "GET")
    public ResponseEntity<Animal> getAnimalById(final int id){
        return animalService.getAnimalById(id);
    }
    // 注意，这里参数animal不能用RequstBody标注，否则接收不到参数
    // @RequestBody只能用在只有一个参数模型的方法中，用于将所有请求体中携带的参数全部映射到这个请求参数模型中
    @RequestMapping(value = "getAnimalsByPage")
    @ApiOperation(value = "分页获取动物们",notes = "分页获取所有动物", httpMethod = "GET")
    public ResponseEntity<Page<Animal>> getAnimalsByPage(@RequestParam final int pageId, @RequestParam final int pageSize, final Animal animal) {
        return animalService.getAnimalPage(animal==null?Animal.builder().build():animal, pageId, pageSize);
    }
    @RequestMapping(value = "updateAnimal")
    @ApiOperation(value = "更新动物", notes = "根据条件更新",httpMethod = "POST")
    public ResponseEntity<Integer> updateAnimals(final Animal animal){
        return animalService.updateAnimal(animal);
    }
}
```
## 高级功能
### 代码生成器

### 分页插件
#### 第一步：添加必要的配置
```java
@EnableTransactionManagement
@Configuration
@MapperScan("com.example.springbootdemo.plusmapper")
public class MyBatisPlusConfig {
    @Bean // mybatis-plus分页插件
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```
#### 第二步：添加Mapper
```java
public interface AnimalRepository extends BaseMapper<Animal> {
}
```
#### 第三步：添加service
```java
@Service
@Log4j2
public class AnimalService {
    @Autowired
    private AnimalRepository animalRepository;
    //...
    public Page<Animal> getAnimalsByPage(int pageId,int pageSize) {
        Page page = new Page(pageId, pageSize);
        return (Page<Animal>)animalRepository.selectPage(page,null);
    }
}
```
### 逻辑删除
    所谓逻辑删除是相对于物理删除而言的，MyBatis Plus默认的删除操作是物理删除，即直接调用数据库的delete操作，直接将数据从数据库删除，但是，一般情况下，我们在项目中不会直接操作delete，为了保留记录，我们只是将其标记为删除，并不是真的删除，也就是需要逻辑删除，MyBatis Plus也提供了实现逻辑删除的功能，通过这种方式可以将底层的delete操作修改成update操作。
#### 第一步：添加必要的配置
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```
#### 第二步：添加必要的配置类
```java
@Configuration
public class MyBatisPlusConfiguration {

    @Bean
    public ISqlInjector sqlInjector() {
        return new LogicSqlInjector();
    }
}
```
#### 第三步：添加字段isDel和注解
```java
@TableLogic
private Integer isDel;
```
    如此一来，我们再执行delete相关操作的时候，底层就会变更为update操作，将isDel值修改为1。
> 注意：通过此种方式删除数据后，实际数据还存在于数据库中，只是字段isDel值改变了，虽然如此，但是再通过MyBatis Plus查询数据的时候却会将其忽略，就好比不存在一般。
> 即通过逻辑删除的数据和物理删除的外在表现是一致的，只是内在机理不同罢了。
### 枚举自动注入
#### 第一种方式
    使用注解@EnumValue
    使用方式：定义普通枚举，并在其中定义多个属性，将该注解标注于其中一个枚举属性之上，即可实现自动映射，使用枚举name传递，实际入库的却是添加了注解的属性值，查询也是如此，可以将库中数据与添加注解的属性对应，从而获取到枚举值name。
#### 第二种方式
    Mybatis Plus中定义了IEnum用来统筹管理所有的枚举类型，我们自定义的枚举只要实现IEnum接口即可，在MyBatis Plus初始化的时候，会自动在MyBatis中handler缓存中添加针对IEnum类型的处理器，我们的自定义的枚举均可使用这个处理器进行处理来实现自动映射。
##### 步骤一：添加必要的配置
```properties
mybatis-plus.type-enums-package: com.example.springbootdemo.enums
```
##### 步骤二：自定义枚举
```java
public enum AnimalType implements IEnum {
    CAT("1","猫"),DOG("2","狗"),TIGER("3","虎"),MOUSE("4","鼠"),MONKEY("5","猴"),LOAN("6","狮"),OTHER("7","其他");
    private final String value;
    private final String desc;
    AnimalType(final String value,final String desc){
        this.value=value;
        this.desc = desc;
    }
    @Override
    public Serializable getValue() {
        return value;
    }
    public String getDesc() {
        return desc;
    }
}
```
> 注意：一定要实现IEnum接口，否则无法实现自动注入。

### Sql自动注入

### 性能分析插件
#### 第一步：添加必要的配置
```java
@EnableTransactionManagement
@Configuration
@MapperScan("com.example.springbootdemo.plusmapper")
public class MyBatisPlusConfig {
    //...
    //sql执行效率插件（性能分析插件）
    @Bean
    @Profile({"dev","test"})// 设置 dev test 环境开启
    public PerformanceInterceptor performanceInterceptor() {
        return new PerformanceInterceptor();
    }
}
```
> 说明：
> 性能分析拦截器，用于输出每条 SQL 语句及其执行时间:
> - maxTime：SQL 执行最大时长，超过自动停止运行，有助于发现问题。
> - format：SQL SQL是否格式化，默认false。

> 注意：性能分析工具最好不要在生产环境部署，只在开发、测试环境部署用于查找问题即可。
### 乐观锁插件
#### 第一步：添加必要的配置
```java
@EnableTransactionManagement
@Configuration
@MapperScan("com.example.springbootdemo.plusmapper")
public class MyBatisPlusConfig {
    //...
    // mybatis-plus乐观锁插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```
#### 第二步：添加@Version
```java
@Version
private int version;
```
> 注意：
> - @Version支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime；
> - 整数类型下 newVersion = oldVersion + 1；
> - newVersion 会回写到 entity 中
> - 仅支持 updateById(id) 与 update(entity, wrapper) 方法
> - 在 update(entity, wrapper) 方法下, wrapper 不能复用!!!

### 实体主键配置
```java
@Getter
public enum IdType {
    /**
     * 数据库ID自增
     */
    AUTO(0),
    /**
     * 该类型为未设置主键类型
     */
    NONE(1),
    /**
     * 用户输入ID
     * 该类型可以通过自己注册自动填充插件进行填充
     */
    INPUT(2),

    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 全局唯一ID (idWorker)
     */
    ID_WORKER(3),
    /**
     * 全局唯一ID (UUID)
     */
    UUID(4),
    /**
     * 字符串全局唯一ID (idWorker 的字符串表示)
     */
    ID_WORKER_STR(5);

    private int key;

    IdType(int key) {
        this.key = key;
    }
}
```
- AUTO：自增，适用于类似MySQL之类自增主键的情况
- NONE：不设置？？？
- INPUT：通过第三方进行逐渐递增，类似Oracle数据库的队列自增
- ID_WORKER：全局唯一ID，当插入对象ID为空时，自动填充
- UUID：全局唯一ID，当插入对象ID为空时，自动填充，一般情况下UUID是无序的
- ID_WORKER_STR：字符串全局唯一ID，当插入对象ID为空时，自动填充
### 注意事项
    最好不要和devTools一起使用，因为devTools中的RestartClassLoader会导致MyBatis Plus中的枚举自动映射失败，因为类加载器的不同从而在MyBatis的TypeHasnlerRegistry的TYPE_HANDLER_MAP集合中找不到对应的枚举类型（存在这个枚举类型，只不过是用AppClassLoader加载的，不同的加载器导致类型不同）
    MyBatis Plus和JPA分页有些不同，前者从1开始计页数，后者则是从0开始。

整合源码：[样例代码](https://github.com/qe2592008/springboot-integration/tree/develop/src/main/java/com/dh/springbootintegration/mybatisplus)