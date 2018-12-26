# SpringBoot整合MyBatis分页插件PageHelper
## 步骤
### 第一步：首先整合MyBatis
参照之前SpringBoot整合MyBatis.md
### 第二步：添加必要的依赖
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.1.6</version>
</dependency>
```
### 第三步：添加必要的配置
无
### 第四步：添加必要的配置类
```java
@Configuration
public class PageHelperConfig {
    @Bean
    public PageHelper pageHelper(){
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("offsetAsPageNum","true");
        properties.setProperty("rowBoundsWithCount","true");
        properties.setProperty("reasonable","true");
        properties.setProperty("dialect","mysql");    //配置mysql数据库的方言
        pageHelper.setProperties(properties);
        return pageHelper;
    }
}
```
### 第五步：使用插件
#### 6-1 定义mapper，延用之前的mapper
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
BookRepository.java
```java
public interface BookRepository {
    
    //省略多余内容
    
    List<Book> getBooks(Book book);
    int count(Book book);
}
```
#### 6-2 定义service
```java
@Service
@Log4j2
public class BookService {
    
    @Autowired
    private BookRepository bookRepository;
    
    // 省略多余内容
    
    public ResponseEntity<PageInfo<Book>> getBooksByPageHelper(int pageId, int pageSize) {
        PageHelper.startPage(pageId, pageSize);
        List<Book> books = bookRepository.getBooks(Book.builder().build());
        int totalNum  = bookRepository.count(Book.builder().build());
        PageInfo<Book> page = new PageInfo<>();
        page.setPageNum(pageId);
        page.setPageSize(pageSize);
        page.setSize(totalNum);
        page.setList(books);
        return ResponseEntity.ok(page);
    }
}
```
> 此处使用PageHelper提供的PageInfo来承载分页信息，你也可以自定义分页模型来进行承载，但一般情况下使用给定的完全能满足要求
#### 6-3 定义controller
```java
@RestController
@RequestMapping("/book")
@Api(description = "书籍接口")
@Log4j2
public class BookApi {
    
    @Autowired
    private BookService bookService;
    
    // 省略多余内容
    
    @RequestMapping(value = "/getBooksByPageHelper", method = RequestMethod.GET)
    @ApiOperation(value = "分页获取书籍", notes = "通过PageHelper分页获取书籍", httpMethod = "GET")
    public ResponseEntity<PageInfo<Book>> getBooksByPageHelper(final int pageId, final int pageNum){
        return bookService.getBooksByPageHelper(pageId, pageNum);
    }

}
```
(结束)
