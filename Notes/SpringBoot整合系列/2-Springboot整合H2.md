# SpringBoot整合H2内存数据库
    一般我们在测试的时候习惯于使用内存内存数据库，这里我们整合h2数据库。
## 步骤
### 第一步：添加必要的jar包
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
    还可以添加一些额外的工具jar包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```
    前者是必备jar包，后者是辅助jar包，用于查看内存数据库。
### 第二步：添加必要的配置
```properties
#h2配置
spring.jpa.show-sql = true #启用SQL语句的日志记录
spring.jpa.hibernate.ddl-auto = update  #设置ddl模式
##数据库连接设置
spring.datasource.url = jdbc:h2:mem:dbtest  #配置h2数据库的连接地址
spring.datasource.username = sa  #配置数据库用户名
spring.datasource.password = sa  #配置数据库密码
spring.datasource.driverClassName =org.h2.Driver  #配置JDBC Driver
##数据初始化设置
spring.datasource.schema=classpath:db/schema.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/schema.sql文件，对数据库的结构进行操作。
spring.datasource.data=classpath:db/data.sql  #进行该配置后，每次启动程序，程序都会运行resources/db/data.sql文件，对数据库的数据操作。
##h2 web console设置
spring.datasource.platform=h2  #表明使用的数据库平台是h2
spring.h2.console.settings.web-allow-others=true  # 进行该配置后，h2 web consloe就可以在远程访问了。否则只能在本机访问。
spring.h2.console.path=/h2  #进行该配置，你就可以通过YOUR_URL/h2访问h2 web consloe。YOUR_URL是你程序的访问URl。
spring.h2.console.enabled=true  #进行该配置，程序开启时就会启动h2 web consloe。当然这是默认的，如果你不想在启动程序时启动h2 web consloe，那么就设置为false。
```
### 第三步：添加数据库结构与数据脚本
resources/db/schema.sql
```sql
create table if not exists USER (
USE_ID int not null primary key auto_increment,
USE_NAME varchar(100),
USE_SEX varchar(1),
USE_AGE NUMBER(3),
USE_ID_NO VARCHAR(18),
USE_PHONE_NUM VARCHAR(11),
USE_EMAIL VARCHAR(100),
CREATE_TIME DATE,
MODIFY_TIME DATE,
USE_STATE VARCHAR(1));
```
resourses/db/data.sql
```sql
INSERT INTO USER (USE_ID,USE_NAME,USE_SEX,USE_AGE,USE_ID_NO,USE_PHONE_NUM,USE_EMAIL,CREATE_TIME,MODIFY_TIME,USE_STATE) VALUES(
1,'赵一','0',20,'142323198610051234','12345678910','qe259@163.com',sysdate,sysdate,'0');
INSERT INTO USER (USE_ID,USE_NAME,USE_SEX,USE_AGE,USE_ID_NO,USE_PHONE_NUM,USE_EMAIL,CREATE_TIME,MODIFY_TIME,USE_STATE) VALUES(
2,'钱二','0',22,'142323198610051235','12345678911','qe259@164.com',sysdate,sysdate,'0');
INSERT INTO USER (USE_ID,USE_NAME,USE_SEX,USE_AGE,USE_ID_NO,USE_PHONE_NUM,USE_EMAIL,CREATE_TIME,MODIFY_TIME,USE_STATE) VALUES(
3,'孙三','1',24,'142323198610051236','12345678912','qe259@165.com',sysdate,sysdate,'0');
```
### 第四步：查看h2-console
浏览器输入：
```txt
http://localhost:8080/h2
```
可以打开h2数据库管理器登录界面：
[登录界面]()
输入配置的数据库信息，点击登录，即可打开操作界面：
[操作界面]()
(结束)