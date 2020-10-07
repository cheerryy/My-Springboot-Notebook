在Spring中对象注册到容器主要有两种方法：

 1. 使用xml配置文件对类进行配置后自动创建对象
 2. 使用注解的方式进行Bean注册

接下来将分别展示两种方式的注册过程

### 1 xml配置文件进行注册
总的来说，这种方法的步骤为：

 1. 保存xml配置文件的路径
 2. 根据位置读到配置文件，解析成DOM对象
 3. DOM结构的对象转换+注册成BeanDefinition
 4. 将BeanDefinition存入beanDefinitionMap
 5. 需要实例化时从beanDefinitionMap中取出

> A *BeanDefinition* describes a bean instance, which has property values, constructor argument values, and further information supplied by concrete implementations.
> 
> *BeanDefinition* 描述了一个bean的实例，它含有属性值，构造方法，其他方法和由实际应用提供的进一步信息。

接下来进行源码的详细解读和分析。

首先定义变量：
```java
public class User {
    private String userName;
    private String password;
}
```

在xml文件进行配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userBean" class="org.kxg.springDemo.User">
        <property name="userName" value="jack" />
        <property name="password" value="123456" />
    </bean>
</beans>
```
读取配置文件并运行：

```java
public class XmlBeanTest {
    public static void main(String[] args){
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
        User user = (User) applicationContext.getBean("userBean");
        System.out.println(user.getUserName());
    }
}
```

可以看到，运行过程是通过`ClassPathXmlApplicationContext`函数从容器中读取到变量，因此分析应从该函数入手。

函数定义如下：

```java
//一个配置文件
public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

//多个配置文件	
public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
	this(configLocations, true, null);
}
```

观察`ClassPathXmlApplicationContext`的构造函数，如下所示：
```java
public ClassPathXmlApplicationContext(
			String[] configLocations, 
			boolean refresh,
			@Nullable ApplicationContext parent)throws BeansException {
				super(parent);
				//将传入的xml配置位置信息保存到configLocations
				setConfigLocations(configLocations);
				if (refresh) {
					refresh();
				}
	}
```

该函数传入——

 1. xml文件的路径（string数组）
 2. 是否refresh（布尔）
 3. parent

内部方法——

 1. 调用`setConfigLocations`方法将传入的xml路径设为configLocations
 2. 如果refresh==true，调用`refresh()`函数

而`refresh()`函数如下：

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// ！！！
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

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
				resetCommonCaches();
			}
		}
	}
```

其中：`ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();`
这句话调用了`obtainFreshBeanFactory()`函数，返回一个bean的factory（类似于[工厂模式](https://blog.csdn.net/weixin_43883815/article/details/108897946)中，构建一个生产bean的工厂）。

该函数定义如下：

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}
```
可见，是通过调用refreshBeanFactory();这个方法来构建工厂的。

进而查看refreshBeanFactory()方法，如下：

```java
protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
		    //创建一个BeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			//这里进行Bean的加载
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```
可见，该函数利用`createBeanFactory`函数创建一个beanFactory对象，接着对这个beanFactory操作，包括调用`loadBeanDefinitions(beanFactory)`方法来加载beanFactory。

接下来查看`loadBeanDefinitions(beanFactory)`方法是如何加载beanFactory的。代码如下：

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// 构造一个XmlBeanDefinitionReader，用于读到xml中配置的bean
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// 配置XmlBeanDefinitionReader
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		//初始化XmlBeanDefinitionReader
		initBeanDefinitionReader(beanDefinitionReader);
		//加载Bean
		loadBeanDefinitions(beanDefinitionReader);
	}
```
可见，该函数的方式是构造一个reader（`beanDefinitionReader`），用于读取beanFactory的东西。先对这个reader操作（配置好环境等，再初始化一下），最后利用同名的`loadBeanDefinitions(beanDefinitionReader);`
加载BeanDefinitions。

这个同名的函数实现如下：

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
	}
```
这个函数将分别处理位置信息和资源信息。
用`getConfigResources`获取配置的资源，放到名为`configResources`的`Resource`类型数组里面，判断获得的东西是不是空，如果不是，就把获得的资源传入`reader.loadBeanDefinitions(configResources);`

同理，还用`getConfigLocations`获得了配置资源的位置，用`configLocations`（String数组）保存，不为空时，调用`reader.loadBeanDefinitions(configLocations);`

**至此经过一番操作，容器已经获得了配置文件的位置`configLocations`，接下来将根据`configLocations`来讲对象解析成DOM对象，最终目的是要将对象注册成`BeanDefinition`**。

观察上一步中，处理位置信息的`loadBeanDefinitions`，它的实现如下：

```java
@Override
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
	Assert.notNull(locations, "Location array must not be null");
	int count = 0;
	for (String location : locations) {
		count += loadBeanDefinitions(location);
	}
	return count;
}
```
它传入一个`configLocations`（stirng数组），
对于`configLocations`里面的每个location，count+=`loadBeanDefinitions`对location的处理结果（也是int），最后返回count（int类型）

刚刚处理单个location 的函数`loadBeanDefinitions`，实现如下：

```java
public int loadBeanDefinitions(String location, @Nullable Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot load bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				int count = loadBeanDefinitions(resources);
				if (actualResources != null) {
					Collections.addAll(actualResources, resources);
				}
				if (logger.isTraceEnabled()) {
					logger.trace("Loaded " + count + " bean definitions from location pattern [" + location + "]");
				}
				return count;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int count = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isTraceEnabled()) {
				logger.trace("Loaded " + count + " bean definitions from location [" + location + "]");
			}
			return count;
		}
	}
```
这个函数首先用`getResourceLoader`，得到一个`resourceLoader`（资源加载器）。再判断`resourceLoader`是否为`ResourcePatternResolver`的一个实例（`instance of`），这个判断的目的是判断后面调用`getResources(location)`时，是传入`Resource`类型的数组还是变量。

如果是实例，则：
用`getResources(location)`函数，返回一个名为resources的Resource数组，再`int count = loadBeanDefinitions(resources);`

否则：
用`getResources(location)`函数，返回一个名为resource的Resource变量
再`int count = loadBeanDefinitions(resource);`

总之都是调用`loadBeanDefinitions(resource)`。

这个`loadBeanDefinitions(resource)`函数，定义如下：

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Loading XML bean definitions from " + encodedResource);
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				//进行BeanDefinations加载
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
```
把传入的resource，调用`getResource`函数+`getInputStream`函数，变成一个`InputStream` 类型的变量`inputStream`，再用`InputSource`方法把`inputStream`变成`InputSource` 类型的变量`inputSource`
最后返回：`doLoadBeanDefinitions(inputSource, encodedResource. getResource())`的结果。

而这个`doLoadBeanDefinitions`实现如下：

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {

		try {
		    //构造xml的Document结构，解析DOM结构
			Document doc = doLoadDocument(inputSource, resource);
			//注册BeanDefinition
			int count = registerBeanDefinitions(doc, resource);
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + count + " bean definitions from " + resource);
			}
			return count;
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```
内部调用`doLoadDocument`函数，传入`inputSource`和`resource`，返回`Document`类型的变量`doc`，再调用`registerBeanDefinitions`函数（传入doc和resource，返回一个int值）。


**至此，已经将配置文件解析转换成DOM对象了，接下来要正式进行注册了（将DOM转换成`BeanDefinition`类型）。**


刚刚被调用的`registerBeanDefinitions`函数，实现如下：

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```
它构造了一个`BeanDefinitionDocumentReader`类型的变量，调用`registerBeanDefinitions`函数注册BeanDefinition，并且返回了本次注册Bean的数量。

而实现注册的方法为：`doRegisterBeanDefinitions`
该方法实现如下：
```java
protected void doRegisterBeanDefinitions(Element root) {
		// Any nested <beans> elements will cause recursion in this method. In
		// order to propagate and preserve <beans> default-* attributes correctly,
		// keep track of the current (parent) delegate, which may be null. Create
		// the new (child) delegate with a reference to the parent for fallback purposes,
		// then ultimately reset this.delegate back to its original (parent) reference.
		// this behavior emulates a stack of delegates without actually necessitating one.
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				// We cannot use Profiles.of(...) since profile expressions are not supported
				// in XML config. See SPR-12458 for details.
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isDebugEnabled()) {
						logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}

		preProcessXml(root);
		//进行BeanDefinition转换，将DOM结构的对象转换成BeanDefinition
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}
```
可以看到里面的`parseBeanDefinitions`就是将DOM转换成`BeanDefinition`的函数。

这个`parseBeanDefinitions`的实现如下：

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
				Node node = nl.item(i);
				if (node instanceof Element) {
					Element ele = (Element) node;
					if (delegate.isDefaultNamespace(ele)) {
					    //Spring默认元素转换
						parseDefaultElement(ele, delegate);
					}
					else {
					    //xml中自定义的Element进行解析
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```
可见，它内部调用parseDefaultElement方法，对元素进行配置，如下：

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```
可以看到，它对import、alias、bean、beans这几个元素进行了分析，这几个也是xml配置文件的默认配置元素。

这里面调用的`processBeanDefinition`用于对bean元素加工处理，如下：

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```


processXXX函数内部调用的`registerBeanDefinition`将会把bean正式进行注册，如下：

```java
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
		//注册BeanDefinition
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
```

**至此，已经成功的将DOM对象注册成为`BeanDefinition`，接下来将`BeanDefinition`进行存储。**

刚刚的函数内部又调用了同名的`registerBeanDefinition`，这个同名的函数将会把BeanDefinition存储到`beanDefinitionMap`中，实现如下：

```java
// DefaultListableBeanFactory
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			//将beanDefinition放入beanDefinitionMap中
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
				    //将beanDefinition放入beanDefinitionMap中
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// Still in startup registration phase
				//将beanDefinition放入beanDefinitionMap中
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}
```

该函数把注册的`BeanDefinition`存到一个键为`beanName`，值为`beanDefinition`对象的Map集合中。

这个Map的定义为：

```java
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```
**至此，正式完成xml配置文件方式的注册。**

完成存储以后，即真正注册到容器中了。当需要使用的时候，再进行实例化从容器（Map）中取出。


### 2 注解方式下Bean的注册
总的来说，该方法的主要步骤为：

 1. 根据包名，扫描包下面带注解的类
 2. 将带注解的都用`registerBeanDefinition`进行注册，转换成`BeanDefinition`
 3. 将`BeanDefinition`都存储到set中



同样的，先创建对象，如下：
```java
@Component
public class AnnotionConfig {
    @Bean(name = "userBean")
    public User getUserBean(){
        User user = new User();
        user.setUserName("Lucy");
        return user;
    }
}
```
测试容器是否拿到对象：
```java
public class AnnotionBeanTest {
    public static void main(String[] args){
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext("org.kxg.springDemo");
        User user = (User) applicationContext.getBean("userBean");
        System.out.println(user.getUserName());
    }
}
```
这里调用的是`AnnotationConfigApplicationContext`的构造函数，传入包名，自动扫描包下面的Spring注解，然后将其注册到容器中。

它的构造方法具体如下：

```java
public AnnotationConfigApplicationContext(String... basePackages) {
		this();
		//主要是scan方法完成bean的注册
		scan(basePackages);
		//同样的调用了refresh方法
		refresh();
	}
```

可见，该函数内部调用`scan`函数，根据传入的包名，扫描包名对应的包的内容，如下：

```java
public int scan(String... basePackages) {
		int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
        //扫描包，进行Bean注册
		doScan(basePackages);

		// Register annotation config processors, if necessary.
		if (this.includeAnnotationConfig) {
			AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
		}

		return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
	}
```

而这个`scan`函数又调用`doScan(basePackages)`函数。实现如下：

```java
//ClassPathBeanDefinitionScanner
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
		Assert.notEmpty(basePackages, "At least one base package must be specified");
		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
		for (String basePackage : basePackages) {
		    //扫描包下打了注解的类，并将其转换成BeanDefinition
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
			for (BeanDefinition candidate : candidates) {
				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
				candidate.setScope(scopeMetadata.getScopeName());
				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
				if (candidate instanceof AbstractBeanDefinition) {
					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
				}
				if (candidate instanceof AnnotatedBeanDefinition) {
					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
				}
				if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					definitionHolder =
							AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
					//进行BeanDefinition注册
					registerBeanDefinition(definitionHolder, this.registry);
				}
			}
		}
		return beanDefinitions;
	}
```

可见，`doScan`函数会扫描包下打了注解的类，进行`BeanDefinition`注册（和上一个一样，也是调用`registerBeanDefinition`函数），并将其转换成`BeanDefinition`，最后返回一个名为`beanDefinitions`的集合`Set<BeanDefinitionHolder>`。

其中调用的`registerBeanDefinition`函数，会根据bean的name，alias等把它注册成为一个正式的有定义的bean变量。如下：

```java
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {

		// Register bean definition under primary name.
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

		// Register aliases for bean name, if any.
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String alias : aliases) {
				registry.registerAlias(beanName, alias);
			}
		}
	}
```
**至此，完成了注解方式下的bean注册。**

