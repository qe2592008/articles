# SpringBoot基础系列-创建可执行jar包
## 步骤
### 第一步：使用Maven插件：spring-boot-maven-plugin
在pom.xml中添加如下maven插件，创建SpringBoot项目会自动添加的
```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```
### 第二步：运行maven命令进行打包
```youtrack
mvn package
```
### 第三步：运行jar包
打包成功后，你可以在项目的target目录中找到打包的结果，一个fat jar。
执行以下命令运行这个jar
```youtrack
java -jar target/xxx-0.0.1-SNAPSHOT.jar
```
(结束)