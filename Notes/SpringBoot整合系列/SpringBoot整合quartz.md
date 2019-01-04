# SpringBoot整合系列-SpringBoot整合Quartz
## 步骤
### 第一步：添加必要的依赖
```xml
<!-- 整合Schedule Quartz-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```
### 第二步：添加必要的配置
可以不进行任何的自定义配置，使用默认的配置即可进行使用
```yaml
spring: 
  quartz:
    job-store-type: jdbc #配置job保存方式，默认是memory，保存到内存，jdbc表示保存到数据库
    auto-startup: true #默认为true，表示应用启动后，自动执行定时任务
    jdbc: #这个配置是在job-store-type=jdbc的情况下设置的
      comment-prefix: -- #配置SQL初始化脚本中单行注释的前缀。
      initialize-schema: always #数据库模式下的初始化方式
      schema:  #用于初始化数据库模式的SQL文件的路径。
    overwrite-existing-jobs: false #配置是否覆盖job
    properties: #用于配置额外的其他定时器配置属性
    scheduler-name: test-scheduler #配置定时器名称
    startup-delay: 0s #配置定时任务在应用初始化完成后开始的延时时间，默认为0s
    wait-for-jobs-to-complete-on-shutdown: false #配置应用关闭时，是否需要等待定时任务执行完毕，默认为false
  task:
    scheduling:
      thread-name-prefix: donghao- #用于新创建线程名称的前缀，默认为scheduling-
      pool:
        size: 1 #允许的最大线程数,默认为1
```
### 第三步：添加必要的配置类
```java
@EnableScheduling// 开启定时器
@Configuration
public class ScheduleConfig {
}
```
### 第四步：使用
```java
@Component
public class TestJob {
    @Scheduled(cron = "0/10 * * * * ?")// 具体定时操作
    public void execute1(){
        //do some things
        System.out.println("开始执行定时任务1,当前时间为" + LocalDateTime.now());
    }
    @Scheduled(cron = "0/10 * * * * ?")// 具体定时操作
    public void execute2(){
        //do some things
        System.out.println("开始执行定时任务2,当前时间为" + LocalDateTime.now());
    }
}
```
（结束）