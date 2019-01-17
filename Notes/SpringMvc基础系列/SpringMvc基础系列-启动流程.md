# SpringMvc基础系列-启动流程
## 一、概述
我说的容器启动流程涉及两种情况，SSM开发模式和Springboot开发模式。

SSM开发模式中，需要配置web.xml文件用作启动配置文件，而Springboot开发模式中由main方法直接启动。

下面是web项目中容器启动的流程，起点是web.xml中配置的ContextLoaderListener监听器。
## 二、调用流程图
![调用图](..\images\Springweb调用栈.png)
## 三、流程解析
### 3.1 IOC容器初始化
Tomcat服务器启动时会读取项目中web.xml中的配置项来生成ServletContext，在其中注册的ContextLoaderListener是ServletContextListener接口的实现类，它会时刻监听ServletContext的动作，包括创建和销毁，ServletContext创建的时候会触发其contextInitialized()初始化方法的执行。而Spring容器的初始化操作就在这个方法之中被触发。

源码1-来自：ContextLoaderListener
```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    /**
     * Initialize the root web application context.
     */
    @Override
    public void contextInitialized(ServletContextEvent event) {
        initWebApplicationContext(event.getServletContext());
    }
    //...
}
```
ContextLoaderListener是启动和销毁Spring的根WebApplicationContext（web容器或者web应用上下文）的引导器，其中实现了ServletContextListener的contextInitialized容器初始化方法与contextDestoryed销毁方法，用于引导根web容器的创建和销毁。

上面方法中contextInitialized就是初始化根web容器的方法。其中调用了initWebApplicationContext方法进行Spring web容器的具体创建。

源码2-来自：ContextLoader
```java
public class ContextLoader {
    //...
    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
        // SpringIOC容器的重复性创建校验
        if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
            throw new IllegalStateException(
                    "Cannot initialize context because there is already a root application context present - " +
                    "check whether you have multiple ContextLoader* definitions in your web.xml!");
        }

        Log logger = LogFactory.getLog(ContextLoader.class);
        servletContext.log("Initializing Spring root WebApplicationContext");
        if (logger.isInfoEnabled()) {
            logger.info("Root WebApplicationContext: initialization started");
        }
        // 记录Spring容器创建开始时间
        long startTime = System.currentTimeMillis();

        try {
            // Store context in local instance variable, to guarantee that
            // it is available on ServletContext shutdown.
            if (this.context == null) {
                // 创建Spring容器实例
                this.context = createWebApplicationContext(servletContext);
            }
            if (this.context instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
                if (!cwac.isActive()) {
                    // 容器只有被刷新至少一次之后才是处于active（激活）状态
                    if (cwac.getParent() == null) {
                        // 此处是一个空方法，返回null,也就是不设置父级容器
                        ApplicationContext parent = loadParentContext(servletContext);
                        cwac.setParent(parent);
                    }
                    // 重点操作：配置并刷新容器
                    configureAndRefreshWebApplicationContext(cwac, servletContext);
                }
            }
            // 将创建完整的Spring容器作为一条属性添加到Servlet容器中
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

            ClassLoader ccl = Thread.currentThread().getContextClassLoader();
            if (ccl == ContextLoader.class.getClassLoader()) {
                // 如果当前线程的类加载器是ContextLoader类的类加载器的话，也就是说如果是当前线程加载了ContextLoader类的话，则将Spring容器在ContextLoader实例中保留一份引用
                currentContext = this.context;
            }
            else if (ccl != null) {
                // 添加一条ClassLoader到Springweb容器的映射
                currentContextPerThread.put(ccl, this.context);
            }

            if (logger.isDebugEnabled()) {
                logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
                        WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
            }
            if (logger.isInfoEnabled()) {
                long elapsedTime = System.currentTimeMillis() - startTime;
                logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
            }

            return this.context;
        }
        catch (RuntimeException ex) {
            logger.error("Context initialization failed", ex);
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
            throw ex;
        }
        catch (Error err) {
            logger.error("Context initialization failed", err);
            servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
            throw err;
        }
    }
    //...
}
```
这里十分明了的显示出了ServletContext和Spring root ApplicationContext的关系：后者只是前者的一个属性，前者包含后者。

ServletContext表示的是一整个应用，其中囊括应用的所有内容，而Spring只是应用所采用的一种框架。从ServletContext的角度来看，Spring其实也算是应用的一部分，属于和我们编写的代码同级的存在，只是相对于我们编码人员来说，Spring是作为一种即存的编码架构而存在的，即我们将其看作我们编码的基础，或者看作应用的基础部件。虽然是基础部件，但也是属于应用的一部分。所以将其设置到ServletContext中，而且是作为一个单一属性而存在，但是它的作用却是很大的。

源码中WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE的值为：WebApplicationContext.class.getName() + ".ROOT"，这个正是Spring容器在Servlet容器中的属性名。

在这段源码中主要是概述Spring容器的创建和初始化，分别由两个方法实现：**createWebApplicationContext方法和configureAndRefreshWebApplicationContext方法**。

首先，我们需要创建Spring容器，我们需要决定使用哪个容器实现。

源码3-来自：ContextLoader
```java
public class ContextLoader {
    //...
    protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
        // 决定使用哪个容器实现
        Class<?> contextClass = determineContextClass(sc);
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
                    "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
        }
        // 反射方式创建容器实例
        return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
    }
    //...
    // 我们在web.xml中以 <context-param> 的形式设置contextclass参数来指定使用哪个容器实现类，
    // 若未指定则使用默认的XmlWebApplicationContext，其实这个默认的容器实现也是预先配置在一个
    // 叫ContextLoader.properties文件中的
    protected Class<?> determineContextClass(ServletContext servletContext) {
        // 获取Servlet容器中配置的系统参数contextClass的值，如果未设置则为null
        String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
        if (contextClassName != null) {
            try {
                return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
            }
            catch (ClassNotFoundException ex) {
                throw new ApplicationContextException(
                        "Failed to load custom context class [" + contextClassName + "]", ex);
            }
        }
        else {
            // 获取预先配置的容器实现类
            contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
            try {
                return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
            }
            catch (ClassNotFoundException ex) {
                throw new ApplicationContextException(
                        "Failed to load default context class [" + contextClassName + "]", ex);
            }
        }
    }
    //...
}
```
BeanUtils是Spring封装的反射实现，instantiateClass方法用于实例化指定类。

我们可以在web.xml中以 <context-param> 的形式设置contextclass参数手动决定应用使用哪种Spring容器，但是一般情况下我们都遵循Spring的默认约定，使用ContextLoader.properties中配置的org.springframework.web.context.WebApplicationContext的值来作为默认的Spring容器来创建。

源码4-来自：ContextLoader.properties
```properties
# Default WebApplicationContext implementation class for ContextLoader.
# Used as fallback when no explicit context implementation has been specified as context-param.
# Not meant to be customized by application developers.

org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext
```
可见，一般基于Spring的web服务默认使用的都是XmlWebApplicationContext作为容器实现类的。

到此位置容器实例就创建好了，下一步就是配置和刷新了。

源码4-来自：ContextLoader
```java
public class ContextLoader {
    //...
    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            // The application context id is still set to its original default value
            // -> assign a more useful id based on available information 
            String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
            if (idParam != null) {
                wac.setId(idParam);
            }
            else {
                // Generate default id...
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                        ObjectUtils.getDisplayString(sc.getContextPath()));
            }
        }
        // 在当前Spring容器中保留对Servlet容器的引用
        wac.setServletContext(sc);
        // 设置web.xml中配置的contextConfigLocation参数值到当前容器中
        String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
        if (configLocationParam != null) {
            wac.setConfigLocation(configLocationParam);
        }

        // The wac environment's #initPropertySources will be called in any case when the context
        // is refreshed; do it eagerly here to ensure servlet property sources are in place for
        // use in any post-processing or initialization that occurs below prior to #refresh
        // 在容器刷新之前，提前进行属性资源的初始化，以备使用，将ServletContext设置为servletContextInitParams
        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
        }
        // 取得web.xml中配置的contextInitializerClasses和globalInitializerClasses对应的初始化器，并执行初始化操作，需自定义初始化器
        customizeContext(sc, wac);
        // 刷新容器
        wac.refresh();
    }
    //...
}
```
首先将ServletContext的引用置入Spring容器中，以便可以方便的访问ServletContext；然后从ServletContext中找到“contextConfigLocation”参数的值，配置是在web.xml中以<context-param>形式配置的。

在Spring中凡是以<context-param>配置的内容都会在web.xml加载的时候优先存储到ServletContext之中，我们可以将其看成ServletContext的配置参数，将参数配置到ServletContext中后，我们就能方便的获取使用了，就如此处我们就能直接从ServletContext中获取“contextConfigLocation”的值，用于初始化Spring容器。

在Java的web开发中，尤其是使用Spring开发的情况下，基本就是一个容器对应一套配置，这套配置就是用于初始化容器的。ServletContext对应于<context-param>配置，Spring容器对应applicationContext.xml配置，这个配置属于默认的配置，如果自定义名称就需要将其配置到<context-param>中备用了，还有DispatchServlet的Spring容器对应spring-mvc.xml配置文件。

Spring容器的environment表示的是容器运行的基础环境配置，其中保存的是Profile和Properties，其initPropertySources方法是在ConfigurableWebEnvironment接口中定义的，是专门用于web应用中来执行真实属性资源与占位符资源（StubPropertySource）的替换操作的。

StubPropertySource就是一个占位用的实例，在应用上下文创建时，当实际属性资源无法及时初始化时，临时使用这个实例进行占位，等到容器刷新的时候执行替换操作。

上面源码中customizeContext方法的目的是在刷新容器之前对容器进行自定义的初始化操作，需要我们实现ApplicationContextInitializer<C extends ConfigurableApplicationContext>接口，然后将其配置到web.xml中即可生效。

源码5-来自：ContextLoader
```java
public class ContextLoader {
    //...
    protected void customizeContext(ServletContext sc, ConfigurableWebApplicationContext wac) {
        // 获取初始化器类集合
        List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> initializerClasses =
                determineContextInitializerClasses(sc);

        for (Class<ApplicationContextInitializer<ConfigurableApplicationContext>> initializerClass : initializerClasses) {
            Class<?> initializerContextClass =
                    GenericTypeResolver.resolveTypeArgument(initializerClass, ApplicationContextInitializer.class);
            if (initializerContextClass != null && !initializerContextClass.isInstance(wac)) {
                throw new ApplicationContextException(String.format(
                        "Could not apply context initializer [%s] since its generic parameter [%s] " +
                        "is not assignable from the type of application context used by this " +
                        "context loader: [%s]", initializerClass.getName(), initializerContextClass.getName(),
                        wac.getClass().getName()));
            }
            // 实例化初始化器并添加到集合中
            this.contextInitializers.add(BeanUtils.instantiateClass(initializerClass));
        }
        // 排序并执行，编号越小越早执行
        AnnotationAwareOrderComparator.sort(this.contextInitializers);
        for (ApplicationContextInitializer<ConfigurableApplicationContext> initializer : this.contextInitializers) {
            initializer.initialize(wac);
        }
    }
    //...
    protected List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>>
            determineContextInitializerClasses(ServletContext servletContext) {

        List<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>> classes =
                new ArrayList<Class<ApplicationContextInitializer<ConfigurableApplicationContext>>>();
        // 通过<context-param>属性配置globalInitializerClasses获取全局初始化类名
        String globalClassNames = servletContext.getInitParameter(GLOBAL_INITIALIZER_CLASSES_PARAM);
        if (globalClassNames != null) {
            for (String className : StringUtils.tokenizeToStringArray(globalClassNames, INIT_PARAM_DELIMITERS)) {
                classes.add(loadInitializerClass(className));
            }
        }
        // 通过<context-param>属性配置contextInitializerClasses获取容器初始化类名
        String localClassNames = servletContext.getInitParameter(CONTEXT_INITIALIZER_CLASSES_PARAM);
        if (localClassNames != null) {
            for (String className : StringUtils.tokenizeToStringArray(localClassNames, INIT_PARAM_DELIMITERS)) {
                classes.add(loadInitializerClass(className));
            }
        }

        return classes;
    }
    //...
}
```
initPropertySources操作用于配置属性资源，其实在refresh操作中也会执行该操作，这里提前执行，目的为何，暂未可知。

到达refresh操作我们先暂停。refresh操作是容器初始化的操作。是通用操作，而到达该点的方式确实有多种，每种就是一种Spring的开发方式。
### 3.2 DispatcherServlet初始化
Spring MVC应用除了上面讲述的Root IOC容器的创建之外，还有DispatcherServlet的初始化过程。

DispatcherServlet是配置到web.xml中的，在ContextLoaderListener完成Root IOC容器的创建、所有Filter加载完成之后，就是Servlet的加载了。

加载Servlet，从调用其init方法开始。纵观DispatcherServlet继承结构（见附录），init方法位于HttpServletBean抽象类中：

源码6-来自：HttpServletBean
```java
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
    //...
    @Override
    public final void init() throws ServletException {

        // Set bean properties from init parameters.
        // 从ServletConfig中获取初始化参数，将其设置到Servlet中
        // 获取ServletConfig中的所有初始化参数（init-param）
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
        if (!pvs.isEmpty()) {
            try {
                // 
                BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
                ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
                bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
                initBeanWrapper(bw);// 这是个空方法，留待扩展
                bw.setPropertyValues(pvs, true);// 将初始化参数设置到Servlet中
            }
            catch (BeansException ex) {
                if (logger.isErrorEnabled()) {
                    logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
                }
                throw ex;
            }
        }

        // Let subclasses do whatever initialization they like.
        // 将下面的初始化操作留给子类来完成
        initServletBean();
    }
    //...
}
```
首先进行参数设置，将web.xml中配置的init-param设置到Servlet的对应属性中。

然后进行初始化操作，这里的初始化指的是针对当前Servlet的子容器。

初始化流程为：首先尝试获取Root IOC容器，也就是根容器，这里是一定能获取到的，毕竟咱们上面的源码就是在创建这个根容器，然后通过反射调用XmlWebApplicationContext的构造器创建子容器实例，子容器创建完成后，为其设置环境Environment、父容器、容器ID、ServletContext引用，ServletConfig引用，命名空间、应用监听器（监听容器刷新事件），之后执行指定的容器初始化器，最后就是刷新容器了。

源码7-来自：FrameworkServlet
```java
public abstract class FrameworkServlet extends HttpServletBean implements ApplicationContextAware {
    //...
	@Override
	protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
		if (logger.isInfoEnabled()) {
			logger.info("Initializing Servlet '" + getServletName() + "'");
		}
		long startTime = System.currentTimeMillis();

		try {
			this.webApplicationContext = initWebApplicationContext();// 初始化子容器（相对上面的IOC容器而言）
			initFrameworkServlet();// 空方法，留待扩展
		}
		catch (ServletException | RuntimeException ex) {
			logger.error("Context initialization failed", ex);
			throw ex;
		}

		if (logger.isDebugEnabled()) {
			String value = this.enableLoggingRequestDetails ?
					"shown which may lead to unsafe logging of potentially sensitive data" :
					"masked to prevent unsafe logging of potentially sensitive data";
			logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
					"': request parameters and headers will be " + value);
		}

		if (logger.isInfoEnabled()) {
			logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
		}
	}
	//...
    protected WebApplicationContext initWebApplicationContext() {
	    // 尝试获取根容器
	    // 在我们分析的项目中，一定是存在根容器的，它就在之前刚刚创建完成
        WebApplicationContext rootContext =
                WebApplicationContextUtils.getWebApplicationContext(getServletContext());
        WebApplicationContext wac = null;
        // 一般来说，启动时这里webApplicaitonContext是空的
        if (this.webApplicationContext != null) {
            // A context instance was injected at construction time -> use it
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
                if (!cwac.isActive()) {
                    // The context has not yet been refreshed -> provide services such as
                    // setting the parent context, setting the application context id, etc
                    if (cwac.getParent() == null) {
                        // The context instance was injected without an explicit parent -> set
                        // the root application context (if any; may be null) as the parent
                        cwac.setParent(rootContext);
                    }
                    configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }
        if (wac == null) {
            // No context instance was injected at construction time -> see if one
            // has been registered in the servlet context. If one exists, it is assumed
            // that the parent context (if any) has already been set and that the
            // user has performed any initialization such as setting the context id
            // 如果为Servlet自定义了容器实例，则直接从ServletContext中获取
            wac = findWebApplicationContext();
        }
        if (wac == null) {
            // No context instance is defined for this servlet -> create a local one
            // 一般我们也不会自定义容器，所以通常都会走这里
            wac = createWebApplicationContext(rootContext);
        }
        // 下面开始初始化DispatcherServlet
        if (!this.refreshEventReceived) {
            // Either the context is not a ConfigurableApplicationContext with refresh
            // support or the context injected at construction time had already been
            // refreshed -> trigger initial onRefresh manually here.
            synchronized (this.onRefreshMonitor) {
                onRefresh(wac);
            }
        }

        if (this.publishContext) {
            // Publish the context as a servlet context attribute.
            String attrName = getServletContextAttributeName();
            getServletContext().setAttribute(attrName, wac);
        }

        return wac;
    }
    //...
    protected WebApplicationContext createWebApplicationContext(@Nullable WebApplicationContext parent) {
        return createWebApplicationContext((ApplicationContext) parent);
    }
    //...
    protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
        // 获取上下文的类，默认为：XmlWebApplicationContext.class
	    Class<?> contextClass = getContextClass();
	    // 校验获取到的上下文类与ConfigurableWebApplicationContext类的关系，是否继承关系
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException(
                    "Fatal initialization error in servlet with name '" + getServletName() +
                    "': custom WebApplicationContext class [" + contextClass.getName() +
                    "] is not of type ConfigurableWebApplicationContext");
        }
        // 创建容器实例，该容器为XmlWebApplicationContext类型，这里指向ConfigurableWebApplicationContext类型
        ConfigurableWebApplicationContext wac =
                (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
        // 为容器实例设置环境，这个环境是一个新建的StandardServletEnvironment
        // 这个environment中默认保存着：系统属性、系统环境变量、ServletContext初始化参数、ServletConfig初始化参数
        wac.setEnvironment(getEnvironment());
        // 设置父级容器，即根容器
        wac.setParent(parent);
        String configLocation = getContextConfigLocation();
        if (configLocation != null) {
            wac.setConfigLocation(configLocation);
        }
        // 配置并刷新新建的容器
        configureAndRefreshWebApplicationContext(wac);

        return wac;
    }
    //...
    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
	    // 设置容器ID
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            // The application context id is still set to its original default value
            // -> assign a more useful id based on available information
            if (this.contextId != null) {
                wac.setId(this.contextId);
            }
            else {
                // Generate default id...
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                        ObjectUtils.getDisplayString(getServletContext().getContextPath()) + '/' + getServletName());
            }
        }
        // 当前容器中持有对ServletContext的引用
        wac.setServletContext(getServletContext());
        // 当前容器中持有对ServletConfig的引用
        wac.setServletConfig(getServletConfig());
        wac.setNamespace(getNamespace());// 设置命名空间
        // 添加应用监听器，监听容器刷新事件：ContextRefreshedEvent
        wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));

        // The wac environment's #initPropertySources will be called in any case when the context
        // is refreshed; do it eagerly here to ensure servlet property sources are in place for
        // use in any post-processing or initialization that occurs below prior to #refresh
        // 这里调用initPropertySources方法的目的是提前检查一下，确保Servlet属性源已就绪
        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
        }
        // 容器刷新前的后处理操作，空的，留待扩展
        postProcessWebApplicationContext(wac);
        // 容器刷新之前执行初始化器：ApplicationContextInitializer
        applyInitializers(wac);
        // 执行容器刷新
        wac.refresh();
    }
    //...
    protected void applyInitializers(ConfigurableApplicationContext wac) {
	    // 获取初始化参数中配置的容器初始化器
        String globalClassNames = getServletContext().getInitParameter(ContextLoader.GLOBAL_INITIALIZER_CLASSES_PARAM);
        if (globalClassNames != null) {
            for (String className : StringUtils.tokenizeToStringArray(globalClassNames, INIT_PARAM_DELIMITERS)) {
                this.contextInitializers.add(loadInitializer(className, wac));
            }
        }

        if (this.contextInitializerClasses != null) {
            for (String className : StringUtils.tokenizeToStringArray(this.contextInitializerClasses, INIT_PARAM_DELIMITERS)) {
                this.contextInitializers.add(loadInitializer(className, wac));
            }
        }
        // 排序
        AnnotationAwareOrderComparator.sort(this.contextInitializers);
        for (ApplicationContextInitializer<ConfigurableApplicationContext> initializer : this.contextInitializers) {
            // 循环执行所有的容器初始化器
            initializer.initialize(wac);
        }
    }
}	
```
创建好子容器之后，重点来了，下面就是进行DispatcherServlet的初始化工作了：
```java
public class DispatcherServlet extends FrameworkServlet {
    //...
    @Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}

	/**
	 * Initialize the strategy objects that this servlet uses.
	 * <p>May be overridden in subclasses in order to initialize further strategy objects.
	 */
	protected void initStrategies(ApplicationContext context) {
	    // 初始化Multipart处理器（MultipartResolver），如果BeanFactory中没有指定name（multipartResolver）的Bean，则不执行初始化
		initMultipartResolver(context);
		// 初始化Locale处理器（LocaleResolver），如果beanFactory中没有指定的name（localeResolver）的Bean，则使用AcceptHeaderLocaleResolver
		initLocaleResolver(context);
		// 初始化theme处理器（ThemeResolver），如果beanFactory中没有指定的name（themeResolver）的Bean，则使用FixedThemeResolver
		initThemeResolver(context);
		// 初始化HandlerMappings，如果没有则初始化一个默认的BeanNameUrlHandlerMapping和，必须保证至少有一个HandlerMapping
		initHandlerMappings(context);
		// 初始化HandlerAdapters，如果没有则初始化一个默认的SimpleControllerHandlerAdapter,必须要在至少有一个HandlerAdapter
		initHandlerAdapters(context);
		// 初始化异常解析器（HandlerExceptionResolvers），如果没有，那么就不初始化
		initHandlerExceptionResolvers(context);
		// 初始化RequestToViewNameTranslator，如果没有就初始化默认的DefaultRequestToViewNameTranslator，name为viewNameTranslator
		initRequestToViewNameTranslator(context);
		// 初始化视图解析器，ViewResolver，如果没有就初始化默认的InternalResourceViewResolver
		initViewResolvers(context);
		// 初始化FlushMapManager,如果没有则使用默认的DefaultFlashMapManager
		initFlashMapManager(context);
	}
	//...
}
```
DispatcherServlet的初始化，其实就是初始化它所用到的九大组件：
1. MultipartResolver：文件上传的解析器，在需要实现文件上传功能的情况下，初始化它，否则不用
2. LocaleResolver：区域配置解析器，这个如果没有手动设置，我们需要使用默认设置，默认的配置在DispatcherServlet.properties文件中
3. ThemeResolver：主题解析器，如果没有手动设置，我们需要使用默认值，默认的配置在DispatcherServlet.properties文件中
4. HandlerMapping：处理器映射器，主要用于通过URI来映射找到指定的handler。
    如果没有找到自定义的HandlerMapping，需要采用默认的，DispatcherServlet中至少要有一个HandlerMapping
    其实整个MVC应用中也只有一个HandlerMapping实例，由这一个HandlerMapping实例来处理所有的控制器的映射
    自动创建的HandlerMapping有两种：SimpleUrlHandlerMapping、BeanNameUrlHandlerMapping，默认的为BeanNameUrlHandlerMapping和RequestMappingHandlerMapping
5. HandlerAdapter
    HandlerAdapter是处理器适配器，主要用于调用找到的Handler来执行逻辑
    如果找不到HandlerAdapter，需要使用默认的配置，DispatcherServlet中至少要有若干HandlerMapping，不能一个没有
6. HandlerExceptionResolver：控制器执行异常解析器，可以使用@ExceptionHandler来实现自动义的异常解析器，配合@ControllerAdvice，完美
    如果没有自定义，则需要至少有一些解析器，采用默认配置
7. RequestToViewNameTranslator：从请求路径解析得视图名称，可以在处理器返回的View为空时使用它根据Request获取viewName。
    需要默认值
8. ViewResolver：视图解析器，用于解析视图，至少要有一个，如果为自定义，使用默认配置
9. FlashMapManager：FlushMap管理器，主要用于重定向传参，如果未自定义，则使用默认配置，
我们来看看DispatcherServlet.properties的内容：
```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```
除了此处的web开发方式，还有Springboot开发方式，貌似就两种。。。下面说说Springboot启动的流程，最后统一说refresh流程。
详见[SpringBoot基础系列-启动流程]()、[Spring基础系列-刷新容器]()

附录：
DispatcherServlet继承结构如下图：
![DispatcherServlet继承结构]()
