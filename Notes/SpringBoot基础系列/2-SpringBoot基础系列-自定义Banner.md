# SpringBoot基础系列-自定义Banner
## 自定义方式
### 第一种：classpath路径下添加banner.txt文件
banner.txt文件中有要展示的内容，我们可以使用占位符变量：

|序号|变量|说明|
|---|----|----|
|1|${application.version}|应用版本|
|2|${applicaiton.formatted-version}|格式化应用版本|
|3|${spring-boot.version}|springboot版本|
|4|${spring-boot.formatted-version}|格式化springboot版本|
|5|${Ansi.NAME}or${AnsiColor.NAME}or${AnsiBackground.NAME}or${AnsiStyle.NAME}|ANSI转码名称|
|6|${application.title}|应用名称|
### 第二种：自定义banner.txt文件，使用如下属性指向该文件
```properties
spring.banner.location=/xxx/banner.txt
#如果文件不是UTF-8编码，需要这里手动指定编码类型
spring.banner.charset=
```
### 第三种：classpath路径下添加banner.gif、banner.jpg、banner.png文件
### 第四种：自定义banner.gif、banner.jpg、banner.png文件，使用如下属性指向该文件
```properties
spring.banner.image.location=/xxx/banner.gif
```
### 第五种：编程方式
```java
@SpringBootApplication
public class SpringbootdemoApplication {
    public static void main(String[] args) {
        Banner banner = new Banner() {
            @Override
            public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
                out.println("就是要大于你");
            }
        };
        SpringApplication application = new SpringApplication(SpringbootdemoApplication.class);
        application.setBanner(banner);
        application.run(args);
    }
}
```
(结束)