# SpringAppication的初始化

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

1 设置 primarySources 为启动类

2 根据 deduceFromClasspath 查看当前项目所属环境 （非web环境、web环境、reactive环境三种）即this.webApplicationType = SERVLET

```java
static WebApplicationType deduceFromClasspath() {
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : SERVLET_INDICATOR_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
```

3 setInitializers 之前首先通过getSpringFactoriesInstances 主要是先拿到当前线程的 ClassLoader ，再根据 classType和ClassLoader加载对应的META-INF/spring.factories文件，拿到加载应用上下文初始化的类名,并根据反射生成对应实例。

```java

//此方法将在loadSpringFactories加载的所有类名列表中获取某种类型集合列表
//例如factoryClass 为 org.springframework.context.ApplicationContextInitializer
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}

//此方法将通过ClassLoader加载到的jar包下META-INF/spring.factories文件，并读取文件里面的类名列表
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}
		try {
            //FACTORIES_RESOURCE_LOCATION = META-INF/spring.factories
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryClassName = ((String) entry.getKey()).trim();
					for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryClassName, factoryName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

//resource中将是各个jar包中的META-INF/spring.factories资源，setInitializers 从而实现初始化

![img](file:///C:\Users\mrchen\AppData\Roaming\Tencent\Users\1142170725\QQ\WinTemp\RichOle\~QEWX1$_`FVSJ500~LQ[}QL.png)



4 setListeners  先像之前一样通过getSpringFactoriesInstances 拿到需要加载的类名集合，只不过之前是通过ApplicationContextInitializer和ClassLoader或许需要初始化的类，这里是通过ApplicationListener和ClassLoader拿到需要加载的监听器类名，随后在由createSpringFactoriesInstances通过反射生成对应类名的实例。

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
		// 同样根据 loadFactoryNames 读取项目文件中需要加载类的 names
		Set<String> names = new LinkedHashSet<>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

	@SuppressWarnings("unchecked")
	private <T> List<T> createSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
			Set<String> names) {
		List<T> instances = new ArrayList<>(names.size());
        //根据getSpringFactoriesInstances获取的names，通过反射生成对应实例。
		for (String name : names) {
			try {
				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
				Assert.isAssignable(type, instanceClass);
				Constructor<?> constructor = instanceClass
						.getDeclaredConstructor(parameterTypes);
				T instance = (T) BeanUtils.instantiateClass(constructor, args);
				instances.add(instance);
			}
			catch (Throwable ex) {
				throw new IllegalArgumentException(
						"Cannot instantiate " + type + " : " + name, ex);
			}
		}
		return instances;
	}
```

5 最后deduceMainApplicationClass通过方法栈中寻找main方法加载对应类名，找到指定的mainApplicationClass类，SpringApplication的初始化才算完成而springboot真正的启动才刚刚开始。



# SpringAppication的Run

```java
public ConfigurableApplicationContext run(String... args) {
  		//时间监控
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		//java.awt.headless是J2SE的一种模式用于在缺少显示屏、键盘或者鼠标时的系统配置，很多监控工具		 //如jconsole 需要将该值设置为true，系统变量默认为true
  		configureHeadlessProperty();
  		//获取spring.factories中的监听器变量，args为指定的参数数组，默认为当前类SpringApplication
		//第一步：获取并启动监听器
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
            //第二步：构造容器环境
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
          	//设置需要忽略的bean
			configureIgnoreBeanInfo(environment);
          	//打印banner
			Banner printedBanner = printBanner(environment);
          	//第三步：创建容器
			context = createApplicationContext();
          	//第四步：实例化SpringBootExceptionReporter.class，用来支持报告关于启动的错误
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
          	//第五步：准备容器
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
          	//第六步：刷新容器
			refreshContext(context);
          	//第七步：刷新容器后的扩展接口
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		return context;
	}
```

## 1 获取监听器

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
				SpringApplicationRunListener.class, types, this, args));
	}
```

```java
public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}
```

```java
SpringApplicationRunListeners(Log log,
      Collection<? extends SpringApplicationRunListener> listeners) {
   this.log = log;
   this.listeners = new ArrayList<>(listeners);
}
```

本质还是和之前一样，通过getSpringFactoriesInstances获取需要类名并通过反射生成实例，不过此处的class参数为SpringApplicationRunListener，也就是获取对应spring.factories里面对应的监听器。SpringApplicationRunListener本身是一个接口，通过反射构造实例时将创建EventPublishingRunListener实例对象。构造该实例时，它将通过application.getListeners()拿到全部的Listeners，并在SpringApplicationRunListeners构造实例时将所有Listeners传递。

##2 启动监听器



```java
@Override
public void starting() {
   this.initialMulticaster.multicastEvent(
         new ApplicationStartingEvent(this.application, this.args));
}
```

```java
@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
      	//getApplicationListeners将根据发布的事件类型从上述所有监听器中选择对应的监听器进行事件发布
      	//例如此时 tpye 为 ApplicationStartingEvent
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			////获取线程池，如果为空则同步处理。这里线程池为空，还未没初始化。
          	Executor executor = getTaskExecutor();
			if (executor != null) {
              	//异步启动
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
              	//同步启动
				invokeListener(listener, event);
			}
		}
	}
```

```java
public void onApplicationEvent(ApplicationEvent event) {
  		//在springboot启动的时候
        if (event instanceof ApplicationStartingEvent) {
            this.onApplicationStartingEvent((ApplicationStartingEvent)event);
        }
		//在springboot容器的准备
        if (event instanceof ApplicationPreparedEvent) {
            this.onApplicationPreparedEvent((ApplicationPreparedEvent)event);
        }
		//在springboot容器的准备完成以后
        if (event instanceof ApplicationReadyEvent || event instanceof ApplicationFailedEvent) {
            Restarter.getInstance().finish();
        }
  		//容器启动失败的时候
        if (event instanceof ApplicationFailedEvent) {
            this.onApplicationFailedEvent((ApplicationFailedEvent)event);
        }

    }
```

## 3 环境构建

```java
private ConfigurableEnvironment prepareEnvironment(
			SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
  		// 此处是servlet返回new StandardServletEnvironment();
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		//第二次监听器的事件发布
  		//最终将执行ConfigFileApplicationListener的onApplicationEnvironmentPreparedEvent
  		//该监听器非常核心，主要用来处理项目配置。项目中的 properties 和yml文件都是其内部类所加载。
  		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader())
					.convertEnvironmentIfNecessary(environment, deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}


```

```java
private void onApplicationEnvironmentPreparedEvent(
			ApplicationEnvironmentPreparedEvent event) {
  		//读spring.factories 文件获取 EnvironmentPostProcessor
		List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
		postProcessors.add(this);
		AnnotationAwareOrderComparator.sort(postProcessors);
  		//在ConfigFileApplicationListener中，将实例化 Loader
		for (EnvironmentPostProcessor postProcessor : postProcessors) {
			postProcessor.postProcessEnvironment(event.getEnvironment(),
					event.getSpringApplication());
		}
	}
```

```java
Loader(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
			this.environment = environment;
			this.placeholdersResolver = new PropertySourcesPlaceholdersResolver(
					this.environment);
			this.resourceLoader = (resourceLoader != null) ? resourceLoader
					: new DefaultResourceLoader();
  			//还是将通过读spring.factories 文件， 读取PropertySourceLoader类型的类名列表
  			//并通过反射创建实例
			this.propertySourceLoaders = SpringFactoriesLoader.loadFactories(
					PropertySourceLoader.class, getClass().getClassLoader());
		}
```

## 4 创建容器

```java
public static final String DEFAULT_SERVLET_WEB_CONTEXT_CLASS = "org.springframework.boot."
			+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";

protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, "
								+ "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```

创建容器的类型 还是根据`webApplicationType`进行判断



##4 报告错误信息

```java
exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
```

![img](file:///C:\Users\mrchen\AppData\Roaming\Tencent\Users\1142170725\QQ\WinTemp\RichOle\5RQWCA6V$J100AG$`R[2M9U.png)

跟之前一样，还是通过getSpringFactoriesInstances获取对应的实例，这里SpringBootExceptionReporter的实例



## 5 准备容器

**将启动类注入容器，为后续开启自动化配置奠定基础。**

```java
private void prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		//设置容器环境，包括各种变量
  		context.setEnvironment(environment);
  		//执行容器后置处理
		postProcessApplicationContext(context);
  		//执行容器中的ApplicationContextInitializer（包括 spring.factories和自定义的实例）
		applyInitializers(context);
  		//发送容器已经准备好的事件，通知各监听器
		listeners.contextPrepared(context);
  		//打印log
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
  		//注册启动参数bean，这里将容器指定的参数封装成bean，注入容器
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		//设置banner
  		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		// Load the sources
  		//获取我们的启动类指定的参数，可以是多个
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
  		//加载我们的启动类，将启动类注入容器
		load(context, sources.toArray(new Object[0]));
  		//发布容器已加载事件。
		listeners.contextLoaded(context);
	}
```



## 6 刷新容器

执行到这里，springBoot相关的处理工作已经结束，接下的工作就交给了spring。

`refresh`方法在spring整个源码体系中举足轻重，是实现 ioc 和 aop的关键。

```java
synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			/**
			 * 刷新上下文环境
			 * 初始化上下文环境，对系统的环境变量或者系统属性进行准备和校验
			 * 如环境变量中必须设置某个值才能运行，否则不能运行，这个时候可以在这里加这个校验，
			 * 重写initPropertySources方法就好了
			 */
			prepareRefresh();
 
			// Tell the subclass to refresh the internal bean factory.
			/**
			 * 初始化BeanFactory，解析XML，相当于之前的XmlBeanFactory的操作，
			 */
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
 
			// Prepare the bean factory for use in this context.
			/**
			 * 为上下文准备BeanFactory，即对BeanFactory的各种功能进行填充，如常用的注解@Autowired @Qualifier等
			 * 设置SPEL表达式#{key}的解析器
			 * 设置资源编辑注册器，如PerpertyEditorSupper的支持
			 * 添加ApplicationContextAwareProcessor处理器
			 * 在依赖注入忽略实现*Aware的接口，如EnvironmentAware、ApplicationEventPublisherAware等
			 * 注册依赖，如一个bean的属性中含有ApplicationEventPublisher(beanFactory)，则会将beanFactory的实例注入进去
			 */
			prepareBeanFactory(beanFactory);
 
			try {
				// Allows post-processing of the bean factory in context subclasses.
				/**
				 * 提供子类覆盖的额外处理，即子类处理自定义的BeanFactoryPostProcess
				 */
				postProcessBeanFactory(beanFactory);
 
				// Invoke factory processors registered as beans in the context.
				/**
				 * 激活各种BeanFactory处理器,包括BeanDefinitionRegistryBeanFactoryPostProcessor和普通的BeanFactoryPostProcessor
				 * 执行对应的postProcessBeanDefinitionRegistry方法 和  postProcessBeanFactory方法
				 */
				invokeBeanFactoryPostProcessors(beanFactory);
 
				// Register bean processors that intercept bean creation.
				/**
				 * 注册拦截Bean创建的Bean处理器，即注册BeanPostProcessor，不是BeanFactoryPostProcessor，注意两者的区别
				 * 注意，这里仅仅是注册，并不会执行对应的方法，将在bean的实例化时执行对应的方法
				 */
				registerBeanPostProcessors(beanFactory);
 
				// Initialize message source for this context.
				/**
				 * 初始化上下文中的资源文件，如国际化文件的处理等
				 */
				initMessageSource();
 
				// Initialize event multicaster for this context.
				/**
				 * 初始化上下文事件广播器，并放入applicatioEventMulticaster,如ApplicationEventPublisher
				 */
				initApplicationEventMulticaster();
 
				// Initialize other special beans in specific context subclasses.
				/**
				 * 给子类扩展初始化其他Bean
				 */
				onRefresh();
 
				// Check for listener beans and register them.
				/**
				 * 在所有bean中查找listener bean，然后注册到广播器中
				 */
				registerListeners();
 
				// Instantiate all remaining (non-lazy-init) singletons.
				/**
				 * 设置转换器
				 * 注册一个默认的属性值解析器
				 * 冻结所有的bean定义，说明注册的bean定义将不能被修改或进一步的处理
				 * 初始化剩余的非惰性的bean，即初始化非延迟加载的bean
				 */
				finishBeanFactoryInitialization(beanFactory);
 
				// Last step: publish corresponding event.
				/**
				 * 初始化生命周期处理器DefaultLifecycleProcessor，DefaultLifecycleProcessor含有start方法和stop方法，spring启动的时候调用start方法开始生命周期，
				 * spring关闭的时候调用stop方法来结束生命周期，通常用来配置后台程序，启动有一直运行，如一直轮询kafka
				 * 启动所有实现了Lifecycle接口的类
				 * 通过spring的事件发布机制发布ContextRefreshedEvent事件，以保证对应的监听器做进一步的处理，即对那种在spring启动后需要处理的一些类，这些类实现了
				 * ApplicationListener<ContextRefreshedEvent> ,这里就是要触发这些类的执行(执行onApplicationEvent方法)另外，spring的内置Event有ContextClosedEvent、ContextRefreshedEvent、ContextStartedEvent、ContextStoppedEvent、RequestHandleEvent
				 * 完成初始化，通知生命周期处理器lifeCycleProcessor刷新过程，同时发出ContextRefreshEvent通知其他人
				 */
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
				resetCommonCaches();
			}
		}

```



## 7 刷新容器后的扩展接口

```
protected void afterRefresh(ConfigurableApplicationContext context,
			ApplicationArguments args) {
	}
```

扩展接口，设计模式中的模板方法，默认为空实现。如果有自定义需求，可以重写该方法。比如打印一些启动结束log，或者一些其它后置处理。