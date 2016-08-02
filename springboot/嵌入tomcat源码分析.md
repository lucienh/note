# tomcat


在启动`spring boot`工程时利用`@SpringBootApplication`注解，该注解启动`@EnableAutoConfiguration`自动配置，加载`META-INF/spring.factories`文件

```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
... 
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
 ...
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration

```

其中`EmbeddedServletContainerAutoConfiguration`被加载


```java

@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
public class EmbeddedServletContainerAutoConfiguration {
  ...
}
```

`@ConditionalOnWebApplication`注解表明只有在`web`环境下才会创建容器相关信息，因此应用无需容器则使用

`new SpringApplicationBuilder(Xxx.class).web(false).run(args)`实现。

<hr>

`TomcatEmbeddedServletContainerFactory.java`

```java

	@Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

	}

```
优先创建`TomcatEmbeddedServletContainerFactory`,由于存在`@ConditionalOnMissingBean`因此优先使用用户自定义的`EmbeddedServletContainerFactory`,

此时创建了工厂，但是`tomcat`是如何启动的呢？


在`spring boot`中常使用的上下文为`AnnotationConfigEmbeddedWebApplicationContext`,通过前面的文章也知道加载`BeanDefinition`是在
`AbstractApplicationContext#refresh()`方法中，具体详细的实现可以参考这里[1](https://github.com/liaokailin/spring-boot) [2](https://github.com/liaokailin/spring-framework),有阅读源码的备注信息.

```java

public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {  //同步锁
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {

				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);  //为BeanFactory设置后处理器，用依拓展

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);  //执行BeanFactoryPostProcessor,因此在执行BeanFactoryPostProcessor子类时，bean是没有被实例化的

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();  //释放缓存
			}
		}
	}
```


其中的`onRefresh();`交由子类实现`EmbeddedWebApplicationContext`（`public class AnnotationConfigEmbeddedWebApplicationContext
		extends EmbeddedWebApplicationContext`）

```java
	@Override
	protected void onRefresh() {
		super.onRefresh();
		try {
			createEmbeddedServletContainer();
		}
		catch (Throwable ex) {
			throw new ApplicationContextException("Unable to start embedded container",
					ex);
		}
	}

```

调用方法

```java
 /**
     * 创建内嵌容器
     */
	private void createEmbeddedServletContainer() {
		EmbeddedServletContainer localContainer = this.embeddedServletContainer;
		ServletContext localServletContext = getServletContext();
		if (localContainer == null && localServletContext == null) {
			EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();  //获取自动加载的工厂
			this.embeddedServletContainer = containerFactory
					.getEmbeddedServletContainer(getSelfInitializer());
		}
		else if (localServletContext != null) {
			try {
				getSelfInitializer().onStartup(localServletContext);
			}
			catch (ServletException ex) {
				throw new ApplicationContextException("Cannot initialize servlet context",
						ex);
			}
		}
		initPropertySources();
	}
```


`getEmbeddedServletContainerFactory` 获取容器工厂，通过

```java
containerFactory.getEmbeddedServletContainer(getSelfInitializer());
```
创建一个内建容器，这里的`getSelfInitializer()`返回一个`ServletContextInitializer`,其实现为

```java
return new ServletContextInitializer() {
			@Override
  prepareContext(tomcat.getHost(), initializers);			public void onStartup(ServletContext servletContext) throws ServletException {
				selfInitialize(servletContext);
			}
		}; customizer.customize(bean)
```
其中的`selfInitialize(servletContext);`等后续再回过来看,这里很关键。


继续看`containerFactory.getEmbeddedServletContainer(getSelfInitializer());`方法，默认调用`TomcatEmbeddedServletContainerFactory`中的方法：
```java

@Override
	public EmbeddedServletContainer getEmbeddedServletContainer(
			ServletContextInitializer... initializers) {
		Tomcat tomcat = new Tomcat();  //构建tomcat实例
		File baseDir = (this.baseDirectory != null ? this.baseDirectory
				: createTempDir("tomcat"));
		tomcat.setBaseDir(baseDir.getAbsolutePath());
		Connector connector = new Connector(this.protocol);
		tomcat.getService().addConnector(connector);
		customizeConnector(connector);
		tomcat.setConnector(connector);
		tomcat.getHost().setAutoDeploy(false);
		tomcat.getEngine().setBackgroundProcessorDelay(-1);
		for (Connector additionalConnector : this.additionalTomcatConnectors) {
			tomcat.getService().addConnector(additionalConnector);
		}
		prepareContext(tomcat.getHost(), initializers);
		return getTomcatEmbeddedServletContainer(tomcat);
	}
```

首先构造一个`tomcat`实例，设置connector信息，在`customizeConnector(connector)`中会设置端口等其他配置信息，那么有个疑问来了，`tomcat`中的配置信息是怎么加载的？ 


那么又要回到`EmbeddedServletContainerAutoConfiguration`类，其申明`@Import(BeanPostProcessorsRegistrar.class)`，那么需要看下`BeanPostProcessorsRegistrar`,其实现很简单

```java

public static class BeanPostProcessorsRegistrar
			implements ImportBeanDefinitionRegistrar, BeanFactoryAware {

		private ConfigurableListableBeanFactory beanFactory;

		/**
		 * 在解析import的时候自动绑定各种aware
		 * @param beanFactory
		 * @throws BeansException
		 */
		@Override
		public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
			if (beanFactory instanceof ConfigurableListableBeanFactory) {
				this.beanFactory = (ConfigurableListableBeanFactory) beanFactory;
			}
		}

		@Override
		public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
				BeanDefinitionRegistry registry) {
			if (this.beanFactory == null) {
				return;
			}
			if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(
					EmbeddedServletContainerCustomizerBeanPostProcessor.class, true,
					false))) {
				registry.registerBeanDefinition(
						"embeddedServletContainerCustomizerBeanPostProcessor",
						new RootBeanDefinition(
								EmbeddedServletContainerCustomizerBeanPostProcessor.class));

			}
			if (ObjectUtils.isEmpty(this.beanFactory.getBeanNamesForType(
					ErrorPageRegistrarBeanPostProcessor.class, true, false))) {
				registry.registerBeanDefinition("errorPageRegistrarBeanPostProcessor",
						new RootBeanDefinition(
								ErrorPageRegistrarBeanPostProcessor.class));

			}
		}

	}
```

在`registerBeanDefinitions`注册了一个名称为`embeddedServletContainerCustomizerBeanPostProcessor`的bean,其类型为`EmbeddedServletContainerCustomizerBeanPostProcessor`（在自定义`Beandefinition`时可以采用`BeanDefinitionBuilder`工具类）,该`bean`为一个`bean`的后处理器

```java

public class EmbeddedServletContainerCustomizerBeanPostProcessor
		implements BeanPostProcessor, ApplicationContextAware {
}
```
覆写`postProcessBeforeInitialization`和`postProcessAfterInitialization`方法，其大概的调用顺序可以简单理解为：

```
 调用顺序
  BeanFactoryPostProcessor#postProcessBeanFactory ->
  构造方法 ->
  ApplicationContextAware#setApplicationContext ->
  BeanPostProcessor#postProcessBeforeInitialization->
  PostConstruct注解方法 ->
  InitializingBean#afterPropertiesSet ->
  BeanPostProcessor#postProcessAfterInitialization
 
```


```java

@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName)
			throws BeansException {
		if (bean instanceof ConfigurableEmbeddedServletContainer) {
			postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
		}
		return bean;
	}
```

调用`postProcessBeforeInitialization`方法

```java

private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
		for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {  //
			customizer.customize(bean);
		}
	}
```

需要获取`getCustomizers`


```java

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
		if (this.customizers == null) {
			// Look up does not include the parent context
			this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
					this.applicationContext
							.getBeansOfType(EmbeddedServletContainerCustomizer.class,
									false, false)
							.values());
			Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
			this.customizers = Collections.unmodifiableList(this.customizers);
		}
		return this.customizers;
	}
```

获取`EmbeddedServletContainerCustomizer`类型的`bean`，调用其`customizer.customize(bean)`,那么来看下`ServerProperties`的实现

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties
		implements EmbeddedServletContainerCustomizer, EnvironmentAware, Ordered {

}
```

其`customize`方法为
```java
public void customize(ConfigurableEmbeddedServletContainer container) {
		if (getPort() != null) {
			container.setPort(getPort());
		}
		if (getAddress() != null) {
			container.setAddress(getAddress());
		}
		if (getContextPath() != null) {
			container.setContextPath(getContextPath());
		}
		if (getDisplayName() != null) {
			container.setDisplayName(getDisplayName());
		}
		if (getSession().getTimeout() != null) {
			container.setSessionTimeout(getSession().getTimeout());
		}
		container.setPersistSession(getSession().isPersistent());
		container.setSessionStoreDir(getSession().getStoreDir());
		if (getSsl() != null) {
			container.setSsl(getSsl());
		}
		if (getJspServlet() != null) {
			container.setJspServlet(getJspServlet());
		}
		if (getCompression() != null) {
			container.setCompression(getCompression());
		}
		container.setServerHeader(getServerHeader());
		if (container instanceof TomcatEmbeddedServletContainerFactory) {
			getTomcat().customizeTomcat(this,
					(TomcatEmbeddedServletContainerFactory) container);
		}
		if (container instanceof JettyEmbeddedServletContainerFactory) {
			getJetty().customizeJetty(this,
					(JettyEmbeddedServletContainerFactory) container);
		}

		if (container instanceof UndertowEmbeddedServletContainerFactory) {
			getUndertow().customizeUndertow(this,
					(UndertowEmbeddedServletContainerFactory) container);
		}
		container.addInitializers(new SessionConfiguringInitializer(this.session));
		container.addInitializers(new InitParameterConfiguringServletContextInitializer(
				getContextParameters()));
	}

```
为`container`设置了各种属性值，至此，内嵌容器属性赋值解释完毕，继续看前面的`prepareContext(tomcat.getHost(), initializers);`方法


```java

protected void prepareContext(Host host, ServletContextInitializer[] initializers) {
		File docBase = getValidDocumentRoot();
		docBase = (docBase != null ? docBase : createTempDir("tomcat-docbase"));
		TomcatEmbeddedContext context = new TomcatEmbeddedContext();  //上下文，继承StandardContext
		context.setName(getContextPath());
		context.setDisplayName(getDisplayName());
		context.setPath(getContextPath());
		context.setDocBase(docBase.getAbsolutePath());
		context.addLifecycleListener(new FixContextListener());
		context.setParentClassLoader(
				this.resourceLoader != null ? this.resourceLoader.getClassLoader()
						: ClassUtils.getDefaultClassLoader());
		try {
			context.setUseRelativeRedirects(false);
			context.setMapperContextRootRedirectEnabled(true);
		}
		catch (NoSuchMethodError ex) {
			// Tomcat is < 8.0.30. Continue
		}
		SkipPatternJarScanner.apply(context, this.tldSkip);
		WebappLoader loader = new WebappLoader(context.getParentClassLoader());
		loader.setLoaderClass(TomcatEmbeddedWebappClassLoader.class.getName());
		loader.setDelegate(true);
		context.setLoader(loader);
		if (isRegisterDefaultServlet()) {
			addDefaultServlet(context);
		}
		if (shouldRegisterJspServlet()) {
			addJspServlet(context);
			addJasperInitializer(context);
			context.addLifecycleListener(new StoreMergedWebXmlListener());
		}
		ServletContextInitializer[] initializersToUse = mergeInitializers(initializers);
		configureContext(context, initializersToUse);
		host.addChild(context);
		postProcessContext(context);
	}
```
首先设置`TomcatEmbeddedContext`上下文，它为`StandardContext`子类，后续设置上下文的若干属性，例如上下文路径等,

执行`configureContext`

```java
protected void configureContext(Context context,
			ServletContextInitializer[] initializers) {
		TomcatStarter starter = new TomcatStarter(initializers);
		if (context instanceof TomcatEmbeddedContext) {
			// Should be true
			((TomcatEmbeddedContext) context).setStarter(starter);
		}
		context.addServletContainerInitializer(starter, NO_CLASSES);
		for (LifecycleListener lifecycleListener : this.contextLifecycleListeners) {
			context.addLifecycleListener(lifecycleListener);
		}
		for (Valve valve : this.contextValves) {
			context.getPipeline().addValve(valve);
		}
		for (ErrorPage errorPage : getErrorPages()) {
			new TomcatErrorPage(errorPage).addToContext(context);
		}
		for (MimeMappings.Mapping mapping : getMimeMappings()) {
			context.addMimeMapping(mapping.getExtension(), mapping.getMimeType());
		}
		configureSession(context);
		for (TomcatContextCustomizer customizer : this.tomcatContextCustomizers) {
			customizer.customize(context);
		}
	}
```

执行`TomcatStarter starter = new TomcatStarter(initializers);`然后将其加入到`context`中`context.addServletContainerInitializer(starter, NO_CLASSES);`,则会在`tomcat`启动时会调用`start`中的`onStartup`方法

```java
@Override
	public void onStartup(Set<Class<?>> classes, ServletContext servletContext)
			throws ServletException {
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
				logger.error("Error starting Tomcat context. Exception: "
						+ ex.getClass().getName() + ". Message: " + ex.getMessage());
			}
		}
	}
```

调用了`initializer.onStartup(servletContext);` 则可以回到前面提到的``selfInitialize(servletContext);`了

```java

private void selfInitialize(ServletContext servletContext) throws ServletException {
		prepareEmbeddedWebApplicationContext(servletContext);
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		ExistingWebApplicationScopes existingScopes = new ExistingWebApplicationScopes(
				beanFactory);  //设置web scope
		WebApplicationContextUtils.registerWebApplicationScopes(beanFactory,
				getServletContext());
		existingScopes.restore();
		WebApplicationContextUtils.registerEnvironmentBeans(beanFactory,
				getServletContext());
		for (ServletContextInitializer beans : getServletContextInitializerBeans()) {  //核心方法
			beans.onStartup(servletContext);  //servlet、filter和listen都会注册到ServletContext上
		}
	}
```

* `registerWebApplicationScopes`注册了各种属于`web`的`scope`
* `registerEnvironmentBeans`注册了`web`特定的`contextParameters`,`contextAttributes`等

`getServletContextInitializerBeans()`实现为

```java
protected Collection<ServletContextInitializer> getServletContextInitializerBeans() {
		return new ServletContextInitializerBeans(getBeanFactory());
	}
```

`ServletContextInitializerBeans`为`Collection`的子类，继承了`AbstractCollection`,调用构造方法

```java
	public ServletContextInitializerBeans(ListableBeanFactory beanFactory) {
		this.initializers = new LinkedMultiValueMap<Class<?>, ServletContextInitializer>();
		addServletContextInitializerBeans(beanFactory);  //处理ServletContextInitializer
		addAdaptableBeans(beanFactory);  //核心方法，将所有申明的Servlet ，Filter等转换成对应的XxxRegistrationBean
		List<ServletContextInitializer> sortedInitializers = new ArrayList<ServletContextInitializer>();
		for (Map.Entry<?, List<ServletContextInitializer>> entry : this.initializers
				.entrySet()) {
			AnnotationAwareOrderComparator.sort(entry.getValue());
			sortedInitializers.addAll(entry.getValue());
		}
		this.sortedList = Collections.unmodifiableList(sortedInitializers);
	}
```

首先看`this.initializers = new LinkedMultiValueMap<Class<?>, ServletContextInitializer>();` 其中的`initializers`为一个多值的`map`结构，
简单来说就是`map`中的`key`对应多个`value`，其内部实现看`LinkedMultiValueMap`,利用`Map<K, List<V>> targetMap`内部属性来实现。
```java
public class LinkedMultiValueMap<K, V> implements MultiValueMap<K, V>, Serializable {

	private static final long serialVersionUID = 3801124242820219131L;

	private final Map<K, List<V>> targetMap;
	@Override
	public void add(K key, V value) {
		List<V> values = this.targetMap.get(key);
		if (values == null) {
			values = new LinkedList<V>();
			this.targetMap.put(key, values);
		}
		values.add(value);
	}

}

...

```

更多集合操作可以使用[guava](https://github.com/google/guava)避免重复造轮子


`addServletContextInitializerBeans()`方法


```java

private void addServletContextInitializerBeans(ListableBeanFactory beanFactory) {
		for (Entry<String, ServletContextInitializer> initializerBean : getOrderedBeansOfType(
				beanFactory, ServletContextInitializer.class)) {
			addServletContextInitializerBean(initializerBean.getKey(),
					initializerBean.getValue(), beanFactory);
		}
	}
```

获取所有类型为`ServletContextInitializer`，进入如下处理

```java

private void addServletContextInitializerBean(String beanName,
			ServletContextInitializer initializer, ListableBeanFactory beanFactory) {
		if (initializer instanceof ServletRegistrationBean) {
			Servlet source = ((ServletRegistrationBean) initializer).getServlet();
			addServletContextInitializerBean(Servlet.class, beanName, initializer,
					beanFactory, source);
		}
		else if (initializer instanceof FilterRegistrationBean) {
			Filter source = ((FilterRegistrationBean) initializer).getFilter();
			addServletContextInitializerBean(Filter.class, beanName, initializer,
					beanFactory, source);
		}
		else if (initializer instanceof DelegatingFilterProxyRegistrationBean) {
			String source = ((DelegatingFilterProxyRegistrationBean) initializer)
					.getTargetBeanName();
			addServletContextInitializerBean(Filter.class, beanName, initializer,
					beanFactory, source);
		}
		else if (initializer instanceof ServletListenerRegistrationBean) {
			EventListener source = ((ServletListenerRegistrationBean<?>) initializer)
					.getListener();
			addServletContextInitializerBean(EventListener.class, beanName, initializer,
					beanFactory, source);
		}
		else {
			addServletContextInitializerBean(ServletContextInitializer.class, beanName,
					initializer, beanFactory, null);
		}
	}
```

如果类型为`FilterRegistrationBean`,`DelegatingFilterProxyRegistrationBean`,`ServletRegistrationBean`,`ServletListenerRegistrationBean`分别对应到`Filter`,`Filter`,`Servlet`,`EventListener`,调用

```java

private void addServletContextInitializerBean(Class<?> type, String beanName,
			ServletContextInitializer initializer, ListableBeanFactory beanFactory,
			Object source) {
		this.initializers.add(type, initializer);
		if (source != null) {
			// Mark the underlying source as seen in case it wraps an existing bean
			this.seen.add(source);
		}
		if (ServletContextInitializerBeans.logger.isDebugEnabled()) {
			String resourceDescription = getResourceDescription(beanName, beanFactory);
			int order = getOrder(initializer);
			ServletContextInitializerBeans.logger.debug("Added existing "
					+ type.getSimpleName() + " initializer bean '" + beanName
					+ "'; order=" + order + ", resource=" + resourceDescription);
		}
	}

```

将其类型`type`作为`key`存储在`initializers`中，`seen`记录处理过的`source`避免重复处理。
这里将`spring boot`中的`XxxRegistrationBean`与`web`中的`servlet`,`filter`,`listen`等对应起来，因此后期处理`web`中的元素，只需要处理`XxxRegistrationBean`即可。

继续看`addAdaptableBeans(beanFactory);`方法

```java

private void addAdaptableBeans(ListableBeanFactory beanFactory) {
		MultipartConfigElement multipartConfig = getMultipartConfig(beanFactory);
		addAsRegistrationBean(beanFactory, Servlet.class,
				new ServletRegistrationBeanAdapter(multipartConfig));  //将所有类型的Servlet对应bean转换成ServletRegistrationBean
		addAsRegistrationBean(beanFactory, Filter.class,
				new FilterRegistrationBeanAdapter());
		for (Class<?> listenerType : ServletListenerRegistrationBean
				.getSupportedTypes()) {  //处理servlet中支持的监听
			addAsRegistrationBean(beanFactory, EventListener.class,
					(Class<EventListener>) listenerType,
					new ServletListenerRegistrationBeanAdapter());
		}
	}
```

要看懂这块代码，首先要知道`addAsRegistrationBean`的作用

```java

private <T, B extends T> void addAsRegistrationBean(ListableBeanFactory beanFactory,
			Class<T> type, Class<B> beanType, RegistrationBeanAdapter<T> adapter) {
		List<Map.Entry<String, B>> beans = getOrderedBeansOfType(beanFactory, beanType,
				this.seen);  //this.seen被排除掉，前面已处理
		for (Entry<String, B> bean : beans) {
			if (this.seen.add(bean.getValue())) {
				int order = getOrder(bean.getValue());
				String beanName = bean.getKey();
				// One that we haven't already seen
				RegistrationBean registration = adapter.createRegistrationBean(beanName,
						bean.getValue(), beans.size());
				registration.setName(beanName);
				registration.setOrder(order);
				this.initializers.add(type, registration);
				if (ServletContextInitializerBeans.logger.isDebugEnabled()) {
					ServletContextInitializerBeans.logger.debug(
							"Created " + type.getSimpleName() + " initializer for bean '"
									+ beanName + "'; order=" + order + ", resource="
									+ getResourceDescription(beanName, beanFactory));
				}
			}
		}
	}
```
FilterRegistration.Dynamic
组合起来看，发现其功能:将所有的`servlet`,`filter`,`listener`对应的`bean`适配成`XxxRegistrationBean`,然后存入`initializers`集合中。

通过前面代码可以发现，在`spring boot`中可以直接申明的`servlet`,`fiter`或者`listener`，只要将其申明为`bean`后`spring boot`自然识别，
因此在`spring boot`中申明`filter`有两种方式(`servlet`,`listener`)一样

伪代码如下：

* 方式一

```java

@Component
public MyFilter implements Filter{
...
}

```
* 方式二

```java
@Bean
public FilterRegistrationBean myFilter(){
  FilterRegistrationBean bean = new FilterRegistrationBean();
  bean.setFilter(MyFilter.class);
  bean.addUrlPatterns("/*");
  return bean;  
}


这种将外部对象统一适配为内部对象后，只要处理内部对象即可完成对内部对象+外部对象的处理思路值得学习。

<hr>

继续来看前面的代码

```java

List<ServletContextInitializer> sortedInitializers = new ArrayList<ServletContextInitializer>();
		for (Map.Entry<?, List<ServletContextInitializer>> entry : this.initializers
				.entrySet()) {
			AnnotationAwareOrderComparator.sort(entry.getValue());
			sortedInitializers.addAll(entry.getValue());
		}
		this.sortedList = Collections.unmodifiableList(sortedInitializers);
```

这部分代码将`initializers`排序后赋值给`sortedList`，`sortedList`为该集合`ServletContextInitializerBeans`核心属性，遍历集合时则遍历的为
`sortedList`

```java

	@Override
	public Iterator<ServletContextInitializer> iterator() {
		return this.sortedList.iterator();
	}  //迭代时遍历sortedList

	@Override
	public int size() {
		return this.sortedList.size();
	}

```

至此处理完成`ServletContextInitializerBeans`，回到前面的
```java

for (ServletContextInitializer beans : getServletContextInitializerBeans()) {  //核心方法
			beans.onStartup(servletContext);  //servlet、filter和listen都会注册到ServletContext上
		}
```
调用`onStartup`方法

针对`FilterRegistrationBean`执行

```java
public void onStartup(ServletContext servletContext) throws ServletException {
		Filter filter = getFilter();
		Assert.notNull(filter, "Filter must not be null");
		String name = getOrDeduceName(filter);
		if (!isEnabled()) {
			this.logger.info("Filter " + name + " was not registered (disabled)");
			return;
		}
		FilterRegistration.Dynamic added = servletContext.addFilter(name, filter);  //动态添加filter
		if (added == null) {
			this.logger.info("Filter " + name + " was not registered "
					+ "(possibly already registered?)");
			return;
		}
		configure(added);  //filter映射到/*
	}

```

通过`FilterRegistration.Dynamic`动态添加`filter`


`ServletRegistrationBean`,`ServletListenerRegistrationBean`代码逻辑和`FilterRegistrationBean`逻辑类似。


<hr/>

又要回到`TomcatEmbeddedServletContainerFactory#getEmbeddedServletContainer`
















