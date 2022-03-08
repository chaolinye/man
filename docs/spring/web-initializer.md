# ServletContainerInitializer and ServletContextInitializer

## ServletContainerInitializer

- `javax.servlet.ServletContainerInitializer` 是 servlet 3.0 规范中引入的接口，能够让 web 应用程序在 servelt 容器启动后做一些自定义的操作
    > 主要是提供了编程式地注册 `servlet`、`filter`、`listener`、`context-param` 等的能力

    ```java
    public interface ServletContainerInitializer {

        /**
        * Receives notification during startup of a web application of the classes
        * within the web application that matched the criteria defined via the
        * {@link javax.servlet.annotation.HandlesTypes} annotation.
        *
        * @param c     The (possibly null) set of classes that met the specified
        *              criteria
        * @param ctx   The ServletContext of the web application in which the
        *              classes were discovered
        *
        * @throws ServletException If an error occurs
        */
        void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
    }
    ```

    `ServletContainerInitializer` 基于 `SPI` 实现，因此你需要在你的 jar 包目录下添加 `META-INF/services/javax.servlet.ServletContainerInitializer` 文件，内容就是 `ServletContainerInitializer` 实现类的全限定名。

    Tomcat 中的实现

    ```java
    public class ContextConfig implements LifecycleListener {
        // ...

        protected void processServletContainerInitializers() {

            List<ServletContainerInitializer> detectedScis;
            try {
                WebappServiceLoader<ServletContainerInitializer> loader = new WebappServiceLoader<>(context);
                // 通过 SPI获取
                detectedScis = loader.load(ServletContainerInitializer.class);
            } catch (IOException e) {
                log.error(sm.getString(
                        "contextConfig.servletContainerInitializerFail",
                        context.getName()),
                    e);
                ok = false;
                return;
            }

            for (ServletContainerInitializer sci : detectedScis) {
                initializerClassMap.put(sci, new HashSet<>());

                HandlesTypes ht;
                try {
                    ht = sci.getClass().getAnnotation(HandlesTypes.class);
                } catch (Exception e) {
                    if (log.isDebugEnabled()) {
                        log.info(sm.getString("contextConfig.sci.debug",
                                sci.getClass().getName()),
                                e);
                    } else {
                        log.info(sm.getString("contextConfig.sci.info",
                                sci.getClass().getName()));
                    }
                    continue;
                }
                if (ht == null) {
                    continue;
                }
                Class<?>[] types = ht.value();
                if (types == null) {
                    continue;
                }

                for (Class<?> type : types) {
                    if (type.isAnnotation()) {
                        handlesTypesAnnotations = true;
                    } else {
                        handlesTypesNonAnnotations = true;
                    }
                    Set<ServletContainerInitializer> scis =
                            typeInitializerMap.get(type);
                    if (scis == null) {
                        scis = new HashSet<>();
                        typeInitializerMap.put(type, scis);
                    }
                    scis.add(sci);
                }
            }
        }

        // ...
    }
    ```

    > `ContextConfig` 详见 [Tomcat 教程](../tomcat/tomcat-source)

- `org.springframework.web.SpringServletContainerInitializer` 是 spring mvc 提供的 `ServletContainerInitializer` 实现

    ```java
    @HandlesTypes(WebApplicationInitializer.class)
    public class SpringServletContainerInitializer implements ServletContainerInitializer {

        @Override
        public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
                throws ServletException {

            List<WebApplicationInitializer> initializers = Collections.emptyList();

            if (webAppInitializerClasses != null) {
                initializers = new ArrayList<>(webAppInitializerClasses.size());
                for (Class<?> waiClass : webAppInitializerClasses) {
                    // Be defensive: Some servlet containers provide us with invalid classes,
                    // no matter what @HandlesTypes says...
                    if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
                            WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                        try {
                            initializers.add((WebApplicationInitializer)
                                    ReflectionUtils.accessibleConstructor(waiClass).newInstance());
                        }
                        catch (Throwable ex) {
                            throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
                        }
                    }
                }
            }

            if (initializers.isEmpty()) {
                servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
                return;
            }

            servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
            AnnotationAwareOrderComparator.sort(initializers);
            for (WebApplicationInitializer initializer : initializers) {
                initializer.onStartup(servletContext);
            }
        }

    }
    ```

    `@HandlesTypes` 注解也是由 `ContextConfig` 处理，会从类路径中找到 `@HandlesTypes` 指定的类传入 `ServletContainerInitializer` 实现类的 `onStartup` 方法

- `org.springframework.web.WebApplicationInitializer` 是 `SpringServletContainerInitializer` 调用的初始化器

    Spring 中 `SpringServletContainerInitializer` 只是一个空壳，真正的初始化动作是交给 `WebApplicationInitializer` 来完成的

    ```java
    public interface WebApplicationInitializer {
        /**
        * Configure the given {@link ServletContext} with any servlets, filters, listeners
        * context-params and attributes necessary for initializing this web application. See
        * examples {@linkplain WebApplicationInitializer above}.
        * @param servletContext the {@code ServletContext} to initialize
        * @throws ServletException if any call against the given {@code ServletContext}
        * throws a {@code ServletException}
        */
        void onStartup(ServletContext servletContext) throws ServletException;
    }
    ```

    为了简化使用，实践开发中一般都是基于 Spring 提供的 `AbstractAnnotationConfigDispatcherServletInitializer` 来开发 `WebApplicationInitializer`

- `org.springframework.boot.web.servlet.support.SpringBootServletInitializer`: Spring Boot 提供的 `WebApplicationInitializer` 实现，适用于 Spring Boot 使用 **外部容器** 的场景

    Spring Boot 一般使用内嵌的 Servlet，但也提供了使用外部容器的能力

    > Spring Boot 打包成 war 包，部署到外部容器

    `SpringBootServletInitializer` 就是 Spring Boot 提供的 `WebApplicationInitializer` 实现，用于 **外部化容器** 初始化 Spring Boot 相关能力

    基本用法:

    ```java
    @SpringBootApplication
    public class DemoApplication extends SpringBootServletInitializer {
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(DemoApplication.class);
        }
    }
    ```

    兼容内嵌式和外部式

    ```java
    @SpringBootApplication
    public class DemoApplication extends SpringBootServletInitializer {

        // 使用内嵌容器时，main 方法入口
        public static void main(String[] args) {
            SpringApplication.run(DemoApplication.class, args);
        }

        // 使用内嵌容器时，不会被调用
        // 外部容器时被调用
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
            return builder.sources(DemoApplication.class);
        }
    }
    ```

## ServletContextInitializer

    Spring Boot 的内嵌 Tomcat 不会通过 SPI 来寻找 `ServletContainerInitializer`，而是硬编码了一个实现 `org.springframework.boot.web.embedded.tomcat.TomcatStarter`

    ```java
    public class TomcatServletWebServerFactory extends AbstractServletWebServerFactory implements ConfigurableTomcatWebServerFactory, ResourceLoaderAware {
        // ...

        protected void configureContext(Context context, ServletContextInitializer[] initializers) {
            TomcatStarter starter = new TomcatStarter(initializers);
            if (context instanceof TomcatEmbeddedContext) {
                TomcatEmbeddedContext embeddedContext = (TomcatEmbeddedContext) context;
                embeddedContext.setStarter(starter);
                embeddedContext.setFailCtxIfServletStartFails(true);
            }
            context.addServletContainerInitializer(starter, NO_CLASSES);

            // ...

            new DisableReferenceClearingContextCustomizer().customize(context);
            for (String webListenerClassName : getWebListenerClassNames()) {
                context.addApplicationListener(webListenerClassName);
            }
            for (TomcatContextCustomizer customizer : this.tomcatContextCustomizers) {
                customizer.customize(context);
            }
        }
    }
    ```

    ```java
    class TomcatStarter implements ServletContainerInitializer {

        private static final Log logger = LogFactory.getLog(TomcatStarter.class);

        private final ServletContextInitializer[] initializers;

        private volatile Exception startUpException;

        TomcatStarter(ServletContextInitializer[] initializers) {
            this.initializers = initializers;
        }

        @Override
        public void onStartup(Set<Class<?>> classes, ServletContext servletContext) throws ServletException {
            try {
                for (ServletContextInitializer initializer : this.initializers) {
                    initializer.onStartup(servletContext);
                }
            }
            catch (Exception ex) {
                this.startUpException = ex;
                // Prevent Tomcat from logging and re-throwing when we know we can
                // deal with it in the main thread, but log for information here.
                if (logger.isErrorEnabled()) {
                    logger.error("Error starting Tomcat context. Exception: " + ex.getClass().getName() + ". Message: "
                            + ex.getMessage());
                }
            }
        }

        Exception getStartUpException() {
            return this.startUpException;
        }

    }
    ```

    而 `TomcatStarter` 是通过传入的 `ServletContextInitializer` 实例进行初始化操作

    > `ServletContextInitializer` 初始化就是向 `ServletContext` 添加 `Servlet` `Filter` `Listener` 等组件

    > 如果是 Spring Boot 外部容器场景，ServletContext 实例是 SpringBootWebApplicationInitializer 传入的外部实例

    ```java
    public class ServletWebServerApplicationContext extends GenericWebApplicationContext
		implements ConfigurableWebServerApplicationContext {

            private void createWebServer() {
                WebServer webServer = this.webServer;
                ServletContext servletContext = getServletContext();
                if (webServer == null && servletContext == null) {
                    StartupStep createWebServer = this.getApplicationStartup().start("spring.boot.webserver.create");
                    ServletWebServerFactory factory = getWebServerFactory();
                    createWebServer.tag("factory", factory.getClass().toString());
                    this.webServer = factory.getWebServer(getSelfInitializer());
                    createWebServer.end();
                    getBeanFactory().registerSingleton("webServerGracefulShutdown",
                            new WebServerGracefulShutdownLifecycle(this.webServer));
                    getBeanFactory().registerSingleton("webServerStartStop",
                            new WebServerStartStopLifecycle(this, this.webServer));
                }
                // 外部容器场景
                else if (servletContext != null) {
                    try {
                        getSelfInitializer().onStartup(servletContext);
                    }
                    catch (ServletException ex) {
                        throw new ApplicationContextException("Cannot initialize servlet context", ex);
                    }
                }
                initPropertySources();
            }

            private org.springframework.boot.web.servlet.ServletContextInitializer getSelfInitializer() {
                return this::selfInitialize;
            }

            private void selfInitialize(ServletContext servletContext) throws ServletException {
                prepareWebApplicationContext(servletContext);
                registerApplicationScope(servletContext);
                WebApplicationContextUtils.registerEnvironmentBeans(getBeanFactory(), servletContext);
                for (ServletContextInitializer beans : getServletContextInitializerBeans()) {
                    beans.onStartup(servletContext);
                }
            }

            protected Collection<ServletContextInitializer> getServletContextInitializerBeans() {
                // ServletContextInitializerBeans 是个 ServletContextInitializer 集合，组合模式
                return new ServletContextInitializerBeans(getBeanFactory());
            }
    }
    ```

    `ServletContextInitializerBeans` 是个 `ServletContextInitializer` 集合，在 `BeanFactory` 中查找 `ServletContextInitializer` 类型的 bean 放到集合中

    Spring Boot 提供了不少 `ServletContextInitializer` 的实现类，可以基于这些实现类来简化自定义 `ServletContextInitializer` Bean 的开发

    ![](../images/spring-servletcontextinitializer)

    `ServletContextInitializer` 的优势是由 Spring 进行管理的（本身属于 Spring Bean），可以结合 Spring 做更多的事情

## 实践建议

- Spring Boot 嵌入容器场景基于 Spring Boot 提供的 `ServletContextInitializer` 的实现类进行 web 初始化
- Spring Boot 外部容器场景基于 `SpringBootServletInitializer` + `ServletContextInitializer` 进行 web 初始化
- Spring 场景基于 `AbstractAnnotationConfigDispatcherServletInitializer` 进行 web 初始化

## References

- [ServletContainerInitializer、ServletContextInitializer有什么区别](https://segmentfault.com/a/1190000022763768)