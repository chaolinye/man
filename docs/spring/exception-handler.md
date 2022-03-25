# @ExceptionHandler 源码导读

## 基本用法

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {

	/**
	 * Exceptions handled by the annotated method. If empty, will default to any
	 * exceptions listed in the method argument list.
	 */
	Class<? extends Throwable>[] value() default {};

}
```

`@ExceptionHandler` 注解作用于方法上，使该方法成为 Spring MVC 的异常处理器

作用范围：

- `@Controller` Bean 的方法，只处理当前 `Controller` 的相应异常

- `@ControllerAdvice` Bean 的方法，处理 `@ControllerAdvice` 指定的 `Controller` 的相应异常

> `@ControllerAdvice` 未指定 `Controller`, 就是作用于全部 `Controller`

两个模板例子:

```java
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

```java
@ControllerAdvice
public class ExampleAdvice1 {
    
    @ExceptionHandler({FileSystemException.class, RemoteException.class})
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

## 源码导读

### HandlerExceptionResolver 调用 @ExceptionHandler 方法处理异常

负责处理 `@ExceptionHandler` 的 `HanlderExceptionResolver` 使 `ExceptionHandlerExceptionResolver`

```java
public class ExceptionHandlerExceptionResolver extends AbstractHandlerMethodExceptionResolver
		implements ApplicationContextAware, InitializingBean {
    // ...

    @Override
	@Nullable
	protected ModelAndView doResolveHandlerMethodException(HttpServletRequest request,
			HttpServletResponse response, @Nullable HandlerMethod handlerMethod, Exception exception) {
        // 根据方法对应的 controller 以及异常类型找到匹配的异常处理方法
		ServletInvocableHandlerMethod exceptionHandlerMethod = getExceptionHandlerMethod(handlerMethod, exception);
        // 省略后续对方法的执行
    }

    protected ServletInvocableHandlerMethod getExceptionHandlerMethod(
			@Nullable HandlerMethod handlerMethod, Exception exception) {

		Class<?> handlerType = null;

		if (handlerMethod != null) {
			// Local exception handler methods on the controller class itself.
			// To be invoked through the proxy, even in case of an interface-based proxy.
			handlerType = handlerMethod.getBeanType();
			ExceptionHandlerMethodResolver resolver = this.exceptionHandlerCache.get(handlerType);
			if (resolver == null) {
				resolver = new ExceptionHandlerMethodResolver(handlerType);
				this.exceptionHandlerCache.put(handlerType, resolver);
			}
			Method method = resolver.resolveMethod(exception);
			if (method != null) {
				return new ServletInvocableHandlerMethod(handlerMethod.getBean(), method, this.applicationContext);
			}
			// For advice applicability check below (involving base packages, assignable types
			// and annotation presence), use target class instead of interface-based proxy.
			if (Proxy.isProxyClass(handlerType)) {
				handlerType = AopUtils.getTargetClass(handlerMethod.getBean());
			}
		}

        // 在 ControllerAdvice 中查找匹配 Controller 类型和异常类型的 @ExceptionHandler 方法
		for (Map.Entry<ControllerAdviceBean, ExceptionHandlerMethodResolver> entry : this.exceptionHandlerAdviceCache.entrySet()) {
			ControllerAdviceBean advice = entry.getKey();
			if (advice.isApplicableToBeanType(handlerType)) {
				ExceptionHandlerMethodResolver resolver = entry.getValue();
				Method method = resolver.resolveMethod(exception);
				if (method != null) {
					return new ServletInvocableHandlerMethod(advice.resolveBean(), method, this.applicationContext);
				}
			}
		}

		return null;
	}
}
```

> `ExceptionHandlerExceptionResolver` 从 `Controller` 以及 `ControllerAdvice` 中查找匹配的 `@ExceptionHandler` 方法，然后执行

`ControllerAdvice` Bean 的收集

```java
public class ExceptionHandlerExceptionResolver extends AbstractHandlerMethodExceptionResolver
		implements ApplicationContextAware, InitializingBean {
    // ...

    @Override
	public void afterPropertiesSet() {
		// Do this first, it may add ResponseBodyAdvice beans
		initExceptionHandlerAdviceCache();

		if (this.argumentResolvers == null) {
			List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
			this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
		}
		if (this.returnValueHandlers == null) {
			List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
			this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
		}
	}

	private void initExceptionHandlerAdviceCache() {
		if (getApplicationContext() == null) {
			return;
		}

        // 获取 ControllerAdvice Bean
		List<ControllerAdviceBean> adviceBeans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());
		for (ControllerAdviceBean adviceBean : adviceBeans) {
			Class<?> beanType = adviceBean.getBeanType();
			if (beanType == null) {
				throw new IllegalStateException("Unresolvable type for ControllerAdviceBean: " + adviceBean);
			}
            // 解析 @ExceptionAdvice 方法
			ExceptionHandlerMethodResolver resolver = new ExceptionHandlerMethodResolver(beanType);
			if (resolver.hasExceptionMappings()) {
				this.exceptionHandlerAdviceCache.put(adviceBean, resolver);
			}
			if (ResponseBodyAdvice.class.isAssignableFrom(beanType)) {
				this.responseBodyAdvice.add(adviceBean);
			}
		}

	}
    // ...
}
```

由上知，无论是 Controller 还是 ControllerAdvice 都是使用 `ExceptionHandlerMethodResolver` 解析和提取 Bean 中的 `@ExceptionHandler` 方法

```java
public class ExceptionHandlerMethodResolver {

	/**
	 * A filter for selecting {@code @ExceptionHandler} methods.
	 */
	public static final MethodFilter EXCEPTION_HANDLER_METHODS = method ->
			AnnotatedElementUtils.hasAnnotation(method, ExceptionHandler.class);

    // 提取 @ExceptionHandler 方法，建立 Exception -> method 的映射
    public ExceptionHandlerMethodResolver(Class<?> handlerType) {
		for (Method method : MethodIntrospector.selectMethods(handlerType, EXCEPTION_HANDLER_METHODS)) {
			for (Class<? extends Throwable> exceptionType : detectExceptionMappings(method)) {
				addExceptionMapping(exceptionType, method);
			}
		}
	}

    private List<Class<? extends Throwable>> detectExceptionMappings(Method method) {
		List<Class<? extends Throwable>> result = new ArrayList<>();
		detectAnnotationExceptionMappings(method, result);
		if (result.isEmpty()) {
			for (Class<?> paramType : method.getParameterTypes()) {
				if (Throwable.class.isAssignableFrom(paramType)) {
					result.add((Class<? extends Throwable>) paramType);
				}
			}
		}
		if (result.isEmpty()) {
			throw new IllegalStateException("No exception types mapped to " + method);
		}
		return result;
	}

	private void detectAnnotationExceptionMappings(Method method, List<Class<? extends Throwable>> result) {
		ExceptionHandler ann = AnnotatedElementUtils.findMergedAnnotation(method, ExceptionHandler.class);
		Assert.state(ann != null, "No ExceptionHandler annotation");
		result.addAll(Arrays.asList(ann.value()));
	}

	private void addExceptionMapping(Class<? extends Throwable> exceptionType, Method method) {
		Method oldMethod = this.mappedMethods.put(exceptionType, method);
		if (oldMethod != null && !oldMethod.equals(method)) {
			throw new IllegalStateException("Ambiguous @ExceptionHandler method mapped for [" +
					exceptionType + "]: {" + oldMethod + ", " + method + "}");
		}
	}

    // 根据异常类型获取对应的 Method
    @Nullable
	public Method resolveMethodByExceptionType(Class<? extends Throwable> exceptionType) {
		Method method = this.exceptionLookupCache.get(exceptionType);
		if (method == null) {
			method = getMappedMethod(exceptionType);
			this.exceptionLookupCache.put(exceptionType, method);
		}
		return (method != NO_MATCHING_EXCEPTION_HANDLER_METHOD ? method : null);
	}

	/**
	 * Return the {@link Method} mapped to the given exception type, or
	 * {@link #NO_MATCHING_EXCEPTION_HANDLER_METHOD} if none.
	 */
	private Method getMappedMethod(Class<? extends Throwable> exceptionType) {
		List<Class<? extends Throwable>> matches = new ArrayList<>();
		for (Class<? extends Throwable> mappedException : this.mappedMethods.keySet()) {
			if (mappedException.isAssignableFrom(exceptionType)) {
				matches.add(mappedException);
			}
		}
		if (!matches.isEmpty()) {
			if (matches.size() > 1) {
				matches.sort(new ExceptionDepthComparator(exceptionType));
			}
			return this.mappedMethods.get(matches.get(0));
		}
		else {
			return NO_MATCHING_EXCEPTION_HANDLER_METHOD;
		}
	}
```

### HandlerExceptionResovler 的执行时机

```java
public class DispatcherServlet extends FrameworkServlet {

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		// 只显示了主要代码

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// 执行请求
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                // ...
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            // 调用异常处理器处理异常
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
	}

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
                // 处理异常
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// ...
	}

    protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
			@Nullable Object handler, Exception ex) throws Exception {

		// Check registered HandlerExceptionResolvers...
		ModelAndView exMv = null;
		if (this.handlerExceptionResolvers != null) {
            // 遍历调用 HandlerExceptionResolver 处理异常
			for (HandlerExceptionResolver resolver : this.handlerExceptionResolvers) {
				exMv = resolver.resolveException(request, response, handler, ex);
				if (exMv != null) {
					break;
				}
			}
		}
		
        // ...
	}
}
```

!> 根据执行时机，HandlerExceptionResolver 可以处理 Spring MVC Bean 中产生的异常（Controller 方法内，切面，拦截器等），但无法拦截 Filter 产生的异常

### HandlerExceptionResolver 的创建

> 根据经验 `DispatcherServlet` 的组件多由 `WebMvcConfigurationSupport` 提供的，`HandlerExceptionResolver` 也是如此

```java
public class WebMvcConfigurationSupport implements ApplicationContextAware, ServletContextAware {
    @Bean
	public HandlerExceptionResolver handlerExceptionResolver(
			@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager) {
		List<HandlerExceptionResolver> exceptionResolvers = new ArrayList<>();
        // 添加自定义 HandlerExceptionResolver
		configureHandlerExceptionResolvers(exceptionResolvers);
		if (exceptionResolvers.isEmpty()) {
            // 添加默认的 HandlerExceptionResolver
			addDefaultHandlerExceptionResolvers(exceptionResolvers, contentNegotiationManager);
		}
        // 添加自定义 HandlerExceptionResolver
		extendHandlerExceptionResolvers(exceptionResolvers);
        // 组合模式
		HandlerExceptionResolverComposite composite = new HandlerExceptionResolverComposite();
		composite.setOrder(0);
		composite.setExceptionResolvers(exceptionResolvers);
		return composite;
	}

    protected final void addDefaultHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers,
			ContentNegotiationManager mvcContentNegotiationManager) {
        // 这里创建 ExceptionHandlerExceptionResolver
		ExceptionHandlerExceptionResolver exceptionHandlerResolver = createExceptionHandlerExceptionResolver();
		exceptionHandlerResolver.setContentNegotiationManager(mvcContentNegotiationManager);
		exceptionHandlerResolver.setMessageConverters(getMessageConverters());
		exceptionHandlerResolver.setCustomArgumentResolvers(getArgumentResolvers());
		exceptionHandlerResolver.setCustomReturnValueHandlers(getReturnValueHandlers());
		if (jackson2Present) {
			exceptionHandlerResolver.setResponseBodyAdvice(
					Collections.singletonList(new JsonViewResponseBodyAdvice()));
		}
		if (this.applicationContext != null) {
			exceptionHandlerResolver.setApplicationContext(this.applicationContext);
		}
		exceptionHandlerResolver.afterPropertiesSet();
		exceptionResolvers.add(exceptionHandlerResolver);

		ResponseStatusExceptionResolver responseStatusResolver = new ResponseStatusExceptionResolver();
		responseStatusResolver.setMessageSource(this.applicationContext);
		exceptionResolvers.add(responseStatusResolver);

		exceptionResolvers.add(new DefaultHandlerExceptionResolver());
	}

    protected ExceptionHandlerExceptionResolver createExceptionHandlerExceptionResolver() {
		return new ExceptionHandlerExceptionResolver();
	}
}
```

## 学习资料

- [官方文档](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)

