# SpringBoot基础系列-定制SpringApplicaiton
## 定制方式
我们在SpringBoot应用启动的main方法中创建一个SpringApplicaiton实例，然后就可以对其进行各种定制：
```java
SpringApplication application = new SpringApplication(SpringbootdemoApplication.class);
```
另外，还可以使用Spring提供的SpringApplicaitonBuinder来进行流式定制。
```java
SpringApplication application = new SpringApplicationBuilder()
    .bannerMode(Banner.Mode.CONSOLE).build();
```
## 定制内容
### 定制Banner
定制Banner的基础是我们要有一个实现了Banner类的实例，这里采用内部类的方式定义一个Banner实例
```java
Banner banner = new Banner() {
    @Override
    public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
        out.println("就是要大于你");
    }
};
application.setBanner(banner);// 设置Banner
```
### 定制Banner输出模式
```java
applicaiton.setBannerMode(Banner.Mode.OFF)
```
Banner输出模式有三种：
```java
enum Mode {
    // Disable printing of the banner.
    OFF,
    // Print the banner to System.out.
    CONSOLE,
    // Print the banner to the log file.
    LOG
}
```
### 定制













## 附录：自定义SpringApplicaiton
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
        application.setBanner(banner);// 设置Banner
        application.setAddCommandLineProperties(false);//设置是否使用命令行属性
        application.setAddConversionService(false);// 
        application.setAdditionalProfiles("development");// 添加额外的profile
        application.setAllowBeanDefinitionOverriding(true);// 设置Bean定义可重写
        application.setApplicationContextClass(XmlWebApplicationContext.class);//设置使用的ApplicaitonContext实例类型
        application.setBannerMode(Banner.Mode.CONSOLE);//设置banner输出模式
        application.setMainApplicationClass(SpringbootdemoApplication.class);//设置main方法所在类
        application.setWebApplicationType(WebApplicationType.SERVLET);//设置应用类型
        application.setHeadless(true);// 
        application.setBeanNameGenerator();//设置自定义的bean name生成器
        application.setDefaultProperties();//设置默认的属性
        application.setEnvironment();//设置自定义的environment
        application.setInitializers();//设置自定义的初始化器
        application.setListeners();//设置自定义的监听器
        application.setLogStartupInfo(true);//设置是否显示应用启动信息
        application.setRegisterShutdownHook(true);//设置是否添加关闭狗子
        application.setResourceLoader();//设置自定义的资源加载器
        application.setSources();//设置自定义的资源
        application.run(args);//启动应用
    }
}
```