## SpringMVC配置web.xml文件

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/classes/spring/applicationContext-*.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>

<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 加载springmvc配置 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 
			 配置文件的地址 如果不配置contextConfigLocation， 
			 默认查找的配置文件名称classpath下的：servlet名称+"-serlvet.xml"
			 即：springmvc-serlvet.xml 
		-->
        <param-value>classpath:spring/springmvc.xml</param-value>
    </init-param>

</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!-- 
		可以配置/ ，此工程 所有请求全部由springmvc解析，此种方式可以实现 RESTful方式，
		需要特殊处理对静态文件的解析不能由springmvc解析 
   		可以配置*.do或*.action，所有请求的url扩展名为.do或.action由springmvc解析，
		此种方法常用 不可以/*，如果配置/*，返回jsp也由springmvc解析，这是不对的。 
	-->
    <url-pattern>*.action</url-pattern>
</servlet-mapping>

<!-- restful的配置 -->
<servlet>
    <servlet-name>springmvc_rest</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 加载springmvc配置 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 
			配置文件的地址 如果不配置contextConfigLocation， 
			默认查找的配置文件名称classpath下的：servlet名称+"-serlvet.xml"
			即：springmvc-serlvet.xml 
		-->
        <param-value>classpath:spring/springmvc.xml</param-value>
    </init-param>

</servlet>
<servlet-mapping>
    <servlet-name>springmvc_rest</servlet-name>
    <!-- rest方式配置为 / -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

## org.springframework.web.context.ContextLoaderListener

ContextLoaderListener是javax.servlet.ServletContextListener的实现类，即Spring在ServletContext启动时，会调用javax.servlet.ServletContextListener#contextInitialized方法，销毁时会调用javax.servlet.ServletContextListener#contextDestroyed方法。

```java
// org.springframework.web.context.ContextLoaderListener#contextInitialized
public void contextInitialized(ServletContextEvent event) {
    initWebApplicationContext(event.getServletContext());
}

public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    if (servletContext.getAttribute(
        WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
        throw new IllegalStateException("...");
    }
    if (this.context == null) {
        this.context = createWebApplicationContext(servletContext);
    }
    if (this.context instanceof ConfigurableWebApplicationContext) {
        ConfigurableWebApplicationContext cwac = 
            (ConfigurableWebApplicationContext) this.context;
        if (!cwac.isActive()) {
            if (cwac.getParent() == null) {
                ApplicationContext parent = loadParentContext(servletContext);
                cwac.setParent(parent);
            }
            configureAndRefreshWebApplicationContext(cwac, servletContext);
        }
    }
    servletContext.setAttribute(
        WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

    ClassLoader ccl = Thread.currentThread().getContextClassLoader();
    if (ccl == ContextLoader.class.getClassLoader()) {
        currentContext = this.context;
    } else if (ccl != null) {
        currentContextPerThread.put(ccl, this.context);
    }
}
// ContextLoader#configureAndRefreshWebApplicationContext
protected void configureAndRefreshWebApplicationContext(
    ConfigurableWebApplicationContext wac, ServletContext sc) {
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
        String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
        if (idParam != null) {
            wac.setId(idParam);
        } else {
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                      ObjectUtils.getDisplayString(sc.getContextPath()));
        }
    }
    wac.setServletContext(sc);
    String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
    if (configLocationParam != null) {
        wac.setConfigLocation(configLocationParam);
    }
    ConfigurableEnvironment env = wac.getEnvironment();
    if (env instanceof ConfigurableWebEnvironment) {
        ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
    }
    customizeContext(sc, wac);
    // 执行AbstractApplicationContext#refresh方法
    wac.refresh();
}
```

通过上面的代码可知，ContextLoaderListener#initWebApplicationContext方法完成了Spring容器的启动和解析配置文件。在ContextLoader#configureAndRefreshWebApplicationContext方法中，获取key为contextConfigLocation的value值（bean的配置文件）。

