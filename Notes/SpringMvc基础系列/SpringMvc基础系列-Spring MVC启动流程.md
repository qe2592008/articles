# SpringMvc基础系列-Spring MVC启动流程
## 缘起

## 流程
流程从HttpServletBean的init方法开始说起：
### HttpServletBean
HttpServletBean继承自HttpServlet抽象类，实现了其中的init方法，用于开启整个流程。
```java
public abstract class HttpServletBean 
    extends HttpServlet 
    implements EnvironmentCapable, EnvironmentAware {
    //...
    @Override
    public final void init() throws ServletException {

        // 1-将Servlet配置参数（init-param参数）设置到
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
        if (!pvs.isEmpty()) {
            try {
                BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
                ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
                bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
                initBeanWrapper(bw);
                bw.setPropertyValues(pvs, true);
            }
            catch (BeansException ex) {
                if (logger.isErrorEnabled()) {
                    logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
                }
                throw ex;
            }
        }

        // 2-尤其子类FrameworkServlet来初始化Servlet实例
        initServletBean();
    }
    //...
}
```
第一步的目的是为了将Servlet的初始化配置参数保存起来，这些初始化参数都已经被加载到ServletConfig里了，这里遍历其中的初始化参数，将其保存到MutablePropertyValues里面。
第二步的目的是执行下一步流程，下一步流程不再HttpServletBean中定义，由其子类FrameworkServlet实现。
### FrameworkServlet

```java
public abstract class FrameworkServlet 
    extends HttpServletBean 
    implements ApplicationContextAware { 
    //...
    @Override
    protected final void initServletBean() throws ServletException {
        getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
        if (logger.isInfoEnabled()) {
            logger.info("Initializing Servlet '" + getServletName() + "'");
        }
        long startTime = System.currentTimeMillis();

        try {
            this.webApplicationContext = initWebApplicationContext();
            initFrameworkServlet();
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
    //...
    protected WebApplicationContext initWebApplicationContext() {
        WebApplicationContext rootContext =
                WebApplicationContextUtils.getWebApplicationContext(getServletContext());
        WebApplicationContext wac = null;

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
            wac = findWebApplicationContext();
        }
        if (wac == null) {
            // No context instance is defined for this servlet -> create a local one
            wac = createWebApplicationContext(rootContext);
        }

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
}
```
### DispatcherServlet

## 