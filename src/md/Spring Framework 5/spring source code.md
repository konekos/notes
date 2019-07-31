## 第一部分 核心实现

### 第一章

#### 1.1 Spring的整体架构

![1551335553493](E:\studydyup\notes\src\pic\1551335553493.png)

这些模块被总结为以下几部分。

1．Core Container

Core Container（核心容器）包含有Core、Beans、Context和Expression Language模块。

Core和Beans模块是框架的基础部分，提供IoC（转控制）和依赖注入特性。这里的基础概念是BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。

Core模块主要包含Spring框架基本的核心工具类，Spring的其他组件都要用到这个包里的类，Core模块是其他组件的基本核心。当然你也可以在自己的应用系统中使用这些工具类。

Beans模块是所有应用都要用到的，它包含访问配置文件、创建和管理bean以及进行Inversion of Control / Dependency Injection（IoC/DI）操作相关的所有类。

Context模块构建于Core和Beans模块基础之上，提供了一种类似于JNDI注册器的框架式的对象访问方法。Context模块继承了Beans的特性，为Spring核心提供了大量扩展，添加了对国际化（例如资源绑定）、事件传播、资源加载和对Context的透明创建的支持。Context模块同时也支持J2EE的一些特性，例如EJB、JMX和基础的远程处理。ApplicationContext接口是Context模块的关键。

2．Data Access/Integration

Data Access/Integration层包含JDBC、ORM、OXM、JMS和Transaction模块。

3．Web

4．AOP

AOP模块提供了一个符合AOP联盟标准的面向切面编程的实现，它让你可以定义例如方法拦截器和切点，从而将逻辑代码分开，降低它们之间的耦合性。利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中，这有点像.Net技术中的attribute概念。

通过配置管理特性，Spring AOP模块直接将面向切面的编程功能集成到了Spring框架中，所以可以很容易地使Spring框架管理的任何对象支持AOP。Spring AOP模块为基于Spring的应用程序中的对象提供了事务管理服务。通过使用Spring AOP，不用依赖EJB组件，就可以将声明性事务管理集成到应用程序中。
Aspects模块提供了对AspectJ的集成支持。
Instrumentation模块提供了class instrumentation支持和classloader实现，使得可以在特定的应用服务器上使用。

5．Test
Test模块支持使用JUnit和TestNG对Spring组件进行测试。

#### 1.2 环境搭建

选择版本5.0.1 git pull

build失败解决方法

1、缺失jar： gradle cglibRepackJar、gradle objenesisRepackJar

2、安装AspectJ

### 第2章　容器的基本实现

#### 2.1　容器基本用法

bean

```java
package bean;

/**
 * @author @Jasu
 * @date 2019-02-27 17:05
 */
public class MyTestBean {
	private String testStr = "testStr";

	public String getTestStr() {
		return testStr;
	}

	public void setTestStr(String testStr) {
		this.testStr = testStr;
	}
}
```

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="myTestBean" class="bean.MyTestBean">
	</bean>
</beans>
```

Test class

```java
import bean.MyTestBean;
import org.junit.Test;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.core.io.ClassPathResource;

import static org.junit.Assert.assertEquals;

/**
 * @author @Jasu
 * @date 2019-02-27 17:07
 */
@SuppressWarnings("deprecation")
public class MyBeanTests {
	@Test
	public void testSimpleLoad(){
		BeanFactory bf = new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));
		MyTestBean bean=(MyTestBean) bf.getBean("myTestBean");
		assertEquals("testStr",bean.getTestStr());
	}
}
```

#### 2.2　功能分析

读取配置文件beanFactoryTest.xml。
根据beanFactoryTest.xml中的配置找到对应的类的配置，并实例化。
调用实例化后的实例。

#### 2.4　Spring的结构组成

##### 2.4.1　beans包的层级结构

##### 2.4.2　核心类介绍

1．DefaultListableBeanFactory

![1551337261278](E:\studydyup\notes\src\pic\1551337261278.png)

XmlBeanFactory继承自DefaultListableBeanFactory，而DefaultListableBeanFactory是整个bean加载的核心部分，是Spring注册及加载bean的默认实现，而对于XmlBeanFactory与DefaultListableBeanFactory不同的地方其实是在XmlBeanFactory中使用了自定义的XML读取器XmlBeanDefinitionReader，实现了个性化的BeanDefinitionReader读取，DefaultListableBeanFactory继承了AbstractAutowireCapableBeanFactory并实现了ConfigurableListableBeanFactory以及BeanDefinitionRegistry接口。图2-4是ConfigurableListableBeanFactory的层次结构图，图2-5是相关类图。

#### 2.5 容器的基础XmlBeanFactory

分析BeanFactory    bf = new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));

![1551434084274](E:\studydyup\notes\src\pic\1551434084274.png)

##### 2.5.1　配置文件封装

Resource extends InputStreamSource

```java
public interface InputStreamSource {
	InputStream getInputStream() throws IOException;

}
```

```java
public interface Resource extends InputStreamSource {
	boolean exists();
    boolean isReadable();
    boolean isOpen();
    URL getURL() throws IOException;
    URI getURI() throws IOException;
    File getFile() throws IOException;
    long lastModified() throws IOException;
    Resource createRelative(String relativePath) throws IOException;
    String getFilename();
    String getDescription();
}
```



Resource接口抽象了所有Spring内部使用到的底层资源：File、URL、Classpath等。

对不同来源的资源文件都有相应的Resource实现：文件（FileSystemResource）、Classpath资源（ClassPathResource）、URL资源（UrlResource）、InputStream资源（InputStreamResource）、Byte数组（ByteArrayResource）等。

![1551435604753](E:\studydyup\notes\src\pic\1551435604753.png)

在日常的开发工作中，资源文件的加载也是经常用到的，可以直接使用Spring提供的类，比如在希望加载文件时可以使用以下代码：
Resource resource=new ClassPathResource("beanFactoryTest.xml");
InputStream inputStream=resource.getInputStream();

有了Resource接口便可以对所有资源文件进行统一处理。至于实现，其实是非常简单的，以getInputStream为例，ClassPathResource中的实现方式便是通过class或者classLoader提供的底层方法进行调用，而对于FileSystemResource的实现其实更简单，直接使用FileInputStream对文件进行实例化。

ClassPathResource.java

```java
@Override
	public InputStream getInputStream() throws IOException {
		InputStream is;
		if (this.clazz != null) {
			is = this.clazz.getResourceAsStream(this.path);
		}
		else if (this.classLoader != null) {
			is = this.classLoader.getResourceAsStream(this.path);
		}
		else {
			is = ClassLoader.getSystemResourceAsStream(this.path);
		}
		if (is == null) {
			throw new FileNotFoundException(getDescription() + " cannot be opened because it does not exist");
		}
		return is;
	}
```

FileSystemResource.java

```java
public InputStream getInputStream() throws IOException {
         return new FileInputStream(this.file);
     }
```

当通过Resource相关类完成了对配置文件进行封装后配置文件的读取工作就全权交给XmlBeanDefinitionReader 来处理了。

```java
@SuppressWarnings({"serial", "all"})
public class XmlBeanFactory extends DefaultListableBeanFactory {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}

}
```

```java
public AbstractAutowireCapableBeanFactory() {
		super();
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
	}
```

ignoreDependencyInterface的主要功能是忽略给定接口的自动装配功能，那么，这样做的目的是什么呢？会产生什么样的效果呢？
举例来说，当A中有属性B，那么当Spring在获取A的Bean的时候如果其属性B还没有初始化，那么Spring会自动初始化B，这也是Spring中提供的一个重要特性。但是，某些情况下，B不会被初始化，其中的一种情况就是B实现了BeanNameAware接口。Spring中是这样介绍的：自动装配时忽略给定的依赖接口，典型应用是通过其他方式解析Application上下文注册依赖，类似于BeanFactory通过BeanFactoryAware进行注入或者ApplicationContext通过ApplicationContextAware进行注入。

##### 2.5.2　加载Bean

this.reader.loadBeanDefinitions(resource);是整个资源加载的切入点。

![1551766227092](E:\studydyup\notes\src\pic\1551766227092.png)

1．封装资源文件。当进入XmlBeanDefinitionReader后首先对参数Resource使用EncodedResource类进行封装。
2．获取输入流。从Resource中获取对应的InputStream并构造InputSource。
3．通过构造的InputSource实例和Resource实例继续调用函数doLoadBeanDefinitions。

public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
​     return loadBeanDefinitions(new EncodedResource(resource));
} 

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
                //org.xml.sax.InputSource
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
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

我们再次整理数据准备阶段的逻辑，首先对传入的resource参数做封装，目的是考虑到Resource可能存在编码要求的情况，其次，通过SAX读取XML文件的方式来准备InputSource对象，最后将准备的数据通过参数传入真正的核心处理部分doLoadBeanDefinitions(inputSource, encodedResource.getResource())。

准备阶段完成

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
      throws BeanDefinitionStoreException {

   try {
      Document doc = doLoadDocument(inputSource, resource);
      int count = registerBeanDefinitions(doc, resource);
      if (logger.isDebugEnabled()) {
         logger.debug("Loaded " + count + " bean definitions from " + resource);
      }
      return count;
   }
   catch () {
   }
}
```

在上面的代码中假如不考虑异常类的代码，其实只做了2件事，这2件事的每一件都必不可少。

- 加载XML文件，并得到对应的Document。

  - ```java
    protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
        //EntityResolver EntityResolver的作用是项目本身就可以提供一个如何寻找DTD声明的方法，即由程序来实现寻找DTD声明的过程，比如我们将DTD文件放到项目中某处，在实现时直接将此文档读取并返回给SAX即可。这样就避免了通过网络来寻找相应的声明。
    		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
    				getValidationModeForResource(resource), isNamespaceAware());
    	}
    ```

    ```java
    @Override
    	public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
    			ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {
    
    		DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
    		if (logger.isTraceEnabled()) {
    			logger.trace("Using JAXP provider [" + factory.getClass().getName() + "]");
    		}
    		DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
    		return builder.parse(inputSource);
    	}
    ```

- 根据返回的Document注册Bean信息。当把文件转换为Document后，接下来的提取及注册bean就是我们的重头戏。继续上面的分析，当程序已经拥有XML文档文件的Document实例对象时，就会被引入下面这个方法。

  XmlBeanDefinitionReader.class

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		//使用DefaultBeanDefinitionDocumentReader实例化BeanDefinitionDocumentReader
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		//在实例化BeanDefinitionReader时候会将BeanDefinitionRegistry传入，默认使用继承自DefaultListableBeanFactory的子类
		//记录统计前BeanDefinition的加载个数
		int countBefore = getRegistry().getBeanDefinitionCount();
		//加载及注册bean
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		//记录本次加载的BeanDefinition个数
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

DefaultBeanDefinitionDocumentReader.class

```java
@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		doRegisterBeanDefinitions(doc.getDocumentElement());
	}
```

![1551781401965](E:\studydyup\notes\src\pic\1551781401965.png)

#####  2.8.1　profile属性的使用

```java
<beans profile="dev">
       ... ...
 </beans>
    <beans profile="production">
        ... ...
    </beans>
</beans>
```

集成到Web环境中时，在web.xml中加入以下代码：

```xml
  <context-param>
    <param-name>Spring.profiles.active</param-name>

<param-value>dev</param-value>
</context-param>

```



上面的代码看起来逻辑还是蛮清晰的，因为在Spring的XML配置里面有两大类Bean声明，一个是默认的，如：
```<bean id="test" class="test.TestBean"/>```
另一类就是自定义的，如：
<tx:annotation-driven/>

而两种方式的读取及解析差别是非常大的，如果采用Spring默认的配置，Spring当然知道该怎么做，但是如果是自定义的，那么就需要用户实现一些接口及配置了。对于根节点或者子节点如果是默认命名空间的话则采用parseDefaultElement方法进行解析，否则使用delegate.parseCustomElement方法对自定义命名空间进行解析。而判断是否默认命名空间还是自定义命名空间的办法其实是使用node.getNamespaceURI()获取命名空间，并与Spring中固定的命名空间http://www.springframework.org/schema/beans进行比对。如果一致则认为是默认，否则就认为是自定义。而对于默认标签解析与自定义标签解析我们将会在下一章中进行讨论。

   

### 第3章　默认标签的解析

之前提到过Spring中的标签包括默认标签和自定义标签两种，而两种标签的用法以及解析方式存在着很大的不同，本章节重点带领读者详细分析默认标签的解析过程。

默认标签的解析是在parseDefaultElement函数中进行的，函数中的功能逻辑一目了然，分别对4种不同标签（import、alias、bean和beans）做了不同的处理。

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

#### 3.1　bean标签的解析及注册

4种标签的解析中，对bean标签的解析最为复杂也最为重要，所以我们从此标签开始深入分析，如果能理解此标签的解析过程，其他标签的解析自然会迎刃而解。首先我们进入函数processBeanDefinition(ele, delegate)。

```java
/**
	 * Process the given bean element, parsing the bean definition
	 * and registering it with the registry.
	 */
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

1．首先委托BeanDefinitionDelegate类的parseBeanDefinitionElement方法进行元素解析，返回BeanDefinitionHolder类型的实例bdHolder，经过这个方法后，bdHolder实例已经包含我们配置文件中配置的各种属性了，例如class、name、id、alias之类的属性。

2．当返回的bdHolder不为空的情况下若存在默认标签的子节点下再有自定义属性，还需要再次对自定义标签进行解析。

3．解析完成后，需要对解析后的bdHolder进行注册，同样，注册操作委托给了Bean- DefinitionReaderUtils的registerBeanDefinition方法。

4．最后发出响应事件，通知相关的监听器，这个bean已经加载完成了。
配合时序图（见图3-1），可能会更容易理解。

![1552467252201](E:\studydyup\notes\src\pic\1552467252201.png)

##### 3.1.1　解析BeanDefinition

下面我们就针对各个操作做具体分析。首先我们从元素解析及信息提取开始，也就是BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele)，进入BeanDefinition- Delegate类的parseBeanDefinitionElement方法。　
BeanDefinitionDelegate.java

```
@Nullable
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
	}

	/**
	 * Parses the supplied {@code <bean>} element. May return {@code null}
	 * if there were errors during parse. Errors are reported to the
	 * {@link org.springframework.beans.factory.parsing.ProblemReporter}.
	 */
	@Nullable
	public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
		//解析id属性
		String id = ele.getAttribute(ID_ATTRIBUTE);
		//解析name属性
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
		//分割name属性
		List<String> aliases = new ArrayList<>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isTraceEnabled()) {
				logger.trace("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}

		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
					//如果不存在beanName根据spring命名规则生成
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						// Register an alias for the plain bean class name, if still possible,
						// if the generator returned the class name plus a suffix.
						// This is expected for Spring 1.2/2.0 backwards compatibility.
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isTraceEnabled()) {
						logger.trace("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}
```

以上便是对默认标签解析的全过程了。当然，对Spring的解析犹如洋葱剥皮一样，一层一层地进行，尽管现在只能看到对属性id以及name的解析，但是很庆幸，思路我们已经了解了。在开始对属性展开全面解析前，Spring在外层又做了一个当前层的功能架构，在当前层完成的主要工作包括如下内容。

1．提取元素中的id以及name属性。
2．进一步解析其他所有属性并统一封装至GenericBeanDefinition类型的实例中。
3．如果检测到bean没有指定beanName，那么使用默认规则为此Bean生成beanName。
4．将获取到的信息封装到BeanDefinitionHolder的实例中。

我们进一步地查看步骤2中对标签其他属性的解析过程。

```java
/**
	 * Parse the bean definition itself, without regard to name or aliases. May return
	 * {@code null} if problems occurred during the parsing of the bean definition.
	 */
	@Nullable
	public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, @Nullable BeanDefinition containingBean) {

		this.parseState.push(new BeanEntry(beanName));

		String className = null;
		//解析class属性
		if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
			className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
		}
		String parent = null;
		if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
			//解析parent属性
			parent = ele.getAttribute(PARENT_ATTRIBUTE);
		}

		try {
			//创建用于承载属性的AbstractBeanDefinition类型的GenericBeanDefinition
			AbstractBeanDefinition bd = createBeanDefinition(className, parent);
			//硬编码解析默认bean的各种属
			parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
			//提取description
			bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
			//解析元数据
			parseMetaElements(ele, bd);
			//解析lookup-method属性
			parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
			//解析lookup-method属
			parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
			//解析构造函数参数
			parseConstructorArgElements(ele, bd);
			//解析property子元素
			parsePropertyElements(ele, bd);
			//解析qualifier子元素
			parseQualifierElements(ele, bd);

			bd.setResource(this.readerContext.getResource());
			bd.setSource(extractSource(ele));

			return bd;
		}
		catch (ClassNotFoundException ex) {
			error("Bean class [" + className + "] not found", ele, ex);
		}
		catch (NoClassDefFoundError err) {
			error("Class that bean class [" + className + "] depends on not found", ele, err);
		}
		catch (Throwable ex) {
			error("Unexpected failure during bean definition parsing", ele, ex);
		}
		finally {
			this.parseState.pop();
		}

		return null;
	}
```

终于，bean标签的所有属性，不论常用的还是不常用的我们都看到了，尽管有些复杂的属性还需要进一步的解析，不过丝毫不会影响我们兴奋的心情。接下来，我们继续一些复杂标签属性的解析。

1．创建用于属性承载的BeanDefinition

BeanDefinition是一个接口，在Spring中存在三种实现：RootBeanDefinition、ChildBean- Definition以及GenericBeanDefinition。三种实现均继承了AbstractBeanDefiniton，其中BeanDefinition是配置文件<bean>元素标签在容器中的内部表示形式。<bean>元素标签拥有class、scope、lazy-init等配置属性，BeanDefinition则提供了相应的beanClass、scope、lazyInit属性，BeanDefinition和<bean>中的属性是一一对应的。其中RootBeanDefinition是最常用的实现类，它对应一般性的<bean>元素标签，GenericBeanDefinition是自2.5版本以后新加入的bean文件配置属性定义类，是一站式服务类。

在配置文件中可以定义父<bean>和子<bean>，父<bean>用RootBeanDefinition表示，而子<bean>用ChildBeanDefiniton表示，而没有父<bean>的<bean>就使用RootBeanDefinition表示。

AbstractBeanDefinition对两者共同的类信息进行抽象。

Spring通过BeanDefinition将配置文件中的<bean>配置信息转换为容器的内部表示，并将这些BeanDefiniton注册到BeanDefinitonRegistry中。Spring容器的**BeanDefinitionRegistry**就像是Spring配置信息的内存数据库，主要是以map的形式保存，后续操作直接从BeanDefinition- Registry中读取配置信息。它们之间的关系如图3-2所示。![1552470137471](E:\studydyup\notes\src\pic\1552470137471.png)

由此可知，要解析属性首先要创建用于承载属性的实例，也就是创建GenericBeanDefinition类型的实例。而代码createBeanDefinition(className, parent)的作用就是实现此功能。

BeanDefinitionParserDelegate.java

```
/**
 * Create a bean definition for the given class name and parent name.
 * @param className the name of the bean class
 * @param parentName the name of the bean's parent bean
 * @return the newly created bean definition
 * @throws ClassNotFoundException if bean class resolution was attempted but failed
 */
protected AbstractBeanDefinition createBeanDefinition(@Nullable String className, @Nullable String parentName)
      throws ClassNotFoundException {

   return BeanDefinitionReaderUtils.createBeanDefinition(
         parentName, className, this.readerContext.getBeanClassLoader());
}
```

BeanDefinitionReaderUtils.java

```java
/**
	 * Create a new GenericBeanDefinition for the given parent name and class name,
	 * eagerly loading the bean class if a ClassLoader has been specified.
	 * @param parentName the name of the parent bean, if any
	 * @param className the name of the bean class, if any
	 * @param classLoader the ClassLoader to use for loading bean classes
	 * (can be {@code null} to just register bean classes by name)
	 * @return the bean definition
	 * @throws ClassNotFoundException if the bean class could not be loaded
	 */
	public static AbstractBeanDefinition createBeanDefinition(
			@Nullable String parentName, @Nullable String className, @Nullable ClassLoader classLoader) throws ClassNotFoundException {

		GenericBeanDefinition bd = new GenericBeanDefinition();
		bd.setParentName(parentName);
		if (className != null) {
			if (classLoader != null) {
				bd.setBeanClass(ClassUtils.forName(className, classLoader));
			}
			else {
				bd.setBeanClassName(className);
			}
		}
		return bd;
	}
```

2．解析各种属性

当我们创建了bean信息的承载实例后，便可以进行bean信息的各种属性解析了，首先我们进入parseBeanDefinitionAttributes方法。parseBeanDefinitionAttributes方法是对element所有元素属性进行解析：

BeanDefinitionParserDelegate.java

```java
public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
      @Nullable BeanDefinition containingBean, AbstractBeanDefinition bd) {
   //scope与singleton两个属性只能指定其中之一，不可以同时出现，否则Spring将会报出异常
   if (ele.hasAttribute(SINGLETON_ATTRIBUTE)) {
      error("Old 1.x 'singleton' attribute in use - upgrade to 'scope' declaration", ele);
   }
   else if (ele.hasAttribute(SCOPE_ATTRIBUTE)) {
      bd.setScope(ele.getAttribute(SCOPE_ATTRIBUTE));
   }
   else if (containingBean != null) {
      // Take default from containing bean in case of an inner bean definition.
      //在嵌入beanDifinition情况下且没有单独指定scope属性则使用父类默认的属性
      bd.setScope(containingBean.getScope());
   }
   //解析abstract属性
   if (ele.hasAttribute(ABSTRACT_ATTRIBUTE)) {
      bd.setAbstract(TRUE_VALUE.equals(ele.getAttribute(ABSTRACT_ATTRIBUTE)));
   }
   //解析lazy-init属性
   String lazyInit = ele.getAttribute(LAZY_INIT_ATTRIBUTE);
   if (isDefaultValue(lazyInit)) {
      lazyInit = this.defaults.getLazyInit();
   }
   //若没有设置或设置成其他字符都会被设置为false
   bd.setLazyInit(TRUE_VALUE.equals(lazyInit));
   //解析autowire属性
   String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
   bd.setAutowireMode(getAutowireMode(autowire));
   //解析dependency-on
   if (ele.hasAttribute(DEPENDS_ON_ATTRIBUTE)) {
      String dependsOn = ele.getAttribute(DEPENDS_ON_ATTRIBUTE);
      bd.setDependsOn(StringUtils.tokenizeToStringArray(dependsOn, MULTI_VALUE_ATTRIBUTE_DELIMITERS));
   }
   //解析autowire-candidate属性
   String autowireCandidate = ele.getAttribute(AUTOWIRE_CANDIDATE_ATTRIBUTE);
   if (isDefaultValue(autowireCandidate)) {
      String candidatePattern = this.defaults.getAutowireCandidates();
      if (candidatePattern != null) {
         String[] patterns = StringUtils.commaDelimitedListToStringArray(candidatePattern);
         bd.setAutowireCandidate(PatternMatchUtils.simpleMatch(patterns, beanName));
      }
   }
   else {
      bd.setAutowireCandidate(TRUE_VALUE.equals(autowireCandidate));
   }

   if (ele.hasAttribute(PRIMARY_ATTRIBUTE)) {
      bd.setPrimary(TRUE_VALUE.equals(ele.getAttribute(PRIMARY_ATTRIBUTE)));
   }
   //解析init-method属性
   if (ele.hasAttribute(INIT_METHOD_ATTRIBUTE)) {
      String initMethodName = ele.getAttribute(INIT_METHOD_ATTRIBUTE);
      bd.setInitMethodName(initMethodName);
   }
   else if (this.defaults.getInitMethod() != null) {
      bd.setInitMethodName(this.defaults.getInitMethod());
      bd.setEnforceInitMethod(false);
   }

   if (ele.hasAttribute(DESTROY_METHOD_ATTRIBUTE)) {
      String destroyMethodName = ele.getAttribute(DESTROY_METHOD_ATTRIBUTE);
      bd.setDestroyMethodName(destroyMethodName);
   }
   else if (this.defaults.getDestroyMethod() != null) {
      bd.setDestroyMethodName(this.defaults.getDestroyMethod());
      bd.setEnforceDestroyMethod(false);
   }
   ///解析factory-method属性
   if (ele.hasAttribute(FACTORY_METHOD_ATTRIBUTE)) {
      bd.setFactoryMethodName(ele.getAttribute(FACTORY_METHOD_ATTRIBUTE));
   }
   //解析factory-bean属性
   if (ele.hasAttribute(FACTORY_BEAN_ATTRIBUTE)) {
      bd.setFactoryBeanName(ele.getAttribute(FACTORY_BEAN_ATTRIBUTE));
   }

   return bd;
}
```

我们可以清楚地看到Spring完成了对所有bean属性的解析，这些属性中有很多是我们经常使用的，同时我相信也一定会有或多或少的属性是读者不熟悉或者是没有使用过的，有兴趣的读者可以查阅相关资料进一步了解每个属性。

3．解析子元素meta

在开始解析元数据的分析前，我们先回顾一下元数据meta属性的使用。

```java
<bean id="myTestBean" class="bean.MyTestBean">
         <meta key="testStr" value="aaaaaaaa"/>
</bean>
```

并不会体现在MyTestBean的属性当中，而是一个额外的声明，当需要使用里面的信息的时候可以通过BeanDefinition的getAttribute(key)方法进行获取。

对meta属性的解析代码如下：

```java
public void parseMetaElements(Element ele, BeanMetadataAttributeAccessor attributeAccessor) {
		NodeList nl = ele.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			if (isCandidateElement(node) && nodeNameEquals(node, META_ELEMENT)) {
				Element metaElement = (Element) node;
				String key = metaElement.getAttribute(KEY_ATTRIBUTE);
				String value = metaElement.getAttribute(VALUE_ATTRIBUTE);
				BeanMetadataAttribute attribute = new BeanMetadataAttribute(key, value);
				attribute.setSource(extractSource(metaElement));
				attributeAccessor.addMetadataAttribute(attribute);
			}
		}
	}
```

4．解析子元素lookup-method

同样，子元素lookup-method似乎并不是很常用，但是在某些时候它的确是非常有用的属性，通常我们称它为获取器注入。引用Spring in Action中的一句话：获取器注入是一种特殊的方法注入，它是把一个方法声明为返回某种类型的bean，但实际要返回的bean是在配置文件里面配置的，此方法可用在设计有些可插拔的功能上，解除程序依赖。我们看看具体的应用。

①．首先我们创建一个父类。
package test.lookup.bean;

public class User {

```
 public void showMe(){
     System.out.println("i am user");
 }
```

}

②．创建其子类并覆盖showMe方法。
package test.lookup.bean;

public class Teacher extends User{
​     public void showMe(){
​             System.out.println("i am Teacher");
​     }
}
③．创建调用方法。
public abstract class GetBeanTest {

public void showMe(){
​     this.getBean().showMe();
 }
 public abstract User getBean();

 }

④．创建测试方法。
package test.lookup;

import org.Springframework.context.ApplicationContext;
import org.Springframework.context.support.ClassPathXmlApplicationContext;

import test.lookup.app.GetBeanTest;

public class Main {
​     public static void main(String[] args) {
​         ApplicationContext bf = 
​                 new ClassPathXmlApplicationContext("test/lookup/lookupTest.xml"); 
​         GetBeanTest test=(GetBeanTest) bf.getBean("getBeanTest");
​         test.showMe();
​     }
}

到现在为止，除了配置文件外，整个测试方法就完成了，如果之前没有接触过获取器注入的读者们可能会有疑问：抽象方法还没有被实现，怎么可以直接调用呢？答案就在Spring为我们提供的获取器中，我们看看配置文件是怎么配置的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.Springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.   
springframework.org/schema/beans/Spring-beans.xsd">
<bean id="getBeanTest" class="test.lookup.app.GetBeanTest">
             <lookup-method name="getBean" bean="teacher"/>
     </bean>

<bean id="teacher" class="test.lookup.bean.Teacher"/>
</beans>
```

在配置文件中，我们看到了源码解析中提到的lookup-method子元素，这个配置完成的功能是动态地将teacher所代表的bean作为getBean的返回值，运行测试方法我们会看到控制台上的输出：
i am Teacher
当我们的业务变更或者在其他情况下，teacher里面的业务逻辑已经不再符合我们的业务要求，需要进行替换怎么办呢？这是我们需要增加新的逻辑类：

package test.lookup.bean;

public class Student extends User {

```
 public void showMe(){
     System.out.println("i am student");
 }

```

同时修改配置文件：
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
​     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
​     xsi:schemaLocation="http://www.Springframework.org/schema/beans http://www.  
springframework.org/schema/beans/Spring-beans.xsd">

```
     <bean id="getBeanTest" class="test.lookup.app.GetBeanTest">    
         <lookup-method name="getBean" bean="student"/>
 </bean>

```

<bean id="teacher" class="test.lookup.bean.Teacher"/>
<bean id="student" class="test.lookup.bean.Student"/>
</beans>

再次运行测试类，你会发现不一样的结果：
i am Student

至此，我们已经初步了解了lookup-method子元素所提供的大致功能，相信这时再次去看它的属性提取源码会觉得更有针对性。

```java
/**
	 * Parse lookup-override sub-elements of the given bean element.
	 */
	public void parseLookupOverrideSubElements(Element beanEle, MethodOverrides overrides) {
		NodeList nl = beanEle.getChildNodes();
		for (int i = 0; i < nl.getLength(); i++) {
			Node node = nl.item(i);
			//仅当在Spring默认bean的子元素下且为lookup-method时有效
			if (isCandidateElement(node) && nodeNameEquals(node, LOOKUP_METHOD_ELEMENT)) {
				Element ele = (Element) node;
				//获取要修饰的方法
				String methodName = ele.getAttribute(NAME_ATTRIBUTE);
				//获取配置返回的bean
				String beanRef = ele.getAttribute(BEAN_ELEMENT);
				LookupOverride override = new LookupOverride(methodName, beanRef);
				override.setSource(extractSource(ele));
				overrides.addOverride(override);
			}
		}
	}
```

5．解析子元素replaced-method

我们可以看到无论是look-up还是replaced-method都是构造了一个MethodOverride，并最终记录在了AbstractBeanDefinition中的methodOverrides属性中。而这个属性如何使用以完成它所提供的功能我们会在后续的章节进行详细的介绍。

6.解析子元素constructor-arg

对构造函数的解析是非常常用的，同时也是非常复杂的，也相信大家对构造函数的配置都不陌生，举个简单的小例子：

```java
    ... ...
     <beans>  
    <!-- 默认的情况下是按照参数的顺序注入，当指定index索引后就可以改变注入参数的顺序 -->  
    <bean id="helloBean" class="com.HelloBean">  
        <constructor-arg index="0">  
            <value>郝佳</value>  
        </constructor-arg>  
        <constructor-arg index="1">  
            <value>你好</value>  
        </constructor-arg>  
    </bean>  
... ...
    </beans>



```

上面的配置是Spring构造函数配置中最基础的配置，实现的功能就是对HelloBean自动寻找对应的构造函数，并在初始化的时候将设置的参数传入进去。那么让我们来看看具体的XML解析过程。
对于constructor-arg子元素的解析，Spring是通过parseConstructorArgElements函数来实现的，具体的代码如下：

```java
public void parseConstructorArgElements(Element beanEle, BeanDefinition bd) {
         NodeList nl = beanEle.getChildNodes();
         for (int i = 0; i < nl.getLength(); i++) {
             Node node = nl.item(i);
             if (isCandidateElement(node) && nodeNameEquals(node, CONSTRUCTOR_ARG_ELEMENT)) {
                 //解析constructor-arg
                 parseConstructorArgElement((Element) node, bd);
             }
         }
}
```

这个结构似乎我们可以想象得到，遍历所有子元素，也就是提取所有constructor-arg，然后进行解析，但是具体的解析却被放置在了另个函数parseConstructorArgElement中，具体代码如下：

```java
public void parseConstructorArgElement(Element ele, BeanDefinition bd) {
         //提取index属性
         String indexAttr = ele.getAttribute(INDEX_ATTRIBUTE);
         //提取type属性
         String typeAttr = ele.getAttribute(TYPE_ATTRIBUTE);
         //提取name属性
         String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
         if (StringUtils.hasLength(indexAttr)) {
             try {
                 int index = Integer.parseInt(indexAttr);
                 if (index < 0) {
                     error("'index' cannot be lower than 0", ele);
                 }else {
                     try {
                         this.parseState.push(new ConstructorArgumentEntry(index));
                         //解析ele对应的属性元素
                         Object value = parsePropertyValue(ele, bd, null);
                         ConstructorArgumentValues.ValueHolder valueHolder = new   
ConstructorArgumentValues.ValueHolder(value);
                         if (StringUtils.hasLength(typeAttr)) {
                             valueHolder.setType(typeAttr);
                         }
                         if (StringUtils.hasLength(nameAttr)) {
                             valueHolder.setName(nameAttr);
                         }
                         valueHolder.setSource(extractSource(ele));
                         //不允许重复指定相同参数
                         if (bd.getConstructorArgumentValues(). hasIndexedArgumentValue   
(index)) {
                             error("Ambiguous constructor-arg entries for index " +   
index, ele);
                         }else {
                             bd.getConstructorArgumentValues(). AddIndexedArgumentValue   
(index, valueHolder);
                         }
                     }finally {
                         this.parseState.pop();
                         }
                 }
             }catch (NumberFormatException ex) {
                 error("Attribute 'index' of tag 'constructor-arg' must be an integer", ele);
             }
         }else {
             //没有index属性则忽略去属性，自动寻找
             try {
                 this.parseState.push(new ConstructorArgumentEntry());
                 Object value = parsePropertyValue(ele, bd, null);
                 ConstructorArgumentValues.ValueHolder valueHolder = new Constructor-   
ArgumentValues.ValueHolder(value);
                 if (StringUtils.hasLength(typeAttr)) {
                     valueHolder.setType(typeAttr);
                 }
                 if (StringUtils.hasLength(nameAttr)) {
                     valueHolder.setName(nameAttr);
                 }
                 valueHolder.setSource(extractSource(ele));
                 bd.getConstructorArgumentValues().addGenericArgumentValue(valueHolder);
             }
             finally {
                 this.parseState.pop();
             }
         }
}
```



 上面一段看似复杂的代码让很多人失去了耐心，但是，涉及的逻辑其实并不复杂，首先是提取constructor-arg上必要的属性（index、type、name）。
如果配置中指定了index属性，那么操作步骤如下。
1．解析Constructor-arg的子元素。
2．使用ConstructorArgumentValues.ValueHolder类型来封装解析出来的元素。
3．将type、name和index属性一并封装在ConstructorArgumentValues.ValueHolder类型中并添加至当前BeanDefinition的constructorArgumentValues的indexedArgumentValues属性中。
如果没有指定index属性，那么操作步骤如下。
1．解析constructor-arg的子元素。
2．使用ConstructorArgumentValues.ValueHolder类型来封装解析出来的元素。
3．将type、name和index属性一并封装在ConstructorArgumentValues.ValueHolder类型中并添加至当前BeanDefinition的constructorArgumentValues的genericArgumentValues属性中。
可以看到，对于是否制定index属性来讲，Spring的处理流程是不同的，关键在于属性信息被保存的位置。
那么了解了整个流程后，我们尝试着进一步了解解析构造函数配置中子元素的过程，进入parsePropertyValue：

7．解析子元素property

parsePropertyElement函数完成了对property属性的提取，property使用方式如下：

```xml
<bean id="test" class="test.TestClass">
<property name="testStr" value="aaa"/>
</bean>
```

或者

```java
<bean id="a">
         <property name="p">
             <list>
                 <value>aa</value>
                 <value>bb</value>
             </list>
         </property>
</bean>
```

解析过程

```java
public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
         NodeList nl = beanEle.getChildNodes();
         for (int i = 0; i < nl.getLength(); i++) {
             Node node = nl.item(i);
             if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
                 parsePropertyElement((Element) node, bd);
             }
         }

```

这个函数我们并不难理解，无非是提取所有property的子元素，然后调用parsePropertyElement处理，parsePropertyElement代码如下：

```java
public void parsePropertyElement(Element ele, BeanDefinition bd) {
         //获取配置元素中name的值
         String propertyName = ele.getAttribute(NAME_ATTRIBUTE);
         if (!StringUtils.hasLength(propertyName)) {
             error("Tag 'property' must have a 'name' attribute", ele);
             return;
         }
         this.parseState.push(new PropertyEntry(propertyName));
         try {
             //不允许多次对同一属性配置
             if (bd.getPropertyValues().contains(propertyName)) {
                 error("Multiple 'property' definitions for property '" + propertyName + "'", ele);
                 return;
             }
             Object val = parsePropertyValue(ele, bd, propertyName);
             PropertyValue pv = new PropertyValue(propertyName, val);
             parseMetaElements(ele, pv);
             pv.setSource(extractSource(ele));
             bd.getPropertyValues().addPropertyValue(pv);
         }
         finally {
             this.parseState.pop();
         }
}
```

可以看到上面函数与构造函数注入方式不同的是将返回值使用PropertyValue进行封装，并记录在了BeanDefinition中的propertyValues属性中。

8．解析子元素qualifier

对于qualifier元素的获取，我们接触更多的是注解的形式，在使用Spring框架中进行自动注入时，Spring容器中匹配的候选Bean数目必须有且仅有一个。当找不到一个匹配的Bean时，Spring容器将抛出BeanCreationException异常，并指出必须至少拥有一个匹配的Bean。
Spring允许我们通过Qualifier指定注入Bean的名称，这样歧义就消除了，而对于配置方式使用如：

```xml
<bean id="myTestBean" class="bean.MyTestBean">
    <qualifier type="org.Springframework.beans.factory.annotation.Qualifier" value="qf"/>
</bean>
```


其解析过程与之前大同小异，这里不再重复叙述。

##### 3.1.2　AbstractBeanDefinition属性

至此我们便完成了对XML文档到GenericBeanDefinition的转换，也就是说到这里，XML中所有的配置都可以在GenericBeanDefinition的实例类中找到对应的配置。

GenericBeanDefinition只是子类实现，而大部分的通用属性都保存在了AbstractBeanDefinition中，那么我们再次通过AbstractBeanDefinition的属性来回顾一下我们都解析了哪些对应的配置。







