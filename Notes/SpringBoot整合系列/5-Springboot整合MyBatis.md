# SpringBoot整合Mybatis
## 步骤
### 第一步：添加必要的jar包
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```
### 第二步：添加必要的配置
application.properties
```properties
##配置数据源
spring.datasource.url = jdbc:h2:mem:dbtest
spring.datasource.username = sa
spring.datasource.password = sa
spring.datasource.driverClassName =org.h2.Driver
```
### 第三步：添加配置类
```java
// 该配置类用于配置自动扫描器，用于扫描自定义的mapper接口,MyBatis会针对这些接口生成代理来调用对应的XMl中的SQL
@Configuration
@MapperScan("com.example.springbootdemo.mapper")
public class MyBatisConfig {
}
```
> 这个注解必须手动配置是因为mapper接口的位置完全就是用户自定义的，自动配置的时候也不可能找到还不存在的位置。
### 第四步：定义实体类型
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@EqualsAndHashCode
@Builder
@ApiModel("书籍模型")
public class Book {
    @ApiModelProperty(value = "书籍ID", notes = "书籍ID",example = "1")
    private Integer bookId;
    @ApiModelProperty(value = "书籍页数", notes = "书籍页数",example = "100")
    private Integer pageNum;
    @ApiModelProperty(value = "书籍名称", notes = "书籍名称",example = "Java编程思想")
    private String bookName;
    @ApiModelProperty(value = "书籍类型", notes = "书籍类型",hidden = false)
    private BookType BookType;
    @ApiModelProperty(value = "书籍简介")
    private String bookDesc;
    @ApiModelProperty(value = "书籍价格")
    private Double bookPrice;
    @ApiModelProperty(value = "创建时间",hidden = true)
    private LocalDateTime createTime;
    @ApiModelProperty(value = "修改时间",hidden = true)
    private LocalDateTime modifyTime;
}
```
还有一个枚举类型
```java
public enum BookType {
    TECHNOLOGY,//技术
    LITERARY,//文学
    HISTORY//历史
    ;
}
```
> 实体类中使用了swagger2和Lombok中的注解，需要添加对应的jar包
### 第五步：定义mapper接口
```java
public interface BookRepository {
    int addBook(Book book);
    int updateBook(Book book);
    int deleteBook(int id);
    Book getBook(int id);
    List<Book> getBooks(Book book);
}
```
### 第六步：定义mapper配置
```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springbootdemo.mapper.BookRepository">
    <insert id="addBook" parameterType="Book">
        INSERT INTO BOOK(
        <if test="pageNum != null">
            PAGE_NUM,
        </if>
        <if test="bookType != null">
            BOOK_TYPE,
        </if>
        <if test="bookName != null">
            BOOK_NAME,
        </if>
        <if test="bookDesc != null">
            BOOK_DESC,
        </if>
        <if test="bookPrice != null">
            BOOK_PRICE,
        </if>
            CREATE_TIME,
            MODIFY_TIME)
        VALUES (
        <if test="pageNum != null">
            #{pageNum},
        </if>
        <if test="bookType != null">
            #{bookType},
        </if>
        <if test="bookName != null">
            #{bookName},
        </if>
        <if test="bookDesc != null">
            #{bookDesc},
        </if>
        <if test="bookPrice != null">
            #{bookPrice},
        </if>
        sysdate,sysdate)
	</insert>
    <update id="updateBook" parameterType="Book">
		UPDATE BOOK SET
		<if test="pageNum != null">
            PAGE_NUM = #{pageNum},
        </if>
        <if test="bookType != null">
            BOOK_TYPE = #{bookType},
        </if>
        <if test="bookDesc != null">
            BOOK_DESC = #{bookDesc},
        </if>
        <if test="bookPrice != null">
            BOOK_PRICE = #{bookPrice},
        </if>
        <if test="bookName != null">
            BOOK_NAME = #{bookName},
        </if>
        MODIFY_TIME=sysdate
        WHERE 1=1
        <if test="bookId != null">
            and BOOK_ID = #{bookId}
        </if>
	</update>
    <delete id="deleteBook" parameterType="int">
		delete from BOOK where BOOK_id=#{bookId}
	</delete>
    <select id="getBook" parameterType="int" resultMap="bookResultMap">
		select * from BOOK where BOOK_ID=#{bookId}
	</select>
    <select id="getBooks" resultMap="bookResultMap">
		select * from BOOK WHERE 1=1
        <if test="bookId != null">
            and BOOK_ID = #{bookId}
        </if>
        <if test="pageNum != null">
            and PAGE_NUM = #{pageNum}
        </if>
        <if test="bookType != null">
            and BOOK_TYPE = #{bookType}
        </if>
        <if test="bookDesc != null">
            and BOOK_DESC = #{bookDesc}
        </if>
        <if test="bookPrice != null">
            and BOOK_PRICE = #{bookPrice}
        </if>
        <if test="bookName != null">
            and BOOK_NAME = #{bookName}
        </if>
	</select>
    <resultMap id="bookResultMap" type="Book">
        <id column="BOOK_ID" property="bookId"/>
        <result column="PAGE_NUM" property="pageNum"/>
        <result column="BOOK_NAME" property="bookName"/>
        <result column="BOOK_TYPE" property="bookType"/>
        <result column="BOOK_DESC" property="bookDesc"/>
        <result column="BOOK_PRICE" property="bookPrice"/>
        <result column="CREATE_TIME" property="createTime"/>
        <result column="MODIFY_TIME" property="modifyTime"/>
    </resultMap>
</mapper>
```
在这个配置文件中我们使用了MyBatis的动态SQL和参数映射
### 第七步：再次添加必要的配置
application.properties
```properties
#配置Xml配置的位置
mybatis.mapper-locations=classpath*:/mapper/*.xml
#配置实体类型别名
mybatis.type-aliases-package=com.example.springbootdemo.entity
```
> 这里的两个配置也和之前的扫描器注解一样，都是自动配置时未知的，需要手动配置，当然可能会存在默认的位置，但是一旦我们自定义了，就必须手动添加配置
### 第八步：定义service和controller
```java
@Service
@Log4j2
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    public ResponseEntity<Book> addBook(final Book book) {
        int num = bookRepository.addBook(book);
        return ResponseEntity.ok(book);
    }
    
    public ResponseEntity<Integer> updateBook(final Book book){
        return ResponseEntity.ok(bookRepository.updateBook(book));
    }
    
    public ResponseEntity<Integer> deleteBook(final int bookId){
        return ResponseEntity.ok(bookRepository.deleteBook(bookId));
    }
    
    public ResponseEntity<Book> getBook(final int bookId) {
        Book book = bookRepository.getBook(bookId);
        return ResponseEntity.ok(book);
    }
    
    public ResponseEntity<List<Book>> getBooks(final Book book){
        return ResponseEntity.ok(bookRepository.getBooks(book));
    }
    
}
```
```java
@RestController
@RequestMapping("/book")
@Api(description = "书籍接口")
@Log4j2
public class BookApi {
    @Autowired
    private BookService bookService;
    
    @RequestMapping(value = "/addBook", method = RequestMethod.PUT)
    @ApiOperation(value = "添加书籍", notes = "添加一本新书籍", httpMethod = "PUT")
    public ResponseEntity<Book> addBook(final Book book){
        return bookService.addBook(book);
    }
    
    @RequestMapping(value = "/updateBook", method = RequestMethod.POST)
    @ApiOperation(value = "更新书籍", notes = "根据条件更新书籍信息", httpMethod = "POST")
    public ResponseEntity<Integer> updateBook(final Book book){
        return bookService.updateBook(book);
    }
    
    @RequestMapping(value = "/deleteBook", method = RequestMethod.DELETE)
    @ApiOperation(value = "获取一本书籍", notes = "根据ID获取书籍", httpMethod = "DELETE")
    public ResponseEntity<Integer> deleteBook(final int bookId){
        return bookService.deleteBook(bookId);
    }
    
    @RequestMapping(value = "/getBook", method = RequestMethod.GET)
    @ApiOperation(value = "获取一本书籍", notes = "根据ID获取书籍", httpMethod = "GET")
    public ResponseEntity<Book> getBook(final int bookId){
        return bookService.getBook(bookId);
    }
    
    @RequestMapping(value = "/getBooks", method = RequestMethod.GET)
    @ApiOperation(value = "获取书籍", notes = "根据条件获取书籍", httpMethod = "GET")
    public ResponseEntity<List<Book>> getBooks(final Book book){
        return bookService.getBooks(book);
    }
}
```
这里使用了swagger2的注解
至此设置完毕。
### 第十步：浏览器访问
```txt
http://localhost:8080/swagger-ui.html
```
通过swagger界面可以看到我们定义的接口。
## 高级功能
### 分页（两种，简单分页RowBounds和拦截器分页，插件）
#### RowBounds分页
使用RowBounds分页适用于小数据量的分页查询
使用方式是在查询的Mapper接口上添加RowBounds参数即可，service传参时需要指定其两个属性，当前页和每页数
##### 1-定义分页模型
```java
@Data
@Builder
@ToString
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class MyPage<T> {
    private Integer pageId;//当前页
    private Integer pageNum;//总页数
    private Integer pageSize;//每页数
    private Integer totalNum;//总数目
    private List<T> body;//分页结果
    private Integer srartIndex;//开始索引
    private boolean isMore;//是否有下一页
}
```
##### 2-定义mapper
```java
public interface BookRepository {
    
    // 省略多余内容
    
    int count(Book book);
    List<Book> getBooks(Book book, RowBounds rowBounds);
}
```
BookRepository.xml
```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springbootdemo.mapper.BookRepository">
   
    <!--省略多余内容-->
    
    <select id="getBooks" resultMap="bookResultMap">
        select * from BOOK WHERE 1=1
        <if test="bookId != null">
            and BOOK_ID = #{bookId}
        </if>
        <if test="pageNum != null">
            and PAGE_NUM = #{pageNum}
        </if>
        <if test="bookType != null">
            and BOOK_TYPE = #{bookType}
        </if>
        <if test="bookDesc != null">
            and BOOK_DESC = #{bookDesc}
        </if>
        <if test="bookPrice != null">
            and BOOK_PRICE = #{bookPrice}
        </if>
        <if test="bookName != null">
            and BOOK_NAME = #{bookName}
        </if>
    </select>
    <select id="count" resultType="int">
        select count(1) from BOOK WHERE 1=1
        <if test="bookId != null">
            and BOOK_ID = #{bookId}
        </if>
        <if test="pageNum != null">
            and PAGE_NUM = #{pageNum}
        </if>
        <if test="bookType != null">
            and BOOK_TYPE = #{bookType}
        </if>
        <if test="bookDesc != null">
            and BOOK_DESC = #{bookDesc}
        </if>
        <if test="bookPrice != null">
            and BOOK_PRICE = #{bookPrice}
        </if>
        <if test="bookName != null">
            and BOOK_NAME = #{bookName}
        </if>
    </select>
    <resultMap id="bookResultMap" type="Book">
        <id column="BOOK_ID" property="bookId"/>
        <result column="PAGE_NUM" property="pageNum"/>
        <result column="BOOK_NAME" property="bookName"/>
        <result column="BOOK_TYPE" property="bookType"/>
        <result column="BOOK_DESC" property="bookDesc"/>
        <result column="BOOK_PRICE" property="bookPrice"/>
        <result column="CREATE_TIME" property="createTime"/>
        <result column="MODIFY_TIME" property="modifyTime"/>
    </resultMap>
</mapper>
```
##### 3-定义service
```java
@Service
@Log4j2
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    // 省略多余内容
    // 使用RowBounds实现分页
    public ResponseEntity<MyPage<Book>> getBooksByRowBounds(int pageId,int pageSize){
        MyPage<Book> myPage = new MyPage<>();
        myPage.setPageId(pageId);
        myPage.setPageSize(pageSize);
        List<Book> books = bookRepository.getBooks(Book.builder().build(), new RowBounds(pageId,pageSize));
        int totalNum = bookRepository.count(Book.builder().build());
        myPage.setBody(books);
        myPage.setTotalNum(totalNum);
        return ResponseEntity.ok(myPage);
    }
}
```
##### 4-定义controller
```java
@RestController
@RequestMapping("/book")
@Api(description = "书籍接口")
@Log4j2
public class BookApi {
   
    @Autowired
    private BookService bookService;
    // 省略多余内容
    @RequestMapping(value = "/getBooksPageByRowBounds", method = RequestMethod.GET)
    @ApiOperation(value = "分页获取书籍", notes = "通过RowBounds分页获取书籍", httpMethod = "GET")
    public ResponseEntity<PageInfo<Book>> getBooksPageByRowBounds(final int pageId, final int pageNum){
        return bookService.getBooksByRowBounds(pageId, pageNum);
    }
    
}
```
#### 拦截器分页
当面对大数据量的分页时，RowBounds就力不从心的，这时需要我们使用分页拦截器实现分页。
这里其实可以直接使用插件PageHelper，其就是以拦截器技术实现的分页查询插件。
### 自定义类型转换器（枚举转换器）
```java
public class BookTypeEnumHandler extends BaseTypeHandler<BookType> {
    /**
     * 用于定义设置参数时，该如何把Java类型的参数转换为对应的数据库类型
     */
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, BookType parameter, JdbcType jdbcType) throws SQLException {
        int j = 0;
        for (BookType bookType : BookType.values()){
            if(bookType.equals(parameter)){
                ps.setString(i, j +"");
                return;
            }
            j++;
        }
    }
    /**
     * 用于定义通过字段名称获取字段数据时，如何把数据库类型转换为对应的Java类型
     */
    @Override
    public BookType getNullableResult(ResultSet rs, String columnName) throws SQLException {
        int j = Integer.valueOf(rs.getString(columnName));
        if(j >= BookType.values().length) {
            return null;
        }
        int i = 0;
        for(BookType bookType:BookType.values()){
            if(j == i){
                return bookType;
            }
            i++;
        }
        return null;
    }
    /**
     * 用于定义通过字段索引获取字段数据时，如何把数据库类型转换为对应的Java类型
     */
    @Override
    public BookType getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return null;
    }
    /**
     * 用定义调用存储过程后，如何把数据库类型转换为对应的Java类型
     */
    @Override
    public BookType getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return null;
    }
}
```
### 使用@Mapper（不常用）
> 注意：使用@Mapper注解的时候是不需要添加xml配置Mapper文件的，SQL脚本在接口方法的注解内部定义
#### 第一步：定义实体类
```java
@Data
@Builder
@ToString
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class Tree {
    private Integer treeId;
    private String treeName;
    private Integer treeAge;
    private Double treeHight;
    private TreeType treeType;
    private TreeState treeState;
    private String treeDesc;
}
```
#### 第二步：定义持久层
```java
@Mapper
public interface TreeRepository {
    
    @Insert("INSERT INTO TREE (TREE_NAME,TREE_AGE,TREE_HIGHT,TREE_TYPE,TREE_STATE,TREE_DESC) VALUES (#{treeName},#{treeAge},#{treeHight},#{treeType},#{treeState},#{treeDesc}) ")
    int addTree(Tree tree);
    
    // 此处treeState是一个枚举，此处执行一直报错
    @Update("UPDATE TREE SET TREE_STATE=#{treeState} WHERE TREE_ID=#{treeId}")
    int updateState(final int treeId, final TreeState treeState);
    
    @Delete("DELETE FROM TREE WHERE TREE_ID=#{treeId}")
    int deleteTree(final int treeId);
    
    @Results({
            @Result(id = true, column = "TREE_ID",property = "treeId"),
            @Result(column = "TREE_NAME",property = "treeName"),
            @Result(column = "TREE_AGE", property = "treeAge"),
            @Result(column = "TREE_HIGHT",property = "treeHight"),
            @Result(column = "TREE_TYPE",property = "treeType",typeHandler = EnumOrdinalTypeHandler.class),
            @Result(column = "TREE_STATE",property = "treeState",typeHandler = EnumOrdinalTypeHandler.class),
            @Result(column = "TREE_DESC", property = "treeDesc")
    })
    @Select("SELECT * FROM TREE WHERE TREE_ID=#{treeId}")
    Tree getTree(final int treeId);
    
    @Results({
            @Result(id = true, column = "TREE_ID",property = "treeId"),
            @Result(column = "TREE_NAME",property = "treeName"),
            @Result(column = "TREE_AGE", property = "treeAge"),
            @Result(column = "TREE_HIGHT",property = "treeHight"),
            @Result(column = "TREE_TYPE",property = "treeType",typeHandler = EnumOrdinalTypeHandler.class),
            @Result(column = "TREE_STATE",property = "treeState",typeHandler = EnumOrdinalTypeHandler.class),
            @Result(column = "TREE_DESC", property = "treeDesc")
    })
    @Select("SELECT * FROM TREE")
    List<Tree> getTrees(RowBounds rowBounds);
}
```
> 注意：重点就在这个接口中，我们添加接口注解@Mapper，表示这是一个持久层Mapper,它的实例化依靠SpringBoot自动配置完成。
> 在接口方法上直接添加对应的执行注解，在注解中直接定义SQL，这种SQL仍然可以使用表达式#{}来获取参数的值。
> 注意@Result注解中定义的两个关于枚举的类型处理器EnumOrdinalTypeHandler，其实其为MyBatis内部自带的两种枚举处理器之一，
> 用于存储枚举序号，还有一个EnumTypeHandler用于存储枚举名称。
#### 第三步：定义service和controller
```java
@Service
@Log4j2
public class TreeService {
    
    @Autowired
    private TreeRepository treeRepository;
    
    public ResponseEntity<Tree> addTree(final Tree tree){
        treeRepository.addTree(tree);
        return ResponseEntity.ok(tree);
    }
    
    public ResponseEntity<Tree> updateTree(final int treeId, final TreeState treeState){
        treeRepository.updateState(treeId,treeState);
        return ResponseEntity.ok(Tree.builder().treeId(treeId).treeState(treeState).build());
    }
    
    public ResponseEntity<Integer> deleteTree(final int treeId){
        return ResponseEntity.ok(treeRepository.deleteTree(treeId));
    }
    
    public ResponseEntity<Tree> getTree(final int treeId){
        return ResponseEntity.ok(treeRepository.getTree(treeId));
    }
   
    public ResponseEntity<MyPage<Tree>> getTrees(final int pageId,final int pageSize){
        List<Tree> trees = treeRepository.getTrees(new RowBounds(pageId,pageSize));
        MyPage<Tree> treeMyPage = new MyPage<>();
        treeMyPage.setPageId(pageId);
        treeMyPage.setPageSize(pageSize);
        treeMyPage.setBody(trees);
        return ResponseEntity.ok(treeMyPage);
    }
}
```
```java
@RestController
@RequestMapping("/tree")
@Api(description = "树木接口")
public class TreeApi {
   
    @Autowired
    private TreeService treeService;
    
    @RequestMapping(value = "/addTree",method = RequestMethod.PUT)
    @ApiOperation(value = "添加树木",notes = "添加新树木",httpMethod = "PUT")
    public ResponseEntity<Tree> addTree(final Tree tree){
        return treeService.addTree(tree);
    }
    
    @RequestMapping(value = "/updateTree",method = RequestMethod.POST)
    @ApiOperation(value = "更新状态",notes = "修改树木状态",httpMethod = "POST")
    public ResponseEntity<Tree> updateTree(final int treeId,final TreeState treeState){
        return treeService.updateTree(treeId,treeState);
    }
    
    @ApiOperation(value = "获取树木",notes = "根据ID获取一棵树",httpMethod = "GET")
    @RequestMapping(value = "/getTree",method = RequestMethod.GET)
    public ResponseEntity<Tree> getTree(final int treeId){
        return treeService.getTree(treeId);
    }
}
```
整合源码：[样例代码](https://github.com/qe2592008/springboot-integration/tree/develop/src/main/java/com/dh/springbootintegration/mybatis)

(暂时结束)