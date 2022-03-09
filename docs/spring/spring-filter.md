# Spring Boot 中创建 Filter 的方法

Spring Boot 是通过 [ServeltContextInitializer](./web-initializer) 来初始化内嵌式容器 Web

因此 `Filter` 的创建也是基于 `ServeltContextInitializer` 实现的

具体 `Filter` 的收集会在 `ServeltContextInitializerBeans` 中完成

## Filter Bean

```java
@Component
public class HelloFilter extends HttpFilter {
    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("Hello Filter");
        chain.doFilter(request, response);
    }
}
```

> 这种方式最简单，但是 `urlMapping` 默认为 `/*` ，无法指定其他值

实现代码导读


```java
public class ServletContextInitializerBeans extends AbstractCollection<ServletContextInitializer> {
    // ...

    protected void addAdaptableBeans(ListableBeanFactory beanFactory) {
		MultipartConfigElement multipartConfig = getMultipartConfig(beanFactory);
		addAsRegistrationBean(beanFactory, Servlet.class, new ServletRegistrationBeanAdapter(multipartConfig));
        // 将 Filter 类型的 Bean 包装成 ServletContextInitializer 的子类 FilterRegistrationBean
		addAsRegistrationBean(beanFactory, Filter.class, new FilterRegistrationBeanAdapter());
		for (Class<?> listenerType : ServletListenerRegistrationBean.getSupportedTypes()) {
			addAsRegistrationBean(beanFactory, EventListener.class, (Class<EventListener>) listenerType,
					new ServletListenerRegistrationBeanAdapter());
		}
	}

    // ...
}
```

## FilterRegistrationBean Bean

```java
@Configuration
public class FilterConfiguration {
    public FilterRegistrationBean<HelloFilter> helloFilterRegistrationBean() {
        FilterRegistrationBean<HelloFilter> bean = new FilterRegistrationBean<>(new HelloFilter());
        bean.addUrlPatterns("/*");
        return bean;
    }
}
```

实现代码导读

```java
public class ServletContextInitializerBeans extends AbstractCollection<ServletContextInitializer> {
    // ...

    private void addServletContextInitializerBeans(ListableBeanFactory beanFactory) {
		for (Class<? extends ServletContextInitializer> initializerType : this.initializerTypes) {
            // 查找 ServletContextInitializer 类型的 Bean，FilterRegistrationBean 就是 ServletContextInitializer 的子类，在这里被找到
			for (Entry<String, ? extends ServletContextInitializer> initializerBean : getOrderedBeansOfType(beanFactory,
					initializerType)) {
				addServletContextInitializerBean(initializerBean.getKey(), initializerBean.getValue(), beanFactory);
			}
		}
	}

    // ...
}
```

## @ServletComponentScan + @WebFilter + Filter

创建一个带有 `@WebFilter` 注解的 `Filter` 实现类

```java
@WebFilter(
        urlPatterns = "/*"
)
public class HelloFilter extends HttpFilter {
    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("Hello Filter");
        chain.doFilter(request, response);
    }
}
```

通过 `@ServletComponentScan` 开启 `@WebFilter` 注解的扫描

```java
@SpringBootApplication
@ServletComponentScan
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

`@ServletComponentScan` 通过扫描类路径指定包下的 class 文件获取相应的 Class 对象

!> `@ServletComponentScan` 和 `@ComponentScan` 是互相隔离的，不会从 `BeanFactory` 获取对象

`@ServletComponentScan` 的作用就是注册一个 `BeanFactoryPostProcessor`: `ServletComponentRegisteringPostProcessor`

```java
class ServletComponentRegisteringPostProcessor implements BeanFactoryPostProcessor, ApplicationContextAware {

	private static final List<ServletComponentHandler> HANDLERS;

	static {
		List<ServletComponentHandler> servletComponentHandlers = new ArrayList<>();
		servletComponentHandlers.add(new WebServletHandler());
        // @WebFilter 由 WebFilterHandler 处理
		servletComponentHandlers.add(new WebFilterHandler());
		servletComponentHandlers.add(new WebListenerHandler());
		HANDLERS = Collections.unmodifiableList(servletComponentHandlers);
	}

    // ...

	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		if (isRunningInEmbeddedWebServer()) {
			ClassPathScanningCandidateComponentProvider componentProvider = createComponentProvider();
			for (String packageToScan : this.packagesToScan) {
				scanPackage(componentProvider, packageToScan);
			}
		}
	}

	private void scanPackage(ClassPathScanningCandidateComponentProvider componentProvider, String packageToScan) {
		for (BeanDefinition candidate : componentProvider.findCandidateComponents(packageToScan)) {
			if (candidate instanceof AnnotatedBeanDefinition) {
				for (ServletComponentHandler handler : HANDLERS) {
					handler.handle(((AnnotatedBeanDefinition) candidate),
							(BeanDefinitionRegistry) this.applicationContext);
				}
			}
		}
	}

	// ...

}
```

`ServletComponentRegisteringPostProcessor` 扫描指定包路径的 class 文件，得到带有 `@WebFilter` 注解的 `Filter` 实现类，并构建 `AnnotatedBeanDefinition` 对象，交给 `WebFilterHandler` 处理

```java
class WebFilterHandler extends ServletComponentHandler {

    // ...

	@Override
	public void doHandle(Map<String, Object> attributes, AnnotatedBeanDefinition beanDefinition,
			BeanDefinitionRegistry registry) {
		BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(FilterRegistrationBean.class);
		builder.addPropertyValue("asyncSupported", attributes.get("asyncSupported"));
		builder.addPropertyValue("dispatcherTypes", extractDispatcherTypes(attributes));
		builder.addPropertyValue("filter", beanDefinition);
		builder.addPropertyValue("initParameters", extractInitParameters(attributes));
		String name = determineName(attributes, beanDefinition);
		builder.addPropertyValue("name", name);
		builder.addPropertyValue("servletNames", attributes.get("servletNames"));
		builder.addPropertyValue("urlPatterns", extractUrlPatterns(attributes));
		registry.registerBeanDefinition(name, builder.getBeanDefinition());
	}

	// ...
}
```

`WebFilterHandler` 构建 `FilterRegistrationBean` Bean，后续就是 `FilterRegistrationBean` 的收集了，和上面一致

