# Core Technologies

Foremost amongst these is the Spring Framework’s Inversion of Control (IoC) container.  

## The IoC container
### 1.1. Introduction to the Spring IoC container and beans
IOC is also known as DI. Dependency Injection.  这是一个对象定义他们的依赖的过程，依赖就是需要协同的其他对象，只有通过构造函数，工厂方法参数或设置在实例上的属性在对象构造后或从工厂方法返回后。容器在创建这些bean时注入这些依赖，是一个完全反转的过程，bean自身通过直接创建类或通过例如Service Locator的机制控制依赖的实例化。  

Ioc容器的基础包为org.springframework.beans 和 org.springframework.context。BeanFactory接口提供了先进的配置机制能管理任何种类的对象。ApplicationContext是BeanFactory的子接口，它可以轻松和aop集成；信息资源管理（国际化）、事件发布；应用层特殊上下文例如WebApplicationContext。  

简而言之，BeanFactory提供配置框架和基础功能，ApplicationContext添加更多专业和特殊的功能。ApplicationContext是BeanFactory的超集，在本章专门用于描述IOC容器。  

Spring中，对象构成了应用的骨架，通过IOC容器管理，被称作beans。一个bean是一个实例化的，可组装的等由IOC管理的对象。否则，这只是应用中对象中的一个普通的一个。Beans和他们的依赖，体现在container使用的配置元数据中。  

### 1.2. Container overview
org.springframework.context.ApplicationContext代表IOC容器负责实例化、配置、组装上述的beans。通过读取配置元数据，container得到对象如何去实例化配置与组装的指令。配置元数据表现为XML、注解、代码。它允许你表达组成应用的对象和这些对象之间的依赖。  

Spring有几个开箱即用的ApplicationContext的实现，在单机应用通常创建一个ClassPathXmlApplicationContext或FileSystemXmlApplicationContext实例。  

在大多数应用场景，用户代码不需要显示去实例化一个或多个IOC container。  


![How　Spring works](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/container-magic.png)  

#### 1.2.1. Configuration metadata
如上图所示，Ioc容器消费了配置元数据；配置元数据描述了你作为应用开发者告诉spring container怎么在应用中实例化、配置、组装objects。  

传统配置使用xml，本章大多使用xml来传达ioc容器的概念与特性。  

Ioc容器与配置元数据是分离的，也有人用 Java-based configuration来配置元数据。  

Spring配置包含至少一个通常不是一个容器必须管理的bean的定义。 xml的bean在beans标签内，或用在一个@Configuration的类上用@Bean注解方法。
这些bean定义对应组成应用的实体。通常定义服务层对象，数据访问层对象，表现层比如Struts Action实例，基础设施对象比如Hibernate SessionFactories，JMS队列等等。通常不在容器配置细粒度的domain对象，因为这通常是daos和业务逻辑去创建和加载domain对象。但是你也可以用Spring集成Aspectj来配置IOC容器外创建的对象。See Using AspectJ to dependency-inject domain objects with Spring.

以下是xml配置元数据基础结构  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
id数据鉴别单个bean的定义。class属性用全限定名定义bean类型。id属性的值是指关联对象。see Dependencies查看协作对象。  

#### 1.2.2. Instantiating a container
实例化容器很简单。提供给ApplicationContext构造器的路径通常是资源字符串，它允许容器从各种外部资源比如本地文件系统、Java Classpath等加载配置元数据。  
```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

以下是service对象配置文件service.xml。  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```
以下是dao对象配置文件dao.xml。  
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的例子中，name属性元素对应JavaBean的属性，ref元素对应另一个bean的定义名字。  

##### Composing XML-based configuration metadata

Bean定义跨多个xml域非常有用。通常一个xml文件代表了一个逻辑层或模块。  

你可以使用application context constructor来从所有的xml片段加载bean的定义。 一种是像如上面例子手动加载多个资源文件。另一种方法是，使用import标签从其他文件加载bean的定义。如下  
```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

如上所示， services.xml、resources/messageSource.xml都是相对路径，最好不要加斜线。导入的文件内容，包含最高级的beans节点，必须是有效的符合Spring Scheme的bean定义。  

#### 1.2.3. Using the container

ApplicationContext是一个高级工厂接口，可以维护不同的beans以及它们的依赖的注册表。使用T getBean(String name, Class<T> requiredType)方法得到bean的实例。  
As follows  
```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```
最灵活的变体GenericApplicationContext 使用reader比如XmlBeanDefinitionReader代表xml文件：
```
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```
这样的reader delegates可以在同一个ApplicationContext混合匹配，从不同的配置源读取bean定义。  

你可以使用getBean得到bean的实例。ApplicationContext接口有一些其他得到对象的方法，但你的应用最好永远不用。其实，你的应用根本不该有getBean方法，从而根本不依赖Spring APIs。比如，Spring与web集成为各种web框架组件提供了依赖注入，比如Controllers，JSF-Managed Beans，允许你通过配置元数据在一个特定的Bean上声明依赖。  
### 1.3. Bean overview

一个Ioc容器管理一个或多个beans。这些beans是container通过提交的配置元数据创建的，比如xml中的beans标签。  

在容器本身内，这些bean的定义被表示为BeanDefinition objects，包含以下元数据：  

- 一个包限定的类名：通常定义的bean的实际实现类
- Bean行为配置元素，表明bean在container的行为（作用域，生命周期回调等等）
- 这个bean工作需要的其他的Beans的引用；这些引用也被叫做collaborators or dependencies。
- 在新建对象的其他的配置，比如，管理连接池的连接的数量，或者池的大小限制。  

这些元数据翻译成属性的集合组成每个bean的定义。

*Table 1. The bean definition*

|Property|Explained in|
|:--|:--|
|class                              |Instantiating beans|
|name                               |Naming beans|
|scope                              |Bean scopes|
|constructor argument               |Dependency injection|
|properties                         |Dependency Injection|
|autowiring mode                    |Autowiring collaborators|
|lazy-initialization mode           |Lazy-initialized beans|
|initialization method              |Initialization callbacks|
|destruction method                 |Destruction callbacks|

除了包含如何创建一个特定的bean的定义外，ApplicationContext的实现也允许由用户注册已经存在并在容器外部创建的对象。通过getBeanFactory()得到ApplicationContext的默认实现BeanFactory的DefaultListableBeanFactory。DefaultListableBeanFactory通过registerSingleton(..) 和 registerBeanDefinition(..)支持这种注册。然而，典型的应用程序只使用元数据定义的bean。  

**为了容器能在autowiring和其他自省步骤能正确解释bean，bean元数据和手动提供的单例实例应该尽早注册。虽然覆盖现有的元数据与单例实例在某种程度上得以支持，在运行时注册新bean（并发访问工厂）没有得到官方支持，可以导致并发访问异常和/或bean容器中不一致的状态**  

#### 1.3.1. Naming beans

每个bean有一个或多个标识符，在承载这些bean的容器内是唯一的。通常只有一个，如果有多个，额外的可被认为是别名。  

在xml配置元数据，使用id or/and name指定bean标识符。id允许你指定一个id。惯例这些名字是字母数字，也可能包含特殊符号。如果想向bean引入别名，也可以在name属性指定，通过用逗号分号或者空格分隔。3.1版本前，id属性被定义为xsd:ID类型，限制可能的字符。3.1后为xsd:string类型，但是bean的id由容器强制唯一，虽然不是由xml解析器去强制。  

你没有必要提供name或者id，若没有明确指出id和name，容器会产生一个唯一的name。然而，如果你想通过ref标签或者Service Loader风格查找引用他，你必须提供名字。使用inner beans和 autowiring collaborators可以让你不提供name。  

##### *Bean Naming Conventions*
*约定是命名时，使用实例字段名的标准Java约定。即bean名字以小写字母开头，然后驼峰命名。beans命名一致有助于阅读与理解，如果你使用aop，在把advice apply到一系列的beans时会很有用。*

**在classpath使用component scanning，Spring会为没有命名的组件生成名字。遵循以下原则：基本上取简单的类名把第一个字母改成小写。然而，不寻常的情况下，开头不止一个字母是大写的话，就会保留原始的形式。这是由java.beans.Introspector.decapitalize定义的。**  

##### Aliasing a bean outside the bean definition  

使用以下指定别名。  

```
<alias name="fromName" alias="toName"/>
```
*Java-configurationa参照Using the @Bean annotation*

#### 1.3.2. Instantiating beans

一个bean的定义实际上是创建一个或者多个对象的recipe。container被请求时，会查看一个命名过的bean的recipe，使用bean定义封装的配置元数据创建（或获取）实例对象。  
如果你使用基于xml的配置元数据，你可以在bean标签下的class属性指定对象类型。这个class属性，内部是在一个BeanDefinition实例的Class属性，通常是强制的。（对于异常，参考Instantiation using an instance factory method 和 Bean definition inheritance）。你可以用2种方式之一来使用Class属性：  

- 通常，在容器本身直接创建bean的情况下，通过其构造函数反射来创建bean，这种指定bean class的方式类似于java代码的new操作符。  
- 指定一个包含静态工厂方法的类，该方法被调用来创建对象，在不太常见的情况下，容器调用一个类上的静态工厂方法来创建bean。从静态工厂方法的调用返回的对象类型可能是相同的类或完全另一个类。  

*
Inner class names
如果你想为静态内部类定义bean，你必须使用静态内部类的binary name。  
比如，com.package.Foo中有个静态内部类Bar，bean定义的class属性应该是com.package.Foo$Bar，使用$分隔内部类。
*

##### Instantiation with a constructor

当你通过构造函数方法创建一个bean，所有普通类都被Spring兼容并可用。就是说被开发的类不需要实现任何特定接口，或被特殊方式编码。简单指明bean类就够了。但是，根据ioc类型，你可能需要一个默认（empty）constructor。  

IOC容器可管理任何你想管理的类，并不限于管理真正的JavaBean。大部分Spring用户倾向于实际的JavaBean，适当的getter和setter以及无参构造方法，你也可以在容器中有更多外来的非bean样式类，比如你要用一个历史遗留的完全不遵循Java规范的连接池，Spring也可以管理它。  

```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
有关向构造器提供参数的机制（如果需要的话）的详细信息，以及在构建对象之后设置对象实例属性，参照 Injecting Dependencies.

##### Instantiation with a static factory method

在定义用静态工厂方法创建的bean时，你使用class属性来指定包含静态工厂方法的类，一个名为factory-method的属性用来指定工厂方法本身的名称。你应该能够调用这个方法（稍后将使用可选的参数）并返回一个live对象，随后就被认为好像通过构造器创建的。这种定义的一种用法是使用遗留代码的静态工厂。  

下面的bean定义指定bean通过工厂方法创建。该定义没有指定返回对象的类型（类），只有包含工厂方法的类。在这个例子中，createInstance（）方法必须是一个静态方法。  
```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
关于向工厂方法提供（可选的）参数的机制，以及在从工厂返回对象之后设定对象实例属性的详细信息，see  Dependencies and configuration in detail.  

##### Instantiation using an instance factory method

类似于通过静态工厂方法进行实例化，实例工厂方法的实例化从容器中调用现有bean的非静态方法来创建一个新bean。使用这个机制，将class属性留空，在工厂bean属性中，指定当前（或父/祖先）容器中的bean名称，该容器包含要调用的实例方法来创建对象，用工厂方法属性设置factory-method。  
```
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以容纳不止一个工厂方法，如下所示  
```
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
这种方法表明，工厂bean本身可以通过依赖注入（DI）来管理和配置,See Dependencies and configuration in detail.  

*在Spring文档中，工厂bean指的是在Spring容器中配置的bean，它将通过实例或静态工厂方法创建对象。相比之下，FactoryBean（注意大小写）是指一个特定于spring的FactoryBean*

### 1.4. Dependencies

典型的企业应用程序不包括单个对象（或Spring用语中的bean）。即使是最简单的程序，也有一些对象协同工作让用户看起来是一个连贯的应用。下一节将解释如何从定义一系列独立的bean到对象协作来达成目标的应用。

#### 1.4.1. Dependency Injection

依赖注入（DI）是一个对象定义其依赖性的过程，也就是说，所使用的其他对象，只能通过构造器参数，工厂方法参数，，或者在从工厂方法构造或返回的对象实例上设置的属性。容器在创建bean时就会注入这些依赖。  该过程根本可逆，对应控制反转，bean本身通过直接构造类或Service Locator pattern来控制其依赖项的实例化或定位。  

代码使用DI原则更简洁，当对象被提供依赖项时，解耦就更有效了。对象不会查找其依赖，也不知道依赖的位置或类。因此，您的类会变得更容易测试，特别是当依赖项在接口或抽象基类时，这允许在单元测试中使用存根或模拟实现。  

DI主要存在于两个变体。 Constructor-based dependency injection and Setter-based dependency injection.  

##### Constructor-based dependency injection

基于构造器的DI是由container调用有一些参数的构造函数完成的，参数代表了依赖。调用带有特定参数的静态工厂方法来构造bean几乎是等价的，把对构造器的参数和静态工厂方法的参数看作近似。下面的例子展示了一个只能依赖于构造函数注入的类。请注意，该类没有什么特别之处，它是一个不依赖于容器特定接口、基类或注解的POJO。  
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

##### Constructor argument resolution

使用参数类型时，发生构造器参数匹配。如果bean定义的构造器参数没有歧义，那么bean定义的构造器参数的order，就是bean实例化的时候被提供给构造器的参数的order。  
```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```
假设Bar和Baz类与继承无关，则不存在潜在的歧义。因此，下面的配置可以很好地工作，并且您不需要在构造器-arg/元素中显式地指定构造器参数索引and/or类型。  
```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```
当另一个bean被引用时，类型是已知的，并且可以发生匹配（就像前面的例子一样）。当使用一个简单的类型时，比如value 是true，Spring不能确定值的类型，因此在没有帮助的情况下无法匹配类型。考虑下面的类：  

```
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### *Constructor argument type matching*
在前面的场景中，如果您使用type属性显式地指定构造器参数的类型，则容器可以使用与简单类型匹配的类型匹配。比如：
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

##### *Constructor argument index*
使用index属性显式指定构造器参数的索引。比如：
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的模糊性之外，指定索引还解决了构造器有相同类型的两个参数的歧义。注意，索引是基于0的。  
##### *Constructor argument name*

你也可以使用构造函数参数名来消除value的歧义。比如：  
```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
请记住，要使这个工作立即可用，您的代码必须使用debug flag进行编译，以便Spring能够从构造函数中查找参数名。如果您不能使用调试标志（或者不想）编译您的代码，那么您可以使用@ConstructorProperties JDK注释来显式地命名构造函数参数。样本如下：  
```
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

##### Setter-based dependency injection

基于Setter的DI是在容器调用无参构造或无参静态工厂方法实例化bean之后，调用setter方法完成的。  
下面的例子展示了一个只能使用纯setter注入的依赖注入的类。这个类是传统的Java类。它是一个不依赖于容器特定接口、基类或注解的POJO。  
```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

ApplicationContext支持基于构造器和Setter的两种DI。在一些依赖通过构造器注入后再使用基于setter的DI注入依赖也是支持的。你可以用BeanDefinition的形式配置依赖，同时使用PropertyEditor将属性从一种格式转换为另一种格式。然而，大多数Spring用户不会直接使用这些类（比如以编程的方式），而是用xml bean定义、注解组件（例如@Component, @Controller）、或有@Configuration注解的Java基类的方法上加@Bean。 这些source随后被内部转换成BeanDefinition的实例，并用于加载整个Spring IoC容器实例。  

##### **Constructor-based or setter-based DI?**

因为你可以混合使用基于constructor和setter的DI，对于强制依赖使用构造器注入可选依赖使用setter注入，这是一个很好的经验。注意，在setter方法上使用@required注释可以用来使属性成为必需的依赖项。  
Spring团队通常提倡构造器注入，因为它能够将应用程序组件作为不可变对象实现，并确保所需的相依性不是空的。而且，构造注入的组件总是返回完全初始化状态下的客户端（调用）代码。附带一点，大量的构造函数参数是一种糟糕的代码气味，这意味着类可能有太多的责任，应该重构，以更好地解决关注点分离问题。  

Setter注入应该主要只用于可选的依赖项，可以在类中分配合理的默认值。否则，必须在代码使用相依性的任何地方执行非null检查。etter注入的一个好处是，setter方法使该类的对象可以稍后重新配置或重新注入。因此，通过JMX mbean的管理是setter注入的一个引人注目的用例。  

对特定类使用最有效的DI方式。有时，在处理没有源代码的第三方类时，选择是你做出的。例如，如果一个第三方类没有公开任何setter方法，那么构造函数注入可能是惟一可用的DI形式。

##### Dependency resolution process

容器执行bean依赖性解析，如下所列：  

- ApplicationContext使用描述所有bean的配置元数据被创建和记载。配置元数据可以是xml，java代码，注解。
- 对于每个bean，它的依赖用属性，构造器参数，或静态工厂方法参数的形式来表达。当bean真正被创建时，依赖会提供给bean。
- 每个属性或构造器参数都是一个确切的值，在容器中被set或引用到另一个bean。
- 每个属性都是一个值，从特殊形式转化为属性或构造器参数特定的形式。默认下，Spring可以将字符串格式提供的值转换为所有内置类型，如int、long、String、boolean。

当容器被创建时，Spring容器验证每个bean的配置。然而，bean属性本身并没有设置，直到bean真正被创建为止。当容器被创建时，单例的bean和被设置成pre-instantiated（默认就是）的bean会被创建。Scope被定义在Bean scopes。否则，bean只有在被请求时才会被创建。随着bean的依赖关系及其依赖项的依赖（等等）被创建和分配，会产生一个beans的图状结构。注意 beans之间的resolution mismatches可能会稍后显现，也就是说，在第一次创建受影响的bean时。  

##### **Circular dependencies**

如果您主要使用构造函数注入，就有可能创建一个不可解析的循环依赖场景。Class A需要通过构造函数注入的类B的实例，而类B需要通过构造函数注入一个A类的实例。如果你为A类和B配置了bean，那么Spring IoC容器会在运行时检测这个循环引用，并抛出BeanCurrentlyInCreationException。  

一种可能的解决方案是由setter而不是构造函数来配置某些类。或者，避免构造器注入和只使用setter注入。换句话说，虽然不推荐使用setter注入，但您可以用setter配置循环依赖项。与典型的情况不同（没有循环依赖），bean A和bean B之间的循环依赖会迫使其中一个bean在完全初始化之前被注入到另一个bean中（一个典型的鸡/蛋场景）。  

你通常可以相信Spring会做正确的事。Spring会检测配置问题，比如在container加载时，会检测到引用不存在的bean，或环形依赖。当一个bean被确实创建，Spring尽可能晚得设置属性和解决依赖。这意味着，正确加载的Spring container之后，当你请求一个对象，这个对象创建的时候有问题或者它的依赖有问题，可能会产生异常。例如，由于缺少或无效的属性，bean抛出异常。这可能推迟隐藏了配置问题的显现，也是ApplicationContext的实现默认为pre-instantiate单例bean的原因。以预支的时间和内存为代价，在beans真正被需要前创建这些beans，会让你在创建ApplicationContext时发现配置问题，而不是创建之后。您仍然可以覆盖这个默认行为，这样单例bean就会延迟初始化，而不是预先实例化。  

如果没有循环依赖关系，当一个或多个协作的bean被注入到从属bean中时，每个协作bean在被注入到从属bean之前都是完全配置的。这意味着如果bean A对bean B有依赖性，那么Spring IoC容器在调用bean A的setter方法之前，完全配置了bean B。换句话说，bean被实例化（如果不是预先实例化的单例），它的依赖被set，并且相关的生命周期方法（如配 configured init method 或 InitializingBean callback method）被调用。  

##### Examples of dependency injection

下面是基于XML的setter的DI配置，

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```
在前面的例子中，setter被声明与XML文件中指定的属性相匹配。下面的基于构造器的DI：  
```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```
现在考虑一下这个例子的一个变体，在这里，Spring被告知要调用静态工厂方法来归还物件的一个实例，而不是使用构造函数。
```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
```
静态工厂的参数通过constructor-arg标签元素提供，和构造器实际使用的参数一致。工厂方法返回的类的类型不必与包含静态工厂方法的类相同，尽管在本例中是相同的。一个实例（非静态）工厂方法将以一种基本相同的方式使用（除了使用factory-bean属性而不是class属性），因此这里不会讨论细节。  

#### 1.4.2. Dependencies and configuration in detail

如前一节提到的，你可以定义bean属性和构造器参数作为其他被管理bean（协作bean）的引用，或作为内嵌值。xml配置元数据在property和constructor-arg节点支持子元素类型。

##### Straight values (primitives, Strings, and so on)

property标签的value属性用字符串形式指明属性或构造器参数。Spring的 conversion service 用来把String类型转换为确切类型的属性或参数。  
```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

下面的例子使用了p名称空间来实现更简洁的XML配置。
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```
这样的xml很简洁但容易发生拼写错误，除非你使用IDEA或者STS这样的IDE，会有提示与补全功能，这样的IDE也是十分推荐的。  

你也可以定义java.util.Properties的实例，比如：  
```
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
Spring容器使用JavaBeans PropertyEditor机制把value里面的文本转化为java.util.Properties实例。这是一个好的捷径，也是Spring团队支持在value节点上使用嵌套的几个地方之一。  

##### The idref element
idref节点是简单的错误校验的方式来把另一个容器中的bean的id（string类型，不是引用）转化成constructor-arg或者property节点。  

```
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

上面的bean定义片段与下列代码片段完全相同（在运行时）：
```
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用idref标签允许容器在部署时验证所引用的、命名的bean实际上存在。在第二种形式中，值传递给client类的targetName时，没有发生验证。当client bean被实际初始化时，只会发生拼写错误（很可能是致命的）。如果client bean是原型bean，拼写错误和导致的异常很可能在部署容器后很长一段时间才会发现。

**idref元素上的本地属性不再在4.0 bean xsd中得到支持，因为它不再为常规bean引用提供价值。在升级到4.0模式时，只需将现有的idref本地引用改为idref bean。**

一个常见的地方（至少在比Spring 2.0版本更早的版本中），在ProxyFactoryBean bean定义中，idref元素带来值是在AOP拦截器的配置中。当您指定拦截器名称时，使用idref元素可以防止您错误地拼写截取程序id。  

#### References to other beans (collaborators)

ref元素是一个constructor-arg或property元素内的最后一个元素。在这里，你将一个bean的指定属性的值设置为引用容器管理的另一个bean（一个协作者）的引用。被引用的bean是被设置属性的bean的依赖，在属性设置之前，它是根据需要初始化的。（如果协作者是单例bean，那么它可能已经被容器初始化了）。所有的引用最终都是对另一个对象的引用。Scoping和校验取决于是否使用bean, local 或 parent属性指明了另一个对象的bean, local, or parent。  

使用ref标签的bean属性是最通常的指定目标bean的方式，允许在同一个容器或父容器中创建对任何bean的引用，不管它是否在同一个XML文件中。ref标签的bean属性的值可能与目标bean的id属性相同，或与目标bean的name属性中的一个值相同。  

```
<ref bean="someBean"/>
```

通过parent指定的目标bean会产生一个在当前容器的父容器里面的bean的引用。parent的值可能与目标bean的id属性相同，或是目标bean的name属性中的一个值，而且目标bean必须在当前容器的父容器里。当你的容器有层级结构时，你可以使用这种bean引用的变体。或者你想通过一个和parent bean name相同的代理来包装一个在父容器中存在的bean。

```
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```
```
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

**ref元素上的local属性不再在4.0 bean xsd中得到支持，因为它不再为常规bean引用提供价值。在升级到4.0模式时，只需将现有的ref本地引用更改为ref bean。**

#### Inner beans

property或constructor-arg节点内的bean元素是一个inner bean。  
```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要id或name;如果指定了，容器也不会用这个值作为标识符。容器也会忽略创建的scope flag：内部bean总是匿名的，并且总是由外部bean创建。不可能把内部类注入到除了封闭bean的协作bean，或者独立访问内部bean。  

as a corner case， 从自定义scope接收销毁回调是可能的，例如，在一个单例bean包含一个请求域的内部bean：内部bean的创建将被绑定到包含它的bean，但销毁回调允许允许它参与请求域的生命周期。这不是一个常见常见；内部bean通常只是简单分享包含他们的bean的scope。  

#### Collections

在list，set，map，props元素，你分别设置Java Collection 类型的List，Set，Map和Properties。  
```
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

map键或值的值，或set值，也可以是以下元素中的任何一个：
```
bean | ref | idref | list | set | map | props | value | null
```

##### Collection merging

Spring容器还支持集合的合并。应用程序开发人员可以定义一个parent风格的list、map、set或props，并且有child风格的list，map，set和props继承和覆盖来自父集合的值。也就是说，child集合的值是合并父与子集合的元素的结果，child的集合元素覆盖了parent集合中指定的值。  

*关于合并的这一部分讨论了父子bean机制。不熟悉父子bean定义的读者可以阅读 Bean definition inheritance 相关部分。*  

下面的例子演示了集合合并：
```
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

注意到在child bean定义的property的props节点使用了merge=true。当child bean被容器解析和实例化，结果实例adminEmails有一个属性集，是merge了父子集合的结果。 

```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

子Properties集继承了所有parent的props，并且子集合会覆盖与父相同名称的value。  

这样的merge行为同样适用与list，map，set集合类型。在特别情况list元素下，语义和List 集合类型关联，就是说，维护了有序集合的值的概念；父类的值优于所有子list的值。Map, Set, 和Properties没有顺序的说法。因此无序语义对容器内部使用的相关的map、set和Properties实现类型的基础有效。  

##### Limitations of collection merging 

你不能merge不同的集合类型，如果这样做就会抛出异常。合并属性必须在较低的、继承的、子定义上指定;在父集合长指定merge是多余的，不会有期望的合并。  

##### Strongly-typed collection 

通过在Java 5中引入泛型类型，您可以使用强类型集合。也就是说，它可以声明一个集合类型，它只能包含字符串（例如）。如果你在使用Spring去注入一个强类型集合的依赖，您可以利用Spring的类型转换支持，这样您的强类型集合实例的元素在被添加到集合之前就被转换成适当的类型。  

```
public class Foo {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

当foo bean的accounts属性被准备注入时，关于强类型的元素类型的泛型信息Map<String, Float>会被反射而可用。因此，Spring的类型转换设施识别出各种各样的Float值元素， 9.99, 2.75,3.99这样的string值被转换成Float。  

#### Null and empty string values 

Spring为属性提供空的参数，就像空字符串一样。下列基于xml的配置元数据片段将电子邮件属性设置为空字符串值（""）。  

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

前面的例子相当于下面的Java代码：

```
exampleBean.setEmail("");
```

null元素处理null值，例如。
```
bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

上面的配置相当于下面的Java代码：  

```
exampleBean.setEmail(null);
```

#### XML shortcut with the p-namespace  

p名称空间使您能够使用bean元素的属性，而不是property节点，来描述你的属性值或/和协作beans。  

Spring支持带有namespaces的可扩展配置形式，是基于xml形式定义。本章中讨论的bean配置格式是在XML模式文档中定义的。然而，p-namespace不是在XSD文件中定义的，并且只存在于Spring的核心中。  

下面的例子展示了两个相同结果的XML片段：第一个使用标准XML格式，第二个使用p名称空间。

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="foo@bar.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="foo@bar.com"/>
</beans>
```

这个例子展示了在bean定义中p名称空间中的一个电子邮件的属性。这将告诉Spring包含一个property声明。如前面提到的，p-namespace没有shema（xsd）定义，所以你可以将属性的名称设置为属性名。  

下一个例子包括两个bean定义，它们都有对另一个bean的引用：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```

正如您所看到的，这个例子不仅有使用p名称空间的属性值，也使用一种特殊的格式来声明属性引用。第一个bean定义用property name="spouse" ref="jane" 来创建了jane的引用。 第二个使用p:spouse-ref="jane"作为一个属性做了同样的事情。在这个例子中spouse是一个property名称，-ref表明了这不是一个实际的值，而是一个对其他bean的引用。  

**p名称空间不像标准XML格式那样灵活。例如，这种声明引用的形式和以Ref结尾的属性冲突，而XML不会。我们建议您仔细选择您的方法并将其传达给您的团队成员，来避免同时使用所有三种方法的XML文档。**  

#### XML shortcut with the c-namespace 

和XML shortcut with the p-namespace相似，c-namespace在Spring3.1引进。允许使用内联属性来配置构造器参数，而不是嵌套的构造-arg元素。  

让我们看看使用c名称空间的Constructor-based dependency injection：  

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>

    <!-- traditional declaration -->
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
        <constructor-arg value="foo@bar.com"/>
    </bean>

    <!-- c-namespace declaration -->
    <bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>

</beans>
```

c名称空间使用与p相同的约定，来通过构造参数的name设置值。同样，它也需要被声明，即使它不是在XSD模式中定义的（但是它存在于Spring核心中）。  

对于罕见的构造函数参数名称不可用的情况（通常字节码在没有debugging information情况下被编译），我们可以使用参数索引的fallback。  

```
<!-- c-namespace index declaration -->
<bean id="foo" class="x.y.Foo" c:_0-ref="bar" c:_1-ref="baz"/>
```

**由于XML语法，索引符号需要"_"开头作为XML属性的name，不能用数字开头（尽管一些ide允许数字开头）**  

在实践中，构造函数解析机制在匹配参数方面是相当有效的，因此，除非您真正需要，我们建议使用名称符号来完成您的配置。  

#### Compound property names 

当你设置bean属性时，你可以使用复合或嵌套属性名，只要路径的所有组件除了最终属性名都不是空的。考虑下面的bean定义。  

```
<bean id="foo" class="foo.Bar">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

foo bean有一个fred属性，fred有一个bob属性，bob有一个sammy属性，最终sammy被设置为123。 为了能够工作，foo的fred和fred的bob在bean被构造后一定不为null，否则抛出NullPointerException。  

### 1.4.3. Using depends-on 

如果一个bean是另一个bean的依赖，通常意味着这个bean被设置为另一个bean的属性。：通常，您可以使用基于xml的配置元数据中的ref来实现这一点。然而，有时bean之间的依赖关系不那么直接;例如，一个类中的静态初始化程序需要被触发，比如数据库驱动注册。depends-on属性可以显式地强制在使用该元素的bean之前初始化一个或多个bean。下面的例子使用depends-on属性来表达对单个bean的依赖：   

```
下面的例子使用依赖项属性来表达对单个bean的依赖：
```

为了表示对多个bean的依赖，提供一个bean名称列表，作为依赖项属性的值，用逗号、空格和分号作为有效分隔符：  

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

*在bean定义中depends-on属性，可以指定初始化时间依赖性，并且仅在单例bean的情况下，对应的销毁时间依赖性。给定bean定义了depends-on属性的依赖beans，beans会先销毁，早于给定bean本身的销毁。因此depends-on也可以控制关闭顺序。*  

### 1.4.4. Lazy-initialized beans  

默认情况下，ApplicationContext的实现会急切地创建和配置所有单例bean，作为初始化过程的一部分。一般来说，这种预实例化是满足需要的，因为配置或周围环境中的错误会立即被发现，而不是几小时甚至几天后。当这种行为不满足需要的时候，您可以防止单例bean的预实例化将bean定义标记为延迟初始化一个延迟初始化的bean告诉IoC容器在第一次请求时创建一个bean实例，而不是在启动时。在XML中，这种行为是由bean节点上的lazy-init属性控制的;例如:  

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当前面的配置被ApplicationContext使用，当ApplicationContext启动时，名为lazy的bean不会预先实例化，not.lazy会预先实例化。  

然而，当一个延迟初始化的bean是一个没有被延迟初始化的单例bean的依赖项时，ApplicationContext在启动时创建 lazy-initialized的bean，因为它必须满足单例的依赖性。lazy-initialized的bean被注入到在其他的地方的单例bean，所以它不是lazy-initialized的了。  

您还可以在容器级别上通过使用default-lazy-init属性来控制容器级别的延迟初始化;例如:

```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 1.4.5. Autowiring collaborators  

Spring容器可以自动连接协作bean之间的关系。您可以通过检查ApplicationContext的内容，允许Spring自动为您的bean解析合作者（其他bean）。Autowiring有如下优势：  

- 自动装配可以显著减少指定属性或构造函数参数的需要。（其他机制，如本章其他地方讨论的bean模板，在这方面也很有价值）
- 当您的对象不断演进时，自动装配可以更新配置。例如，如果您需要向类添加依赖项，依赖可以自动满足，而不需要修改配置。因此，自动装配在开发过程中尤其有用，当代码库变得更加稳定时，不否认切换到显式wiring的选择。  

在使用基于xml的配置元数据时，您可以为bean定义指定autowire模式，使用bean节点的autowire属性。自动装配功能有四种模式。您可以为每个bean指定autowire，因此可以选择要自动连接的哪一个。  

*Table 2. Autowiring modes*

|Mode|Explanation|
|:--|:--|
|no|（默认）不autowiring。Bean引用必须通过ref元素来定义。对于较大的部署，不建议更改默认设置，因为指定合作者显式地提供了更大的控制和清晰性。在某种程度上，它记录了系统的结构。|
|byName|通过属性名byName。Spring寻找一个与需要autowire属性同名的bean。例如，如果一个bean定义被设置为通过name autowire，并且它包含一个master属性（也就是说，它有一个setMaster（..）方法），Spring寻找一个名为master的bean定义，并用它来设置属性|
|byType|如果容器中存在一个属性类型的bean，则允许属性被autowire。如果不止一个存在，就会抛出一个致命的异常，表示你不能用byType的autowire来设置bean。如果没有匹配的bean，什么也不会发生;这个属性没有设置。|
|constructor|类似于byType，但是应用于构造函数参数。如果容器中的构造器参数类型没有一个bean，那么就会出现一个致命错误|

通过byType或constructor的autowire模式，你可以连接数组和类型-集合。在这种情况下，所有在容器内的与预期的类型匹配的autowire候选者都被提供以满足依赖。如果预期的键类型是string，您可以自动连接强类型map（带有泛型）。autowired Map的值会包括所有满足预期类型的实例，键会包含和值对应的bean名字。  

您可以将autowire与依赖检查结合起来，这是在autowire完成后执行的。  

#### Limitations and disadvantages of autowiring  

自动装配在整个项目中一致使用时效果最好。如果一般不使用自动装配，开发人员可能会混淆使用它来只连接一个或两个bean定义。  

考虑一下自动装配的局限性和缺点：  

- property和constructor-arg的显示依赖设置总是覆盖autowire。你不能自动连接所谓的简单属性，例如原语、字符串和类（以及此类简单属性的数组）。限制是故意的。  
- autowire不如显示wire准确。尽管如上表所示，Spring谨慎地避免猜测可能会产生意想不到的结果，您的spring管理对象之间的关系不再被明确地记录下来。  
- 不会再提供给从Spring容器生成文档工具wiring的信息。
- 容器中多个beans的定义，可以匹配setter方法或构造器参数指定的类型。对于数组，集合，或Maps，这并不一定是一个问题。但是对于期望是单个值的依赖，这种模糊性不是随意解决的。如果没有唯一的bean定义，则抛出一个异常。  

在后面的场景中，您有几个选项：  

- 放弃autowire使用显示wire
- 设置autowire-candidate属性为false来避免autowire，如下一节的描述。
- 指定一个单例bean定义为primary候选，通过设置bean节点的primary为true。
- 使用基于注解的配置实现更细粒度的控制， 如Annotation-based container configuration描述。

#### Excluding a bean from autowiring

在每个bean的基础上，您可以将bean从autowire中排除。在Spring的XML形式，设置bean节点的autowire-candidate为false；容器会使这样的bean定义对autowire不可用（包括注解风格配置，如@autowired）。  

**autowire-candidate属性被设计成只影响type-based的autowire，它不影响显式的引用，即使指定的bean没有被标记为autowire候选，它也会得到解决。因此，如果name匹配，由name的autowire还是会注入一颗bean。**  

你也可以基于对bean名称的模式匹配来限制autowire候选。最顶级beans节点在他的default-autowire-candidates属性内接收一个或多个匹配。例如限制Repository结尾的任何bean，提供一个```*Repository```。要提供多个匹配，使用逗号分隔列表。对bean定义的autowire-candidate属性地显示true或false值是总是优先的，对于这样的bean，模式匹配规则并不适用。  

这些技术对于那些您永远不希望通过自动装配将其注入到其他bean中的bean非常有用。这并不意味着被排除的bean本身不能使用autowire进行配置。而是被配置bean的自身不是一个对于其他beans的autowire的候选。  

#### 1.4.6. Method injection

在大多数应用程序场景中，容器中的大多数bean都是单例。当单例bean需要与另一个单例bean合作，或者非单例bean需要与另一个非单例bean合作，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean生命周期不同时，就会出现问题。假设单例bean需要使用no-singleton（prototype）bean B，可能是A的每个方法的调用上。容器只创建单例bean A一次，因此只有一个机会来设置属性。每次需要时，容器都不能提供带有bean B的bean A的新实例。  

一个解决方案是放弃一些控制反转。你可以实现ApplicationContextAware接口，来 make bean A aware of the container，并通过 making a getBean("B") call to the container每次A需要的时候去请求bean B实例（通常是new）。下面是这种方法的一个例子：  

```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不可取的，因为业务代码be aware of 框架并与Spring框架耦合在一起。方法注入，是Spring IoC容器的一个稍微高级的特性，允许以一种干净的方式处理这个用例。  

##### Lookup method injection  

查找方法注入是容器能重写容器管理beans的方法的能力，来返回容器中查找的另一个named bean。他的查找通常涉及到一个原型bean，就像在前一节中描述的场景中一样。Spring框架通过使用来自CGLIB库的字节码生成来实现这种方法注入，通过动态生成覆盖该方法的子类。  

- 为了这种动态子类能工作，Spring bean容器将子类化的类不能被标记为final，被覆盖的方法也不可以是final。  
- 单元测试一个有抽象方法的类需要你自己子类化这个类，并且提供抽象方法的stub implementation。  
- 具体的方法对于component scanning也是必须的，它需要pick up具体的类。  
- 一个关键的限制就是，查找方法不适用于工厂方法，特别是在配置类中不适用@bean方法，因为容器不负责在这种情况下创建实例，因此不能在运行时创建一个运行时生成的子类。  

查看前面代码片段中的CommandManager类，您可以看到Spring容器将动态覆盖createCommand（）方法的实现。您的CommandManager类不会有任何Spring依赖项，正如在重新修订的示例中所看到的那样：  

```
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在client类包含被注入的方法（这个例子的CommandManager），要注入的方法需要以下形式的签名： 

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);  
```

如果方法是abstract的，动态生成的子类会实现这个方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。例如：  

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

这个bean被标识为commandManager，每当需要一个myCommand bean的新实例时，调用它自己的方法createCommand（）。您必须小心地将myCommand bean部署为prototype，如果myCommand真的被需要。如果它是一个单例，那么每次都会返回myCommand bean的同一个实例。  

或者，在基于注解的component模型中，您可以通过@lookup注释声明一个查找方法：  

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更傻瓜的做法，根据目标bean根据查找方法的声明返回类型来解决：  

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

注意你通常声明@lookup的方法是有具体子实现的方法，为了使它们与Spring的组件扫描规则兼容，在这种规则中，抽象类在默认情况下会被忽略。在显式注册或显式导入bean类的情况下，此限制不适用。  

**另一种访问不同范围的目标bean的方法是ObjectFactory/ Provider注入点。看 Scoped beans as dependencies。感兴趣的读者可以看org.springframework.beans.factory.config包下ServiceLocatorFactoryBean的使用。**  

##### Arbitrary method replacement  

与查找方法注入相比，一种不太有用的方法注入方式是用另一种方法的实现去替换一个被管理bean的任意方法。用户可以安全地跳过本节的其余部分，直到实际需要功能为止。  

使用基于xml的配置元数据，replaced-method节点可以替换任意部署bean的实现方法，考虑下面的类，有一个computeValue方法，我们想要覆盖它：  

```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

一个实现org.springframework.beans.factory.support.MethodReplacer接口的类提供新的方法定义。  

```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}

```

用于部署原始类并指定方法覆盖的bean定义如下：  


```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

你可以使用replaced-method节点下的一个或者多个arg-type元素来指出被覆盖的方法的方法签名。只有方法被重载并且类中有多个变体才需要参数的签名。为了方便，参数的类型字符串可能是完全限定类型名称的子串。例如，下列所有都匹配java.lang.string：  

```
java.lang.String
String
Str
```

因为参数数量经常足够区分每种可能的选择，这个快捷的方式通过允许您的输入最短字符串来匹配参数类型，可以节省大量打字。  

### 1.5. Bean scopes

当你创建一个bean定义时，你创建一个recipe来由那个bean定义创建实际实例。 一个bean定义是一个recipe的概念很重要，这意味着，你可以从一个recipe创建很多对象实例，就像一个class。  

你不仅可以控制不同的依赖和配置值，这些将被设置到将由bean定义创建的对象中，还可以控制特定bean定义创建的对象的scope。这种方法灵活且强大，您可以通过配置选择创建的对象的范围，而没有必要从java类级别搞对象的scope。bean可以被定义部署在许多scope中的一个：开箱即用，Spring框架支持六个作用域，其中4个只有当你用web-aware ApplicationContext才有效。  

下面的作用域是开箱即用。你也可以自定义scope。  

*Table 3. Bean scopes*

|Scope|Description|
|:--|:--|
|singleton|(默认)将单个bean定义作用于每个Spring IoC容器的单个对象实例。|
|prototype|将单个bean定义作用于任意数量的对象实例。|
|request|一个bean定义作用于一个HTTP request生命周期；也就是说，每个HTTP请求都有自己的bean实例，它是从single bean定义的后面创建的。只在web-aware 的Spring ApplicationContext有效|
|session|一个bean定义作用于一个HTTP session的生命周期；只在web-aware 的Spring ApplicationContext有效|
|application|一个bean定义作用于一个ServletContext，只在web-aware 的Spring ApplicationContext有效|
|websocket|一个bean定义作用于一个WebSocket生命周期，只在web-aware 的Spring ApplicationContext有效|

**在spring3.0中，thread scope是可用的，但是在默认情况下是不注册的。要了解更多信息，请参阅SimpleThreadScope的文档。有关如何注册这个scope或任何其他自定义scope，请参阅Using a custom scope。**  

#### 1.5.1. The singleton scope  

只有单例bean的一个共享实例被管理，所有带id或ids对beans的请求匹配到这个bean定义，其结果都指向那个由Spring容器返回的特定bean实例。  

换句话说，当你定义一个bean定义时它的作用域是一个单例，Spring IoC容器根据bean定义创建了对象的exactly一个实例。这个单一实例存储在这些单例beans的缓存中，所有随后的请求和引用都将返回缓存的对象。  

![](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/singleton.png)  

Spring singleton bean的概念和在Gang of Four（GoF）模式手册中定义的单例模式不同。GoF Singleton 硬编码了一个对象的范围，这样每个类加载器就会创建一个特定类的一个实例。Spring singleton的scope最好描述为每个容器和每个bean。这意味着，如果你在一个Spring容器中为某个特定的类定义一个bean，然后Spring容器根据bean定义创建有且仅一的类实例。singleton scope是Spring默认scope。要将bean定义为XML中的单例，您可以这样写，例如：

```
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) 等价且多余 -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

#### 1.5.2. The prototype scope 

非单例，prototype scope导致每产生一个特定bean的请求，都会创建一个新的实例。也就是说，bean被注入到另一个bean中，或者您通过容器上的getBean（）方法调用请求它。作为原则，对有状态的beans设置prototype scope，无状态的设置为prototype scope。  

*下图展示了Spring的prototype scope。数据访问对象（DAO）通常不会被配置为prototype scope，因为典型的DAO不具有任何会话状态；重用singleton图的核心对作者来说很轻松*  

![](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/prototype.png)

在xml中定义bean为prototype scope,如下：  

```
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```

和其他scopes相比，Spring没有管理prototype bean的完整的生命周期：容器初始化、配置等来组装一个prototype对象，把它交给client，然后就没有这个prototype实例的进一步记录。因此，尽管initialization 生命周期回调方法在所有的不管是什么scope的对象上都被调用，在prototype的情况下，配置的destruction生命周期回调并没有被调用。客户端代码必须清理prototype-scoped对象，来释放prototype bean(s)持有的资源。要得到Spring容器来释放prototype-scoped beans持有的资源，尝试使用自定义bean post-processor，它包含了对需要清理的bean的引用。  

在一些方面，Spring容器在prototype-scoped bean上的角色，是Java new 操作符的一个替代。所有的生命周期管理都必须由客户端处理。（关于Spring容器中bean的生命周期的详细信息， see  Lifecycle callbacks）  

#### 1.5.3. Singleton beans with prototype-bean dependencies  

当你使用一个单例bean，他的依赖有prototype beans，请注意，*依赖关系在实例化时得到解决*。因此，如果你往一个singleton-scoped里注入prototype-scoped的bean，一个新的prototype-scoped的bean将会实例化然后注入到singleton-scope的bean中。提供到singleton-scope bean的prototype实例是独有的（每次都是新的）实例。  

然而，假如你想在runtime时候让singleton-scoped bean重复得到prototype-scoped bean的新实例，你不可以把prototype-scoped bean注入到singleton-scoped bean，因为注入仅发生一次，是当Spring容器实例化singleton bean并解析并注入它的依赖项的时候。如果您在运行时不止一次需要一个prototype bean的新实例，请参阅 Method injection。  

#### 1.5.4. Request, session, application, and WebSocket scopes  

request, session, application, and websocket scopes仅仅在你使用web-aware ApplicationContext（比如XmlWebApplicationContext）时可用。如果你在常规的Spring IoC容器使用这些scopes，比如ClassPathXmlApplicationContext，IllegalStateException会被抛出并说明未知scope。  

##### Initial web configuration 

为了能支持request, session, application, and websocket 级别(web-scoped beans)的scope的beans，在定义bean之前，需要进行一些较小的初始配置。（这个初始设置不需要标准scopes，singleton和prototype。）  

如何完成这个初始设置取决于特定Servlet环境。  

如果你在Spring Web MVC中访问scoped beans，实际上，是在一个Spring DispatcherServlet处理的请求之内，不需要特殊的设置：DispatcherServlet已经expose所有相关的状态。  

如果您使用一个Servlet 2.5 web容器，在Spring的DispatcherServlet之外处理请求（例如，在使用JSF或Struts时），您需要注册org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于 Servlet 3.0+，WebApplicationInitializer接口会自动做这件事。或者对于较老的容器，在web应用程序的web.xml上添加以下声明:  

```
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者，如果你的监听器设置有问题，考虑使用Spring的RequestContextFilter。过滤器映射依赖于周围的web应用程序配置，因此您必须适当地更改它。  

```
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

DispatcherServlet, RequestContextListener, and RequestContextFilter都做了完全相同的事情，即把HTTP请求对象绑定到服务该请求的Thread。这使得在调用链上的request- and session-scoped的bean变得可用。  

##### Request scope  

请考虑下面的bean 定义XML配置：  

```
<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
```

Spring容器使用LoginAction的bean定义，每次HTTP request时，创建一个LoginAction bean的instance。也就是说loginAction的作用域是HTTP request级别的。你可以随意改变你想要创建的实例的内部状态，因为同一个loginAction bean定义创建的其他的实例，不会看到你这个bean的状态变化。它们是特定于单个request的。当请求完成处理时，作用于request范围的bean被丢弃。  

当使用annotation-driven组件或者Java Config， @RequestScope注解用于将一个组件定义为request scope。  

```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

##### Session scope  

请考虑下面的bean 定义XML配置：  

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

Spring容器使用UserPreferences的bean定义，创建一个UserPreferences bean的实例，用于一个HTTP Session的生命周期。也就是说，userPreferences bean是HTTP Session级别范围有效。request-scoped的beans，你可以任意改变你想创建的bean实例的内部状态，注意到其他other HTTP Session实例用的也是userPreferences bean定义创建的实例，不会看到状态的变化，因为它们对于HTTP Session是独立的。当HTTP会话最终被丢弃时，HTTP Session scope的bean也被丢弃。  

当使用annotation-driven组件或者Java Config， @SessionScope注解用于将一个组件定义为session scope。  

```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

##### Application scope  

请考虑下面的bean 定义XML配置：  

```
<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
```

Spring容器使用AppPreferences的bean定义，创建一个AppPreferences bean的实例，用于一整个web应用的生命周期。也就是说，AppPreferences bean在ServletContext 级别范围有效，存储为常规的ServletContext属性。这有点类似于Spring的singleton bean，但在两个重要方面有所不同：它是每个ServletContext的单例，不是每个Spring 'ApplicationContext'（在任何给定的web应用程序中可能有几个ApplicationContext），它实际上是exposed因而对ServletContext属性可见。

当使用annotation-driven组件或者Java Config， @ApplicationScope注解用于将一个组件定义为application scope。  

```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

##### Scoped beans as dependencies  

Spring IoC容器不仅管理对象（beans）的实例化，也会wire up collaborators（dependencies）。如果您想要注入一个HTTP request scope的bean到另一个长期存在的bean中，你可以选择注入一个aop代理代替scoped bean。也就是说，你需要注入一个代理对象，它暴露了与作用域对象相同的public接口，但也可以从相关scope（比如HTTP request）拿到真实的目标对象，或者委托方法调用得到真实对象。  

**你也可以singleton scope的beans间使用`<aop:scoped-proxy/>`,通过这个引用然后通过一个可序列化的中间代理，从而通过反序列化重新得到目标单例bean。**  

**当对prototype的bean声明`<aop:scoped-proxy/>`，共享代理上的每个方法调用，都会导致一个方法调用所导向的新的目标实例的创建。**  

**此外，scoped proxies不是唯一的方式，以一种生命周期安全的方式，访问shorter scopes的beans。你也可以简单地声明你的注入点（例如，constructor/setter参数，或者autowired field）作为```ObjectFactory<MyTargetBean>```，允许getObject（）调用在每次需要时按需检索当前实例——而不需要保留实例或单独存储它。**  

**作为一个扩展的变体，您可以声明`ObjectProvider<MyTargetBean>`，它提供了几个额外的访问形式，包括`getIfAvailable`和`getIfUnique`。**  

**JSR-330在此叫`Provider`，使用`Provider<MyTargetBean>`声明和一个对应的`get()`调用，用于每一次获取尝试。see Using JSR 330 Standard Annotations for more details on JSR-330 overall。**


下面例子中的配置只是一行，但是理解“为什么”以及它背后的“如何”是很重要的。  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

通过插入一个子`<aop:scoped-proxy/>`节点到scoped bean的定义（see  Choosing the type of proxy to create 和 XML Schema-based configuration）来创建一个这样的代理。为什么对request, session and custom-scope levels的scoped的beans的定义需要`<aop:scoped-proxy/>`节点呢？让我们检查下面的单例bean定义，并将其与上述scoped beans进行对比（注意，下面的userPreferences bean定义是不完整的）。  

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

前例中，单例bean userManager 注入了一个 HTTP Session-scoped bean的引用，即userPreferences。这里的重点是userManager bean是一个单例对象：它只会在每个容器中实例化一次，它的依赖关系（在本例中只有一个，userPreferences bean）也只注入一次。这意味着userManager bean只会在完全相同的userPreferences对象上操作，也就是最初被注入的那个对象。  

这不是你想要的行为，当你把一个shorter-lived scoped bean注入到一个longer-lived scoped bean，例如，把一个HTTP Session-scoped的协作bean作为一个singleton bean的依赖。相当于，你需要一个userManager对象，用于一个HTTP Session的整个生命周期，你需要一个特定于HTTP Session的userPreferences对象。因此容器创建一个对象，它暴露出和UserPreferences class相同的public接口，可以从scoping mechanism（HTTP request, Session, etc.）获取到真正的UserPreferences对象。容器将这个代理对象注入userManager bean中，bean并不知道UserPreferences的引用是一个代理。在此例，当UserManager实例调用DI注入的UserPreferences对象的方法时，它实际上是在代理上调用一个方法。然后代理从HTTP会话中（在本例）获取真实的UserPreferences对象，将方法调用委托给真正接收到的UserPreferences对象。  

因此，当你把request- 和 session-scoped的beans注入到协作对象，您需要以下、正确和完整的配置：  

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

###### Choosing the type of proxy to create 

默认情况下，当Spring容器为带有`<aop:scoped-proxy/>`节点的bean创建代理时，*会创建一个CGLIB-based class proxy*  

**CGLIB代理只拦截公共方法调用！不要用这样的代理调用非公共方法;它们不会被委托给实际作用域的目标对象。**  

你也可以配置Spring容器，来为这些scoped beans创建standard JDK interface-based proxies，通过指定`<aop:scoped-proxy/>`节点的 proxy-target-class 属性为false来做。使用JDK interface-based proxies 意味着在应用程序classpath中不需要额外的库来影响这种代理，但是，它也意味着作用域bean的类必须至少实现一个接口，并且所有注入了scoped bean的协作bean，必须通过它的一个接口引用bean。  

```
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.foo.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择class-based或interface-based代理的更详细信息，see Proxying mechanisms。  

#### 1.5.5. Custom scopes  

bean作用域机制是可扩展的;你可以定义自己的scopes，设置重新定义已存在的scopes，尽管后者被认为是糟糕的实践，并且你不能重写内置的singleton和prototype scopes。  

##### Creating a custom scope  

为了整合你的自定义scope，你需要实现org.springframework.beans.factory.config.Scope接口，如本节描述。关于如何实现你自己的scopes，see Scope implementations，由Spring Framework自己提供，以及the Scope javadocs详细解释了你需要实现的方法。  

Scope 接口有4个方法，来从scope得到对象，从scope移除以及允许他们被销毁。  

下面的方法从underlying scope返回对象。例如，session scope的实现返回session-scoped的bean。（如果不存在，方法在将其绑定到session为了以后引用，然后返回一个bean的新实例。）  

```
Object get(String name, ObjectFactory objectFactory)
```

下面的方法从underlying scope删除对象。例如session scope的实现从underlying session移除underlying session bean。对象应该返回，但是如果没有找到指定名字的对象，则返回null。  

```
Object remove(String name)
```

下面的方法注册了一个scope应该执行的回调，当其被销毁或者scope的特定对象被销毁。有关销毁回调的更多信息，请参阅javadocs或Spring scope implementation。  

```
void registerDestructionCallback(String name, Runnable destructionCallback)
```

下面的方法获得 underlying scope的对话标识符。这个标识符对于每个scope都是不同的。对于session scoped的实现， 这个标识符可以是session identifier。  

```
String getConversationId()
```

##### Using a custom scope 

在您编写和测试一个或多个自定义scope实现之后，你需要让Spring容器知道你的scope。下面的方法是用Spring容器注册新scope的核心方法。  

```
void registerScope(String scopeName, Scope scope);
```

这个方法在ConfigurableBeanFactory接口，在大部分的ApplicationContext的具体实现上都可用，通过Spring的BeanFactory的属性。  

`registerScope(..)`方法的第一个参数是与scope相关的唯一的name；Spring容器中这样的名字例如singleton和prototype。`registerScope(..)`方法的第二个参数是自定义Scope实现的实例，你希望注册和使用它。  

假设你写了自定义Scope实现，然后像下面一样注册。  

**下面的例子用了SimpleThreadScope scope，Spring包含了它，但是默认没有注册。对于你自己的自定义scope实现注册操作是一样的。**  

```
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后你可以创建bean定义，遵循你的自定义scope的规则。  

```
<bean id="..." class="..." scope="thread">
```

有了自定义scope，你不局限于程序化注册scope。你也可以声明式地注册scope，使用CustomScopeConfigurer类。  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="bar" class="x.y.Bar" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="foo" class="x.y.Foo">
        <property name="bar" ref="bar"/>
    </bean>

</beans>
```

**当你把`<aop:scoped-proxy/>`放到FactoryBean的实现，这是FactoryBean本身被scoped，不是`getObject()`返回的对象。**  

### 1.6. Customizing the nature of a bean  

#### 1.6.1. Lifecycle callbacks  

与容器管理bean的生命周期交互，你可以实现Spring的InitializingBean和DisposableBean接口。容器对前者调用`afterPropertiesSet()`，对后者调用`destroy()`，来允许bean在初始化和销毁时执行某些动作。  

**JSR-250，@PostConstruct 和 @PreDestroy注解，在现代java应用，普遍被认为是接收生命周期回调的最佳实践。用这些注释意味着您的bean没有耦合到Spring特定的接口。For details see @PostConstruct and @PreDestroy  如果你不想用JSR-250注解，但你仍然想消除耦合，考虑使用对象定义元数据的 init-method 和 destroy-method。**  

Spring内部使用BeanPostProcessor的实现，来处理任何它可以找到的回调接口，并调用合适的方法。如果你需要Spring没有提供开箱即用的自定义特性和其他生命周期行为，你可以自己实现BeanPostProcessor。更多信息，see  Container Extension Points。  

除了初始化和销毁的回调，Spring管理的对象也可能实现了Lifecycle接口，这样这些对象就可以参与由容器自身生命周期驱动的启动和关闭的过程。  

生命周期回调接口在本节中描述。  

##### Initialization callbacks  

org.springframework.beans.factory.InitializingBean接口允许bean在bean上的所有必要属性都由容器设置之后执行初始化工作。InitializingBean接口只指定了一个方法：  

```
void afterPropertiesSet() throws Exception;
```

建议您不使用InitializingBean接口，因为它不必要地将代码与Spring结合在一起。可以选择用 @PostConstruct注解，或者指定POJO初始化方法。在基于xml的配置元数据的情况下，使用init-method属性指定一个返回类型为void的无参方法。使用Java Config，使用@Bean的initMethod属性，see Receiving lifecycle callbacks。如下：  

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

和下面一样：  

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是不要把代码和Spring耦合。

##### estruction callbacks  

实现org.springframework.beans.factory.DisposableBean接口，允许bean在容器包含它时，被销毁得到一个回调。DisposableBean接口指定了单个方法：  

```
void destroy() throws Exception;
```
建议您不使用DisposableBean接口，因为它不必要地将代码与Spring结合在一起。可以使用@PreDestroy注解，或者由bean定义指定一个方法。xml配置元数据，使用bean节点的destroy-method属性。Java Config使用@Bean的destroyMethod属性， see Receiving lifecycle callbacks。 例如以下定义：  

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

和以下同义：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements DisposableBean {

    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是不要把代码和Spring耦合。  


**bean节点的destroy-method属性，可以被指定一个特殊的（推断）值，可以指示Spring自动检测指定bean上的public的close或shutdown方法（任何实现了java.lang.AutoCloseable或者java.io.Closeable的类都能匹配）。这个特殊的值，也可以被设置在beans的default-destroy-method属性上，将这个行为应用到一组beans中（see Default initialization and destroy methods）。注意这是Java config的默认行为**  

##### Default initialization and destroy methods  

当你写不使用Spring特定的InitializingBean和DisposableBean回调接口的初始化和销毁方法回调，你通常将方法命名为 `init()` `initialize()` `dispose()`等等。理想情况下，这样的生命周期回调方法在项目中标准化，所有开发人员使用相同一致的方法名。  

你可以配置Spring容器在每个bean上查找被命名为初始化和销毁的方法。这意味着，作为开发者，写应用classes，使用叫`init()`的初始化方法，不需要在每个bean的定义都写上init-method="init"。当bean被创建，Spring容器调用那个方法（如前面描述的契约）。这个特性也对初始化和销毁方法回调强制了一致性的命名。  

假设你的初始化回调方法叫`init()`，销毁回调方法叫`destory()`。你的类将类似如下类：  

```
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```
<beans default-init-method="init">

    <bean id="blogService" class="com.foo.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

顶级节点beans的default-init-method的存在，使IOC容器识别beans上的叫init的方法，使之作为初始化方法回调。当一个bean被创建和组装时，如果bean类有这样的方法，那么它将在适当的时候被调用。  

你可以用类似的方式配置销毁方法回调（这是在XML），使用beans节点的default-destroy-method属性。  

当存在的bean classes有和约定不通过的回调方法，你可以通过指定bean的init-method和destroy-method属性覆盖掉默认的。  

Spring容器保证在一个bean被提供所有的依赖后，其配置的初始化回调方法会被立即调用。因此，初始化回调是在raw bean引用上被调用的，这意味着AOP拦截器等等还没有应用到bean中。目标bean*首先*被完全创建，然后带有拦截器的AOP代理（例如）会被applied。如果目标bean和代理是分开定义的，您的代码甚至可以绕过代理，与原始目标bean直接交互。因此，把拦截器应用到init method会导致不一致，因为这样做会使目标bean的生命周期和它的代理/拦截器耦合，当你的代码直接与原始目标bean交互时，会产生奇怪的语义。  

##### Combining lifecycle mechanisms  

自Spring2.5，你有3种方式控制bean生命周期行为：InitializingBean 和 DisposableBean接口；自定义`init()`和`destroy()`方法； @PostConstruct 和 @PreDestroy注解。您可以组合这些机制来控制给定的bean。  

**如果为bean配置了多个生命周期机制，并且每个机制都配置了不同的方法名，那么每个配置的方法都按照下面列出的顺序执行。然而，如果配置了相同的名字，例如，多个机制配置init()为初始化方法，该方法执行一次，如前一节所述。**  

为同一bean配置的多个生命周期机制，具有不同的初始化方法，如下被调用：

- @PostConstruct注解的方法。
- InitializingBean回调接口定义的afterPropertiesSet()
- 自定义的init()方法。  

销毁方法以同样的顺序被调用，

- @PostConstruct注解的方法。
- DisposableBean回调接口定义的destroy()
- 自定义的init()方法。  

##### Startup and shutdown callbacks  

Lifecycle接口定义了任何对象其自身生命周期需要的必要的方法（例如，启动和停止一些后台进程）。   
```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该接口。然后，当ApplicationContext本身接收开始和停止信号时，例如在运行时停止/重启的场景，他会把这些调用串联到所有Context内定义的Lifecycle的实现。这是通过委托给一个LifecycleProcessor完成的：  

```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

注意到LifecycleProcessor接口本身就是Lifecycle接口的扩展。它还添加了另外两种方法来响应正在刷新和关闭的context。  

**注意常规的`org.springframework.context.Lifecycle`接口只是一个简单的显示启动/关闭通知的约定，并不意味在context刷新时自动启动。考虑使用`org.springframework.context.SmartLifecycle`接口代替，来对特定bean自动启动进行细粒度控制（包括启动阶段）。同时注意，在bean销毁前，停止信号不能被保证：在常规shutdown，所有的Lifecycle beans在一般的销毁回调被传播之前，会首先收到终止通知 。然而，在一个context的生命周期，或者在丢弃的刷新尝试的热刷新上，只有销毁方法被调用。**  



启动和关闭调用的顺序可能很重要。如果两个对象之间存在"depends-on" ，依赖别人的一方，会在它的依赖之后启动，并在它的依赖之后停止。然而，有时直接依赖关系是未知的。您可能只知道某种类型的对象应该在另一种类型的对象之前启动。在这种情况下，`SmartLifecycle `接口定义是一种选择，它的`getPhase() `方法定义在其父接口上：  

```
public interface Phased {

    int getPhase();
}
```

```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，lowest phase的对象先启动，停止时，顺序相反。因此，实现了`SmartLifecycle`的对象，它的`getPhase`方法`Integer.MIN_VALUE`会是第一个启动最后一个销毁。在spectrum的另一端，phase的值为`Integer.MAX_VALUE`表明对象最后start，最先stop。考虑phase值时，必要知道任何普通的`Lifecycle`对象（没有实现`SmartLifecycle`接口）的默认值是0。因此，任何负值都表明对象在这些标准组件之前启动（在其后销毁），对正值是相反的。  

由你所见，`SmartLifecycle`接口定义的方法接受回调。任何实现必须在其自身的shutdown过程完成后，调用callback的run方法。这使必要时异步shutdown成为可能，因为默认的`LifecycleProcessor`的实现`DefaultLifecycleProcessor` ，会等待每个phase的对象组的timeout值，来调用回调方法。默认的 per-phase timeout为30s。你可以通过在context内定义一个叫“lifecycleProcessor”的bean来override默认的lifecycle processor实例。如果您只想修改超时，那么定义以下内容就足够了： 

```
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

正如提到的，`LifecycleProcessor`接口也为刷新和关闭context定义了回调方法。后者将简单地驱动shutdown过程，好像`stop()`方法被显示调用了，但是当context关闭时它就会发生。另一方面，“刷新”回调则支持`SmartLifecycle` beans的另一个特性。 context被刷新时（所以对象被初始化和实例化之后），回调会被调用，在那时，默认的lifecycle processor 会检查每个`SmartLifecycle`对象`isAutoStartup()`方法返回的布尔值。如果是“true”，对象就会在那时启动，而不是等待显示调用context或者其自身的`start()`方法（不像context刷新，context start对于标准context实现不自动发生。）。phase的值和depends-on的关系相同方式的启动顺序，如上所述。

##### Shutting down the Spring IoC container gracefully in non-web applications

**这部分只适用于非web应用程序。web-based的ApplicationContext的实现 ，当相关的web应用程序关闭时，已经有了适当的代码来关闭Spring IoC容器。**

如果你在非web端应用环境，比如，富客户端桌面环境；你可以用JVM注册一个shutdown钩子。从而优雅地关闭并调用单例beans的相关销毁方法，来释放所有的资源。当然，您仍然必须正确地配置和实现这些销毁回调。 

调用`ConfigurableApplicationContext`接口的`registerShutdownHook()`方法来注册钩子：  

```
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

#### 1.6.2. ApplicationContextAware and BeanNameAware 

当ApplicationContext创建一个实现`org.springframework.context.ApplicationContextAware`接口的对象实例，实例会提供给ApplicationContext一个引用。

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

beans会编程式地控制创建它们的`ApplicationContext`，通过`ApplicationContext`接口，或者转换引用到这个借口的子类，比如`ConfigurableApplicationContext`，它暴露了额外的功能。一个功能是编程式地得到其他的beans。有时这个会有用；然而，通常你需要避免使用，因为它把代码和Spring耦合了，不符合IOC风格。`ApplicationContext`的其他方法提供了访问文件资源，发布应用实践，访问数据源。这些额外特性描述在 [Additional capabilities of the ApplicationContext](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-introduction) 。

自从Spring2.5，autowirng是ApplicationContext得到引用的另一种选择。"traditional" 的`constructor`和`byType`autowiring模式，可以为ApplicationContext的各自的构造器参数和setter方法参数提供依赖。为了获得更大的灵活性，包括自动连接字段和多种参数方法的能力，可以使用新的基于注解的自动连接特性。查看 [@Autowired](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation) 寻找更多信息。

当ApplicationContext创建一个实现`org.springframework.beans.factory.BeanNameAware`接口的类，这个类提供了其他关联对象定义里该name的引用。

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

回调在所有bean属性设置后被调用，但是在初始化回调之前，比如`InitializingBean`的*afterPropertiesSet* 或者一个自定义的init-method。

#### 1.6.3. Other Aware interfaces

除了以上的`ApplicationContextAware` 和 `BeanNameAware` ，Spring提供了一系列`Aware`接口来允许bean指示容器其需要某一个 *infrastructure*  依赖。最重要的`Aware`接口如下，作为常规-name指示了依赖类型：

| Name                             | Injected Dependency                                          | Explained in…                                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ApplicationContextAware`        | Declaring `ApplicationContext`                               | [ApplicationContextAware and BeanNameAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | Event publisher of the enclosing `ApplicationContext`        | [Additional capabilities of the ApplicationContext](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `BeanClassLoaderAware`           | Class loader used to load the bean classes.                  | [Instantiating beans](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-class) |
| `BeanFactoryAware`               | Declaring `BeanFactory`                                      | [ApplicationContextAware and BeanNameAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BeanNameAware`                  | Name of the declaring bean                                   | [ApplicationContextAware and BeanNameAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware) |
| `BootstrapContextAware`          | Resource adapter `BootstrapContext`the container runs in. Typically available only in JCA aware `ApplicationContext`s | [JCA CCI](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/integration.html#cci) |
| `LoadTimeWeaverAware`            | Defined *weaver* for processing class definition at load time | [Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) |
| `MessageSourceAware`             | Configured strategy for resolving messages (with support for parametrization and internationalization) | [Additional capabilities of the ApplicationContext](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-introduction) |
| `NotificationPublisherAware`     | Spring JMX notification publisher                            | [Notifications](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/integration.html#jmx-notifications) |
| `ResourceLoaderAware`            | Configured loader for low-level access to resources          | [Resources](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#resources) |
| `ServletConfigAware`             | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring `ApplicationContext` | [Spring MVC](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc) |
| `ServletContextAware`            | Current `ServletContext` the container runs in. Valid only in a web-aware Spring `ApplicationContext` | [Spring MVC](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc) |

请再次注意，这些接口的使用将您的代码与Spring API绑定在一起，并且不遵循IOC风格。因此，建议对需要对容器进行编程访问的基础设施bean进行推荐 。 

### 1.7. Bean definition inheritance

子bean继承父bean的元数据，并根据需要复写一些值，是一种模版的形式。

如果你编程式使用`ApplicationContext`，子bean定义由`ChildBeanDefinition`类表示。大多数用户不在这个级别操作，而是声明式地配置bean定义比如`ClassPathXmlApplicationContext`。

```
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

子类一定能接受父类的类型。子类继承了scope，构造器参数，属性值。余下的属性从子bean定义取：*depends on*, *autowire mode*, *dependency check*, *singleton*, *lazy init*。

如果父类没指定类：

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父bean不能自己实例化，因为它是不完整的，并且被显示指明abstract。这样的bean定义只能作为子ban定义的模版。如果你尝试`getBean`会返回错误。容器内部的`preInstantiateSingletons()`方法会忽略标记为abstract的beans定义。

**`ApplicationContext`在默认情况下预先实例化所有单例。所以如果你想让一个父bean只是当做一个模版，它的定义也指定了class，你必须把abstract属性设置为true，否则application context会确实（尝试）预实例化abstract bean**

## 1.8. Container Extension Points

ioc容器可以插入特殊的集成接口进行扩展。

### 1.8.1. Customizing beans using a BeanPostProcessor

`BeanPostProcessor`接口定义了*callback methods*，你可以实现它（或覆盖容器默认的）来提供自己的实例化逻辑，依赖解决逻辑等等。如果您想在Spring容器完成实例化后实现一些自定义逻辑 ，您可以插入一个或多个`BeanPostProcessor`实现 。

你可以配置多个`BeanPostProcessor`实例，也能通过order属性控制执行顺序。只有`BeanPostProcessor`实现了`Ordered`接口，才能设置这个属性。如果你自己实现也需要考虑实现Ordered。更多细节查看`BeanPostProcessor` and `Ordered` 文档。也可以看 [programmatic registration of `BeanPostProcessor`s](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors) 。


- **`BeanPostProcessor`s  是在对象实例上操作的；也就是说ioc容器实例化了bean后，`BeanPostProcessor`s 才工作。**
- **`BeanPostProcessor`s 是每个容器scoped的。只有你使用container继承才有关系。如果你在一个容器定义了`BeanPostProcessor`，他只会在这个容器post-process。一个容器的beans定义，不会被定义在其他容器的`BeanPostProcessor` post-process，除非两个容器都是同一继承的一部分。**
- **要改变bean定义，需要使用 [Customizing configuration metadata with a BeanFactoryPostProcessor](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-extension-factory-postprocessors)描述的`BeanFactoryPostProcessor`**  

`org.springframework.beans.factory.config.BeanPostProcessor`接口就包含2个方法。bean后处理程序通常检查回调接口，或者用代理包装bean。为了提供代理包装逻辑，一些Spring AOP基础设施类被实现为bean后处理程序。 

`ApplicationContext`自动检测所有实现了`BeanPostProcessor`的beans的元数据，把他们注册成post-processors 然后在bean创建后被调用，Bean post-processors可以在容器中像其他beans一样被部署。

注意，当在configuration类上使用`@bean`工厂方法声明BeanPostProcessor时， 工厂方法的返回类型应该是其自身的实现，或者至少是`org.springframework.beans.factory.config.BeanPostProcessor `接口，这清楚地表明了post-processor特性 否则，ApplicationContext将无法在完全创建之前自动检测它。 因为一个`BeanPostProcessor`为了apply到其他context中的beans的实例化，需要尽早实例化，这种预先类型检测是必要的。

**以编程方式注册BeanPostProcessors**

**虽然推荐通过application context自动检测注册`BeanPostProcessor`，使用`ConfigurableBeanFactory`的`addBeanPostProcessor`方法也是可以做到的。这在注册前评估条件逻辑很有用，甚至在一个层次的context复制bean post processors。这里是注册顺序决定执行顺序。还要注意的是，通过编程方式注册的beanpost处理器总是在通过自动检测注册之前进行处理，而不考虑任何显式的排序。**

**BeanPostProcessors and AOP auto-proxying** 

**实现了`BeanPostProcessor`接口的类是特殊的，会被容器特别对待。所有引用的`BeanPostProcessor`s 和beans都在启动时实例化，作为application context启动的一个特殊阶段。然后，所有`BeanPostProcessor`s 会被注册成排好序的形式，然后apply到容器的所有further bean。因为AOP auto-proxying 被实现为`BeanPostProcessor`本身，这些类直接引用的`BeanPostProcessor`s 和beans都被资格auto-proxying，因此不会织入aspect到其中。**

**对于任何这样的Bean，您都应该看到一个信息日志消息：“Bean foo不适合由所有`BeanPostProcessor`接口进行处理（例如：没有资格进行自动代理）。**

 **注意，如果你有使用autowiring或者`@Resource（可能会回到autowiring）`wire到`BeanPostProcessor`的beans，当搜索类型匹配依赖性候选者时，Spring可能会访问意想不到的bean ，因此，使它们没有资格进行自动代理或其他类型的bean后处理。 例如，你有一个依赖注解为`@Resource`，field/setter 的name没有被声明，Spring会通过type匹配来访问其他beans。**

下面的例子展示了如何编写、注册和使用beanpost处理器：  

#### Example: Hello World, BeanPostProcessor-style

第一个例子说明了基本用法。BeanPostProcessor实现，它调用每个bean的toString（）方法，因为它是由容器创建的，并将产生的字符串打印到系统控制台。 

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        http://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

注意InstantiationTracingBeanPostProcessor 的定义，没有name，因为这是个像其他bean一样被dependency-injected 的bean。

下面的简单Java应用程序执行前面的代码和配置： 

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```

前面的应用程序的输出类似于以下内容： 

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

#### Example: The RequiredAnnotationBeanPostProcessor

使用回调接口或注解，和自定义`BeanPostProcessor`的实现一起使用是扩展Spring IoC容器的常用方法。 一个例子是`RequiredAnnotationBeanPostProcessor`，ships with Spring distribution，确保被注解的JavaBean属性必须被设置一个属性。

###  1.8.2. Customizing configuration metadata with a BeanFactoryPostProcessor

下一个扩展点为`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。这个借口的语义和`BeanPostProcessor`类似，有一个主要的不同，它作用于bean的配置元数据；就是说，ioc容器允许一个`BeanFactoryPostProcessor` 在容器实例化除了``BeanFactoryPostProcessor`的beans之前读取并且可能改变配置元数据。

可定义多个``BeanFactoryPostProcesso`，通过设置order属性控制执行顺序。必须实现了Odered接口。

**如果你想要改变实际的bean实例（例如，由配置元数据创建的对象 ）然后你需要使用`BeanPostProcessor`。虽然从技术上讲，可以在`BeanFactoryPostProcessor`中使用bean实例（比如，using `BeanFactory.getBean()` ），但是这样会导致bean过早实例化，违反标准容器生命周期。这可能会导致负面的副作用，比如绕过bean post处理 。**

**此外，`BeanFactoryPostProcessor`也是每个per-container scoped的。** 

一个  bean factory post-processor 当声明在application context里是自动执行的，目的是改变配置元数据。Spring包含许多预定义的 bean factory post-processors ，比如`PropertyOverrideConfigurer` and `PropertyPlaceholderConfigurer` 。也可自定义 `BeanFactoryPostProcessor` ，例如，注册自定义属性编辑器 。

也是自动检测的，适时使用。

与`BeanPostProcessor`s一样，您通常不希望为了懒加载初始化而配置`BeanFactoryPostProcesso`s  。如果没有其他Bean引用 `Bean(Factory)PostProcessor` ，post-processor 不会实例化。因此，将其标记为惰性初始化将被忽略 ，即使您将`Bean(Factory)PostProcessor` 默认的-lazy-init属性设置为true，也会被实例化。 

##### Example: the Class name substitution PropertyPlaceholderConfigurer

使用`PropertyPlaceholderConfigurer`来从文件读取属性，可以定制某些属性，避免修改容器的XML文件。

考虑以下基于xml的配置元数据片段，`DataSource`  使用place holder被定义。这个例子展示了从外部属性文件配置的属性。 在运行时，PropertyPlaceholderConfigurer被应用到元数据中，该元数据将取代数据源的一些属性。 

```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

实际的值来自标准Java属性格式的另一个文件： 

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

他的`PropertyPlaceholderConfigurer`在大多数属性和bean定义的属性中检查占位符。此外，可以定制占位符前缀和后缀。 

使用Spring 2.5提供的`context`  namespace，可以用专用的配置元素配置属性占位符 。一个或多个位置可以作为一个逗号分隔的列表在location属性中提供。

```
<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
```

`ropertyPlaceholderConfigurer` 不仅在你指定的属性文件中寻找属性。 默认情况下，如果不能在指定的属性文件中找到属性，它也会检查Java系统属性。你可以设置配置器的systemPropertiesMode属性定制这种行为，有以下支持的3种整数值： 

- never(0)：从不检查系统属性
- fallback(1)：指定属性文件里没有的属性会去检查系统属性。默认值
- override(2)：在指定特定属性文件之前，先检查系统属性。这允许系统属性覆盖其他属性源。

参阅PropertyPlaceholderConfigurer javadoc查看更多信息。

你可以用PropertyPlaceholderConfigurer代替类名，这在运行时pick up一个特定实现类会有用。例如：

```
<bean class=
"org.springframework.beans.factory.config.PropertyPlaceholderConfigurer
">
<property name="locations">
<value>classpath:com/foo/strategy.properties</value>
</property>
<property name="properties">
<value>custom.strategy.class=com.foo.DefaultStrategy</value>
</property>
</bean>
<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果类运行时不能解析为有效类，当他即将被created，resolution of bean fails，这发生在application context对于non-lazy-init bean的preInstantiateSingletons()阶段发生的。

##### Example: the PropertyOverrideConfigurer

PropertyOverrideConfigurer是另一个bean factory post-processor，与PropertyPlaceholderConfigurer类似，与后者不同的是，原始的定义可以有默认值或者根本没有对bean属性的值。如果一个覆盖的属性文件没有一个对特定的bean属性的入口，使用默认的context定义。

注意bean定义是不知道自己被覆盖的，对于xml定义文件，override configurer的使用不明显。多个PropertyOverrideConfigurer的实例情况下，对同个属性定义了不同的值，最后一个值会因为覆盖机制胜出。

属性文件配置行采用这种格式： 

```
beanName.property=value
```

例如：

```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

这个例子文件可以与包含一个名为dataSource的bean的容器定义一起使用，它有driver和url属性。

复合属性也支持，只要路径上每个组件期望的最终属性已经是non-null的（大概由构造器初始化）。在这个例子：

```
foo.fred.bob.sammy=123
```

foo bean的fred的bob的sammy属性设置为123。

**指定覆盖值通常是字面值；不会翻译成bean引用。这个约定在xml bean定义中原始value指定了bean引用也是适用的。**

使用Spring 2.5的context命名空间，使用一个专门的配置节点来配置属性覆盖。

```
<context:property-override location="classpath:override.properties"/>
```

### 1.8.3. Customizing instantiation logic with a FactoryBean

实现了`org.springframework.beans.factory.FactoryBean`接口的对象，是其自身的工厂。

`FactoryBean`接口是Spring IoC容器的实例化逻辑中的一个可插拔的点 。如果你有复杂的初始化逻辑代码相对于冗长的xml，Java代码会更好，你可以创建自己的`FactoryBean`，在那个类内部写复杂初始化逻辑，把自定义的`FactoryBean`插入到容器里。

`FactoryBean`提供了3个方法：

- `Object getObject()`：返回该工厂创建的对象的一个实例，该实例可以共享，这取决于该工厂是否返回single或prototype。
- `boolean isSingleton()`：如果`FactoryBean`返回的是单例就返回true，否则false。
- ``Class getObjectType()`：返回`getObject()`方法返回的对象类型，如果类型不能提前得知返回null。

FactoryBean概念和接口在Spring框架内的许多地方使用;有超过50个与Spring本身的FactoryBean接口的实现。

当你请求容器得到一个确切的`FactoryBean`实例本身，而不是它产生的bean，在调用`ApplicationContext `的`getBean()`方法时，在bean的id前面加$符号。给出一个id为myBean的bean，调用`getBean("myBean")`返回`FactoryBean`返回的product，但调用`getBean("&myBean") `返回的是`FactoryBean`本身。

## 1.9. Annotation-based container configuration

```html
			Are annotations better than XML for configuring Spring?
看情况，各有优缺点。由开发人员决定那种更适合。不管选择什么，Spring都可以同时容纳两种样式，甚至可以将它们混合在一起。值得提出的是，通过JavaConfig选项，Spring允许以非侵入方式使用注释，不涉及目标组件源代码和工具方面的内容，所有的配置样式都由Spring Tool Suite支持。
```

在相关类、方法或字段声明上使用注释将配置移动到组件类本身中。如 [Example: The RequiredAnnotationBeanPostProcessor](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp-examples-rabpp), 使用`BeanPostProcessor`与注释一起使用是扩展Spring IoC容器的常用方法 。例如Spring 2.0引入 [@Required](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-required-annotation) 。Spring 2.5引入遵循相同的通用方法来驱动Spring的依赖注入。`@Autowired `注解提供和[Autowiring collaborators](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-autowire)相同的功能，但是具有更细粒度的控制和更广泛的适用性。Spring 2.5也加入了JSR-250注解比如`@PostConstruct和` `@PreDestroy`。Spring 3.0加入了JSR-330(Dependency Injection for Java )注解，在`javax.inject `包比如`@Inject` 和`@Named`。在 [relevant section](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-standard-annotations)寻找相关注解详情。

**注解注入是在XML注入之前执行的，因此后一种配置将覆盖前者。**

像往常一样，您可以将它们注册为单独的bean定义 ,也可以隐式注册。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

（隐式注册post-processors 包括 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html),[`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html), [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html), 和上述的[`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html) )

`<context:annotation-config/> `仅仅是查找同一个application context上的定义的beans的注解。如果你把`<context:annotation-config/>`放在一个对于`WebApplicationContext`的`DispatcherServlet`上，只会检查你的controllers的`@Autowire`的beans，不会查你的service。See [The DispatcherServlet](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-servlet) for more information. 

### 1.9.1. @Required

@required注释应用于bean属性setter方法，如下面的例子： 

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

这个注释仅仅表明受影响的bean属性必须在配置时填充 ，通过一个bean定义或通过autowire的显式属性值。如果受影响的bean属性没有被填充，容器将抛出一个异常；这允许出现急切和显式的故障，避免稍后的`nullpointerexception`或类似异常。仍然建议您将断言放入bean类本身，例如，在init方法中。这样做，即使您在容器外部使用类，也会强制要求引用和值。 

### 1.9.2. @Autowired

在下面的例子中JSR-330的`@Inject`注解可以用来代替Spring的`@Inject`注解。详情请见 [here](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-standard-annotations) 。  

您可以将@autowired注解应用于构造函数： 

```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

**自Spring 4.3，像这样在`constructor上的@Autowired`注解，如果目标bean只定义了一个构造函数 ，就不必要了。但是，如果有几个构造函数可用 ，至少要有一个注释来教容器要使用哪一个。**

正如预期的那样，您也可以将`@autowired`注解应用到“传统的”setter方法： 

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注释应用于具有任意名称and/or多个参数的方法： 

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您也可以将@autowired应用到字段中，甚至可以将其与构造函数混合： 

```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

**确保已经声明你的组件声明了`@Autowired`-annotated injection points ，否则运行时会因为没有类型匹配报错**

**对于通过类路径扫描找到的xml定义的bean或组件类 ，容器通常预先知道具体的类型 。但是，对于@bean工厂方法，你需要确保声明的返回类型能充分表达。对于实现了几个接口，或者被它们的实现类型引用的组件，考虑在工厂方法中声明最具体的返回类型 （至少根据你的bean的injection points的要求是一样的 ）**

通过将注解添加到返回类型为数组的方法或者field上，可以从`ApplicationContext`中提供所有beans一个特定类型：

```
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

这同样适用于类型化集合： 

```
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

**如果你想数组或者list里的item按照一样次序排序，你的目标bean可以实现`org.springframework.core.Ordered`接口，或者使用 `@Order` or standard `@Priority` 注解。否则顺序为容器中相应的目标bean定义的注册顺序。** 

**`@order`注释可以在目标类级别上声明，也可以在`@bean`的方法上声明。`@order`值可能影响注入点的优先级， 但是请注意，它们不会影响单例启动顺序， 这是由依赖关系和 `@DependsOn`声明决定好了的。**

**注意标准`javax.annotation.Priority`  注解在`@Bean`级别不可用，因为不能标注在方法上。它的语义可以通过`@order`与`@primary`在单个bean上的每一种类型进行建模。** 

**设置typed map也可以autowire，只要期望的键类型是`String`。Map的values包含期望类型的所有beans，keys包含对应的bean的名字。**

```
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认下，没有候选bean可用是，autowire会失败；默认行为对待注解的方法，构造器和域表示需要依赖。这种行为可以被更改，如下所示。 

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

**每个类只有一个带注释的构造函数可以被标记为required，但是多个non-required的构造函数可以被注解。*：*在这种情况下，每一个都被认为是候选对象，Spring使用最贪婪的构造器，它的依赖性可以得到满足，这是拥有最多参数的构造函数。** 

**`@autowired`的必需属性被推荐在`@required`注释上。所必需的属性表明，该属性不需要自动装配。如果不能自动连接，属性就会被忽略。`@Required`,  另一方面，它更强大，因为它强制执行由容器支持的任何方法所设置的属性。 如果没有注入任何值，就会产生相应的异常。**

或者，您可以通过Java 8的`java.util.Optional`来表达non-required特性：

```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

自Spring 5.0，你可以使用`@Nullable`注解（任何包中的任何一种，例如`javax.annotation.Nullable` from JSR-305 ）：

```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

你也可以在接口使用`@Autowired` 来注解well-known resolvable的依赖：`BeanFactory`, `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher`, and `MessageSource`。这些接口和它们的继承接口，比如`ConfigurableApplicationContext` or `ResourcePatternResolver`, 自动解决，不需要特殊设置。 

```
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

**`@Autowired`, `@Inject`, `@Resource`, and `@Value` 注解由由Spring `BeanPostProcessor`的实现来处理，这就意味着您不能在自己的`BeanPostProcessor`或`BeanFactoryPostProcessor`类型中应用这些注释（如果有的话） 这些类型必须通过XML或者使用Spring`@bean`方法显式地“wire up”。**

### 1.9.3. Fine-tuning annotation-based autowiring with @Primary

因为通过类型的自动装配可能会导致多个候选，所以通常需要对选择过程有更多的控制。一种解决方式是使用`@Primary` 注解。@primary指出，当多个bean都是被自动连接到单值依赖项时，应该优先考虑特定的bean。 如果在候选者中有一个“primary”bean，那么它将是autowire的值。 

让我们假设我们有以下配置，将`firstMovieCatalog`定义为主要的`MovieCatalog`。 

```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

有了这样的配置，接下来的`MovieRecommender`将与`firstMovieCatalog`进行autowire。

```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

 相应的bean定义如下所示。 

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 1.9.4. Fine-tuning annotation-based autowiring with qualifiers

`@Primary`有效解决多个候选者不确定的问题。当需要对选择过程进行更多的控制时，可以用Spring的`@Qualifier`注解。你可以将限定值与特定的参数关联起来，缩小类型匹配的集合，以便为每个参数选择特定的bean。在最简单的情况下，这可以是一个简单的描述性值： 

```
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

@qualifier注解也可以在单独的构造函数参数或方法参数上指定： 

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

相应的bean定义如下所示。带有限定值“main”的bean与具有相同值的构造器参数连接在一起。 

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

对于一个fallback match，bean名称被认为是默认的限定值。 因此，您可以用id“main”来定义bean，而不是嵌套的qualifier节点，从而得到相同的匹配结果。 但是，尽管您可以使用这个约定来引用特定的bean名称，`@Autowired`  本质上是带有optional semantic qualifiers的type-driven的注入。这意味着限定符值，即使qualifier values带有bean name fallback，在类型匹配的集合中总是有缩小的语义；它们在语义上没有表示对唯一bean id的引用 。Good qualifier values are "main" or "EMEA" or "persistent",  表达独立于bean id的特定组件的特征，在一个匿名bean定义的情况下，它可以自动生成，就像前面例子中的那个一样。

Qualifiers 也适用typed collections，如以上讨论，例如`Set<MovieCatalog>`。在这种情况下，根据声明的限定符，所有匹配的bean都被注入到集合中。 这意味着限定符不一定是唯一的；它们只是简单地构成了过滤标准。例如，您可以定义多个MovieCatalog bean，具有相同的限定符值“action” ，所有这些都将被注入到一个带有`@qualifier（“action”）`注释的集合中。

**在type-matching的候选中，使qualifier values选择目标bean name，甚至不需要在注入点使用`@Qualifier` 。如果没有其他resolution indicator（例如 a qualifier or a primary marker ），对于非唯一的依赖情况 ，Spring将与目标bean名称匹配注入点名称（即字段名或参数名），并选择相同名称的候选者，如果有的话。** 

**也就是说，如果你打算用name来表达注解驱动的注入，不要主要使用@autowired，即使它能够在类型匹配的候选者中选择bean名称。 相反，使用JSR-250`@resource`注解，它在语义上定义为通过它的唯一名称来标识一个特定的目标组件 ，声明的类型与匹配过程无关。`@Autowired` 有相当不同的语义：按类型选择候选bean之后，指定的String qualifier value将只在type-selected candidates中考虑，例如，匹配"account" qualifier的beans，要有相同的qualifier label。** 

**对于自身被定义成集合/map或list的bean，`@Resource` 是一个好的解决方案，通过unique name引用特定集合或数组bean。即便如此 ，从Spring 4.3，collection/map and array 类型也可以被`@autowire`类型匹配算法匹配，只要element type信息保存在`@Bean`的返回类型特征或者集合继承层级结构中。在这种情况下，qualifier values 可以用来在同类型的集合中进行选择，如前一段所述。** 

**自Spring 4.3，`@autowired`也考虑注入的自我引用，例如，返回当前注入的bean的引用。请注意，自我注入是一个fallback; 对其他组件的常规依赖总是有优先级的。 从这个意义上说，自我引用不参与常规的候选选择 ，never primary；相反，它们总是以最低优先级的顺序结束 。在实践中，只使用self引用作为最后的手段。例如通过bean的事务代理调用同一实例上的其他方法：在这样的场景中，考虑将受影响的方法分解为单独的委托bean。另一种方法是使用@resource，它可以通过它的唯一名称获得一个代理回当前bean。** 

**`@autowired`应用于field、构造函数和多参数方法，允许在参数级别上通过qualifier注解进行缩窄。 相反，`@resource`仅支持字段和bean属性setter方法，只有一个参数。因此，如果你的注入目标是一个构造函数或多参数方法，就坚持使用qualifiers。** 

您可以创建自己的定制限定符注释。 简单地定义一个注释，并在您的定义中提供@qualifier注解： 

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后您可以在autowire的字段和参数上提供自定义限定符： 

```
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，提供候选bean定义的信息。添加`<qualifier/>`作为`<bean/>`的子标签，指定`type` and `value` 来匹配你的自定义qualifier注解。该类型与注释的完全限定类名相匹配。或者，如果不存在相互冲突的名字的风险， 您可以使用简短的类名。 在下面的示例中演示了这两种方法。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在 [Classpath scanning and managed components](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-classpath-scanning)，您将看到一个基于注解的替代方案，以在XML中提供限定符元数据。 具体, see [Providing qualifier metadata with annotations](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-scanning-qualifiers). 

在某些情况下，它可能足以使用没有值的注释。 注解服务于一个更通用的目的，可以应用于几种不同类型的依存关系可能有用。例如，您可以提供一个离线目录，当没有Internet连接可用时，它将被搜索。首先定义简单的注释： 

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后把注释添加到字段或属性中，以autowire： 

```
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...
}
```

现在bean定义只需要一个`qualifier type`： 

```
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```

你可以可以定义自定义qualifier注解，接受named attributes，除了simple `value`attribute之外或代替simple `value`attribute。如果多个属性值指定在field或者参数上来autowire，一个bean定义必须匹配所有这样被考虑为候选者的值。作为一个例子，考虑下面的注释定义： 

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

这种情况下`Format`是枚举：

```
public enum Format {
    VHS, DVD, BLURAY
}
```

带有custom qualifier的要autowire的field，包括2个属性值：`genre` and `format`。

```
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean定义应该包含相配的qualifier values。这个例子也说明了bean meta属性可能被使用，而不是`<qualifier/>`子节点。如果可用，`<qualifier/>`和它的值是优先的，如果没有qualifier的值提供，自动装配机制会回到`<meta/>`标签内的值上，如下面的例子中最后两个bean定义。 

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

### 1.9.5. Using generics as autowiring qualifiers

除了`@qualifier`注解之外，也可以使用Java泛型类型作为一种隐式的qualification形式。例如，假设您有以下配置： 

```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设上面的bean实现了一个通用接口，例如`Store<String>` and `Store<Integer>` ，你可以`@Autowire`  一个store接口，泛型会被作为qualifier使用。

```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

通用限定符也适用于自动装配Lists，Maps，Arrays：

```
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

### 1.9.6. CustomAutowireConfigurer

 [`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) 是一个`BeanFactoryPostProcessor` 允许您注册您自己的custom qualifier 注解类型，即使它们没有被Spring的`@qualifier`注解。 

```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

`AutowireCandidateResolver` 绝对autowire候选通过：

- 每个bean定义的 `autowire-candidate`  值
- 任何beans节点上的`default-autowire-candidates`  pattern(s)
-  `@Qualifier` 注解的出现和任何通过 `CustomAutowireConfigurer` 注册的自定义注解

当多个bean qualify as autowire候选，"primary" 的决断如下：如果候选者中恰好有一个bean定义有一个primary属性为真，那么它将被选中。 

### 1.9.7. @Resource

Spring也提供使用JSR-250 `@Resource` 注解的注入，在fields或者bean属性setter方法上。

`@Resource` 接收一个name属性，默认情况下，Spring将该值解释为注入的bean名称 。换句话说，它遵循了by-name语义，如本例所示： 

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果没指定名字，默认名称来源于字段名或setter方法。在字段的情况下，它取字段名;在setter方法的情况下，它接受bean属性名。 面的例子有一个名为“movieFinder ”的bean，注入它的setter方法： 

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

**注解提供的name被`ApplicationContext` 解析为bean name，它的`CommonAnnotationBeanPostProcessor`  aware到。如果您显式地配置Spring的[`SimpleJndiBeanFactory`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/jndi/support/SimpleJndiBeanFactory.html)，则可以通过JNDI解析名称。 但是，建议您依赖于默认行为，并简单地使用Spring的JNDI查找功能来保持间接的级别。** 

在唯一`@Resource`  没有指定name的情况下，类似于`@Autowired`，会 finds a primary type match，而不是找specific named bean，并且会解析well-known resolvable dependencies: the `BeanFactory`,`ApplicationContext`, `ResourceLoader`, `ApplicationEventPublisher`, and `MessageSource` 接口。

在下面的例子中 ，`customerPreferenceDao` field，先去查找一个名字为customerPreferenceDao 的bean，然后返回到`CustomerPreferenceDao`类型的一个基本类型匹配。"context"  field的注入based on the known resolvable dependency type `ApplicationContext`。

```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

### 1.9.8. @PostConstruct and @PreDestroy

`CommonAnnotationBeanPostProcessor` 不仅识别`@Resource` 注解也能识别JSR-250生命周期注解。Spring 2.5引入，对这些注解的支持也提供了另一种选择，described in[initialization callbacks](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) and [destruction callbacks](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)。前提是 `CommonAnnotationBeanPostProcessor` 注册在`ApplicationContext`之内，在生命周期的同一点上调用带有其中一个注释的方法，作为相应的Spring生命周期接口方法或显式声明的回调方法。在下面的例子中，缓存将在初始化时预先填充，并在销毁时清除。 

```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

有关组合各种生命周期机制的效果的详细信息，see [Combining lifecycle mechanisms](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-combined-effects)。

## 1.10. Classpath scanning and managed components

本节描述通过扫描类路径来隐式检测候选组件的选项。 候选组件是与筛选标准相匹配的类，并且有一个对应的bean定义在容器中注册。 这就消除了使用XML执行bean注册的需要; 相反，您可以使用注解（例如@component），AspectJ类型表达式 ，或者你自己的已经在容器注册的自定义过滤器标准。

**从Spring 3.0，Spring JavaConfig项目提供的许多特性都是核心Spring框架的一部分。 这允许您使用Java来定义bean，而不是使用传统的XML文件 。看下 `@Configuration`, `@Bean`, `@Import`, and `@DependsOn` 注解的例子了解如何使用这些特性。**

####  1.10.1. @Component and further stereotype annotations

`@repository`注解是任何满足存储库角色或原型的类的标记 （DAO）。该标记的用途之一是 [Exception translation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/data-access.html#orm-exception-translation)中所描述的异常的自动转换。 

Spring提供了更多原型注解： `@Component`, `@Service`, and `@Controller`。@component是任何spring管理组件的通用原型，其余的根据用途。这些原型注解为切入点提供了理想的目标。  `@Repository`, `@Service`, and `@Controller`  在未来可能会有额外的语义。

####  1.10.2. Meta-annotations

Spring提供的许多注解可以在您自己的代码中用作元注解。 元注解只是一个可以应用到另一个注释的注解。 例如，上面提到的`@service`注解带有元注解`@component`： 

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

    // ....
}
```

元注解还可以组合元注解来创建组合注解 。例如，Spring MVC中的`@RestController`注释由`@Controller`和`@ResponseBody`组成。 

此外，组合注解还可以选择性地从元注解重新声明属性，以允许用户定制。 当您只想公开元注解属性的一个子集时，这一点特别有用。 例如，Spring的`@SessionScope` 注解硬编码了scope名称到`session` ，但仍然允许定制`proxyMode。`

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

} 
```

可以在不声明`proxyMode`的情况下使用`@SessionScope` ： 

```
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

或者，`proxyMode`的覆盖值如下： 

```
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

For further details, consult the [Spring Annotation Programming Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) wiki page. 

####  1.10.3. Automatically detecting classes and registering bean definitions

Spring可以自动检测原型类，并使用`ApplicationContext`注册相应的`beandefinition`。 以下两个类有资格进行自动检测：

```
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，你需要向你的`@Configuration` 类添加`@ComponentScan` ，`basePackages`是这两个类的parent package（或者逗号、分号、空格分隔的列表）。

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

为了简洁，上面可能使用了注释的value属性，即`@ComponentScan（“org.example”）` 。

下面是使用XML的另一种选择：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

**`<context:component-scan>` 显式启用了`<context:annotation-config>`。使用了前面的就不需要声明后面的了。**

**对类路径包的扫描需要在类路径中存在相应的目录条目。** 

**在JDK 9的模块路径（Jigsaw）上，Spring的类路径扫描通常按照预期工作。但是，请确保您的组件类被导出到 `module-info` descriptors中；如果你期望Spring调用你的类的非公共成员，确保它们是“opened”（例如，使用`opens`声明代替`exports`在你的 `module-info` descriptor上）。**

另外，当你使用component-scan 节点，`AutowiredAnnotationBeanPostProcessor` and `CommonAnnotationBeanPostProcessor`都被显示包含了。这意味着这两个组件是自动检测和连接在一起的-所有这些都没有在XML中提供bean配置元数据。

**通过包含*annotation-config*的属性为false，可以使`AutowiredAnnotationBeanPostProcessor` and`CommonAnnotationBeanPostProcessor` 注册失效。**

 ####  1.10.4. Using filters to customize scanning

添加*includeFilters* or *excludeFilters* 参数到`@ComponentScan` 注解。每个过滤元素都需要`type`和`expression`属性。下表描述了过滤选项。 

| Filter Type          | Example Expression           | Description                                                  |
| -------------------- | ---------------------------- | ------------------------------------------------------------ |
| annotation (default) | `org.example.SomeAnnotation` | An annotation to be present at the type level in target components. |
| assignable           | `org.example.SomeClass`      | A class (or interface) that the target components are assignable to (extend/implement). |
| aspectj              | `org.example..*Service+`     | An AspectJ type expression to be matched by the target components. |
| regex                | `org\.example\.Default.*`    | A regex expression to be matched by the target components class names. |
| custom               | `org.example.MyTypeFilter`   | A custom implementation of the `org.springframework.core.type .TypeFilter` interface. |

 下面的例子展示了忽略所有`@Repository`注解的配置，而是用 "stub" repositories。

```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

使用等价的XML：

```

```

```
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans> 
```

你可以设置在注解上使用`useDefaultFilters=false` 或者 在`<component-scan/>`节点提供`use-default-filters="false"` 属性来使默认过滤器失效。这会禁用 `@Component`, `@Repository`, `@Service`, `@Controller`, or `@Configuration`的自动检测。

#### 1.10.5. Defining bean metadata within components

Spring组件还可以向容器提供bean定义元数据。 您可以使用相同的`@Bean`标注来定义`@Configuration`注释类中的bean元数据。这里有一个简单的例子： 

```
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

这个类是一个Spring组件，它的`doWork()`方法中包含特定于应用程序的代码。 但是，它还提供了一个有工厂方法的bean定义，引用`publicInstance()`的方法。`@Bean`注解标识工厂方法和其他bean定义属性，例如通过`@Qualifier`注解的qualifier值。其他可以被指定的方法级别的注解为 `@Scope`, `@Lazy`,  和自定义qualifier注解。

**除了组件初始化的角色之外， `@Lazy` 注解也可以放在 标注`@Autowired` or `@Inject` 的注入点上，在这种情况下，它导致了一lazy-resolution proxy的注入。** 

autowire的字段和方法得到了前面讨论的支持，并为`@Bean`方法的自动装配提供了额外的支持： 

```
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

自Spring Framework 4.3，您还可以声明工厂方法参数`InjectionPoint` （更具体是子类 `DependencyDescriptor`），以访问请求注入点，触发当前bean的创建 。注意，这只适用于bean实例的实际创建， 而不是对现有实例的注入。 因此，这个特性对于prototype的bean是最有意义的。对于其他scope，工厂方法只会看到注入点，它触发了给定范围内的新bean实例的创建：  例如，触发了lazy单例bean创建的依赖。在这样的场景中使用提供的注入点元数据和语义处理。 

```
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

普通Spring组件中的`@Bean`方法与Spring`@Configuration`类中的对应方法不同。 不同之处在于，`@Component`类没有通过CGLIB增强来拦截方法和字段的调用。 CGLIB代理通过`@Configuration`类中的`@Bean`方法中的调用方法或字段创建bean元数据引用到协作对象; 这些方法不是用普通的Java语义调用的， 而是通过容器来提供通常的生命周期管理和Spring bean的代理 ，即使是通过编程式调用`@Bean`方法来引用其他bean。相反，在纯`@Component`类中调用`@Bean`方法中的方法或字段具有标准的Java语义，没有特殊的CGLIB处理或其他约束。 

**你可以声明`@Bean`为static，允许不需要创建它们的containing configuration class作为实例来杯调用。这在创建post-processor beans时很有用，例如`BeanFactoryPostProcessor` or `BeanPostProcessor`类型，因为这样的bean将在容器生命周期的早期被初始化 ，在那个时候，应会避免触发配置的其他部分。** 

**请注意，对静态@bean方法的调用永远不会被容器拦截， 甚至在`@Configuration`类中也不行（见上文）。 这是由于技术上的限制 :CGLIB子类化只能覆盖非静态方法。因此，对另一个`@Bean`方法的直接调用将具有标准的Java语义 ，导致产生一个独立的实例，直接从工厂方法本身返回。** 

**Java语言@Bean方法的可见性不会对Spring容器bean定义的结果立即生效。您可以自由地声明您的工厂方法，您认为它适合于非@Configuration类，也可以在任何地方使用静态方法。 然而，常规@Configuration类的@Bean方法需要可以被覆盖，例如，它们一定不能被声明为private或者final。**

**@Bean方法也会在给定组件或配置类的基类上被发现， 以及由组件或配置类实现的接口中声明的Java 8默认方法上。这允许在组合复杂的配置安排方面有很大的灵活性，在Spring 4.2的Java 8默认方法中，甚至可以使用多个继承。**

**最后，请注意，单个类可能为同一bean持有多个@Bean方法， 作为多种工厂方法的一种安排，在运行时依赖于可用的依赖关系。这与在其他配置场景中选择“最贪婪”的构造器或工厂方法的算法是一样的： 在构建时，将选择最大数量的可满足依赖项的变体，类似于容器在多个@autowired构造函数之间选择的方式。** 

#### 1.10.6. Naming autodetected components

当一个组件被自动检测时，它的bean名称是由scanner知道的`BeanNameGenerator` 策略生成的。 默认任何Spring包含一个name值的原型注解（`@Component`, `@Repository`, `@Service`, and`@Controller`）将为相应的bean定义提供该名称。 

如果这样的注解没有提供name的值，或者对于其他任何检测到的组件（例如被自定义filters发现的）也没有，默认bean name生成返回未大写的非限定类名。例如，如果检测到下列组件类，名称将是`myMovieLister`和`movieFinderImpl`：

```
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

**如果您不想依赖默认的bean命名策略，先实现 [`BeanNameGenerator`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanNameGenerator.html) 接口，一定要包含一个默认的无参构造函数。 然后，在配置扫描器时提供完全限定的类名：** 

```
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

作为一般规则，当其他组件可能对其进行显式引用时，请考虑使用注释指定名称。另一方面，每当容器负责wire时，自动生成的名称就足够了。 

#### 1.10.7. Providing a scope for autodetected components

通常都是singleton的，如果需要不同的scope，使用`@Scope` 指定。

```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

**`@Scope` 只在具体的类（被注解的组件）或者工厂方法（@Bean的方法）上自检。与xml bean定义不同，没有关于bean定义继承的概念 ，类级别的继承层次结构与元数据的目的无关。**

**为scope问题提供自定义策略，而不是依赖于基于注解的方法，实现 [`ScopeMetadataResolver`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/ScopeMetadataResolver.html)接口，一定要包含一个默认的无arg构造函数。*：*然后，在配置扫描器时提供完全限定的类名：** 

```
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

当使用一些non-singleton scopes，可能有必要为作用域对象生成代理。原因在[Scoped beans as dependencies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)。出于这个目的，在component-scan元素上可以使用*scoped-proxy*属性。三个可能的值是 no, interfaces, and targetClass 。例如，下面的配置将导致标准JDK动态代理： 

```
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

####  1.10.8. Providing qualifier metadata with annotations

 `@Qualifier`注解在 [Fine-tuning annotation-based autowiring with qualifiers](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers) 讨论。那节的例子展示了`@Qualifier` 的用法，以及自定义 qualifier 注解来提供解决autowire多候选者时候细粒度控制。那是用xml的，你可以在候选类上使用top-level注解提供qualifier的元数据。

```
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}

```

**与大多数基于注解的替代方案一样，请记住，注解元数据绑定到类定义本身，虽然xml允许同类的多个beans在它们的qualifier元数据提供多个变体，因为那个元数据是per-instance提供而不是per-class。**

#### 1.10.9. Generating an index of candidate components

虽然类路径扫描非常快，通过在编译时创建一个静态的候选列表，可以提高大型应用程序的启动性能。 在这种模式下，应用程序的所有模块都必须使用这种机制，当`ApplicationContext`检测到这样的索引时，它会自动使用它，而不是扫描类路径。 

只需在每个模块中添加一个额外的依赖项，模块包含组件扫描指令的目标，来生成索引。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.0.7.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

这个过程将生成一个`META-INF/spring.components` 组件文件，该文件将被包含在jar中。 

**当您在IDE中使用这种模式时，`spring-context-indexer`必须注册为注释处理器，以确保在更新候选组件时索引是最新的。** 

 **`META-INF/spring.components` 在classpath被找到时，索引就会自动生效。如果某个索引部分可用到一些库（或用例），但不能为整个应用程序构建， 您可以返回到常规的classpath arrangement(也就是说，没有任何索引存在 )，通过设置 `spring.index.ignore` 为true要么是系统属性要么是classpath下的`spring.properties`  文件。**

### 1.11. Using JSR 330 Standard Annotations

从Spring 3.0开始，Spring提供了对JSR-330标准注释的支持（依赖注入）。这些注释的扫描方式与Spring注释的方式相同。 您只需要在类路径中拥有相关的jar。 

如果你使用maven， `javax.inject`  如下：

```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

#### 1.11.1. Dependency Injection with @Inject and @Named

不使用 `@Autowired` ，`@javax.inject.Inject`  使用如下：

```
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```

如同 `@Autowired` ， `@Inject` 也可以在field级别、方法级别和构造器参数级别使用。此外，您可以将您的注入点声明为 `Provider`， 允许按需访问shorter scoped的bean或通过`Provider.get()`调用来lazy access其他bean。作为上面例子的一个变体：

```
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        ...
    }
}
```

如果你想要使用一个 qualified name来注入的依赖项，你应该用 `@Named`  注解如下：

```
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如同 `@Autowired` ， `@Inject`  也可以用`java.util.Optional` or `@Nullable`。这在这里更加适用，因为`@Inject` 没有`required` 的属性。 

```
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

```
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

#### 1.11.2. @Named and @ManagedBean: standard equivalents to the @Component annotation

不使用 `@Component` ，`@javax.inject.Named` or `javax.annotation.ManagedBean` 可以如下使用：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在不指定组件名称的情况下使用`@Component`是很常见的。`@named`可以以类似的方式使用： 

```
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

当使用`@Named` or `@ManagedBean`，使用组件扫描的方式与使用Spring注解的方式是完全相同的： 

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

与 `@Component` 相比， JSR-330 `@Named` and the JSR-250 `ManagedBean` 注解是不能组合的。请使用Spring的stereotype model来构建自定义组件注解。

#### 1.11.3. Limitations of JSR-330 standard annotations

在使用标准注解时，重要的是要知道一些重要的特性是不可用的，如下表所示：

*Table 6. Spring component model elements vs. JSR-330 variants* 

| Spring              | javax.inject.*        | javax.inject restrictions / comments                         |
| ------------------- | --------------------- | ------------------------------------------------------------ |
| @Autowired          | @Inject               | `@Inject` has no 'required' attribute; can be used with Java 8’s `Optional` instead. |
| @Component          | @Named / @ManagedBean | JSR-330 does not provide a composable model, just a way to identify named components. |
| @Scope("singleton") | @Singleton            | The JSR-330 default scope is like Spring’s `prototype`. However, in order to keep it consistent with Spring’s general defaults, **a JSR-330 bean declared in the Spring container is a `singleton` by default**. In order to use a scope other than `singleton`, you should use Spring’s `@Scope` annotation. `javax.inject` also provides a [@Scope](http://download.oracle.com/javaee/6/api/javax/inject/Scope.html)annotation. Nevertheless, this one is only intended to be used for creating your own annotations. |
| @Qualifier          | @Qualifier / @Named   | `javax.inject.Qualifier` is just a meta-annotation for building custom qualifiers. Concrete String qualifiers (like Spring’s `@Qualifier` with a value) can be associated through `javax.inject.Named`. |
| @Value              | -                     | no equivalent                                                |
| @Required           | -                     | no equivalent                                                |
| @Lazy               | -                     | no equivalent                                                |
| ObjectFactory       | Provider              | `javax.inject.Provider` is a direct alternative to Spring’s `ObjectFactory`, just with a shorter `get()` method name. It can also be used in combination with Spring’s `@Autowired`or with non-annotated constructors and setter methods. |

###  1.12. Java-based container configuration

####  1.12.1. Basic concepts: @Bean and @Configuration

Spring新Java-configuration的支持的核心artifacts是`@Configuration`-annotated classes and `@Bean`-annotated methods。

`@Bean`注释用来表示一个方法实例化， 配置并初始化一个新对象，由Spring IoC容器管理。对于熟悉Spring的XML配置的人来说，`@Bean`注释扮演的角色与`<bean/> `相同。 你可以在 `@Component`上使用`@Bean`注解方法，然而，最常用在 `@Configuration` 的beans。

用`@Configuration`注释一个类表明它的主要目的是作为bean定义的source。另外，`@Configuration` classes 允许之内的依赖通过简单调用其他在同一个类的`@Bean`方法来定义。最简单的`@Configuration`类将如下： 

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上面的AppConfig类将相当于下面的Spring/XML `<beans/>`  ： 

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

```
				Full @Configuration vs 'lite' @Bean mode?
当@bean方法没有在@Configuration的类中声明时，它们被称为在“lite”模式下进行处理。在@Component中声明的Bean方法，甚至在普通的旧类中，都将被认为是“lite”，带有包含类的a different primary purpose，@Bean method在这里只是a sort of bonus。例如，service组件通过将@Bean方法添加到每个适用的组件类，可以向容器暴露management views。在这种情况下，@Bean方法是一种简单的通用工厂方法机制。

不像full @Configuration，lite @Bean方法不能声明inter-bean依赖。相反，它们对包含组件的内部状态进行操作，并可选择性地在它们可能声明的参数上进行操作。因此，这样的@Bean方法不应该调用其他@Bean方法;每一个这样的方法都只是一个特定的bean引用的工厂方法，没有任何特殊的运行时语义。这里的积极的副作用是，在运行时不需要使用CGLIB子类，因此，在类设计方面没有限制（也就是说，包含的类可能是final的）。

在普通情况下,@Bean方法在@Configuration类中声明，确保始终使用“full”模式，这样cross-method references就会被重定向到容器的生命周期管理。这会防止相同的@Bean方法不小心通过常规Java调用，会减少在'lite'模式下产生的难以追踪的细微bug。
```

`@Bean`和@`Configuration`注释将在下面的小节中深入讨论。 但是，首先，我们将介绍使用基于java的配置创建spring容器的各种方法。 

#### 1.12.2. Instantiating the Spring container using AnnotationConfigApplicationContext

下面部分介绍 `AnnotationConfigApplicationContext` ，Spring 3.0引入。这个通用的`ApplicationContext` 实现不仅能够接受`@Configuration`类作为输入，还有`@Component` 的类以及JSR-330元数据的注解类。

当`@Configuration`类作为输入时， `@Configuration` 类本事被注册成bean定义，并且所有在类中声明的`@Bean`方法也被注册为bean定义。 

当 `@Component` and JSR-330 类被提供作为输入，他们被注册为bean定义，并且假设DI metadata 比如`@Autowired` or `@Inject`  在这些类必要的地方使用。

#####  Simple construction

和Spring xml实例化`ClassPathXmlApplicationContext` 作为输入使用大致相同，`@Configuration` 类在实例化 `AnnotationConfigApplicationContext`时作为输入。他允许完全没有xml的Spring容器的使用：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如上所述，`AnnotationConfigApplicationContext` 不局限于 works with`@Configuration`  类，任何`@Component` or JSR-330 注解类都可以作为输入提供给构造器：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

以上假定`MyServiceImpl`, `Dependency1` and `Dependency2`  使用Spring 依赖注入注解例如 `@Autowired`。

#####  Building the container programmatically using register(Class<?>…)

 `AnnotationConfigApplicationContext` 可以使用无参构造器实例化，然后使用`register()` 方法配置。当编程式构建`AnnotationConfigApplicationContext`时很有用。

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

##### Enabling component scanning with scan(String…)

To enable component scanning, just annotate your `@Configuration` class as follows: 

```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

````
有经验的Spring用户将熟悉Spring的context:` namespace
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
````

 `AnnotationConfigApplicationContext` 暴露了`scan(String…)`  方法允许相同的组件扫描功能。

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

**记住 `@Configuration` 类是[meta-annotated](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-meta-annotations) with `@Component`, 所以它们是组件扫描的候选对象！ 在上个例子，假设`AppConfig`声明在 `com.acme` 包下（或任何底下的包），在调用`scan()`它会被pick up，在`refresh()` 所有的它的`@Bean`方法在容器将被处理并注册为bean定义。**

##### Support for web applications with AnnotationConfigWebApplicationContext

一个 `AnnotationConfigApplicationContext` 形式的`WebApplicationContext`  ，可以用`AnnotationConfigWebApplicationContext`。这个实现可以在配置Spring `ContextLoaderListener`servlet监听器,Spring MVC `DispatcherServlet`等等时使用。下面是一个典型的Spring MVC web应用的`web.xml` 片段。注意`contextClass`  的context-param and init-param 的使用：

```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

####  1.12.3. Using the @Bean annotation

`@Bean` 是方法级别的注解，类比于XML的`<bean/>`节点。注解提供了`<bean/>`给的一些属性，比如 [init-method](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean), [destroy-method](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean), [autowiring](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) and `name`。

你可以在`@Configuration`-annotated or in a `@Component`-annotated class 使用 `@Bean`  注解。

#####  Declaring a bean

声明一个bean，只需用`@Bean`标注一个方法。 使用一种指定为该方法返回值的类型的`ApplicationContex` 注册bean定义。默认，bean名称将与方法名相同。

```
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

等同于

```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

这两种声明都在`ApplicationContext`中提供了一个名为`transferService`的bean，绑定到对象实例类型 `TransferServiceImpl`：

```
transferService -> com.acme.TransferServiceImpl
```

您也可以用一个接口（或基类）返回类型声明您的`@Bean`方法： 

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

然而，这限制了对指定接口类型的提前类型预测的可见性，当受影响的单例bean被实例化时，只有容器才知道完整类型（`TransferServiceImpl`）。non-lazy 单例bean根据它们的声明顺序进行实例化，因此，您可能会看到不同类型匹配的结果，这取决于另一个组件试图通过a non-declared type匹配的情况下（比如 `@Autowired TransferServiceImpl`  只有 "transferService" bean实例化才会resolve ）。

**如果您始终通过声明的服务接口引用您的类型，你的 `@Bean` 的返回类型可以安全加入该设计决策。然而，对于实现几个接口的组件，或者对于其实现类可能引用的组件， 声明最特定的返回类型是更安全的 （至少根据你的bean的注射点的要求是一样的） 。**

##### Bean dependencies

一个 `@Bean` 注解方法，可以有任意数量的参数来描述构建该bean所需的依赖性。比如：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解决机制与基于构造的依赖性注入非常相似， see [the relevant section](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-constructor-injection) for more details。

#####  Receiving lifecycle callbacks

任何用`@Bean`注解的类都支持常规的生命周期回调，可以使用JSR-250的 `@PostConstruct` and `@PreDestroy`。see [JSR-250 annotations](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations) for further details。

 regular Spring [lifecycle](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-nature) callbacks 也是fully支持的。如果一个bean实现 `InitializingBean`, `DisposableBean`, or `Lifecycle` ，它们各自的方法被容器调用。

标准`*Aware`  接口集比如 [BeanFactoryAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-beanfactory), [BeanNameAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware), [MessageSourceAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-functionality-messagesource),[ApplicationContextAware](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-aware), and so on 也是fully supported。

`@Bean`注解支持指定任意初始化和销毁回调方法， 很想Spring XML的bean上的`init-method` and `destroy-method`  属性：

```
public class Foo {

    public void init() {
        // initialization logic
    }
}

public class Bar {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```

```
！！ 
默认情况下，使用具有公共 close or shutdown 方法的Java配置定义的bean会自动加入销毁回调。如果你有一个公共 close or shutdown方法你不希望在容器关闭时被调用，简单地将@Bean（destroyMethod=""）添加到bean定义中，以禁用默认（推断）模式。

默认情况下，您可能想要通过JNDI获得的资源，因为它的生命周期是在应用程序之外进行管理的。特别是，确保始终为数据源做这件事，因为它在Java EE应用服务器上被认为是有问题的。

@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}

另外，通过@Bean方法，您通常会选择使用程序性JNDI查找：要么用JndiTemplate/JndiLocatorDelegate helpers 要么 straight JNDI InitialContext usage，不要用JndiObjectFactoryBean的变体，这会强制您声明返回类型为FactoryBean类型，而不是实际的目标类型，使得其他想使用这个资源的@Bean方法的cross-reference calls更难。
```

当然，在上面的Foo案例中，在构建过程中直接调用`init()`方法同样有效： 

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...
}
```

**当你直接在Java中工作时，你可以用你的对象做任何你喜欢的事情，并且并不总是需要依赖于容器的生命周期！** 

##### Specifying bean scope

###### Using the @Scope annotation 

你可以对`@Bean`注解bean指定一个特定的scope。你可以使用 [Bean Scopes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes)  指定的任何标准scope。

默认`singleton`，如下覆盖：

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
} 
```

###### @Scope and scoped-proxy 

Spring提供了一种通过 [scoped proxies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)来处理scoped dependencies的方便方法。 在使用XML配置时创建这样一个代理的最简单方法 在使用XML配置时创建这样一个代理的最简单方法是`<aop:scoped-proxy/> `节点。@Bean也提供了这种支持。默认是 no proxy( `ScopedProxyMode.NO` )，但你可以指定`ScopedProxyMode.TARGET_CLASS` or `ScopedProxyMode.INTERFACES`。

如果您从XML参考文档中移植了作用域的代理示例到@Bean，如下：

```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

#####  Customizing bean naming

默认情况下，配置类使用@bean方法的名称作为结果bean的名称。然而，这个功能可以用name属性覆盖。 

```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }
}
```

#####  Bean aliasing

如 [Naming beans](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-beanname)讨论，有时候，给一个bean多个名字是可取的， 也就是所谓的bean *aliasing* 。`@Bean`注释的name属性接受一个字符串数组来实现这个目的。 

```
@Configuration
public class AppConfig {

    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

##### Bean description

有时，提供对bean的更详细的文本描述是很有帮助的。 当bean被公开（可能通过JMX）进行监视时，这一点特别有用。 

```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }
}
```

####  1.12.4. Using the @Configuration annotation

`@Configuration`是一个类级别的注释，表明对象是bean定义的来源。 `@Configuration`类通过`@Bean`注释的public方法声明bean。 在`@Configuration`类上对`@Bean`方法的调用也可以用来定义bean间的依赖关系。 See [Basic concepts: @Bean and @Configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts) for a general introduction。

#####  Injecting inter-bean dependencies

当@bean彼此依赖时，表示依赖性就像拥有一个bean方法调用另一个bean一样简单： 

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }
}
```

在上面的例子中，foo bean通过构造函数注入获得对bar的引用。 

**这种声明inner-bean依赖只在`@Configuration` class 有用，`@Component` classes不行。**

#####  Lookup method injection

如前所述, [lookup method injection](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection) 是一个你应该很少使用的高级功能。在singleton-scoped bean有一个 prototype-scoped bean依赖的情况下有用。为这种类型的配置使用Java提供了实现此模式的自然方法。

```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

使用 Java-configuration 支持，你可以创建 `CommandManager`  子类，抽象方法`createCommand()` 方法被如下复写，它查找一个新的（prototype）command对象：

```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### Further information about how Java-based configuration works internally

下面的例子展示了一个`@bean`注解的方法被调用两次： 

```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`clientDao()`一次在`clientService1()` 一次在 `clientService2()`被调用。因为这个方法创建了clientDaoImpl的新实例并返回它，您通常期望有两个实例（每个service对应一个实例）。这肯定会有问题： 在Spring中，实例化bean默认情况下有一个singleton的scope。这就是魔法的地方：所有的`@Configuration`类都在启动时subclassed with CGLIB。 在subclass，child method 在其调用parent方法并创建实例前都会先检查容器的所有cached(scoped) beans。注意，自Spring3.2，不需要再添加CGLIB到classpath，因为已经在Spring-core JAR的`org.springframework.cglib` 。

**根据您的bean的scope，行为可能会有所不同。我们在这里讨论的是单例。** 

**由于CGLIB在启动时动态添加功能，所以有一些限制，特别地，配置类不能是final的。然而，自从Spring 4.3，任何构造器都可以在配置类上 ，包括使用 `@Autowired`  或a single non-default对默认注入的构造器声明。**

**如果你想避免任何cglib的限制， 考虑在非`@Configuration`类声明`@Bean`方法，例如 `@Component` 类。@Bean方法的Cross-method calls不会被拦截，因此，您将不得不在构造函数或方法级别上完全依赖依赖注入。** 

####  1.12.5. Composing Java-based configurations

#####  Using the @Import annotation

就像在Spring XML文件中使用 `<import/>`元素来帮助模块化配置一样，`@Import`注解允许从其他配置类加载bean定义：

```
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，不需要同时指定ConfigA和ConfigB。当实例化context时，只有ConfigB需要显式地提供： 

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器实例化 ，因为只有一个类需要处理，而不是要求开发人员在构建过程中记住大量的`@Configuration`类。

**自从Spring 4.2，`@Import`也支持对常规组件类的引用，类似于`AnnotationConfigApplicationContext.register`方法。如果你想避免组件扫描，这是非常有用的，使用一些配置类作为入口点，用于显式地定义所有组件。**

######  Injecting dependencies on imported @Bean definitions 

上面的例子是有效的，但是过于简单。 在大多数实际场景中，bean之间在配置类之间会有依赖关系。 在使用XML时，这本身并不是问题，因为没有涉及到编译器，并且可以简单地声明ref=“someBean”，并相信Spring会在容器初始化过程中解决这个问题。 当然，当使用`@Configuration`类时，Java编译器会对配置模型设置约束，因为对其他bean的引用必须是有效的Java语法。 

幸运的是，解决这个问题很简单。 如同 [we already discussed](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-java-dependencies) ，`@Bean` 方法可以有任意数量的参数描述依赖，让我们考虑一个更真实的场景，其中有几个`@Configuration`类，每个类依赖于其他的bean： 

```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到同样的效果。请记住，`@Configuration`类最终只是容器中的另一个bean： 这意味着他们可以像其他bean一样利用`@Autowired`和`@Value`等来注入！

**确保您所注入的依赖关系是最简单的类型。 `@Configuration` 类在context实例化时很早被处理，强制以这种方式注入依赖可能会导致意外的早期初始化。 只要有可能，就像上面的例子中使用基于参数的注入。**  

**另外，通过`@Bean`对`BeanPostProcessor`和`BeanFactoryPostProcessor`定义特别小心。这些通常应该被声明为静态的`@Bean`方法，  不触发内含配置类的实例化。 否则，`@Autowired`和`@Value`将不会在configuration类本身上工作，因为它过早地被创建为bean实例。** 

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

`@Configuration`类中的构造器注入只在Spring 4.3后支持。还要注意的是，如果目标bean只定义了一个构造函数，那么就不需要指定`@Autowired`; 在上面的例子中，`@Autowired`在 `RepositoryConfig`  构造器中是不必要的。

###### *Fully-qualifying imported beans for ease of navigation*

在上面的场景中，使用`@Autowired`很好地工作，并提供了所需的模块化，但是，确定autowire bean定义的确切位置仍然有些模糊。 例如，作为一个查看`ServiceConfig`的开发人员，如何确切地知道`@Autowire`的` AccountRepository` 帐户的帐户是在何处声明？ 它在代码中并不明确，这可能还好。记住 [Spring Tool Suite](https://spring.io/tools/sts) 提供了渲染图形展示如何你需要的被wired up的。你的ide会轻松找到 `AccountRepository`类型的使用，快速找到返回那个类型的 `@Bean` 方法的位置。

如果这种模糊性是不可接受的，并且您希望在IDE中从一个`@Configuration`类直接导航到另一个`@Configuration`类，可以考虑autowire配置类本身： 

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在上面的情况中，它是完全明确的，在哪里定义了 `AccountRepository`。 然而，`ServiceConfig`现在与 `RepositoryConfig`紧密耦合; 这是一种折衷。 这种紧密耦合可以通过使用基于接口或抽象类的`@Configuration`类来减轻。 考虑以下：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在`ServiceConfig`与具体的`DefaultRepositoryConfig`是松耦合的,内置的IDE工具仍然很有用： 开发人员很容易就能获得`RepositoryConfig` implementations的类型的层次结构。  通过这种方式，导航`@Configuration`类及其依赖项与通常的基于接口的代码的过程没有什么不同。 

**如果你想影响某些beans的启动创建顺序，考虑声明`@Lazy` (在第一次访问时创建，而不是启动 )或者在其他的bean上声明`@DependsOn` （确保在当前bean之前创建特定的其他bean，超越了后者的直接依赖 ）。**

##### Conditionally include @Configuration classes or @Bean methods

有条件地启用或禁用完整的`@Configuration`类通常是有用的，甚至是单独的`@Bean`方法，基于任意的系统状态。 一个常见的例子是，只有在Spring环境中启用特定概要文件时才使用`@Profile`标注来激活bean。(see [Bean definition profiles](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) for details)

`@Profile`  注解是使用更灵活的叫 [`@Conditional`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html)的注解实现的。`@Conditional`  注解指定`org.springframework.context.annotation.Condition` 实现，在注册@bean之前，应该被咨询。

`Condition`接口的实现只简单提供了一个 `matches(…)`  方法返回true或false。例如，以下是用于@profile的实际情况实现： 

```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```

See the [`@Conditional` javadocs](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html) for more detail. 

#####  Combining Java and XML configuration

Spring的`@Configuration`类支持并不旨在完全替代Spring XML 。一些诸如Spring XML名称空间之类的设施仍然是配置容器的理想方式。 在XML方便或必要的情况下，您可以选择： 要么以"XML-centric" 的方式实例化容器，例如`ClassPathXmlApplicationContext`，要么用"Java-centric"的形式使用`AnnotationConfigApplicationContext` and the `@ImportResource`  注解导入需要的XML注解。

###### XML-centric use of @Configuration classes 

最好从XML中引导Spring容器，并以特别的方式包含`@Configuration`类。 例如，在一个使用Spring XML的大型现有代码库中， 在需要的基础上创建`@Configuration`类会更容易，并且可以在现有的XML文件中包含它们。 下面您将找到在这种"XML-centric" 的情况下使用`@Configuration`类的情况。 

###### *Declaring @Configuration classes as plain Spring `<bean/>` elements* 

请记住，`@Configuration`类终究只是容器中的bean定义。 在这个例子中，我们创建了一个名为`AppConfig`的`@configuration`类，并在 `system-test-config.xml`作为一个 `<bean/>`定义。因为`<context:annotation-config/>`  被打开，容器将识别`@Configuration`注释，并正确处理`AppConfig`中声明的`@Bean`方法。 

```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

**system-test-config.xml**: 

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**jdbc.properties**: 

```
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

**在`system-test-config.xml`  ，`AppConfig` `<bean/>`  没有声明 `id`元素。虽然这样做是可以接受的，但是没有必要使用其他bean来引用它，而且它也不太可能从容器中显式地获取它的名称。 同样，`DataSource` bean也只使用类型自动连接，因此不需要显式的bean `id`。** 

###### *Using <context:component-scan/> to pick up `@Configuration` classes* 

因为`@Configuration` 使用了 `@Component`元注解，`@Configuration`注解类是自动扫描的候选。我们定义`system-test-config.xml`利用组件扫描优势。注意在此例，我们不需要显式地声明 `<context:annotation-config/>`，因为`<context:component-scan/>` 已经有同样效果。

**system-test-config.xml**: 

```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

###### @Configuration class-centric use of XML with @ImportResource 

在应用程序中，`@Configuration`类是配置容器的主要机制， 它仍然可能需要至少使用一些XML。在这些场景中，只需使用`@ImportResource`，并只定义所需的XML。这样做可以实现一个“以java为中心”的方法来配置容器，并将XML保持在最低限度。 

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
} 
```

```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

### 1.13. Environment abstraction

 [`Environment`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/core/env/Environment.html)是集成在容器中的一个抽象，在应用环境的两个关键方面建模：[*profiles*](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) and [*properties*](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction)。

profile是一个命名的、逻辑的bean定义组，只有当给定的概要文件处于活动状态时才会在容器中注册。 bean可以被分配到一个profile中，无论是在XML中定义还是通过注释。环境对象与概要文件关系的作用是确定哪些profile（如果有的话）当前是活动的，哪些profile（如果有的话）应该在默认情况下是活动的。 Properties在几乎所有应用程序中都扮演着重要的角色，可能来自各种各样的来源：properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects, Maps, and so on。带有properties 的`Environment`对象  为用户提供一个方便的服务接口，用于配置属性源并从它们中解析属性。 

#### 1.13.1. Bean definition profiles

Bean定义profiles是核心容器中的一种机制，它允许在不同的环境中注册不同的Bean。  *environment*  这个词对于不同用户可能不同，在很多情况有用：

- 在开发中使用内存中的数据源，在QA或生产中查找来自JNDI的相同数据源 
- 只在将应用程序部署到性能环境时才注册监测基础设施 
- 为客户A和客户B的部署注册定制的bean实现

让我们考虑一个实际应用程序中的第一个用例，它需要一个数据源。 在测试环境中，配置可能如下： 

```
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在让我们考虑一下如何将这个应用程序部署到QA或生产环境中，假定应用程序的数据源将在生产应用服务器的JNDI目录中注册。我们的数据源bean现在看起来是这样的： 

```
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何在使用这两种基于当前环境的变量之间切换。 随着时间的推移，Spring用户已经设计了许多方法来完成这项工作，通常依赖系统环境变量结合包含`${placeholder}`的`<import/>`  声明。根据环境变量的值解析正确的配置文件路径。Bean定义profile是一个核心容器特性，它为这个问题提供了解决方案。 

如果我们将示例用例推广到特定于环境的bean定义之上， 我们最终需要在特定的上下文中注册某些bean定义，而不是在其他上下文中。 你可以说你想在情形A中注册一个bean定义的概要文件，在情形b中有一个不同的概要文件。 让我们首先看看如何更新我们的配置以反映这种需求。 

##### @Profile

 [`@Profile`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/Profile.html) 注解允许您指出当一个或多个指定概要文件处于active状态时，组件才有资格进行注册。使用上面的例子，我们可以按照以下方式重写数据源配置：

```
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

**如前所述，使用 `@Bean` 方法，您通常会选择使用程序性JNDI查找：要么使用Spring的`JndiTemplate`/`JndiLocatorDelegate` helpers 要么用the straight JNDI `InitialContext`，不要用`JndiObjectFactoryBean`  变体，这会强制你返回类型为`FactoryBean` 。**

`@Profile` 可以作为 [meta-annotation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-meta-annotations) ，来创建自定义组合注解。下面的例子定义了一个  `@Production`  注解可以作为 `@Profile("production")`的替代。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

**如果`@Configuration`类用`@Profile`标记，所有 类关联的 `@Bean` methods and `@Import` annotations将被忽略除非指定的profiles是active的。If a `@Component` or `@Configuration` class is marked with `@Profile({"p1", "p2"})`，p1或p2avtive。如果是 p1,!p2。p1active，p2 not active。**

@profile也可以在方法级声明来只包含一个configuration类的特定bean，例如对于特定的bean的替代变体： 

```
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development")
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production")
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

**使用`@Profile` on `@Bean` methods，一个特殊的情况可能会有： 在相同的Java方法名（类似于构造函数重载）的重载@bean方法的情况下 需要在所有重载方法上一致声明`@Profile`条件。如果条件不一致 ，只有在重载方法中第一次声明才会起作用。 `@Profile`  因此不能用于选择重载的方法，并在另一种情况下使用特定的参数签名；对于同一bean的所有工厂方法之间的解析遵循Spring的构造器解析算法。** 

**如果您想要定义具有不同profile条件下的alternative bean， 使用不同的Java方法名，通过@bean name属性指向相同的bean名称，如上面的例子所示。 如果参数签名都是相同的（例如，所有的变体都有无arg工厂方法） ，这是在一个有效的Java类中表示这种安排的惟一方法（因为只有一个特定名称和参数签名的方法）。** 

#####  XML bean definition profiles

 `<beans>` 的profile属性和注解等同。上面的样例配置可以在两个XML文件中重写： 

```
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

 ```
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
 ```

也可以避免在同一个文件中出现分割和嵌套的`<beans/>`元素： 

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

 `spring-bean.xsd` 约束允许像这样的只作为文件最后一个的节点。这应该有助于提供灵活性，而不会导致XML文件中的混乱。 

##### Activating a profile

现在我们已经更新了我们的配置， 我们仍然需要指定Spring的哪个 profile 是active的。 如果我们现在开始我们的样例应用程序，我们会看到`NoSuchBeanDefinitionException` 的抛出，因为容器无法找到名为dataSource的Spring bean。 

激活一个profile可以用几种方式来完成， 但最直接的方法是通过编程方式对`Environment`  API进行编程，这是通过`ApplicationContext`提供的：

```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，概要文件也可以通过`spring.profiles.active` 声明的方式被激活，可通过 system environment variables, JVM system properties, servlet context parameters in `web.xml`, or even as an entry in JNDI (see [PropertySource abstraction](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction)) 。在集成测试中，active profiles可以通过`spring-test`模块中的`@ActiveProfiles`注解声明（see [Context configuration with environment profiles](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/testing.html#testcontext-ctx-management-env-profiles) ）。 

注意，概要文件并不是一个“非此即为”的命题；同时激活多个概要文件是可能的。 以编程的方式，简单地为`setActiveProfiles()`方法提供多个概要文件名，该方法接受`String…` varargs: 

```
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

声明式地`spring.profiles.active`  可以接受一个逗号分隔的概要文件名列表： 

```
-Dspring.profiles.active="profile1,profile2"
```

##### Default profile

默认概要文件表示默认启用的概要文件。考虑以下: 

```
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有active profile，则会创建上面的数据源; 这可以看作是为一个或多个bean提供缺省定义的一种方式。 如果启用了概要文件，则默认概要文件将不适用。 

default profile 的name可以使用 `Environment`  的`setDefaultProfiles()`  或者声明`spring.profiles.default`  属性来改变。

####  1.13.2. PropertySource abstraction

Spring `Environment`  抽象在一个可配置的属性源层次上提供搜索操作。 为了充分解释，请考虑以下： 

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("Does my environment contain the 'foo' property? " + containsFoo);
```

在上面的代码片段中，我们看到了一种高级的询问Spring的方式，即foo属性在当前环境被定义。 为了回答这个问题，`Environment`对象对一组 [`PropertySource`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html)对象进行搜索。 一个 [`PropertySource`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html) 是 对任何来源的key-value的简单抽象，并且Spring的 [`StandardEnvironment`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html) 配置了两个 PropertySource 对象——一个表示JVM系统属性集(*a la* `System.getProperties()` )，一个对应系统环境变量的集合(*a la* `System.getenv()` )。

这些默认的属性源存在于`StandardEnvironment`中，用于standalone应用程序。  [`StandardServletEnvironment`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/web/context/support/StandardServletEnvironment.html) 填充了额外的默认属性源包括servlet config and servlet context parameters 。它可以选择性地启用 [`JndiPropertySource`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/jndi/JndiPropertySource.html)。详情看javadocs。

具体地，我们使用`StandardEnvironment`，调用 `env.containsProperty("foo")`  ，如果 `foo`  系统属性或环境变量在运行时存在则返回true。

**所执行的搜索是分层的。默认情况下，系统属性优先于环境变量，所以如果foo属性碰巧在对 `env.getProperty("foo")`的调用中同时设置，系统属性值将“胜出”，并优先于环境变量返回。请注意，属性值不会被合并，而是被前面的条目完全覆盖。**

 

**对于一个常见的标准 `StandardServletEnvironment`，完整的层次结构如下所列，最高优先级的条目在最顶：**

- **ServletConfig parameters (if applicable, e.g. in case of a `DispatcherServlet` context)**
- **ServletContext parameters (web.xml context-param entries)**
- **JNDI environment variables ("java:comp/env/" entries)**
- **JVM system properties ("-D" command-line arguments)**
- **JVM system environment (operating system environment variables)**

最重要的是，整个机制是可配置的。 也许你有一个定制的属性源你想要集成到这个搜索中。没问题——简单地实现并实例化您自己的PropertySource，并将其添加到当前环境的PropertySource集合中： 

```
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

上面的代码，`MyPropertySource` 添加到搜索的最高优先级 [`MutablePropertySources`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html) API 暴露了一系列方法允许对一组属性源进行精确的操作。 

#### 1.13.3. @PropertySource

 [`@PropertySource`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) 注解提供了一种方便的声明机制，可以将`PropertySource`添加到Spring的`Environment`中。 

给出"app.properties" 文件，包含k-v对`testbean.name=myTestBean`，下面的`@Configuration `类使用`@PropertySource` 这种方式 调用`testBean.getName()`  会返回"myTestBean"。

```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
} 
```

在 `@PropertySource`上的占位符会解析注册在环境的属性集。例如：

```
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设"my.placeholder" 存在于某个属性源且已经被注册，例如system properties or environment variables，占位符将被解析为相应的值。 如果没有，那么"default/path"将被用作默认值，如果没有指定默认值，并且不能解决一个属性，  抛出`IllegalArgumentException` 。

**根据Java 8约定， `@PropertySource`注解可以重复。然而，所有这些`@PropertySource`注解都需要在相同的级别上声明： 要么直接在configuration类上，要么在同一个自定义注解中作为元注解。 不推荐混用direct注解和元注解，因为direct注解实际会覆盖元注解。**

####  1.13.4. Placeholder resolution in statements

历史上，元素中的占位符的值只能通过JVM系统属性或环境变量来解决。 现在不再是这种情况了。 因为环境抽象被集成到整个容器中， 通过它来解决占位符的问题是很容易的。 这意味着您可以以任何您喜欢的方式配置解析过程： 更改通过系统属性和环境变量进行搜索的优先级， 或者完全删除它们 ;在适当的时候添加您自己的属性源。 

具体地说，不管`customer`属性的定义在哪里，只要在`Environment`中可用，下面的语句就可以工作： 

```
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

### 1.14. Registering a LoadTimeWeaver

`LoadTimeWeaver`是Spring用来在类被加载到JVM的时候动态转化它们。

添加`@EnableLoadTimeWeaving`到`@Configuration` 类其中来使load-time weaving可用：

```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

或者使用XML配置`context:load-time-weaver` 节点：

```
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦`ApplicationContext`配置了这个，所有`ApplicationContext`里的bean都会实现`LoadTimeWeaverAware`，因而接收到 the load-time weaver实例的引用。 这对于Spring的JPA支持尤其有用，因为JPA类转换可能需要加载时编织。 Consult the `LocalContainerEntityManagerFactoryBean` javadocs for more detail.   For more on AspectJ load-time weaving, see [Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw)。

###  1.15. Additional capabilities of the ApplicationContext

如前面章节讨论的，`org.springframework.beans.factory`包提供了管理和操作bean的基本功能，包括用编程式的方式。`org.springframework.context`包添加[`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)接口，它继承了`BeanFactory`接口，另外扩展其他接口以提供更多more *application framework-oriented style*的功能。许多人以一种完全声明式的方式使用`ApplicationContext `，甚至不编程式创建它，而是依赖于诸如`ContextLoader`之类的支持类来自动实例化一个`ApplicationContext`作为Java EE web应用程序的正常启动过程的一部分。 

为了用more framework-oriented style的方式增强 `BeanFactory`功能， context 包也提供以下功能：

- *Access to messages in i18n-style*, through the `MessageSource` interface.
- *Access to resources*, such as URLs and files, through the `ResourceLoader` interface.
- *Event publication* to namely beans implementing the `ApplicationListener` interface, through the use of the `ApplicationEventPublisher` interface.
- *Loading of multiple (hierarchical) contexts*, allowing each to be focused on one particular layer, such as the web layer of an application, through the `HierarchicalBeanFactory` interface.

#### 1.15.1. Internationalization using MessageSource

 `ApplicationContext`扩展了`MessageSource`接口，因此提供国际化（i18n）功能。 Spring也提供了 `HierarchicalMessageSource`，分层解析message。这些接口一起提供了Spring效应消息解析的基础。 在这些接口上定义的方法包括： 

- `String getMessage(String code, Object[] args, String default, Locale loc)`:  用于从`MessageSource`检索消息的基本方法。 通过使用标准库提供的MessageFormat功能，传递的任何参数都变成了替换值。 
- `String getMessage(String code, Object[] args, Locale loc)`: no default message can be specified; if the message cannot be found, a `NoSuchMessageException` is thrown. 
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`: 在前面的方法中使用的所有属性也被封装在一个名为`MessageSourceResolvable`的类中，通过该方法使用。

当`ApplicationContext` 被加载，会自动搜索context里的`MessageSource` 。bean必须有`messageSource` name。如果找到了这样的bean，所有对上述方法的调用都被委托给消息源。 如果没有找到任何消息源，则`ApplicationContext`试图找到包含同名bean的parent如果是这样，它就会使用那个bean作为`MessageSource`。如果`ApplicationContext`找不到任何消息源，  为了能够接受对上面定义的方法的调用，一个空的`DelegatingMessageSource `被实例化。 

Spring提供了两个`MessageSource`实现， `ResourceBundleMessageSource` and `StaticMessageSource`。两者都实现了`HierarchicalMessageSource`，以便进行嵌套消息传递。 `StaticMessageSource` 很少用，但是提供了编程式向source添加消息的方法。`ResourceBundleMessageSource` 如下所示：

```
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

在这个例子中，假设您有三个资源包，在类路径中定义为`format`， `exceptions` and `windows`。任何解决消息的请求都将以JDK标准的方式处理，通过resourcebundle来解析消息。 出于本例的目的，假设上述两个资源包文件的内容是…… 

```
# in format.properties
message=Alligators rock!
```

```
# in exceptions.properties
argument.required=The {0} argument is required.
```

下一个示例中显示了执行`MessageSource`功能的程序。 请记住，所有的`ApplicationContext`实现都是`MessageSource`实现，因此可以将其转换为`MessageSource`接口。 

```
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", null);
    System.out.println(message);
}
```

由此产生的输出结果将是。 

```
Alligators rock!
```

总而言之，`MessageSource`是在一个名为 `beans.xml`的在你的classpath的文件中定义的。`messageSource`定义通过其`basenames`属性引用了一系列 resource bundles。列表中传递给`basenames`的三个文件在classpath分别叫做 `format.properties`, `exceptions.properties`, and `windows.properties`。

下一个示例展示了传递给消息查找的参数; 这些参数将被转换为字符串，并插入到查找消息中的占位符中。 

```
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.foo.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", null);
        System.out.println(message);
    }
}
```

由`execute()`方法调用产生的输出将是。 

```
The userDao argument is required.
```

关于国际化（i18n）， Spring的各种 `MessageSource`实现和JDK `ResourceBundle`遵循相同的 locale resolution and fallback rules。简而言之，并继续使用前面定义的messageSource的示例， 如果您想要解决针对英国（en-GB）地区的消息， 你要创建`format_en_GB.properties`, `exceptions_en_GB.properties`, and `windows_en_GB.properties` 。

通常，, locale resolution是由应用周围环境来管理的。在本例中，将解析（英式）消息的场所是手动指定的。 

```
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the {0} argument is required, I say, required.
```

```
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

运行上述程序的结果将是…… 

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```

您还可以使用`MessageSourceAware`接口来获得对已定义的`MessageSource`的引用。 所有定义在 `ApplicationContext`的bean都实现了`MessageSourceAware`接口，当bean被创建和配置时注入了application context 的`MessageSource`。

**作为`ResourceBundleMessageSource`的另一种选择， Spring提供了`ReloadableResourceBundleMessageSource`类。 该变种支持相同的bundle文件格式，但是比基于标准JDK的`ResourceBundleMessageSource`实现更灵活。 特别地，它允许从任何Spring资源位置读取文件（不仅仅是从类路径中读取） ，支持热重载 bundle property files（同时有效地将它们缓存）。 Check out the `ReloadableResourceBundleMessageSource` javadocs for details。**

#### 1.15.2. Standard and custom events

 `ApplicationContext`的Event handling通过 `ApplicationEvent`类和`ApplicationListener` 接口提供。如果一个实现`ApplicationListener`接口的bean被部署在context，每次`ApplicationEvent` 发布到`ApplicationContext`，那个bean就会被通知。本质上，这是标准的观察者设计模式。 

**自Spring 4.2，事件基础设施得到了显著改善并且提供了一个[annotation-based model](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-functionality-events-annotation) 和能够发布任意事件的能力，那是个不需要继承`ApplicationEvent`的对象。当这样的对象被发布时，我们将它封装在一个事件中。** 

Spring provides the following standard events: 

*Table 7. Built-in Events*

| Event                   | Explanation                                                  |
| ----------------------- | ------------------------------------------------------------ |
| `ContextRefreshedEvent` | 当  `ApplicationContext`初始化或被刷新被发布，例如使用 `ConfigurableApplicationContext`接口的 `refresh()`方法。这里的“初始化”意味着 所有bean被加载， post-processor beans被检测和激活，单例bean被 pre-instantiated，  `ApplicationContext` 对象已经可用。只要  context还没被关闭，如果选中的 `ApplicationContext`  支持这样的  "hot" refreshes ，一个 refresh 可以被触发多次。例如 `XmlWebApplicationContext` 支持热刷新，但是 `XmlWebApplicationContext`不支持。 |
| `ContextStartedEvent`   | 在 `ApplicationContext` 被Started，使用  `ConfigurableApplicationContext` 接口上的 `start()` 方法时被调用。 "Started" 这里意味着 all `Lifecycle` beans 接收一个start信号。 通常，这个信号用于在显式停止后重新启动bean， 但是它也可以用来启动没有为自动启动配置的组件， 例如，那些还没有在初始化时启动的组件。 |
| `ContextStoppedEvent`   | 当 `ApplicationContext`被stopped时发布，使用  `ConfigurableApplicationContext`接口上的 `stop()`  方法。这里的"stopped"意味着 all `Lifecycle` beans 接收一个显示的stop信号。一个已经stopped的context可以调用 `start()`重启。 |
| `ContextClosedEvent`    | 当 `ApplicationContext` 被closed时发布，使用  `ConfigurableApplicationContext`接口的`close()`方法。这里的"Closed"意味着所有的单例bean摧毁。一个closed的context到达生命终结；不能被刷新和重启。 |
| `RequestHandledEvent`   | 一个web特定的事件告诉所有beans，一个HTTP请求已经被serviced。这个事件在请求完成之后被发布。 这个事件只适用于使用Spring的`DispatcherServlet`的web应用程序。 |

您还可以创建和发布您自己的自定义事件。这个例子展示了一个简单的类，它扩展了Spring的`ApplicationEvent`基类： 

```
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String test;

    public BlackListEvent(Object source, String address, String test) {
        super(source);
        this.address = address;
        this.test = test;
    }

    // accessor and other methods...
}
```

要发布一个自定义的`ApplicationEvent`，请在`ApplicationEventPublisher`上调用`publish event()`方法。通常是创建一个类实现`ApplicationEventPublisherAware`并作为一个bean注册。下面的例子演示了这样一个类： 

```
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String text) {
        if (blackList.contains(address)) {
            BlackListEvent event = new BlackListEvent(this, address, text);
            publisher.publishEvent(event);
            return;
        }
        // send email...
    }
} 
```

在配置时，Spring容器将检测到`EmailService`实现`ApplicationEventPublisherAware`，并将自动调用`setApplicationEventPublisher()`。在现实中，传递进来的参数将是Spring容器本身; 您只是通过它的`ApplicationEventPublisher`接口与application context交互。  

要接收自定义`ApplicationEvent`， 创建一个类实现`ApplicationListener`把它注册为Spring bean。下面的例子演示了这样一个类： 

```
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

注意 `ApplicationListener`类参数化了你的自定义事件 `ApplicationListener`。这意味着`onApplicationEvent()`方法可以保持类型安全，避免了向下转型的需要。您可以按照自己的意愿注册尽可能多的事件监听器，但是请注意，在默认情况下，事件监听器会同步接收事件。 这意味着`publishEvent()`方法会阻塞，直到所有listener都完成了事件处理。 这种同步和单线程方法的一个优点是，当侦听器接收到事件时， 如果事务上下文可用，它在发布者的事务上下文中操作。 如果事件发布的另一种策略变得必要，请参考Spring的`ApplicationEventMulticaster`接口的javadoc。 

下面的例子展示了用于注册和配置上面的每个类的bean定义： 

```
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```

将它们放在一起，当`emailService` bean的`sendEmail()`方法被调用时， 如果有任何邮件应该被列入黑名单，自定义事件`BlackListEvent` 就被发布。`blackListNotifier`  bean被注册为`ApplicationListener`因此可以接收`BlackListEvent`, 在这点通知appropriate parties。

Spring的事件机制是为在相同的application context中的bean之间的简单通信而设计的。 然而，对于更复杂的企业集成需求， 独立维护的[Spring Integration](https://projects.spring.io/spring-integration/)为building lightweight, [pattern-oriented](http://www.enterpriseintegrationpatterns.com/), event-driven architectures提供了完整的支持，

#####  Annotation-based event listeners

自Spring 4.2，事件listener可以通过`EventListener`注解注册在任何managed bean的pubic方法上。`BlackListNotifier`可以重写如下： 

```
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

如上面看到的，方法签名再次声明了其监听事件的类型，但是这一次使用了一个灵活的名称，并且没有实现特定的listener接口。 事件类型可以被通过泛型缩小范围，只要事件实际类型在实现层级上加上泛型参数。

如果你的方法要监听几个事件，如果你想要定义它没有参数， 事件类型(s)也可以在注释本身上指定： 

```
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    ...
}
```

也可以通过注解的 `condition` 属性添加运行时过滤器，这定义了一个 [`SpEL`expression](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#expressions) 应该匹配调用特殊实际的实际方法。

例如，我们的notifier可以被重写，只有当事件的`test`属性等于foo时才会被调用： 

```
@EventListener(condition = "#blEvent.test == 'foo'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个SpEL表达式再次计算一个专用context。下表列出了可用于上下文的项目，以便可以在条件事件处理中使用它们： 

*Table 8. Event SpEL available metadata*

| Name            | Location           | Description                                                  | Example                                                      |
| --------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Event           | root object        | The actual `ApplicationEvent`                                | `#root.event`                                                |
| Arguments array | root object        | The arguments (as array) used for invoking the target        | `#root.args[0]`                                              |
| *Argument name* | evaluation context | Name of any of the method arguments. If for some reason the names are not available (e.g. no debug information), the argument names are also available under the `#a<#arg>` where *#arg* stands for the argument index (starting from 0). | `#blEvent` or `#a0` (one can also use `#p0` or `#p<#arg>`notation as an alias). |

注意 `#root.event` 允许你访问底层事件，即使你的方法签名实际上是指一个被发布的任意对象。

 如果您需要发布一个事件作为处理另一个事件的结果， 只需改变方法签名以返回应该发布的事件，比如： 

```
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

**这个特性不支持 [asynchronous listeners](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-functionality-events-async)。**

这个方法将为每一个`BlackListEvent`  发布一个新的 `ListUpdateEvent` 。如果您需要发布几个事件，只需返回一组事件。 

##### Asynchronous Listeners

如果您想要一个特定的listener处理异步事件，只需重用 [regular `@Async` support](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/integration.html#scheduling-annotation-support-async): ： 

```
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```

在使用异步事件时要注意以下限制： 

1. 如果事件监听器抛出一个异常，它将不会被传播到调用者， 请查看 `AsyncUncaughtExceptionHandler` 了解更多细节。 
2. 这样的事件监听器不能发送回复。 如果你需要发送另一个事件作为处理的结果，注入`ApplicationEventPublisher`以手动发送事件。 

##### Ordering listeners

如果你需要在另一个之前调用监听器， 只需在方法声明中添加@order注释： 

```
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

#####  Generic events

您还可以使用泛型来进一步定义事件的结构。 考虑一个`EntityCreatedEvent<T>`，其中T是被创建的实体的类型。 您可以创建下列listener定义，只接收 `Person` 的`EntityCreatedEvent`： 

```
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    ...
}
```

由于类型擦除, 只有被触发的事件使用了事件监听器过滤的泛型参数（s） 才有用（就像class `PersonCreatedEvent extends EntityCreatedEvent<Person> { … }`) 

在某些情况下，如果所有事件都遵循相同的结构，这可能会变得相当乏味 （就像上面的事件一样 ）。在这种情况下，您可以实现`ResolvableTypeProvider`，以指导框架超出运行时环境所提供的范围： 

```
public class EntityCreatedEvent<T>
        extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(),
                ResolvableType.forInstance(getSource()));
    }
}
```

**这不仅适用于`ApplicationEvent`，也适用于您作为事件发送的任意对象。** 

####  1.15.3. Convenient access to low-level resources

为了优化使用和理解application contexts，用户通常应该熟悉Spring的`Resource`抽象，as described in the chapter [Resources](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#resources)。

一个application context 是一个`ResourceLoader`，用于加载 `Resource`s 。一个`Resource`本质上是JDK class`java.net.URL`更丰富特性的版本，实际上，`Resource` 的实现适当包装了一个`java.net.URL`实例。资源可以以透明的方式从几乎任何位置获得低层次的资源， 包括从类路径、文件系统位置、任何可以用标准URL描述的地方，以及其他一些变体。 如果资源位置字符串是一个没有任何特殊前缀的简单路径， 这些资源的来源是特定的，并且适用于实际的application context类型。 

您可以配置部署bean到 application context来实现特殊的回调接口， `ResourceLoaderAware`，在初始化阶段被自动回调，application context本身作为`ResourceLoader`被传入。你也可以暴露`Resource`的属性，用于访问静态资源; 它们会像其他任何属性一样被注入到里面。 你可以把这些`Resource`属性指定为简单的字符串路径， 并且依赖于一个特殊的`JavaBean PropertyEditor`，它是由context自动注册的， 在部署bean时，将这些文本字符串转换成实际的资源对象。 

提供给`ApplicationContext`构造函数的位置路径(s)实际上是资源字符串，在简单的形式中，对特定的context实现进行适当的处理。 `ClassPathXmlApplicationContext`将一个简单的位置路径视为类路径位置。 您还可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或URL加载定义，不管具体的context类型。

#### 1.15.4. Convenient ApplicationContext instantiation for web applications

您可以声明地创建`ApplicationContext`实例通过例如`ContextLoader`。当然你也可以用`ApplicationContext`实现编程式创建。

使用`ContextLoaderListener`  注册 `ApplicationContext`如下：

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

监听器检查`contextConfigLocation` 参数。如果参数不存在，则listener使用`/WEB-INF/applicationContext.xml`作为默认。当参数确实存在时，listener通过使用预定义的分隔符（逗号、分号和空格）分隔字符串，使用这些值作为搜索应用context的位置。 ant样式的路径模式也得到了支持。 例子是`/WEB-INF/*Context.xml`，`/WEB-INF/**/*Context.xml `。

####  1.15.5. Deploying a Spring ApplicationContext as a Java EE RAR file

可以将Spring ApplicationContext部署为RAR文件， 封装context，和在Java EE RAR部署单元中所有必需的bean类和库jar 。这就相当于引导一个standalone ApplicationContext，只是在Java EE环境中托管， 能够访问Java EE服务器设施。 对于部署一个无头的WAR文件，RAR部署是更自然的选择， 实际上，一个WAR文件没有任何HTTP入口点，只用于在Java EE环境中引导Spring ApplicationContext。 

RAR部署对于不需要HTTP入口点的应用程序上下文来说是理想的，但它只包含消息端点和预定的作业。 在这样的环境中，bean可以使用诸如JTA事务管理器和JNDI绑定JDBC数据源和JMS ConnectionFactory实例之类的应用程序服务器资源，也可以通过Spring的标准事务管理和JNDI和JMX支持工具进行注册。 应用程序组件还可以通过Spring的TaskExecutor抽象与应用服务器的JCA WorkManager交互。 

Check out the javadoc of the [`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) class for the configuration details involved in RAR deployment. 

为了将Spring ApplicationContext简单地部署为Java EE RAR文件：将所有应用程序类打包成RAR文件，这是一个标准的JAR文件，具有不同的文件扩展名。 将所有必需的库jar添加到RAR文件的root。添加一个"META-INF/ra.xml"部署描述符（as shown in `SpringContextResourceAdapter`s javadoc ）和对应的Spring XML bean定义文件（通常是 "META-INF/applicationContext.xml "），然后将产生的RAR文件放入应用程序服务器的部署目录中。 

**这样的RAR部署单元通常是自包含的; 他们不会把组件暴露给外部，甚至不包括同一应用程序的其他模块。 与基于极少的应用程序上下文的交互通常通过它与其他模块共享的JMS有目的地进行。  一个基于rar的应用程序上下文也可以，例如，调度一些作业，对文件系统中的新文件做出反应（或者类似的）。如果它需要允许外部的同步访问，它可以例如导出RMI端点，这样当然也可以被其他应用程序模块在同一台机器上使用。**  

###  1.16. The BeanFactory

 `BeanFactory`为Spring IoC容器提供了底层基础，但它只在与其他第三方框架的集成中直接使用，对于Spring的大多数用户来说，现在基本成为历史。 `BeanFactory`  和相关接口比如 `BeanFactoryAware`, `InitializingBean`, `DisposableBean` ，在Spring中仍然存在，目的是向后兼容与Spring集成的大量第三方框架。 通常，第三方组件不能使用更现代的等价物，比如`@PostConstruct` or `@PreDestroy` ，以避免对JSR-250的依赖。 

本节提供`BeanFactory` and `ApplicationContext`之间差异的背景，以及如何通过经典的单例查找直接访问IoC容器。 

####  1.16.1. BeanFactory or ApplicationContext?

除非你有充分的理由不这样做，否则使用`ApplicationContext`。 

 `ApplicationContext`  包含 `BeanFactory` 全部功能，它通常是在`BeanFactory`中推荐的 ，除了一些情况，比如在资源受限的设备上运行的嵌入式应用程序，内存消耗可能非常重要，而额外的kilobytes 可能会带来不同。 然而，对于大多数典型的企业应用程序和系统来说，`ApplicationContext`是您想要使用的。 Spring大量使用[`BeanPostProcessor` extension point](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp) （来影响代理，等等 ）。如果你只使用原生的`BeanFactory，` 大量的支持如事务和AOP将不会生效，至少在你的部分上没有额外的步骤。这种情况可能会让人感到困惑，因为配置本身并没有什么问题。 

下表列出了`BeanFactory`和`ApplicationContext`接口和实现提供的特性。 

*Table 9. Feature Matrix*

| Feature                                          | `BeanFactory` | `ApplicationContext` |
| ------------------------------------------------ | ------------- | -------------------- |
| Bean instantiation/wiring                        | Yes           | Yes                  |
| Automatic `BeanPostProcessor`registration        | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor`registration | No            | Yes                  |
| Convenient `MessageSource` access (for i18n)     | No            | Yes                  |
| `ApplicationEvent` publication                   | No            | Yes                  |

要显式地用`BeanFactory`实现来显式地注册一个`BeanPostProcessor`，您需要编写这样的代码： 

```
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
MyBeanPostProcessor postProcessor = new MyBeanPostProcessor();
factory.addBeanPostProcessor(postProcessor);

// now start using the factory
```

要在使用`BeanFactory`实现时显式地注册`BeanFactoryPostProcessor`，您必须编写这样的代码： 

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertyPlaceholderConfigurer cfg = new PropertyPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在这两种情况下，显式注册步骤都不方便， 这就是为什么在绝大多数spring支持的应用程序中，不同的`ApplicationContext`实现比普通的`BeanFactory`实现更受欢迎的原因，特别是在使用特别是在使用`BeanFactoryPostProcessor`s 和`BeanPostProcessor`s。这些机制实现了重要的功能，比如属性占位符替换和AOP。 

## Resources

###  2.1. Introduction

不幸的是，Java的标准Java.net.URL类和各种URL前缀的标准处理程序还不足以满足所有底层资源的访问。 例如，没有标准化的URL实现，可以用来访问从类路径，或者相对于ServletContext获得资源。 虽然可以为专门的URL前缀注册新的handlers（类似于诸如http：），这通常是相当复杂的，URL接口仍然缺少一些令人满意的功能，比如检查所指向的资源的存在的方法。 

###  2.2. The Resource interface

Spring的资源接口是一个更有能力的接口，用于抽象对底层资源的访问。 

```
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();

}
```

```
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;

}
```

资源接口中一些最重要的方法是： 

- `getInputStream()`：定位并打开资源，返回一个InputStream，以便从资源中读取数据。预期每次调用都会返回一个新的`InputStream`。调用者有责任关闭这个stream。
- `exists()`: 返回一个布尔值，指示该资源是否实际物理形式存在于中。 
- `isOpen()`: 返回一个布尔值，指示这个资源是否代表一个带有开放流的句柄。 如果true，`InputStream`不能被多次读取，并且必须只读取一次，然后关闭以避免资源泄漏。对所有常用的资源实现为false，带有`InputStreamResource`的异常。
- `getDescription()`: 返回对该资源的描述， 在处理资源时，用于错误输出。 这通常是完全限定的文件名或资源的实际URL。 

其他的方法允许您获得一个实际的URL或文件对象，表示资源 （如果底层实现是兼容的，并且支持该功能 ）。

`Resource`  抽象在Spring被广泛使用，当需要资源时，在许多方法签名中作为参数类型。 一些Spring api中的其他方法 （例如，各种`ApplicationContext`实现的构造函数 ），取一个未装饰或简单形式的字符串，用于创建适合该context实现的`Resource`，或者通过`String`路径上的特殊前缀，允许调用者指定必须创建和使用特定的资源实现。 

虽然资源接口在Spring和Spring中使用了很多，但在您自己的代码中，作为一个通用的实用程序类，甚至当您的代码不知道或不关心Spring的其他部分时，它实际上是非常有用的。 虽然这将您的代码与Spring结合在一起，但它实际上只将它与这一小组实用程序类结合在一起，它们可以作为更有能力的URL替换，并且可以被认为与您用于此目的的任何其他库都是等同的。 

需要注意的是，资源抽象并不能替代功能：它可以在可能的地方包装它。例如，UrlResource包装一个URL，并使用包装的URL来完成它的工作。 

###  2.3. Built-in Resource implementations

Spring提供了几个开箱即用的`Resource` 实现：

####  2.3.1. UrlResource

`UrlResource` wraps a `java.net.URL`, 可以用来访问任何通过URL访问的对象，比如文件、HTTP目标、FTP目标等等。 所有的URL都有一个标准化的字符串表示，这样就可以使用适当的标准化前缀来表示另一个URL类型。 这个包含文件：`http:`   `ftp:` 等。

UrlResource是由Java代码显式地使用UrlResource构造函数创建的， 但是，当你调用一个API方法时，它通常会被隐式地创建，这个方法接受一个字符串参数，它代表一条路径。 对于后一种情况，`JavaBeans PropertyEditor`将最终决定要创建哪种类型的资源。   如果路径字符串包含一些众所周知的（对它来说就是）前缀，比如`classpath`： 它将为该前缀创建一个适当的专门资源。但是，如果它不识别前缀，它将假定这只是一个标准的URL字符串，并将创建一个`UrlResource`。 

####  2.3.2. ClassPathResource

该类代表了应该从classpath中获得的资源。 它使用线程context class loader、given class loader 或given class for loading resources。 

如果class path资源驻留在文件系统中， 这个资源实现支持作为`java.io.File`的解决方案，但不用于驻留在jar中的classpath资源并没有被扩展到filesystem（通过servlet引擎，或者任何环境 ）。 为了解决这个问题，各种资源实现总是支持解析为`java.net.URL`。 

`ClassPathResource`是由Java代码显式地使用`ClassPathResource`构造器创建的，但是当您调用一个API方法时，它通常会被隐式地创建，该方法接受一个字符串参数，该参数是用来表示路径的。 对于后一种情况，`JavaBeans PropertyEditor`将识别特殊的前缀类路径：在字符串路径上，并在这种情况下创建`ClassPathResource`。 

####  2.3.3. FileSystemResource

这是一个为了`java.io.File` handles的 `Resource` 实现。它明显支持resolution as a `File`, and as a `URL`。

#### 2.3.4. ServletContextResource

这是`ServletContext` resources的`Resource`  实现，在相关web应用程序的根目录中解释相关路径。 。

它总是支持流访问和URL访问， 当web应用程序存档被展开，资源物理存在于文件系统上时，只允许`java.io.File`  访问。无论它是不是在像这样的文件系统中展开，还是直接从jar或其他地方如DB访问（这是可能的），实际上都依赖于Servlet容器。 

####  2.3.5. InputStreamResource

对给定 `InputStream`的`Resource`  实现。只有在没有特定的资源实现的情况下，才应该使用这种方法。 特别地，在可能的情况下，更倾向`ByteArrayResource`或任何基于文件的资源实现。 

与其他资源实现不同，这是一个已经打开的资源的描述符——因此从isOpen（）返回true。 *：*如果您需要将资源描述符保留在某个地方，或者您需要多次读取一条流，则不要使用它。 

#### 2.3.6. ByteArrayResource

这是给定字节数组的资源实现。 它为给定的字节数组创建`ByteArrayInputStream`。它对于从任何给定的字节数组中加载内容非常有用，而不需要使用单一的`InputStreamResource`。 

###  2.4. The ResourceLoader

 `ResourceLoader`  接口由能返回（例如加载）`Resource` 实例的对象实现。

```
public interface ResourceLoader {

    Resource getResource(String location);

}
```

所有application contexts 实现了`ResourceLoader`接口，因此，所有application contexts都可以用来获取资源实例。 

特定application context调用 `getResource()`，指定的位置路径没有特定的前缀 ，您将得到一个适合于特定应用程序上下文的资源类型。 例如，假设下列代码片段是针对`ClassPathXmlApplicationContext`实例执行的： 

```
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

返回的将是一个`ClassPathResource`; 如果对`FileSystemXmlApplicationContext`  执行相同的方法， 得到的是 `FileSystemResource`。对于`WebApplicationContext`，你得到`ServletContextResource`。

因此，您可以以适合于特定应用程序上下文的方式加载资源。 

另一方面，您也可以通过指定`classpath:` 前缀，不管应用程序上下文类型如何，强制使用ClassPathResource。 

```
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

类似地，可以通过指定任何标准的`java.net.URL`前缀来强制使用`UrlResource`。 

```
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```
Resource template = ctx.getResource("http://myhost.com/resource/path/myTemplate.txt");
```

下表总结了将`String`s转换为 `Resource`s 的策略：

| Prefix     | Example                          | Explanation                                                  |
| ---------- | -------------------------------- | ------------------------------------------------------------ |
| classpath: | `classpath:com/myapp/config.xml` | Loaded from the classpath.                                   |
| file:      | `file:///data/config.xml`        | Loaded as a `URL`, from the filesystem. [[3](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#_footnote_3)] |
| http:      | `http://myserver/logo.png`       | Loaded as a `URL`.                                           |
| (none)     | `/data/config.xml`               | Depends on the underlying `ApplicationContext`.              |

### 2.5. The ResourceLoaderAware interface

`ResourceLoaderAware` 是一个特殊的标记接口，识别那些能期望提供`ResourceLoader` 引用的对象。

```
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

当一个类实现 `ResourceLoaderAware`被部署到application context(spring managed bean)，会被识别为 `ResourceLoaderAware` 。application context将然后调用`setResourceLoader(ResourceLoader)`,  把自己作为参数（记住，spring里所有 application contexts 实现了`ResourceLoader`  接口）。

当然，既然 `ApplicationContext` 就是一个 `ResourceLoader` ，bean还可以实现`ApplicationContextAware`接口，并直接使用提供的应用程序上下文来加载资源， 但一般来说，如果需要的话，最好使用专用的`ResourceLoader` 接口。 代码将被耦合到资源加载接口，它可以被认为是一个实用接口，而不是整个Spring` ApplicationContext`接口。  

自spring 2.5，你可以依赖 autowire`ResourceLoader`  作为实现`ResourceLoaderAware` 接口的替代。"traditional" `constructor`  和`byType` autowiring  modes（as described in [Autowiring collaborators](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) ）现在可以分别为构造器参数或setter方法参数提供类型是`ResourceLoader`的依赖。 为了获得更多的灵活性（包括自动连接字段和多种参数方法的能力），可以考虑使用新的基于注解的自动装配特性。 在这种情况下， `ResourceLoader` 会被autowire到一个field，构造器参数，或者是方法参数，它期望`ResourceLoader`类型和field，constructor或者method一致。For more information, see [@Autowired](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation).   

### 2.6. Resources as dependencies  

如果bean本身将通过某种动态过程来确定和提供资源路径， 使用`ResourceLoader`接口来加载资源可能是有意义的。 以某种形式的模板的加载为例，其中需要取决于用户角色的特定资源。如果资源是静态的，那么完全消除 `ResourceLoader` 接口的使用是有意义的，  只需让bean暴露需要的`Resource`  ，会被期望注入到里面。

让接下来注入属性微不足道的是，所有的应用程序上下文都注册并使用一个特殊的`JavaBeans PropertyEditor`，它可以将`String` paths 转化为 `Resource` objects。因此，如果`myBean`有一个template资源的模板属性，那么它可以为该资源配置一个简单的字符串，如下所列： 

```
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>

```

注意资源没前缀，application context自身作为 `ResourceLoader`被使用，然后资源会从`ClassPathResource`, `FileSystemResource`, or `ServletContextResource` (as appropriate)  取决于context类型被加载。

如果需要一个特定的`Resource` 来用，要使用前缀。下面两个例子展示了如何强制使用`ClassPathResource`和`UrlResource`（后者用于访问文件系统文件）。 

```
<property name="template" value="classpath:some/resource/path/myTemplate.txt">

```

```
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>

```

### 2.7. Application contexts and Resource paths  

#### 2.7.1 Constructing application contexts

应用程序上下文构造器（针对特定的应用程序上下文类型）通常使用字符串或字符串数组作为资源（s）的位置路径（s），比如构成上下文定义的XML文件。 

当这样的位置路径没有前缀时，从该路径构建的特定资源类型并用于加载bean定义，依赖于特定的应用程序上下文。例如，如果您创建一个`ClassPathXmlApplicationContext`，如下所列： 

```
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

bean定义将从类路径中加载，因为将使用`ClassPathResource`。但是如果您创建了一个类似于`FileSystemXmlApplicationContext`： 

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```

bean的定义将从文件系统位置加载，在本例中是相对于当前工作目录的。 

请注意，在位置路径上使用特殊的classpath前缀或标准URL前缀将覆盖为加载定义而创建的默认类型的资源。所以这`FileSystemXmlApplicationContext`…… 

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

实际上会从类路径加载它的bean定义。然而，它仍然是一个文档的应用程序上下文。如果它随后被用作资源性器，那么任何未预先确定的路径都将被视为文件系统路径。 

#####  Constructing ClassPathXmlApplicationContext instances - shortcuts

ClassPathXmlApplicationContext公开了大量的构造函数，以支持方便的实例化。其基本思想是，一个只提供一个字符串数组，其中只包含XML文件本身的文件名（没有前导路径信息），另一个提供类;ClassPathXmlApplicationContext将从提供的类派生路径信息。 

```
com/
  foo/
    services.xml
    daos.xml
    MessengerService.class
```

```
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```

请参阅ClassPathXmlApplicationContext javadocs以了解各种构造函数的详细信息。 

#### 2.7.2. Wildcards in application context constructor resource paths

使用通配符

#####  Ant-style Patterns

当路径位置包含一个ant样式模式时，例如： 

```
/WEB-INF/*-context.xml
com/mycompany/**/applicationContext.xml
file:C:/some/path/*-context.xml
classpath:com/mycompany/**/applicationContext.xml
```

##### Implications on portability 

如果指定路径已经是文件URL ，然后，wildcarding保证以一种完全可移植的方式工作。 如果指定的路径是一个classpath位置，那么解析器必须通过classloader.getresource（）调用获得最后一个非通配符路径段URL。 由于这只是路径的一个节点（而不是末端的文件），所以它实际上是未定义的（在类加载器javadocs中），在这种情况下返回的是什么类型的URL。 在实践中，它总是一个java.io。文件表示目录，其中classpath资源解析为文件系统位置，或某种类型的jar URL，其中类路径资源解析为jar位置。尽管如此，这个操作还是有一个可移植性的问题。 

##### The classpath*: prefix

当构造一个基于xml的应用程序上下文时，位置字符串可能会使用特殊的`classpath*:` 前缀：

```
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```

这个特殊的前缀指定了匹配给定名称的所有classpath资源都必须获得（在内部，这实际上是通过`classloader.getresources(...)`调用来实现的），然后合并形成最终的应用程序上下文定义。 

**通配符`classpath`依赖底层classloader的`getResources()`方法。由于现在大多数应用服务器都提供自己的类加载器实现， 在处理jar文件时，这种行为可能会有所不同。 检查类路径是否有效的一个简单测试是使用类加载器从类路径上的jar中加载文件： `getClass().getClassLoader().getResources("<someFileInsideTheJar>")`。用具有相同名称的文件来尝试这个测试，但是将它们放在两个不同的位置。如果不合适的结果被返回，  检查应用程序服务器文档，以了解可能影响类加载器行为的设置。** 

`classpath*:`前缀也可以在位置路径的其余部分中与`PathMatcher`模式相结合，例如`classpath*:META-INF/*-beans.xml`。在这种情况下，解决策略相当简单： 先调用 `ClassLoader.getResources()` 在class loader层次匹配，然后在每个资源上使用上面描述的相同的路径匹配策略用于通配符子路径。 

#####  Other notes relating to wildcards

注意`classpath*:` when combined with Ant-style patterns will only work reliably with at least one root directory before the pattern starts, unless the actual target files reside in the file system. 像 `classpath*:*.xml` 可能不会从jar文件的根部检索文件，而只从扩展目录的根目录中检索文件。 

Spring检索类路径条目的能力源自JDK的`classloader.getresources()`方法，该方法只返回传入空字符串的文件系统位置（指示潜在的根搜索）。 Spring评估 `URLClassLoader`运行时配置，以及在jar文件里的 "java.class.path" manifest也不能保证会导致便携性行为。 

**对类路径包的扫描需要在类路径中存在相应的目录条目。 当你 build JARs with Ant, 确保您没有激活JAR任务的files-only的开关。另外，类路径目录可能不会根据某些环境中的安全策略而被公开，例如JDK 1.5.7.0和更高版本的独立应用程序。 （在你的清单中需要“托管库”的设置;参见http://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources ）**

在JDK 9的模块路径（Jigsaw）上， Spring的类路径扫描通常按预期工作。 把资源放到专用的目录中也是非常值得推荐的， 避免前面提到的搜索jar文件根级别引起的可移植性问题。 

ant风格的类路径：如果要搜索的根包在多个类路径位置可用，资源不能保证找到匹配的资源。 这是因为资源比如：

```
com/mycompany/package1/service-context.xml
```

可能只在一个位置，但是当一个路径，例如 

```
classpath:com/mycompany/**/service-context.xml
```

被使用来解析，解析器会得到`getResource("com/mycompany")`返回的（第一个）URL。如果这个基本包节点存在于多个类加载器位置， 实际的最终资源可能不在下面。 因此，最好使用" `classpath*:`"  ，在这种情况下使用相同的ant样式模式， 它会搜索包含根包的所有类路径位置。 

####  2.7.3. FileSystemResource caveats

一个没连接到`FileSystemApplicationContext`的`FileSystemResource`（也就是说，一个`FileSystemApplicationContext` 不是确切的 `ResourceLoader`）会像你期望的那样对待绝对和相对的路径。 相对路径是相对于当前工作目录的，而绝对路径是相对于文件系统的根的。 

然而，为了向后兼容（历史）原因， 这种情况随着 `FileSystemApplicationContext` 是`ResourceLoader`而发生变化。 `FileSystemApplicationContext`  简单地强制所有连接的`FileSystemResource` 实例将所有路径视为相对的，不管它们是不是以“/”开头。在实践中，这意味着以下内容是等价的： 

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/context.xml");
```

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("/conf/context.xml");
```

下面是这样的：（尽管这对他们来说不相同是有意义的，因为一个是相对的，另一个是绝对的。） 

```
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

```
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```

在实践中，如果需要真正的绝对文件系统路径， 最好放弃`FileSystemResource`/ `FileSystemXmlApplicationContext`的绝对路径的使用，简单地通过使用 `file:`URL前缀，强制使用`UrlResource`。 

```
// actual context type doesn't matter, the Resource will always be UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("file:///conf/context.xml");
```

##  3. Validation, Data Binding, and Type Conversion

###  3.1. Introduction

```
			JSR-303/JSR-349 Bean Validation
Spring 4.0在程序上支持 Bean Validation 1.0 (JSR-303) and Bean Validation 1.1 (JSR-349)，并适配Validator接口。

应用程序可以选择在全局范围内启用Bean验证，如 Spring Validation所述，并将其专门用于所有验证需求。

一个应用程序还可以为每个DataBinder实例注册额外的Spring Validator实例，如Configuring a DataBinder所述。这可能有助于在不使用注释的情况下插入验证逻辑。
```

考虑将验证作为业务逻辑的优缺点，Spring提供了验证（和数据绑定）的设计，这并不排除其中任何一个。具体的验证不应该与web层绑定， 应该易于本地化，并且应该可以插入任何可用的验证器。 考虑到上述情况，Spring已经提出了一个验证器接口，它在应用程序的每一层都是基本的，并且非常有用。 

数据绑定对于允许用户输入动态绑定到应用程序的领域模型（或任何用于处理用户输入的对象）非常有用。 Spring提供所谓的 `DataBinder` 来做这个。`Validator` and the`DataBinder`  组成了`validation` 包，它主要用于但不限于MVC框架。 

 `BeanWrapper`是Spring框架中的一个基本概念且在很多地方使用。但是，您可能不需要直接使用`BeanWrapper`。 因为这是参考文档，我们觉得有些解释可能是有序的。 我们将在这一章中解释这个`BeanWrapper`，  如果您打算使用它，那么当试图将数据绑定到对象时，您很可能会这样做。 

Spring的 DataBinder and the lower-level BeanWrapper都用了PropertyEditors来转化和形成属性值。`PropertyEditor`概念是JavaBeans规范的一部分，这一章也解释了。Spring 3引入了"core.convert"包提供通用类型转换工具，以及用于格式化UI字段值的高级“格式”包。 这些包可能被用作 PropertyEditors的更简单的替代品，这一章也将讨论。 

### 3.2. Validation using Spring’s Validator interface

Spring features 一个 `Validator`接口来验证对象。`Validator` 使用一个 `Errors` 对象工作，当验证时，验证器可以向`Errors`对象报告验证失败。 

让我们考虑一个小数据对象： 

```
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

我们要给Person类提供验证行为，通过实现`org.springframework.validation.Validator`的以下两个方法：

- `supports(Class)` - 这个 `Validator` 能验证 supplied `Class`的实例吗?
- `validate(Object, org.springframework.validation.Errors)` - 验证给定对象，只要有验证错误，注册到Errors对象。

实现`Validator` 很简单，特别是当您知道Spring框架还提供的`ValidationUtils`helper类时。 

```
public class PersonValidator implements Validator {

    /**
     * This Validator validates *just* Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```

如你所见，`ValidationUtils`  的`static` `rejectIfEmpty(..)`  方法用于注入'name' 属性如果是null或空字符串看看ValidationUtils javadocs，看看它提供了什么功能，除了前面所展示的例子。 

当然，可以实现单个确认器类来验证rich对象中的每个嵌套对象，在它自己的验证器实现中封装每个嵌套的对象类的验证逻辑可能会更好。 一个“富”对象的简单示例是由两个字符串属性（第一个和第二个名称）和一个复杂的Address对象组成的客户。 `Address` 对象可能独立于`Customer` 对象，因此，一个独特的`AddressValidator`被实现。 如果您希望`CustomerValidator`重用`AddressValidator`类中包含的逻辑，而不需要使用复制粘贴， 您可以在`CustomerValidator`中依赖注入或实例化一个`AddressValidator`，并像这样使用它： 

```
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```

验证错误被报告给传递给validator的`Errors`  对象，在Spring Web MVC的情况下，您可以使用`<spring:bind/>`  标签来检查错误消息。当然，您也可以自己检查错误对象。 关于它提供的方法的更多信息可以在javadocs中找到。 

###  3.3. Resolving codes to error messages

我们已经讨论了数据绑定和验证。 输出与验证错误对应的消息是我们需要讨论的最后一件事。 在上面的例子中，我们拒绝了name和age field。 如果我们要使用MessageSource来输出错误消息，我们将使用我们在拒绝字段时给出的错误代码来实现 。当您调用（直接或间接地，使用ValidationUtils类）`rejectValue`或来自`Errors`接口的其他`reject`方法时， 底层实现不仅会注册您所传递的代码，而且但也有一些额外的error代码。 它注册的什么error代码是由所使用的`MessageCodesResolver`决定的。 默认情况下，使用`DefaultMessageCodesResolver`，例如，它不仅用您给出的代码注册一条消息，还包括传递给reject方法的字段名的消息。 所以如果你用`rejectValue("age", "too.darn.old")`来reject一个field，除了 `too.darn.old` 代码外，Spring也注册 `too.darn.old.age`和`too.darn.old.age.int`（所以第一个包含字段名第二个将包括字段类型 ）；这样做是为了方便开发人员锁定错误消息和诸如此类的信息。 

关于`MessageCodesResolver` 和默认策略see [`MessageCodesResolver`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/validation/MessageCodesResolver.html) and [`DefaultMessageCodesResolver`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html)。

###  3.4. Bean manipulation and the BeanWrapper

`org.springframework.beans` 包用户Oracle的JavaBeans规范。 refer to Oracle’s website ( [javabeans](https://docs.oracle.com/javase/6/docs/api/java/beans/package-summary.html)). 

beans包一个重要的类是`BeanWrapper` interface和它对应的实现( `BeanWrapperImpl`) 。正如javadocs所引用的，  `BeanWrapper`  提供了设置和获取属性值（单独或批量）的功能，获得属性描述符，并查询属性以确定它们是否可读或可写。 此外，BeanWrapper还提供了对嵌套属性的支持，使得子属性上的属性设置为无限深度的。 然后，`BeanWrapper`支持标准JavaBeans `propertychangelistener`和`vetoablechangelistener`的功能，不需要在目标类中代码支持。最后但并非最不重要的是，BeanWrapper提供了对索引属性设置的支持。 `BeanWrapper`通常不会直接被应用程序代码使用，而是由`DataBinder`和`BeanFactory`使用。 

bean包装器的工作方式部分是由它的名称来表示的：它包装一个bean来执行bean上的操作，比如设置和检索属性。 

#### 3.4.1. Setting and getting basic and nested properties

使用`setPropertyValue(s)` and `getPropertyValue(s)` 方法get和set属性，两个方法都有一些重载的变体。它们都在Spring的javadocs中得到了更详细的描述。 重要的是要知道，有一些约定表示对象的属性。几个例子:

*Table 11. Examples of properties*

| Expression             | Explanation                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `name`                 | Indicates the property `name` corresponding to the methods `getName()` or `isName()` and `setName(..)` |
| `account.name`         | Indicates the nested property `name` of the property `account`corresponding e.g. to the methods `getAccount().setName()`or `getAccount().getName()` |
| `account[2]`           | Indicates the *third* element of the indexed property `account`. Indexed properties can be of type `array`, `list` or other *naturally ordered* collection |
| `account[COMPANYNAME]` | Indicates the value of the map entry indexed by the key *COMPANYNAME* of the Map property `account` |

下面您将找到一些使用`BeanWrapper`来获取和设置属性的示例。 

（如果您不打算直接使用`BeanWrapper`，那么下一节对您来说并不重要。 如果您只是使用`DataBinder`和`BeanFactory`及其开箱即用的实现，那么您应该跳过有关`propertyeditor`的部分 ）

Consider the following two classes: 

```
public class Company {

    private String name;
    private Employee managingDirector;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Employee getManagingDirector() {
        return this.managingDirector;
    }

    public void setManagingDirector(Employee managingDirector) {
        this.managingDirector = managingDirector;
    }
}
```

```
public class Employee {

    private String name;

    private float salary;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getSalary() {
        return salary;
    }

    public void setSalary(float salary) {
        this.salary = salary;
    }
}
```

下面的代码片段展示了如何检索和操作实例化`Companies` and `Employees`的一些属性的示例： 

```
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

#### 3.4.2. Built-in PropertyEditor implementations

Spring使用propertyeditor的概念来影响对象和字符串之间的转换。如果你仔细想想，有时候用一种不同的方式来表示属性可能会很方便，而不是对象本身。例如，一个日期可以以人类可读的方式表示（作为字符串'2007-14-09'），而我们仍然能够将人类可读的形式转换回原来的日期（或者更好：将以人类可读的形式输入的日期转换为日期对象）。这种行为可以通过注册*custom editors*来实现，即 `java.beans.PropertyEditor`类型。如前一章所提到的，在一个bean包装器上注册自定义编辑器，或者在一个特定的IoC容器中交替注册，这使它知道如何将属性转换为所需的类型。在java的javadocs中阅读更多关于propertyeditor的信息。由Oracle提供的`java.beans`包。 

在Spring中使用属性编辑的几个例子： 

- *setting properties on beans* is done using `PropertyEditors`。当`java.lang.String` 作为XML bean属性，Spring will（如果对应属性的setter有一个类参数）使用`ClassEditor`尝试将参数解析为类对象。 
- *parsing HTTP request parameters* in Spring’s MVC framework。使用各种各样的`PropertyEditors` ，你可以手动绑定到`CommandController`的所有子类。

Spring有许多内置的属性编辑器，可以让生活变得简单。以下列表都在`org.springframework.beans.propertyeditors`包下。大多数，但不是全部（如下所示）， 在默认情况下是由`BeanWrapperImpl`注册的。 如果属性编辑器在某种程度上是可配置的，那么当然您仍然可以注册您自己的变体来覆盖默认值： 

*Table 12. Built-in PropertyEditors*

| Class                     | Explanation                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `ByteArrayPropertyEditor` | Editor for byte arrays. Strings will simply be converted to their corresponding byte representations. Registered by default by `BeanWrapperImpl`. |
| `ClassEditor`             | Parses Strings representing classes to actual classes and the other way around. When a class is not found, an `IllegalArgumentException` is thrown. Registered by default by `BeanWrapperImpl`. |
| `CustomBooleanEditor`     | Customizable property editor for `Boolean` properties. Registered by default by `BeanWrapperImpl`, but, can be overridden by registering custom instance of it as custom editor. |
| `CustomCollectionEditor`  | Property editor for Collections, converting any source `Collection` to a given target `Collection` type. |
| `CustomDateEditor`        | Customizable property editor for java.util.Date, supporting a custom DateFormat. NOT registered by default. Must be user registered as needed with appropriate format. |
| `CustomNumberEditor`      | Customizable property editor for any Number subclass like `Integer`, `Long`, `Float`, `Double`. Registered by default by `BeanWrapperImpl`, but can be overridden by registering custom instance of it as a custom editor. |
| `FileEditor`              | Capable of resolving Strings to `java.io.File` objects. Registered by default by `BeanWrapperImpl`. |
| `InputStreamEditor`       | One-way property editor, capable of taking a text string and producing (via an intermediate `ResourceEditor` and `Resource`) an `InputStream`, so `InputStream` properties may be directly set as Strings. Note that the default usage will not close the `InputStream` for you! Registered by default by `BeanWrapperImpl`. |
| `LocaleEditor`            | Capable of resolving Strings to `Locale` objects and vice versa (the String format is *[country]*[variant], which is the same thing the toString() method of Locale provides). Registered by default by `BeanWrapperImpl`. |
| `PatternEditor`           | Capable of resolving Strings to `java.util.regex.Pattern`objects and vice versa. |
| `PropertiesEditor`        | Capable of converting Strings (formatted using the format as defined in the javadocs of the `java.util.Properties` class) to `Properties` objects. Registered by default by `BeanWrapperImpl`. |
| `StringTrimmerEditor`     | Property editor that trims Strings. Optionally allows transforming an empty string into a `null` value. NOT registered by default; must be user registered as needed. |
| `URLEditor`               | Capable of resolving a String representation of a URL to an actual `URL` object. Registered by default by `BeanWrapperImpl`. |

 Spring使用`java.beans.PropertyEditorManager`为可能需要的属性编辑器设置搜索路径。搜索路径还包括  `sun.bean.editors`，其中包括字体、颜色和大多数原始类型等类型的PropertyEditor实现。 还要注意的是，标准JavaBeans基础设施将自动发现`PropertyEditor`类（无需显式地注册它们），如果与它们处理的类相同，并且具有与该类相同的名称，并附加了'Editor' ;例如，可以有以下类和包结构，这足以让`FooEditor`类被识别并用作`Foo`-typed属性的`PropertyEditor`。 

```
com
  chank
    pop
      Foo
      FooEditor // the PropertyEditor for the Foo class
```

注意，您也可以在这里使用标准的JavaBeans机制的`BeanInfo `（described [in not-amazing-detail here](https://docs.oracle.com/javase/tutorial/javabeans/advanced/customization.html) ），下面是一个使用`BeanInfo`机制的例子，它使用关联类的属性显式地注册一个或多个`PropertyEditor`实例。 

```
com
  chank
    pop
      Foo
      FooBeanInfo // the BeanInfo for the Foo class
```

下面是所引用的`FooBeanInfo`类的Java源代码。 这将把`CustomNumberEditor`与`Foo`类的`age`属性关联起来。 

```
public class FooBeanInfo extends SimpleBeanInfo {

    public PropertyDescriptor[] getPropertyDescriptors() {
        try {
            final PropertyEditor numberPE = new CustomNumberEditor(Integer.class, true);
            PropertyDescriptor ageDescriptor = new PropertyDescriptor("age", Foo.class) {
                public PropertyEditor createPropertyEditor(Object bean) {
                    return numberPE;
                };
            };
            return new PropertyDescriptor[] { ageDescriptor };
        }
        catch (IntrospectionException ex) {
            throw new Error(ex.toString());
        }
    }
}
```

##### Registering additional custom PropertyEditors

When setting bean properties as a string value, a Spring IoC container ultimately uses standard JavaBeans `PropertyEditors` to convert these Strings to the complex type of the property. Spring pre-registers a number of custom `PropertyEditors` (for example, to convert a classname expressed as a string into a real `Class` object). Additionally, Java’s standard JavaBeans `PropertyEditor` lookup mechanism allows a `PropertyEditor` for a class simply to be named appropriately and placed in the same package as the class it provides support for, to be found automatically. 

When setting bean properties as a string value, a Spring IoC container ultimately uses standard JavaBeans `PropertyEditors` to convert these Strings to the complex type of the property. Spring pre-registers a number of custom `PropertyEditors` (for example, to convert a classname expressed as a string into a real `Class` object). Additionally, Java’s standard JavaBeans `PropertyEditor` lookup mechanism allows a `PropertyEditor` for a class simply to be named appropriately and placed in the same package as the class it provides support for, to be found automatically. 

Note that all bean factories and application contexts automatically use a number of built-in property editors, through their use of something called a `BeanWrapper` to handle property conversions. The standard property editors that the `BeanWrapper`registers are listed in [the previous section](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-beans-conversion). Additionally, `ApplicationContexts` also override or add an additional number of editors to handle resource lookups in a manner appropriate to the specific application context type. 

Standard JavaBeans `PropertyEditor` instances are used to convert property values expressed as strings to the actual complex type of the property. `CustomEditorConfigurer`, a bean factory post-processor, may be used to conveniently add support for additional `PropertyEditor` instances to an `ApplicationContext`. 

Consider a user class `ExoticType`, and another class `DependsOnExoticType` which needs `ExoticType` set as a property: 

```
package example;

public class ExoticType {

    private String name;

    public ExoticType(String name) {
        this.name = name;
    }
}

public class DependsOnExoticType {

    private ExoticType type;

    public void setType(ExoticType type) {
        this.type = type;
    }
}
```

When things are properly set up, we want to be able to assign the type property as a string, which a `PropertyEditor` will behind the scenes convert into an actual `ExoticType` instance: 

```
<bean id="sample" class="example.DependsOnExoticType">
    <property name="type" value="aNameForExoticType"/>
</bean>
```

The `PropertyEditor` implementation could look similar to this: 

```
// converts string representation to ExoticType object
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

    public void setAsText(String text) {
        setValue(new ExoticType(text.toUpperCase()));
    }
}
```

Finally, we use `CustomEditorConfigurer` to register the new `PropertyEditor` with the `ApplicationContext`, which will then be able to use it as needed: 

```
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="customEditors">
        <map>
            <entry key="example.ExoticType" value="example.ExoticTypeEditor"/>
        </map>
    </property>
</bean>
```

 ##### Using PropertyEditorRegistrars

Another mechanism for registering property editors with the Spring container is to create and use a `PropertyEditorRegistrar`. This interface is particularly useful when you need to use the same set of property editors in several different situations: write a corresponding registrar and reuse that in each case. `PropertyEditorRegistrars` work in conjunction with an interface called `PropertyEditorRegistry`, an interface that is implemented by the Spring `BeanWrapper` (and `DataBinder`). `PropertyEditorRegistrars` are particularly convenient when used in conjunction with the `CustomEditorConfigurer` (introduced [here](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-beans-conversion-customeditor-registration)), which exposes a property called `setPropertyEditorRegistrars(..)`: `PropertyEditorRegistrars` added to a`CustomEditorConfigurer` in this fashion can easily be shared with `DataBinder` and Spring MVC `Controllers`. Furthermore, it avoids the need for synchronization on custom editors: a `PropertyEditorRegistrar` is expected to create fresh `PropertyEditor`instances for each bean creation attempt.

Using a `PropertyEditorRegistrar` is perhaps best illustrated with an example. First off, you need to create your own `PropertyEditorRegistrar` implementation:

```
package com.foo.editors.spring;

public final class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {

    public void registerCustomEditors(PropertyEditorRegistry registry) {

        // it is expected that new PropertyEditor instances are created
        registry.registerCustomEditor(ExoticType.class, new ExoticTypeEditor());

        // you could register as many custom property editors as are required here...
    }
}
```

See also the `org.springframework.beans.support.ResourceEditorRegistrar` for an example `PropertyEditorRegistrar`implementation. Notice how in its implementation of the `registerCustomEditors(..)` method it creates new instances of each property editor. 

Next we configure a `CustomEditorConfigurer` and inject an instance of our `CustomPropertyEditorRegistrar` into it: 

```
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="propertyEditorRegistrars">
        <list>
            <ref bean="customPropertyEditorRegistrar"/>
        </list>
    </property>
</bean>

<bean id="customPropertyEditorRegistrar"
    class="com.foo.editors.spring.CustomPropertyEditorRegistrar"/>
```

Finally, and in a bit of a departure from the focus of this chapter, for those of you using [Spring’s MVC web framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc), using `PropertyEditorRegistrars` in conjunction with data-binding `Controllers` (such as `SimpleFormController`) can be very convenient. Find below an example of using a `PropertyEditorRegistrar` in the implementation of an `initBinder(..)` method: 

```
public final class RegisterUserController extends SimpleFormController {

    private final PropertyEditorRegistrar customPropertyEditorRegistrar;

    public RegisterUserController(PropertyEditorRegistrar propertyEditorRegistrar) {
        this.customPropertyEditorRegistrar = propertyEditorRegistrar;
    }

    protected void initBinder(HttpServletRequest request,
            ServletRequestDataBinder binder) throws Exception {
        this.customPropertyEditorRegistrar.registerCustomEditors(binder);
    }

    // other methods to do with registering a User
}
```

This style of `PropertyEditor` registration can lead to concise code (the implementation of `initBinder(..)` is just one line long!), and allows common `PropertyEditor` registration code to be encapsulated in a class and then shared amongst as many`Controllers` as needed. 

### 3.5. Spring Type Conversion

Spring 3引入`core.convert` 包提供一般类型转换系统。他的系统定义了一个SPI来实现类型转换逻辑，以及在运行时执行类型转换的API。 在Spring容器中，这个系统可以作为propertyeditor的替代，将外部化的bean属性值字符串转换为必需的属性类型。 公共API也可以在您的应用程序的任何地方使用，在那里需要类型转换。 

#### 3.5.1. Converter SPI

实现类型转换逻辑的SPI是简单的强类型：

```
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);

}
```

要创建自己的转换器， 简单地实现上面的接口。 转换 a collection or array也是可以的，前提是 a delegating array/collection converter也被注册了（）`DefaultConversionService` 默认做这个事情。

对于每次调用`convert(S)`，源参数保证不是NULL。 如果转换失败，您的转换器可能会抛出任何未经检查的异常; 具体地说，应该抛出一个非法的参数异常来报告一个无效的源值。 注意确保您的转换器实现是线程安全的。 

作为便捷在`core.convert.support`提供了几个converter 实现。这些包括从字符串到数字和其他常见类型的转换器。 将`StringToInteger`看作是典型转换器实现的一个示例： 

```
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }

}
```

#### 3.5.2. ConverterFactory

当您需要将整个类层次结构的转换逻辑集中起来时，例如，当从字符串转换到java.lang.Enum对象，实现`ConverterFactory`: 

```
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

    <T extends R> Converter<S, T> getConverter(Class<T> targetType);

}
```

S是你要转化的类型，R是被转化成类的基类。然后实现`getConverter(Class<T>) `，T是R的子类。

考虑`StringToEnum` ConverterFactory 例子：

```
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

    public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }

    private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }
}
```

#### 3.5.3. GenericConverter

当您需要一个复杂的转换器实现时， 考虑GenericConverter接口。有一个更灵活但不那么强类型的签名，generic转换器支持在多个源和目标类型之间进行转换。 此外，一个generic转换器提供了在实现转换逻辑时可以使用的源和目标字段上下文。 这样的上下文允许由字段标注驱动类型转换，或者在字段签名上声明的通用信息。 

```
package org.springframework.core.convert.converter;

public interface GenericConverter {

    public Set<ConvertiblePair> getConvertibleTypes();

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

要实现 GenericConverter，让getConvertibleTypes()返回支持的source-target type pairs。然后实现 `convert(Object, TypeDescriptor, TypeDescriptor)  `来实现转换逻辑。源TypeDescriptor提供了对源字段的访问，该字段持有转换的值。 目标TypeDescriptor提供对目标字段的访问，其中转换后的值将被设置。 

一个GenericConverter很好的例子是 between a Java Array and a Collection的转换。这样的ArrayToCollectionConverter会对声明目标集合类型的字段进行内省，以解决集合的元素类型。这使得源数组中的每个元素都可以被转换成托收元素类型，然后在目标字段上设置集合。 

因为GenericConverter是一个更复杂的SPI接口，只有在需要时才使用它。赞同Converter or ConverterFactory为了基本类型转换需求。  

##### ConditionalGenericConverter

有时候你想特定条件是true时执行converter。例如，如果目标字段上有特定的注释，您才想执行一个转换器。 或者，如果一个特定的方法，比如静态`valueOf`方法，在目标类上定义，您才可能想执行一个转换器。 `ConditionalGenericConverter` 结合了`GenericConverter` and `ConditionalConverter`  接口允许你定义自定义匹配规则：

```
public interface ConditionalConverter {

    boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);

}

public interface ConditionalGenericConverter
    extends GenericConverter, ConditionalConverter {

}
```

一`ConditionalGenericConverter`的很好的例子是an persistent entity identifier 和 an entity reference 之间的EntityConverter。这样的EntityConverter只在目标实体类型声明一个静态查找器方法，例如findAccount（Long）才匹配，You would perform such a finder method check in the implementation of `matches(TypeDescriptor, TypeDescriptor)`. 

#### 3.5.4. ConversionService API

ConversionService定义了一个统一的API，用于在运行时执行类型转换逻辑。 转换器通常在这个facade接口后面执行： 

```
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

大多数ConversionService实现也实现了`ConverterRegistry`， 为注册转换器提供一个SPI。 在内部，一个ConversionService实现委托它的注册转换器来执行类型转换逻辑。 

一个健壮的 ConversionService实现在`core.convert.support`包提供。 `GenericConversionService` 是适用于大多数环境的通用实现。 ConversionServiceFactory提供了一个方便的工厂来创建常见的ConversionService配置。 

####  3.5.5. Configuring a ConversionService

ConversionService是一个无状态的对象，它被设计为在应用程序启动时实例化， 然后在多个线程之间共享。 在Spring应用程序中，您通常为每个Spring容器（或ApplicationContext）配置ConversionService实例。 ConversionService将在Spring中被选中，然后在需要由框架执行类型转换时使用。 您也可以将此ConversionService注入任何bean并直接调用它。 

**在Spring中没有注册ConversionService，使用原始的PropertyEditor-based system。** 

要用Spring注册一个默认的ConversionService，请在id ConversionService中添加以下bean定义： 

```
<bean id="conversionService"
    class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```

*：*默认的ConversionService可以在字符串、数字、枚举、集合、maps和其他常见类型之间进行转换。 用您自己的自定义转换器（s）来补充或覆盖缺省转换器，设置`converters` 属性。属性值可以实现Converter, ConverterFactory, or GenericConverter接口。  

```
<bean id="conversionService"
        class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="example.MyCustomConverter"/>
        </set>
    </property>
</bean>
```

在Spring MVC应用程序中使用ConversionService也很常见。 . See [Conversion and Formatting](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-conversion) in the Spring MVC chapter. 

In certain situations you may wish to apply formatting during conversion. See [FormatterRegistry SPI](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#format-FormatterRegistry-SPI) for details on using`FormattingConversionServiceFactoryBean`. 

#### 3.5.6. Using a ConversionService programmatically

要以编程的方式使用ConversionService实例， 简单地给它注入一个引用，就像你对其他bean一样： 

```
@Service
public class MyService {

    @Autowired
    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    public void doIt() {
        this.conversionService.convert(...)
    }
}
```

对于大多数用例来说，指定targetType的convert方法可以被使用，但是它不会使用更复杂的类型，比如参数化元素的集合。 例如，如果您想要以编程的方式将一个整数列表转换成字符串列表，那么您需要提供源和目标类型的正式定义。 

幸运的是，TypeDescriptor提供了各种选项，使其变得简单： 

```
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ....
cs.convert(input,
    TypeDescriptor.forObject(input), // List<Integer> type descriptor
    TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

请注意，DefaultConversionService会自动注册转换器，这适用于大多数环境。 这包括集合转换器、标量转换器，以及字符串转换器的基本对象。 在DefaultConversionService类上使用静态adddefault转换器方法，可以在任何ConverterRegistry中注册相同的转换器。 

值类型的转换器将被用于数组和集合， 假设标准的收集处理是合适的，不需要创建一个特定的转换器从S集合转换成T的集合。

###  3.6. Spring Field Formatting

如前所讨论， [`core.convert`](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#core-convert) 是一个通用类型转换系统。它提供了一个统一的ConversionService API，以及一个强类型的转换器SPI，用于实现从一种类型到另一种类型的转换逻辑。 Spring容器使用这个系统来绑定bean属性值。 此外，Spring表达式语言（SpEL）和DataBinder都使用这个系统来绑定字段值。 例如，当 SpEL 需要把一个Short强转为Long来完成`expression.setValue(Object bean, Object value)` ，core.convert system 执行强转。

现在考虑一个典型的客户端环境的类型转换需求，例如web或桌面应用程序。 在这样的环境中，您通常会从字符串转换到支持客户端回发过程，以及返回到字符串来支持视图呈现过程。另外，您通常需要本地化字符串值。 通常的 *core.convert* 转换器SPI并没有直接解决这种格式要求。  为了直接解决这些问题，Spring 3引入了一个方便的Formatter SPI，它为客户端环境提供了一个简单而健壮的替代方案。 

一般来说，当您需要实现通用类型转换逻辑时，使用转换器SPI; 例如在Long和Data之间的转换。当您在客户端环境中工作时，例如web应用程序，并且需要解析和打印本地化的字段值，使用Formatter SPI。 ConversionService为两种SPIs提供了一个统一的类型转换API。 

####  3.6.1. Formatter SPI

Formatter SPI实现字段格式化逻辑是简单而强类型的： 

```
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

Formatter继承Printer and Parser building-block接口：

```
public interface Printer<T> {
    String print(T fieldValue, Locale locale);
}
```

```
import java.text.ParseException;

public interface Parser<T> {
    T parse(String clientValue, Locale locale) throws ParseException;
}
```

要创建您自己的格式化程序，只需实现上面的Formatter接口。 T是你想转换的类型，例如，`java.util.Date`。实现`print()` 操作来print一个T的实例展示在客户端环境。实现`parse()` 操作把T的实例转换，从客户端返回的格式化形式。如果格式化失败抛出a ParseException or IllegalArgumentException。请注意确保您的格式化程序实现是线程安全的。 

为了方便起见，在格式子包中提供了几个`format`子包实现。number包提供`NumberFormatter`, `CurrencyFormatter`, and `PercentFormatter`，使用`java.text.NumberFormat`， 格式化 `java.lang.Number` 对象。`datetime` 提供`DateFormatter` 格式化带有`java.text.DateFormat` 的`java.util.Date`对象。`datetime.joda`  包提供全面日期格式化支持 based on the [Joda-Time library](http://joda-time.sourceforge.net/)。

把`DateFormatter`看作`DateFormatter`的实现：

```
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

    private String pattern;

    public DateFormatter(String pattern) {
        this.pattern = pattern;
    }

    public String print(Date date, Locale locale) {
        if (date == null) {
            return "";
        }
        return getDateFormat(locale).format(date);
    }

    public Date parse(String formatted, Locale locale) throws ParseException {
        if (formatted.length() == 0) {
            return null;
        }
        return getDateFormat(locale).parse(formatted);
    }

    protected DateFormat getDateFormat(Locale locale) {
        DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
        dateFormat.setLenient(false);
        return dateFormat;
    }

}
```

####  3.6.2. Annotation-driven Formatting

正如您将看到的，字段格式化可以通过field type或注解进行配置。要将注解绑定到formatter，请实现注解格式工厂： 

```
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);

}
```

参数A为和转换逻辑关联的域注解类型，例如`org.springframework.format.annotation.DateTimeFormat`。让`getFieldTypes()` 返回注解用在其上的域类型。`getPrinter()` 返回Printer来打印注解域的值。`getParser()` 返回一个Parser来转换一个来自注解域的clientValue。

下例AnnotationFormatterFactory实现把 @NumberFormat 绑定到了一个formatter。

```
public final class NumberFormatAnnotationFormatterFactory
        implements AnnotationFormatterFactory<NumberFormat> {

    public Set<Class<?>> getFieldTypes() {
        return new HashSet<Class<?>>(asList(new Class<?>[] {
            Short.class, Integer.class, Long.class, Float.class,
            Double.class, BigDecimal.class, BigInteger.class }));
    }

    public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    private Formatter<Number> configureFormatterFrom(NumberFormat annotation,
            Class<?> fieldType) {
        if (!annotation.pattern().isEmpty()) {
            return new NumberFormatter(annotation.pattern());
        } else {
            Style style = annotation.style();
            if (style == Style.PERCENT) {
                return new PercentFormatter();
            } else if (style == Style.CURRENCY) {
                return new CurrencyFormatter();
            } else {
                return new NumberFormatter();
            }
        }
    }
}
```

To trigger formatting, simply annotate fields with @NumberFormat: 

```
public class MyModel {

    @NumberFormat(style=Style.CURRENCY)
    private BigDecimal decimal;

}
```

##### Format Annotation API

轻便的format注解api在`org.springframework.format.annotation` 包，使用 @NumberFormat 转换 java.lang.Number fields 。使用@DateTimeFormat 转换 java.util.Date, java.util.Calendar, java.util.Long, or Joda-Time fields。

```
public class MyModel {

    @DateTimeFormat(iso=ISO.DATE)
    private Date date;

}
```

#### 3.6.3. FormatterRegistry SPI

Review the FormatterRegistry SPI below:

```
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

    void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

    void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

    void addFormatterForFieldType(Formatter<?> formatter);

    void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);

}
```

As shown above, Formatters can be registered by fieldType or annotation.

The FormatterRegistry SPI allows you to configure Formatting rules centrally, instead of duplicating such configuration across your Controllers. For example, you might want to enforce that all Date fields are formatted a certain way, or fields with a specific annotation are formatted in a certain way. With a shared FormatterRegistry, you define these rules once and they are applied whenever formatting is needed.

####  3.6.4. FormatterRegistrar SPI

The FormatterRegistrar is an SPI for registering formatters and converters through the FormatterRegistry: 

```
package org.springframework.format;

public interface FormatterRegistrar {

    void registerFormatters(FormatterRegistry registry);

}
```

A FormatterRegistrar is useful when registering multiple related converters and formatters for a given formatting category, such as Date formatting. It can also be useful where declarative registration is insufficient. For example when a formatter needs to be indexed under a specific field type different from its own <T> or when registering a Printer/Parser pair. The next section provides more information on converter and formatter registration. 

#### 3.6.5. Configuring Formatting in Spring MVC

See [Conversion and Formatting](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-conversion) in the Spring MVC chapter.

### 3.7. Configuring a global date & time format

默认没有标注`@DateTimeFormat`的是用`DateFormat.SHORT`风格转换的。如果您愿意，您可以通过定义自己的全局格式来改变这一点。 

您将需要确保Spring不会注册默认格式化器，相反，您应该手动注册所有格式化程序。使用`org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar` or`org.springframework.format.datetime.DateFormatterRegistrar`  取决于你是否使用了joda时间库。 

例如，下面的Java配置将注册一个全局的“yyyyMMdd”格式。这个例子不依赖于joda-Time库： 

```
@Configuration
public class AppConfig {

    @Bean
    public FormattingConversionService conversionService() {

        // Use the DefaultFormattingConversionService but do not register defaults
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        // Ensure @NumberFormat is still supported
        conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

        // Register date conversion with a specific global format
        DateFormatterRegistrar registrar = new DateFormatterRegistrar();
        registrar.setFormatter(new DateFormatter("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

如果您喜欢基于XML的配置，您可以使用FormattingConversionServiceFactoryBean。这里是同样的例子，这次使用Joda-Time： 

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd>

    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="registerDefaultFormatters" value="false" />
        <property name="formatters">
            <set>
                <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
                    <property name="dateFormatter">
                        <bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
                            <property name="pattern" value="yyyyMMdd"/>
                        </bean>
                    </property>
                </bean>
            </set>
        </property>
    </bean>
</beans>
```

**joda-Time提供了单独的不同类型来表示 `date`, `time` and `date-time` 值。`JodaTimeFormatterRegistrar`  的 `dateFormatter`, `timeFormatter` and `dateTimeFormatter`  属性应该应该用于为每种类型配置不同的格式。 `DateTimeFormatterFactoryBean` 为创建formatter提供了便利的方式。**

如果您正在使用Spring MVC，请记住要显式地配置所使用的转换服务。 对于Java-Based的`@Configuration` 意味着扩展 `WebMvcConfigurationSupport` 类并覆盖 `WebMvcConfigurationSupport` 方法。For XML you should use the `'conversion-service'` attribute of the `mvc:annotation-driven` element. See [Conversion and Formatting](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-conversion) for details. 

###  3.8. Spring Validation

Spring 3引入了对其验证支持的一些增强。 首先，完全支持了JSR-303 Bean Validation API ，其次，当编程式使用时，Spring的DataBinder现在可以验证对象并绑定它们。 第三，Spring MVC现在支持声明式验证`@Controller`输入。 

####  3.8.1. Overview of the JSR-303 Bean Validation API

JSR-303标准化了Java平台的验证约束声明和元数据。使用这个API，您可以使用说明性的验证约束来注释领域模型属性，而运行时则强制执行它们。您可以利用一些内置的约束。您还可以定义自己的定制约束。 

为了说明这一点，请考虑一个具有两个属性的简单的PersonForm模型： 

```
public class PersonForm {
    private String name;
    private int age;
}
```

JSR-303允许您定义声明性验证约束这些属性： 

```
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;

}
```

For general information on JSR-303/JSR-349, see the [Bean Validation website](http://beanvalidation.org/). For information on the specific capabilities of the default reference implementation, see the [Hibernate Validator](https://www.hibernate.org/412.html) documentation. To learn how to setup a Bean Validation provider as a Spring bean, keep reading. 

#### 3.8.2. Configuring a Bean Validation Provider

Spring为Bean验证API提供了完全的支持。 这包括方便地支持将JSR-303/JSR-349 Bean验证提供者作为Spring Bean来引导。 这允许你需要的时候注入 `javax.validation.ValidatorFactory` or `javax.validation.Validator`  。

使用 `LocalValidatorFactoryBean` 作为默认校验器作为bean。

```
<bean id="validator"
    class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```

上面的基本配置将触发Bean验证来初始化使用它的默认引导机制。JSR-303/JSR-349提供者，如Hibernate验证器，预计将出现在类路径中，并将自动检测到。 

#####  Injecting a Validator

`LocalValidatorFactoryBean`实现了`javax.validation.ValidatorFactory` and `javax.validation.Validator`, as well as Spring’s `org.springframework.validation.Validator`。您可以将这些接口中的任何一个引用注入到需要调用验证逻辑的bean中。 

注入一个引用到`javax.validation.Validator` 如果你更喜欢直接用Bean Validation API：

```
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
```

注入引用到`org.springframework.validation.Validator` ，如果你的bean需要 Spring Validation API ：

```
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;

}
```

#####  Configuring Custom Constraints

每个Bean验证约束由两个部分组成。第一`@Constraint` 注解表明约束和可配置的属性。第二，`javax.validation.ConstraintValidator` 接口的实现，实现约束的行为。将声明与实现相关联，每个`@Constraint` 注解引用一个对应的ValidationConstraint 实现类。运行时，你的domain model遇到了注解，一个`ConstraintValidatorFactory` 实例化引用的实现。

默认 `LocalValidatorFactoryBean` 配置一个`SpringConstraintValidatorFactory` ，使用spring创建ConstraintValidator实例。这允许您的定制约束验证器从依赖注入中受益，就像任何其他Spring bean一样。 

下面是个自定义 `@Constraint` 声明的例子，紧接着是使用spring依赖注入的 `ConstraintValidator`的实现。

```
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

```
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

    @Autowired;
    private Foo aDependency;

    ...
}
```

如你看到的，ConstraintValidator可以有自己的依赖，像其他bean一样@Autowired 。

##### Spring-driven Method Validation

Bean Validation 1.1支持的方法验证，同样由Hibernate Validator 4.3的自定义扩展，可以通过 `MethodValidationPostProcessor` bean定义集成到 Spring context。

```
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```

为了符合Spring驱动的方法验证，所有的目标类都需要用Spring的`@Validated`进行注解， 可选地声明要使用的验证组 。. Check out the `MethodValidationPostProcessor` javadocs for setup details with Hibernate Validator and Bean Validation 1.1 providers. 

##### Additional Configuration Options

对于大多数情况，默认的LocalValidatorFactoryBean配置应该是足够的。 There are a number of configuration options for various Bean Validation constructs, from message interpolation to traversal resolution. See the`LocalValidatorFactoryBean` javadocs for more information on these options. 

#### 3.8.3. Configuring a DataBinder

从Spring 3开始，可以使用验证器来配置DataBinder实例。一旦配置，Validator会被调用`binder.validate()`. 任何验证Errors会被绑定到binder’s BindingResult 。

当编程式使用 DataBinder，可以用来在绑定后调用验证逻辑：

```
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

通过`dataBinder.addValidators` and `dataBinder.replaceValidators`配置多个`Validator`  实例，This is useful when combining globally configured Bean Validation with a Spring `Validator`configured locally on a DataBinder instance. See [[validation-mvc-configuring\]](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#validation-mvc-configuring). 

#### 3.8.4. Spring MVC 3 Validation

See [Validation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-validation) in the Spring MVC chapter. 

## 4. Spring Expression Language (SpEL)

## 5. Aspect Oriented Programming with Spring

### 5.1. Introduction

*Aspect-Oriented Programming* (AOP)  是 Object-Oriented Programming (OOP) 的补足，提供另一种思考程序结构的方法。OOP的模块化关键单元是类，在AOP是*aspect*。切面使得关注的模块比如事务管理跨越多个类型和对象。（在AOP文献中，这种关注通常被称为横切关注点*crosscutting* 。 ）

Spring的关键组件之一是AOP框架。 虽然Spring IoC容器并不依赖于AOP， 也就是说，如果你不想使用AOP，你就不需要使用，AOP补充了Spring IoC提供了一个非常有能力的中间件解决方案。 

```
								Spring 2.0+ AOP 
spring 2.0引入一种更简单、更强大的编写定制切面的方法，using either a schema-based approach or the @AspectJ annotation style.这两种方式都提供了完全类型的advice和使用AspectJ pointcut语言，当为weaving使用spring AOP的时候。

Spring 2.0+ schema- and @AspectJ-based AOP支持在本节讨论。 lower-level AOP支持，在Spring 1.2的应用程序中，将在接下来的章节中讨论。
```

AOP在Spring框架中被用于...

- 提供声明企业服务，特别是EJB declarative services的替代。最重要的service是 [*declarative transaction management*](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#transaction-declarative) 。
- 自定义切面，用AOP补足OOP。

如果您只对一般的声明性服务或其他预打包的声明性中间件服务感兴趣，比如 pooling，您不需要直接与Spring AOP打交道，并且可以跳过这一章的大部分内容。 

####  5.1.1. AOP concepts

让我们首先定义一些核心的AOP概念和术语。这些术语不是特定于spring的...不幸，AOP术语并不是特别直观; 然而，如果Spring使用了自己的术语，则会更加令人困惑。 

- *Aspect*: 一个关注的模块，切入许多类。在企业Java应用程序 Transaction management是横切关注点的一个很好的例子。在Spring AOP中，切面是使用常规类实现的（the [schema-based approach](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-schema)）或者带有 `@Aspect`注解的常规类(the [`@AspectJ` style](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-ataspectj) )。
- *Join point*: 在执行程序时的一个点，例如执行一个方法或处理一个异常。 在Spring AOP中，连接点 *always*表示 method execution。 
- *Advice*: 在指定join point上aspect采取的action。不同类型的advice包括 "around", "before" and "after" advice。(Advice types are discussed below.) 很多AOP框架，包括Spring，把advice建模为 *interceptor*，在 join point周围维护一系列的 *interceptor*。
- *Pointcut*: 与join points匹配的predicate。advice和pointcut表达式相关联，在任何与pointcut匹配的join point上运行（例如，一个带有特定名称的方法的执行）。pointcut表达式匹配join points的概念是AOP的核心，在默认情况下，Spring使用AspectJ pointcut表达式语言。 
- *Introduction*:  为类型声明额外的方法或field。Spring AOP允许向任何advised对象引入新的接口（和对应实现）。例如，你可以使用一个introduction来让bean实现`IsModified`接口，来简化缓存。  （在AspectJ社区中， introduction被称为inter-type declaration）
- *Target object*: 被一个或多个切面advised的对象。简称之 advised object。由于Spring AOP是使用运行时代理实现的， 这个对象总是一个被代理的对象。 
- *AOP proxy*:  由 AOP framework创建的对象，来实现aspect contracts（advise方法的执行等等）。在 Spring Framework，AOP proxy将是JDK dynamic proxy或CGLIB proxy。
- *Weaving*: 将aspects和其他应用类型或对象相关联，来创建advised object。这可以在编译时 （例如，使用AspectJ编译器 ），加载时，或者运行时完成。Spring AOP和其他纯Java AOP框架一样，在运行时执行编织。 

Types of advice: 

- *Before advice*:  在join point之前执行的advice，它没能力阻止执行流到join point（除非它抛出异常 ）
- *After returning advice*:  join point正常完成后执行的join point：例如，如果一个方法不抛出异常返回。 
- *After throwing advice*:  如果一个方法抛出异常退出，而执行的advice。
- *After (finally) advice*:  不管连接点退出的方法是什么，advice都执行（正常或异常返回）
- *Around advice*:  围绕一个join point的advice，例如一个方法调用。这是最强大的advice。Around advice可以在方法调用之前和之后执行自定义行为。 它还负责选择是否继续进行 join point或缩短被建议的方法执行，通过返回它自己的返回值或抛出异常。

Around advice是最通常类型的advice。因为Spring AOP像AspectJ一样，提供了一系列advice类型，我们建议使用 least powerful的advice来实现需要的行为。例如，如果您只需要用方法的返回值来更新缓存，你最好实现after returning advice，而不是around advice，虽然around advice可以做到同样的事情。使用最具体的advice类型提供一个更简单的编程模型，其错误的可能性更小。 你不需要调用用于around advice的`JoinPoint` 上的 `proceed()`方法，因此，不会调用失败。

在Spring 2.0，所有advice参数是静态类型的，这样你就可以使用合适类型的advice参数（例如，从方法执行的返回值的类型 ），而不是`Objects`数组。

join points的概念，与pointcuts相匹配，是AOP的关键，它将其与只提供拦截的旧技术区分开来。  Pointcuts 使advice独立于面向对象结构。例如，一个around advice提供声明式的事务管理，可以应用于一组跨越多个对象的方法（比如service层中的所有业务操作 ）。

#### 5.1.2. Spring AOP capabilities and goals

Spring AOP是纯Java实现的。 不需要一个特别的编译过程。 Spring AOP不需要控制类加载器层次结构，因此适合在Servlet容器或应用服务器中使用。

Spring AOP目前只支持method execution join points（advising the execution of methods on Spring beans ）。Filed拦截没有实现，尽管可以在不破坏核心Spring AOP api的情况下添加对field拦截的支持。 如果你需要advice field access和更新join points，考虑比如AspectJ的语言。

Spring AOP对AOP的方法不同于大多数的其他AOP框架。其目的不是提供最完整的AOP实现（虽然Spring AOP相当能干）；它更倾向于在AOP实现和Spring IoC之间提供一个紧密的集成来解决企业应用中的常见问题。

因此，例如，Spring框架的AOP功能通常与Spring IoC容器一起使用。aspects是用普通bean定义语法配置的（虽然这允许强大的"autoproxying"能力）；这是与其他AOP实现的关键区别。有些事情你不能用AOP轻松有效地做到，比如advice非常细粒度的对象（比如特别是 domain objects ）：在这种情况下，AspectJ是最好的选择。然而，我们的经验是，Spring AOP为企业Java应用程序中的大多数问题提供了一个很好的解决方案，这些问题都是面向AOP的。 

Spring AOP不会力图和AspectJ竞争以提供全面的AOP解决方案。我们相信，像Spring AOP这样的基于代理的框架和像AspectJ这样的成熟框架都是有价值的，他们是互补的而不是竞争关系。Spring无缝地将Spring AOP和IoC与AspectJ集成在一起，让所有的AOP的使用迎合基于spring的应用程序体系结构。这种集成不会影响Spring AOP API或AOP Alliance API：Spring AOP保持向后兼容。See [the following chapter](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-api) for a discussion of the Spring AOP APIs. 

**Spring Framework的一个核心准则是无侵入性；这个想法是你不应该引入特定于框架的类和接口到你的business/domain model。然而，在某些地方，Spring框架确实为您提供了将Spring框架特定的依赖关系引入到代码库中的选项：给你这样的选择的理由是，在某些情况下，用这种方式阅读或编码某些特定的功能可能会更容易一些。Spring框架（几乎）总是为您提供选择：您可以自由地做出明智的决定，选择哪种选项最适合您的特定用例或场景。** 

**与本章相关的一个选择是选择哪种AOP框架（以及AOP风格）。 您可以选择AspectJ和/或Spring AOP， 你也可以选择@Aspectj注解风格的方法或者Spring XML配置样式的方法。事实是这章选择引入 @AspectJ-style方法，不应该被认为Spring相比XML更喜欢@AspectJ annotation-style。**

**See [Choosing which AOP declaration style to use](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-choosing) for a more complete discussion of the whys and wherefores of each style.** 

####  5.1.3. AOP Proxies

Spring默认对AOP代理使用校准JDK *dynamic proxies*。这使得任何接口（或接口集）都可以被代理。 

Spring AOP也可以用CGLIB proxies。这对于代理类而不是接口是必要的。如果业务对象没有实现接口，则默认使用CGLIB。 对接口而不是类进行编程是一种很好的做法；业务类通常实现一个或多个业务接口。有可能强制使用CGLIB，在那些（希望是罕见的）情况下，您需要为一个没有在接口上声明的方法提供advise，或者您需要将一个proxied对象传递给一个具体类型的方法。 

理解Spring AOP是*proxy-based*的事实是很重要的。see  [Understanding AOP proxies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-understanding-aop-proxies)  彻底检查这个实现细节的确切含义。 

### 5.2. @AspectJ support

@AspectJ指的是一种声明aspects的方式，在常规Java类上标注此注解。 @AspectJ 风格是由 [AspectJ project](https://www.eclipse.org/aspectj)引入，作为AspectJ 5的一部分。*：*Spring解释了与AspectJ 5相同的注解，使用AspectJ提供的库来进行pointcut的解析和匹配。AOP运行时仍然是纯Spring AOP，并且对AspectJ编译器或weaver没有依赖关系。 

**使用AspectJ编译器和编织器可以使用完整的AspectJ语言，and is discussed in [Using AspectJ with Spring applications](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-using-aspectj).** 

#### 5.2.1. Enabling @AspectJ Support

要在Spring配置中使用@Aspectj切面，您需要启用Spring支持，以便基于@Aspectj切面来配置Spring AOP，根据这些是否被Aspect advised来确定自动代理beans。通过自动代理是说，如果Spring绝对一个bean被一个或多个aspect advised，它将自动生成一个代理，以便拦截方法调用，并确保根据需要执行建议。 

 @AspectJ support可以用 XML or Java style configuration来启用。不管哪种情况你都要确认AspectJ’s `aspectjweaver.jar`库在classpath（version 1.8+）。

##### Enabling @AspectJ Support with Java configuration

add the `@EnableAspectJAutoProxy`  ：

```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

##### Enabling @AspectJ Support with XML configuration

 use the `aop:aspectj-autoproxy` element: 

```
<aop:aspectj-autoproxy/> 
```

这假设您在用 [XML Schema-based configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-schemas) 描述的schema support 。 See [the AOP schema](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-schemas-aop) for how to import the tags in the `aop` namespace. 

####  5.2.2. Declaring an aspect

 启用了@AspectJ support，任何定义在application context的bean，是一个 @AspectJ 的aspect，将会被自动检测，并且被用于配置aop。下面是一个 not-very-useful Aspect的最小定义：

常规的application context bean定义，指向一个带有 `@Aspect` 的类：

```
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of aspect here as normal -->
</bean>
```

 `NotVeryUsefulAspect` 类定义，带有 `org.aspectj.lang.annotation.Aspect` 注解：

```
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

Aspects （带有 `@Aspect`注解的类）可能和其他类一样带有方法和field。他们也可能包含pointcut，advice，introduction(inter-type)声明。

**Autodetecting aspects through component scanning** 

**你可以注册aspects类作为beans到xml，像其他beans一样自动类路径扫描。然而，注意*@Aspect* 注解不足于自动检测：为了能自动检测，你需要添加 *@Component* （或者是一个定制的原型注解，根据Spring的组件扫描器的规则 ）**

**Advising aspects with other aspects?** 

**在Spring AOP，让aspects本事成为来自其他aspects的advice的目标是不可能的。*@Aspect* 注解在类上标注他为一个aspect，因此把他从auto-proxying排除。**

#### 5.2.3. Declaring a pointcut

回想一下pointscuts绝对感兴趣的join points，因此当advice执行时，让我们控制。Spring AOP只支持Spring beans的方法执行join points，所以你可以把pointcut想象成与Spring bean上的方法的执行相匹配。pointcut  声明有2个部分，一个包含name和任意参数的签名，一个pointcut表达式决定了哪个*exactly*我们关注的方法执行。在 @AspectJ annotation-style of AOP ，一个pointcut签名由常规方法定义提供，pointcut表达式使用 `@Pointcut`指示（作为pointcut签名的方法必须是void返回类型 ）。

一个例子清楚区分pointcut signature and a pointcut expression 。下例定义了一个叫`'anyOldTransfer'` 的pointcut，匹配任何叫'transfer' 方法的执行。

```
@Pointcut("execution(* transfer(..))")// the pointcut expression
private void anyOldTransfer() {}// the pointcut signature
```

形成`@Pointcut`注解的值的切入点表达式是一个常规的AspectJ 5切入点表达式。 关于AspectJ的切入点语言的完整讨论，see the [AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html) (and for extensions, the [AspectJ 5 Developers Notebook](https://www.eclipse.org/aspectj/doc/released/adk15notebook/index.html)) or one of the books on AspectJ such as "Eclipse AspectJ" by Colyer et. al. or "AspectJ in Action" by Ramnivas Laddad. 

##### Supported Pointcut Designators

Spring AOP支持以下AspectJ pointcut designators (PCD) 用于pointcut切点表达式：

```
						Other pointcut types
完整的AspectJ pointcut language支持额外的pointcut designators，而Spring不支持。它们是：call, get, set, preinitialization, staticinitialization, initialization, handler, adviceexecution, withincode, cflow, cflowbelow, if, @this, and @withincode. 在Spring用这些会抛出IllegalArgumentException。
Spring AOP支持的 pointcut designators在将来可能会扩展
```

- *execution* - 用于匹配 join points方法执行，是你使用aop最基本的pointcut designator
- *within* - 在几种类型限制匹配join points（简单地说，在使用Spring AOP时，在匹配类型中声明的方法的执行 ）
- *this* - 限制匹配到join points时（the execution of methods when using Spring AOP），bean reference（Spring AOP proxy） 是一个给定类型的实例
- *target* -限制匹配到join points时（the execution of methods when using Spring AOP），目标对象（application object being proxied）是一个给定类型的实例
- *args* -限制匹配到join points时（the execution of methods when using Spring AOP），参数是给定类型的实例
- *@target* - 限制匹配到join points时（the execution of methods when using Spring AOP），执行对象的类有一个给定类型的注解。
- *@args* - 限制匹配到join points时（the execution of methods when using Spring AOP），传递的确切运行时参数的类型有给定类型的注解。
- *@within* -限制匹配到join points在给定注解类型内（使用aop时，方法执行声明在给定注解的类型上）
- *@annotation* - 限制匹配到join points，join point的subject（AOP里执行的方法）有给定注解。

因为Spring AOP只在方法执行join points限制匹配，上面的pointcut designators的讨论给出了一个比AspectJ编程指南中所能找到的更窄的定义。另外，AspectJ有基于类型的语义，并且在执行连接点 `this` and `target` 引用同一个对象-执行方法的对象。Spring AOP是proxy-based system。区分代理对象本身（绑定到`this`）和代理背后的目标对象（绑定到`target`）。 

**由于Spring AOP框架的基于代理的特性，目标对象中的调用根据定义没有被拦截。对于JDK代理，只有在代理上调用的public接口方法才能被拦截。使用CGLIB，代理上调用的public and protected方法会被拦截，甚至必要时包可见方法也会被拦截。然而，普通的通过代理的交互总是通过public签名设计的。**

**注意pointcut 定义通常与被拦截的方法相匹配。如果一个pointcut严格意义上是public-only，即使是使用CGLIB的场景使用潜在的通过代理的no-public交互，它需要对应被定义。**

**如果你的拦截需要方法调用甚至在目标类中的构造器，考虑使用Spring-driven [native AspectJ weaving](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) 而不是Spring’s proxy-based AOP framework。这构成了具有不同特征的AOP使用的不同模式，所以，在做决定之前一定要先熟悉一下weaving。** 

Spring AOP也支持额外的PCD命名的`bean`.PCD 允许限制一个join points匹配到一个特定名字的bean，或者是一系列名字的beans（当使用通配符）。`bean` PCD  形式如下：

```
bean(idOrNameOfBean)
```

 `idOrNameOfBean` token可以是任意Spring bean的name：支持 `*`  作为通配符，因此，如果您为Spring bean建立了一些命名约定，您可以很容易地编写一个bean PCD表达式来挑选它们。 与其他切入点指示器的情况一样，bean PCD可以是  &&'ed, ||'ed, and ! (negated) 。

**注意 `bean` PCD 只在Spring AOP支持，native AspectJ weaving并不支持。这是Spring对AspectJ定义的standard PCDs的特定扩展，在`@Aspect`  声明的模型里不适用。**

 **`bean` PCD 在 实例 级别操作（以Spring bean名称的概念为基础  ），而不是只在类型级别上 （ weaving-based AOP的限制）。 Instance-based pointcut designators是Spring’s proxy-based AOP framework的特性，它与Spring bean factory的紧密集成，用名称来标识特定的bean是自然而直接的。** 

##### Combining pointcut expressions

using '&&', '||' and '!' 结合Pointcut expressions。也可以通过name引用pointcut expressions。下面的例子展示了三个切入点表达式：`anyPublicOperation`（匹配任意public方法）  ；`inTrading`  （匹配 in the trading module 的方法执行）；`tradingOperation`（两种合并）  。

```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```

从较小的命名组件中构建更复杂的切入点表达式是一种最佳实践。当以name引用切入点时，普通Java可见性规则适用（你可以在同一类型中看到私有切入点，在层次结构中受保护的切入点，任何地方的公共切入点等等 ）。可见性并不影响切入点匹配。

##### Sharing common pointcut definitions

在处理企业应用程序时，您通常想要从几个aspect引用应用程序的模块和特定的操作集。我们建议定义"SystemArchitecture" Aspect，捕获常见的pointcut表达式。典型的这样一个Aspect： 

```
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.someapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.someapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.someapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
     * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

在这样一个方面定义的切入点可以被引用到任何需要切入点表达式的地方。例如，为了使服务层事务化，您可以写： 

```
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

The `<aop:config>` and `<aop:advisor>` elements are discussed in [Schema-based AOP support](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-schema). The transaction elements are discussed in [Transaction Management](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/data-access.html#transaction). 

##### Examples

Spring 用户最经常可能使用`execution` pointcut designator。execution 表达式格式：

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
            throws-pattern?)
```

所有部分除了returning type pattern（上面的ret-type-pattern）， name pattern和 parameters pattern都是可选的。returning type pattern决定了该方法的返回类型必须是什么，以便匹配连接点。 通常使用`*`作为返回类型，匹配任意返回类型。只有当方法返回给定的类型时，才会匹配全限定类型名称。 name pattern 匹配方法名。您可以使用通配符`*`作为name pattern的全部或部分。 如果指定一个声明的type pattern包含一个`.`结尾，连接到name pattern组件。parameters pattern 稍稍变得复杂：`()`  匹配一个没有参数的方法，`(..)` 匹配任意数量参数的方法（0 or more）。`(*)`  匹配有一个任意参数的方法，`(*,String)`  匹配第一个参数任意，第二个为String的方法。Consult the[Language Semantics](https://www.eclipse.org/aspectj/doc/released/progguide/semantics-pointcuts.html) section of the AspectJ Programming Guide for more information. 

一些常见pointcut expressions例子如下，

- the execution of any public method: 

  ```
  execution(public * *(..))
  ```

- the execution of any method with a name beginning with "set": 

  ```
  execution(* set*(..))
  ```

- the execution of any method defined by the `AccountService` interface: 

  ```
  execution(* com.xyz.service.AccountService.*(..))
  ```

- the execution of any method defined in the service package: 

  ```
  execution(* com.xyz.service.*.*(..))
  ```

- the execution of any method defined in the service package or a sub-package: 

  ```
  execution(* com.xyz.service..*.*(..))
  ```

- any join point (method execution only in Spring AOP) within the service package: 

  ```
  within(com.xyz.service.*)
  ```

- any join point (method execution only in Spring AOP) within the service package or a sub-package: 

  ```
  within(com.xyz.service..*)
  ```

- any join point (method execution only in Spring AOP) where the proxy implements the `AccountService` interface: 

  ```
  this(com.xyz.service.AccountService)
  ```

  **`this`和`target`和`args`和`@target`和`@within`和`@annotation `和`@args` 通常以绑定的形式使用：看下节，怎么让代理对象在advice body可用。**

- any join point (method execution only in Spring AOP) where the target object implements the `AccountService` interface: 

  ```
  target(com.xyz.service.AccountService)
  ```

- any join point (method execution only in Spring AOP) which takes a single parameter, and where the argument passed at runtime is `Serializable`: 

  ```
  args(java.io.Serializable)
  ```

  注意给出的整个例子和 `execution(* *(java.io.Serializable))`是不同的，如果在运行时传递的参数是 Serializable，args 版本会匹配，execution版本匹配的是方法签名为一个`Serializable`的参数。

- any join point (method execution only in Spring AOP) where the target object has an `@Transactional` annotation: 

  ```
  @target(org.springframework.transaction.annotation.Transactional)
  ```

- any join point (method execution only in Spring AOP) which takes a single parameter, and where the runtime type of the argument passed has the `@Classified` annotation: 

  ```
  @args(com.xyz.security.Classified)
  ```

- any join point (method execution only in Spring AOP) on a Spring bean named `tradeService`: 

  ```
  bean(tradeService)
  ```

- any join point (method execution only in Spring AOP) on Spring beans having names that match the wildcard expression `*Service`: 

  ```
  bean(*Service)
  ```

##### Writing good pointcuts

在编译过程中，AspectJ处理pointcut以尝试和优化匹配表现。检查代码并确定每个join point匹配（静态或动态地）给定pointcut是一个代价高昂的过程。（动态匹配意味着匹配不能从静态分析中完全确定，并且将在代码中放置一个测试，以确定代码是否在运行时是否有真正的匹配 ）。当第一次遇到pointcut 声明，AspectJ将把它重写用于匹配过程的最佳形式。这意味着什么？基本上，pointcuts 被重写成DNF（Disjunctive Normal Form ），pointcut的组件被排序，这样这些组件被认定为廉价而首先被检测。这意味着，你不需要担心理解各种pointcut designators的性能表现，可以在切入点声明中以任何顺序提供它们。 

然而，AspectJ只能使用它被告知的内容，为了达到最佳匹配的性能你应该考虑他们想要达到的目标并尽可能缩小匹配的搜索空间。存在的designators 自然地分成3组：kinded, scoping and context: 

- Kinded designators 选择一种特定种类的 join point。例如：execution, get, set, call, handler 
- Scoping designators 选择一组关注的 join points（可能很多种）。例如：within, withincode 
- Contextual designators 基于context匹配（bind可选）。例如：this, target, @annotation 

一个写的好的pointcut至少包含前两种。如果希望根据joinpoint context匹配可以同时包含contextual designators ，或者在advice中绑定context以使用。仅提供一个kinded designator 或者contextual designator 会工作，但是由于所有额外的处理和分析，可能会影响编织性能（使用的时间和内存）。Scoping designators匹配非常快，它们的使用意味着AspectJ可以非常快速地忽略不应该进一步处理的连接点组——这就是为什么一个好的切入点应该总是包含一个这个连接点。  

#### 5.2.4. Declaring advice

advice和pointcut expression关联，并且在由pointcut匹配的方法执行上before, after, or around runs。pointcut expression可以是一个命名的pointcut的简单引用，或者合适地方声明的pointcut expression 。

##### Before advice

Before advice是声明在一个切面里，使用`@Before` 注解：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

如果就地使用 pointcut表达式，可以写成：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```

#####  After returning advice

After returning advice在 matched method execution普通返回时执行。使用 `@AfterReturning`声明：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

**注意：当然有可能有多个advice声明，以及其他成员，都在同一个Aspect。我们只展示了一个advice声明在这些例子，关注正在讨论的问题。** 

有时候，您需要在advice体中访问返还的实际值。 可以使用 `@AfterReturning` 绑定返回值的形式：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```

 `returning` 属性中用的name，要和advice中的参数name一致。当一个方法执行返回了，返回值将作为对应参数值返回给advice方法。 `returning`语句约束了只匹配返回特定类型的方法执行（此例是`Object`，匹配任何返回值  ）。

请注意，当使用after-returning advice，不可能返回完全不同的引用。

##### After throwing advice

After throwing advice当匹配方法抛出异常执行。使用`@AfterThrowing` 注解：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }

}
```

通常，只有当给定类型的异常被抛出匹配方法执行退出时，您才希望建议只运行，你也经常需要在advice body访问抛出的异常。使用`throwing`  属性来同时限制匹配（如果需要，可以使用 `Throwable`作为异常类型 ）和把抛出异常绑定到advice参数。

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```

在`throwing` 属性用的name和advice方法参数的name一致。当一个方法执行通过抛出一个异常来退出时，异常会被传递给advice方法作为对应的参数值。`throwing` 语句也限制了匹配，只会匹配抛出了给定异常的方法执行（本例为`DataAccessException`  ）。

##### After (finally) advice

After (finally) advice当匹配方法执行退出时执行。使用`@After` 注解。After advice 一定要准备好去处理正常返回和异常返回两种情况。它通常用于释放资源，等等。 

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```

##### Around advice

最后是around advice类型。Around advice 在匹配方法执行的“around”执行。他有机会做both before and after the method executes的工作，并决定何时，如何甚至这个方法是否真的执行。Around advice 通常用于在一个方法执行的前后share state，以线程安全的方式（例如启动和停止一个timer）。总是使用最不强大的advice形式来满足你的需求（例如如果简单before advice就能做，那么别用around advice）。

使用`@Around` 注解声明Around advice。advice 方法第一个参数必须是`ProceedingJoinPoint`。在advice body内，调用`ProceedingJoinPoint` 上的`proceed()`导致底层方法的执行。`proceed` 方法也可以被调用以Object[]形式传递——数组中的值会被用作方法执行的参数。

**当带有Object[]对象的调用被proceed，和 AspectJ 编译器编译的around advice的proceed过程不同。对于使用传统 AspectJ语言写的around advice，传递给proceed 的参数一定要匹配传递给around advice的参数（不是底层join point记录的参数），在给定的参数位置传递给proceed的值在 join point 代替原始值，对于值要绑定到上去的实体（如果这没有意义，别担心）。Spring采用的方法更简单，更好地匹配proxy-based, execution only的语义。如果您正在编译为Spring编写的@AspectJ切面，并使用AspectJ编译器和weaver的参数，那么您只需要知道这种差异。 有一种方法可以在Spring AOP和AspectJ中编写百分百兼容的Aspect，这将在接下来的关于advice参数的小节中讨论。** 

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
public class AroundExample {

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
        // start stopwatch
        Object retVal = pjp.proceed();
        // stop stopwatch
        return retVal;
    }

}
```

 around advice 返回的值，将会是这个方法的调用者看到的返回值。例如一个简单的缓存Aspect可以从缓存里返回值如果有，调用 proceed() 如果没有。注意around advice body内的proceed可能会被调用一次，多次，或者一次没有，所有的情况都是合法的。

##### Advice parameters

Spring提供了全类型的 advice ——意味着你可以在 advice 签名声明需要的参数（正如我们在上面看到的返回和抛出的例子 ）而不是总是使用`Object[]` 。我们将会看到如何把参数和其他上下文的值对advice body可用。首先看一下如何写一个通用advice，找出advice正在advising的方法。

###### Access to the current JoinPoint

一个advice method可能声明它的第一个参数，一种`org.aspectj.lang.JoinPoint` 参数类型（请注意around advice需要声明第一个参数类型是`ProceedingJoinPoint，`他是`JoinPoint`的子类。`JoinPoint` 接口提供了一些有用的方法比如 `getArgs()` （返回方法参数）， `getThis()`（返回代理对象）， `getTarget()`  （返回目标对象），`getSignature()`  （返回被advised方法的描述），`toString()`  （打印一个有用的被advised方法的描述信息））。Please do consult the javadocs for full details. 

 ###### Passing parameters to advice 

我们已经看到如何绑定返回的值或者异常值（使用after returning and after throwing advice ）。为了使参数值对advice body可用，你可以使用`args`的形式绑定。如果一个参数name在args表达式代替了一个type name，对应参数的值会被作为参数值传递当advice被调用时。1个例子应该使这个更清楚。 假设你想advise dao  operations  的执行，把Account 对象作为第一个参数，你需要在advice body访问account。

```
@Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```

`args(account,..)` 作为pointcut表达式的一部分有2个目的：限制了匹配，只有那些至少一个方法，the argument passed to that parameter 是一个`Account`的实例的方法执行才匹配；第二，使`Account` 对象通过`account` 参数对advice可用。

另一种写法，声明一个pointcut 当它匹配到一个Join point“提供“ `Account` 对象，然后仅仅引用被命名的来自advice的pointcut。

```
@Pointcut("com.xyz.myapp.SystemArchitecture.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```

The interested reader is once more referred to the AspectJ programming guide for more details. 

代理对象( `this`) ，目标对象 ( `target`) ，和注解( `@within, @target, @annotation, @args`)  都可以以相同的方式绑定。下例展示了怎么匹配带有 `@Auditable` 的方法执行。

首先定义`@Auditable`  ：

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}
```

匹配 `@Auditable`方法执行的advice：

```
@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

###### Advice parameters and generics 

Spring AOP可以处理用在class和方法参数声明的泛型。假设你有：

```
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```

你可以通过某些参数类型限制方法类型匹配，通过：

```
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}
```

这是很明显的，就像我们已经讨论过的。 然而，值得指出的是，这对于泛型集合来说是行不通的。 所以你不能定义这样的切入点： 

```
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```

为了完成这项工作，我们必须检查集合中的每一个元素，这是不合理的，因为我们也不能决定如何一般地对待null值。 为了达到相同效果，你必须把参数改为`Collection<?>` 且手动检查元素的类型。

 ###### Determining argument names 

绑定在advice调用的参数依赖于把切入点表达式中使用的name匹配到方法签名（通知和切入点）中声明的参数名。参数names通过Java反射是不可用的，Spring AOP使用以下策略绝对参数name：

- 如果参数names被用户显示指定，则使用指定名称：advice and the pointcut annotations都有可选的 "argNames"  属性用于指定注解方法的参数names-这些参数在运行时可用。

  ```
  @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
          argNames="bean,auditable")
  public void audit(Object bean, Auditable auditable) {
      AuditCode code = auditable.value();
      // ... use code and bean
  }
  ```

  如果第一个参数是`JoinPoint`, `ProceedingJoinPoint`, or `JoinPoint.StaticPart` 类型的，您可以从“argNames”属性的值中省略参数的名称。例如，如果您修改前面的advice来接收pointcut对象，那么“argNames”属性不需要包括：

  ```
  @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
          argNames="bean,auditable")
  public void audit(JoinPoint jp, Object bean, Auditable auditable) {
      AuditCode code = auditable.value();
      // ... use code, bean, and jp
  }
  ```

  对第一个参数类型为 `JoinPoint`, `ProceedingJoinPoint`, and `JoinPoint.StaticPart` 的处理对advice很方便，不要收集任何其他连接点上下文。在这种情况下，您可能只是省略了“argname”属性。例如，以下advice不必声明“argname”属性： 

  ```
  @Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
  public void audit(JoinPoint jp) {
      // ... use jp
  }
  ```

- 使用'argNames' 有点笨拙，因此如果'argNames' 没有被指定， Spring AOP 会查找类的debug信息，从局部变量表决定参数name。只要使用调试信息编译类，这些信息就会出现。（最低限度'-g:vars' ）。用这面标识进行编译的后果是： 1.您的代码将会更容易理解（逆向工程）2.类文件的大小会稍微大一些（通常是无关紧要的）；删除未使用的局部变量的优化将不会被编译器应用。 换句话说，你应该在这个标识上没有遇到任何困难。

  **如果一个@AspectJ的Aspect已经被AspectJ compiler (ajc)编译，甚至没有debug信息，那么不需要添加argNames属性，编译器会返回需要的信息。**

- 如果代码被编译并且没有必要的debug信息， Spring AOP  会尝试推导出绑定变量与参数的配对（例如，如果切点表达式只有一个变量，方法就一个参数，那就很明显了）。如果给定的信息，变量的绑定是模糊的 ，抛出 `AmbiguousBindingException` 。

- 如果以上策略都失败了，抛出 `IllegalArgumentException` 。

 ###### Proceeding with arguments 

我们之前说过，我们将描述如何用Spring AOP和AspectJ描述一个持续运行的参数的调用。他的解决方案只是为了确保建议签名能够按顺序绑定每个方法参数。 例如：

```java
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.myapp.SystemArchitecture.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)")
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```

在很多情况下，您将会执行这个绑定（如上面的示例） 。

##### Advice ordering

当多个advice都想在同一个 join point上运行时会发生什么？ Spring AOP遵循和AspectJ一样的优先级规则决定advice的执行顺序。最高优先级的advice先运行 "on the way in"  （给定了两个before advice，最高优先级的是先运行）。从一个join point "On the way out"  ，最高优先级的advice最后运行。（给定两个after advice，最后优先级的第二个运行。）

当两个不同Aspect定义的建议都需要在同一个join point上运行，除非您指定否则执行的顺序是未定义的。 您可以通过指定优先级来控制执行顺序。这是用正常的Spring方法完成的 ，在Aspect类实现 `org.springframework.core.Ordered` 接口或者使用`Order` 注解。给定两个Aspect， `Ordered.getValue()`（或注解值）返回较低的值有较高的优先级。

当定义在同一Aspect的两个advice要在同一个join point运行，顺序未定义（因为没办法对javac编译的类通过反射检索顺序声明）。考虑在每个Aspect类中，将这些方法折叠成一个advice方法，或者将advice的各个部分重构到单独的Aspect类中——这些类可以在Aspect级别上排序。 

####  5.2.5. Introductions

Introductions （在AspectJ中被称为内部类型声明 ）使一个Aspect声明被advised的对象实现给定接口，并代表这些对象提供该接口的实现。 

使用 `@DeclareParents` 注解做一个introduction 。这个注释用于声明匹配类型有一个新的parent（对应名字）。例如，给定`UsageTracked`接口，一个此接口的实现`DefaultUsageTracked`，以下Aspect声明所有service接口的实现者都要实现`UsageTracked` 接口（例如为了通过JMX公开统计信息 ）。

```java
@Aspect
public class UsageTracking {

    @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
    public static UsageTracked mixin;

    @Before("com.xyz.myapp.SystemArchitecture.businessService() && this(usageTracked)")
    public void recordUsage(UsageTracked usageTracked) {
        usageTracked.incrementUseCount();
    }

}
```

要实现的接口由被注解field类型决定。 `@DeclareParents` 注解的`value` 属性是一个 AspectJ type pattern：所有匹配类型会实现UsageTracked 接口。请注意，在上面的示例的before advice中，service beans可以直接作为`UsageTracked` 接口的实现被使用。如果以编程方式访问bean，您将编写以下内容： 

```
UsageTracked usageTracked = (UsageTracked) context.getBean("myService"); 
```

####  5.2.6. Aspect instantiation models

（这是一个高级的主题，所以如果您刚开始使用AOP，您可以安全地跳过它，直到以后。） 

默认情况下，在应用程序上下文中会有每个Aspect的单例实例。AspectJ称其为单例实例化模型。可以用不同的生命周期来定义各个Aspect：Spring支持AspectJ的 `perthis` and `pertarget`实例化模型（`percflow, percflowbelow,` and `pertypewithin`  目前不支持）。

一个 "perthis"切面通过指定 `@Aspect` 注解里的`perthis` 语句声明。让我们看一个例子，然后我们将解释它是如何工作的。 

```
@Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
public class MyAspect {

    private int someState;

    @Before(com.xyz.myapp.SystemArchitecture.businessService())
    public void recordServiceUsage() {
        // ...
    }

}
```

'perthis' 语句的效果是：每一个唯一的service对象执行一个business service，就创建一个Aspect实例（每个唯一对象在join point绑定到 'this' ，通过切点表示匹配）。aspect 实例第一次被创建是在service对象上的一个方法被调用。当service对象超出范围时，Aspect就超出了范围。 在创建Aspect实例之前，它里面的advice都没有执行。一旦创建了Aspect实例，内部声明的advice将在匹配的连接点上执行，但只能是service对象是Aspect关联的对象才行。See the AspectJ programming guide for more information on per-clauses. 

'pertarget' 实例化模型和perthis一样的方式工作，但是在匹配的连接点上为每个独特的目标对象创建一个Aspect实例。 

#### 5.2.7. Example

现在您已经了解了所有的组成部分是如何工作的，让我们把它们放在一起做一些有用的事情！ 

由于并发问题，业务服务的执行有时会失败（例如， deadlock loser ）。如果操作被重新尝试 ，很有可能在下次的时候成功。对于业务服务，在这样的条件下进行重试是合适的（不需要返回用户解决冲突的幂等操作 ），我们希望透明地重试操作，避免客户端看到`PessimisticLockingFailureException`。这明显是个在service层切入多个service的需求，因此，通过一个aspect来实现是很理想的。

因为我们想要重试操作，所以我们需要使用around advice，这样我们就可以调用多次了。 以下是基本aspect实现：

```java
@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.SystemArchitecture.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }

} 
```

改进切面，只重试幂等操作，定义 `Idempotent`注解：

```
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

使用注解标注业务操作的实现。改变切面只尝试幂等操作，只需要重新定义切点表达式只匹配`@Idempotent`  ：

```
@Around("com.xyz.myapp.SystemArchitecture.businessService() && " +
        "@annotation(com.xyz.myapp.service.Idempotent)")
public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
    ...
}
```

###  5.3. Schema-based AOP support

如果你更喜欢 XML-based形式...

要使用本节中描述的aop名称空间标记，需要导入`spring-aop` schema as described in [XML Schema-based configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-schemas)。See [the AOP schema](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-schemas-aop) for how to import the tags in the `aop` namespace. 

在你的Spring配置里，所有aspect and advisor都要在`<aop:config>` 内（一个application context可以有多个`<aop:config>`）。一个`<aop:config>`  可以包括pointcut, advisor, and aspect elements（注意必须按顺序声明）。

 **`<aop:config>`  配置风格大量使用了 [auto-proxying](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-autoproxy) 机制。如果你已经使用`BeanNameAutoProxyCreator` 或类似的显示得auto-proxying，可能会导致问题（比如advice没有被织入）。推荐的使用模式要么只使用 `<aop:config>`样式，要么仅仅`AutoProxyCreator`  风格。** 

#### 5.3.1. Declaring an aspect

 an aspect is simply a regular Java object defined as a bean in your Spring application context.

```
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
```

#### 5.3.2. Declaring a pointcut

```
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```

或者，假设你有`SystemArchitecture`  切面as described in [Sharing common pointcut definitions](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-common-pointcuts). 

```
<aop:config>

    <aop:pointcut id="businessService"
        expression="com.xyz.myapp.SystemArchitecture.businessService()"/>

</aop:config>
```

```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        ...

    </aop:aspect>

</aop:config>
```

```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..)) &amp;&amp; this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...

    </aop:aspect>

</aop:config>
```

使用`and`, `or`, and `not` 代替`&&`, `||`, and `!` 。

```
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service..(..)) and this(service)"/>

        <aop:before pointcut-ref="businessService" method="monitor"/>

        ...
    </aop:aspect>
</aop:config>
```

请注意，以这种方式定义的切入点是由它们的XML id引用的，不能用作命名切入点来形成复合切入点 因此，基于约束的定义风格中命名的切入点支持比@aspectj风格所提供的更有限。 

#### 5.3.3. Declaring advice

#####  Before advice

```
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

这里`dataAccessOperation` 是一个pointcut，定义在顶级`<aop:config>`级别。使用一个 `pointcut`  代替 `pointcut-ref`  ：

```
<aop:aspect id="beforeExample" ref="aBean">

    <aop:before
        pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

正如我们在讨论@Aspectj风格时所指出的，使用命名切入点可以显著提高代码的可读性。 

#####  After returning advice

```
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

```
<aop:aspect id="afterReturningExample" ref="aBean">

    <aop:after-returning
        pointcut-ref="dataAccessOperation"
        returning="retVal"
        method="doAccessCheck"/>

    ...

</aop:aspect>
```

doAccessCheck 必须有一个叫 `retVal`的参数，和@AfterReturning限制匹配的方式相同。

```
public void doAccessCheck(Object retVal) {...
```

##### After throwing advice

```
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        method="doRecoveryActions"/>

    ...

</aop:aspect>
```

```
<aop:aspect id="afterThrowingExample" ref="aBean">

    <aop:after-throwing
        pointcut-ref="dataAccessOperation"
        throwing="dataAccessEx"
        method="doRecoveryActions"/>

    ...

</aop:aspect>
```

```
public void doRecoveryActions(DataAccessException dataAccessEx) {...
```

##### Around advice

最后一种建议是关于建议的。围绕建议运行“围绕”匹配的方法执行。

```
<aop:aspect id="aroundExample" ref="aBean">

    <aop:around
        pointcut-ref="businessService"
        method="doBasicProfiling"/>

    ...

</aop:aspect>
```

`doBasicProfiling`  实现和@AspectJ例子一样。

```
public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
    // start stopwatch
    Object retVal = pjp.proceed();
    // stop stopwatch
    return retVal;
}
```

##### Advice parameters

. See [Advice parameters](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-ataspectj-advice-params) for details. 如果你想要显式地为建议方法指定参数名 ， using the `arg-names` attribute of the advice element ，和as described in [Determining argument names](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-ataspectj-advice-params-names) 一样。

```
<aop:before
    pointcut="com.xyz.lib.Pointcuts.anyPublicMethod() and @annotation(auditable)"
    method="audit"
    arg-names="auditable"/>
```

下面是一个稍微复杂一点的xsd的方法的例子，演示了一些与大量强类型参数一起使用的advice。

```
package x.y.service;

public interface FooService {

    Foo getFoo(String fooName, int age);
}

public class DefaultFooService implements FooService {

    public Foo getFoo(String name, int age) {
        return new Foo(name, age);
    }
}
```

下一个是aspect。注意 `profile(..)` 方法接受很多强类型参数，

```
package x.y;

import org.aspectj.lang.ProceedingJoinPoint;
import org.springframework.util.StopWatch;

public class SimpleProfiler {

    public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
        StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
        try {
            clock.start(call.toShortString());
            return call.proceed();
        } finally {
            clock.stop();
            System.out.println(clock.prettyPrint());
        }
    }
}
```

最后，这里是为特定连接点执行上述建议所需的XML配置： 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
    <bean id="fooService" class="x.y.service.DefaultFooService"/>

    <!-- this is the actual advice itself -->
    <bean id="profiler" class="x.y.SimpleProfiler"/>

    <aop:config>
        <aop:aspect ref="profiler">

            <aop:pointcut id="theExecutionOfSomeFooServiceMethod"
                expression="execution(* x.y.service.FooService.getFoo(String,int))
                and args(name, age)"/>

            <aop:around pointcut-ref="theExecutionOfSomeFooServiceMethod"
                method="profile"/>

        </aop:aspect>
    </aop:config>

</beans>
```

如果我们有下面的驱动脚本，我们会在标准输出中得到这样的输出： 

```
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import x.y.service.FooService;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        BeanFactory ctx = new ClassPathXmlApplicationContext("x/y/plain.xml");
        FooService foo = (FooService) ctx.getBean("fooService");
        foo.getFoo("Pengo", 12);
    }
}
```

```
StopWatch 'Profiling for 'Pengo' and '12'': running time (millis) = 0
-----------------------------------------
ms     %     Task name
-----------------------------------------
00000  ?  execution(getFoo)
```

#####  Advice ordering

as described in [Advice ordering](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-ataspectj-advice-ordering) 。 `Order`  注解或者实现 `Order`  接口。

#### 5.3.4. Introductions

介绍（在AspectJ中称为内部类型声明）使一个方面能够声明建议的对象实现给定的接口，并代表这些对象提供该接口的实现。 

使用`aop:declare-parents` element inside an `aop:aspect` 。这个元素用于声明匹配类型有一个新的父类（因此得名） 。

```
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

    <aop:declare-parents
        types-matching="com.xzy.myapp.service.*+"
        implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
        default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

    <aop:before
        pointcut="com.xyz.myapp.SystemArchitecture.businessService()
            and this(usageTracked)"
            method="recordUsage"/>

</aop:aspect>
```

```
public void recordUsage(UsageTracked usageTracked) {
    usageTracked.incrementUseCount();
}
```

service beans是`UsageTracked` 实现。 

```
UsageTracked usageTracked = (UsageTracked) context.getBean("myService");
```

####  5.3.5. Aspect instantiation models

xml只支持singleton model 。其他将来可能实现。

####  5.3.6. Advisors

 "advisors"的概念是从Spring中定义的AOP支持中提出的，在AspectJ中没有直接的等价。一个advisor像一个 self-contained的 aspect 有一条advice。advice本身是bean的形式，必须实现advice接口之一 described in [Advice types in Spring](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-api-advice-types)。不过，advisor可以利用AspectJ切入点表达式。 

Spring支持advisor使用`<aop:advisor>`  ，您通常会看到它与事务advice一起使用 ，它在Spring中也有自己的名称空间支持。

```
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

    <aop:advisor
        pointcut-ref="businessService"
        advice-ref="tx-advice"/>

</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

为了定义advisor的优先级，以便advice可以参与排序，使用order属性来定义advisor的有序值。 

#### 5.3.7. Example

上面例子的XML配置：

```
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }

}
```

```
<aop:config>

    <aop:aspect id="concurrentOperationRetry" ref="concurrentOperationExecutor">

        <aop:pointcut id="idempotentOperation"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        <aop:around
            pointcut-ref="idempotentOperation"
            method="doConcurrentOperation"/>

    </aop:aspect>

</aop:config>

<bean id="concurrentOperationExecutor"
    class="com.xyz.myapp.service.impl.ConcurrentOperationExecutor">
        <property name="maxRetries" value="3"/>
        <property name="order" value="100"/>
</bean>
```

```
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    // marker annotation
}
```

```
<aop:pointcut id="idempotentOperation"
        expression="execution(* com.xyz.myapp.service.*.*(..)) and
        @annotation(com.xyz.myapp.service.Idempotent)"/>
```

### 5.4. Choosing which AOP declaration style to use

取决于application requirements, development tools, and team familiarity with AOP 。

####  5.4.1. Spring AOP or full AspectJ?

Use the simplest thing that can work。如果您只需要建议在Spring bean上执行操作，那么Spring AOP是正确的选择。 Spring AOP比full AspectJ更简单，不需要引入AspectJ compiler / weaver into your development and build processes。如果你只需要，advise the execution of operations on Spring beans ，Spring AOP就是正确选择。如果您需要建议未由Spring容器管理的对象（特别是比如domain object） ，那么你需要使用AspectJ。如果你需要advise join point而不是简单方法执行（例如，field get or set join points 等等），也需要AspectJ。

当使用AspectJ，你将选择使用 AspectJ language syntax 或 @AspectJ annotation style 。如果没用java 5+，不能用注解。如果Aspect在你的设计中起了很大的作用，能使用[AspectJ Development Tools (AJDT)](https://www.eclipse.org/ajdt/) plugin ，AspectJ language syntax 更好一些：它更简洁，更简单，因为语言是为写Aspect而设计的 。只有少数几个方面在您的应用程序中没有发挥主要作用，consider using the @AspectJ style ，在您的IDE中坚持常规的Java编译，并在构建脚本中添加一个方面编织阶段。 

####  5.4.2. @AspectJ or XML for Spring AOP?

有一些权衡：

xml对Spring用户最熟悉，当使用AOP作为配置企业服务的工具时，XML是一个不错的选择（一个好的测试是您是否认为切入点表达式是您的配置的一部分，您可能希望独立地进行更改 ）。有了XML风格，从您的配置中可以清楚地看到系统中存在哪些Aspect。  

XML样式有两个缺点。首先，它不能完全将处理的需求的实现封装在一个地方。DRY原则认为应该有一个单一的，明确的，权威的对系统中任何知识的表示。当使用the XML style，对如何需求被实现的认识通过支持bean类的声明和xml中的配置被分割。当使用@AspectJ style 有一个single module—— the aspect ——在这里信息被封装了。其次，XML样式比@Aspectj风格稍微限制了一些 ：只支持"singleton" 切面实例化模型，不能结合声明在xml的named pointcuts 。例如： @AspectJ style you can write something like: 

```
@Pointcut(execution(* get*()))
public void propertyAccess() {}

@Pointcut(execution(org.xyz.Account+ *(..))
public void operationReturningAnAccount() {}

@Pointcut(propertyAccess() && operationReturningAnAccount())
public void accountPropertyAccess() {}
```

```
<aop:pointcut id="propertyAccess"
        expression="execution(* get*())"/>

<aop:pointcut id="operationReturningAnAccount"
        expression="execution(org.xyz.Account+ *(..))"/>
```

xml不能定义`accountPropertyAccess`  通过结合两个。

@Aspectj风格支持额外的实例化模型，and  richer pointcut composition 。它的优点是将方面保持为一个模块化单元。 它还有一个优点，即@AspectJ切面可以被Spring AOP和AspectJ理解（从而被消耗）——因此，如果您稍后决定需要AspectJ的功能来实现额外的需求，那么很容易迁移到基于AspectJ的方法 。总的来说，当您拥有的Aspect不仅仅是简单的企业服务“配置”时，Spring团队更喜欢@Aspectj风格。 

### 5.5. Mixing aspect types

使用相同的底层支持机制。

### 5.6. Proxying mechanisms

Spring AOP使用JDK动态代理或CGLIB来为给定的目标对象创建代理（当你有选择的时候，JDK动态代理是首选的 ）。 

如果要被proxied的目标对象实现至少一个接口，那么就会使用JDK动态代理。目标类型实现的所有接口都将被代理。如果目标对象没有实现任何接口，那么就会创建CGLIB代理。 

如果你想强制使用CGLIB代理（例如，要代理为目标对象定义的每个方法，而不仅仅是由其接口实现 ），你也可以这么做。然而，有一些问题需要考虑： 

- `final` 方法不能被advised，因为不可覆盖。
- 自Spring 3.2，不需要加 CGLIB到classpath，因为CGLIB类被重新打包到org.springframework  并且直接包含在spring-core JAR。这意味着CGLIB-based代理支持 “就像” JDK dynamic proxies的方式一样工作。
- 自Spring 4.0，你的代理对象的构造函数将不再被调用两次，因为CGLIB代理实例将通过Objenesis创建。只有当您的JVM不允许constructor bypassing ，您可能会在Spring的AOP支持中看到双重调用和相应的调试日志条目。 

强制使用CGLIB代理 ：

```
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

当使用 @AspectJ autoproxy support，强制CGLIB代理：

```
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

**多个`<aop:config/>` 部分会在运行时合并成一个统一的auto-proxy creator ，应用了最强的代理设置。也适用于 `<tx:annotation-driven/>` and `<aop:aspectj-autoproxy/>`。**

**要明确：使用`proxy-target-class="true"`  在 `<tx:annotation-driven/>`, `<aop:aspectj-autoproxy/>` or `<aop:config/>` 会强制它们所有都用CGLIB。**

#### 5.6.1. Understanding AOP proxies

Spring AOP是*proxy-based*。在您编写自己的aspect或使用Spring框架提供的任何Spring基于aop的aspect之前，您必须掌握最后声明的含义，这是非常重要的。 

考虑第一个情况，你有一个plain-vanilla, un-proxied, nothing-special-about-it, straight object reference ，如下代码片段所示。 

```
public class SimplePojo implements Pojo {

    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        this.bar();
    }

    public void bar() {
        // some logic...
    }
}
```

如果你在对象引用上调用一个方法，该方法直接在该对象引用上调用，如下所示：

![](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/aop-proxy-plain-pojo-call.png)

```
public class Main {

    public static void main(String[] args) {

        Pojo pojo = new SimplePojo();

        // this is a direct method call on the 'pojo' reference
        pojo.foo();
    }
}
```

当客户端代码的引用是一个代理时，事情会发生轻微的变化。考虑下面的图表和代码片段。 

![](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/aop-proxy-call.png)

```
public class Main {

    public static void main(String[] args) {

        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();

        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

理解这里的关键是`Main` 类里的`main(..)` 里的客户端代码有对代理的引用。这意味着对该对象引用的方法调用将会在代理上调用，因此代理能够委托给所有的和特定方法调用相关的拦截器（advice）。然而，一旦调用最终到达目标对象，在这个例子的 `SimplePojo` ，任何在他自己的方法调用，比如`this.bar()` or `this.foo()` ，将会在*this*引用上去调用，而不是代理上。这有重要意义。这意味着自我调用不会导致与advice相关联的方法调用获得执行的机会。 

好了，那么我们该怎么做呢？ 最好的方法（最好一词在这用的很宽松）是重构您的代码，使自我调用不会发生。当然，这确实需要你做一些工作，但这是最好的，最小侵入性的方法。 下一种方法绝对是可怕的，我几乎不愿意指出它，因为它太可怕了。 您可以（窒息！）完全将您的类中的逻辑与Spring AOP结合在一起： 

```
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```

这完全将您的代码与Spring AOP结合在一起，这使得类本身意识到它正被用于AOP上下文中，它在AOP的表面上运行。 它还需要在创建代理时进行一些额外的配置： 

```
public class Main {

    public static void main(String[] args) {

        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.adddInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);

        Pojo pojo = (Pojo) factory.getProxy();

        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

最后，必须指出的是，AspectJ没有这种自我调用问题，因为它不是基于代理的AOP框架。 

### 5.7. Programmatic creation of @AspectJ Proxies

除了在配置中使用`<aop:config>` or `<aop:aspectj-autoproxy>` 声明aspect，也可以编程式创建advise对象的代理。有关Spring AOP API的全部细节，请参阅下一章。 在这里，我们想要关注的是使用@Aspectj切面自动创建代理的能力。 

`org.springframework.aop.aspectj.annotation.AspectJProxyFactory` 类可以为被一个或多个@AspectJ aspects advise的目标对象创建代理。这个类的基本用法非常简单，如下所示。请参阅javadocs以获得完整的信息。 

```
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```

###  5.8. Using AspectJ with Spring applications

这一章中至今我们讨论的所有内容都是纯Spring AOP。在本节中，我们将讨论如何使用AspectJ编译器/weaver替代AOP，或者是作为额外。如果您的需求超出了Spring AOP提供的功能。 

Spring附带一个小的AspectJ切面库，在你的发行版中，它是独立于spring的。为了使用其中的各个方面，您需要将其添加到类路径中。  [Using AspectJ to dependency inject domain objects with Spring](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-atconfigurable) and [Other Spring aspects for AspectJ](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-ajlib-other)  讨论这个库的内容以及如何使用它 。 [Configuring AspectJ aspects using Spring IoC](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-configure)  讨论了如何依赖注入使用AspectJ compiler编织的 AspectJ aspect。最后，[Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) 提供了使用AspectJ的Spring应用程序加载时编织的介绍。 

####  5.8.1. Using AspectJ to dependency inject domain objects with Spring

Spring容器实例化和配置在应用程序上下文中定义的bean。也可以要求bean工厂配置一个预先存在的对象，对象被给出了bean定义的name包含要使用的配置。`spring-aspects.jar`  包含一个annotation-driven aspect，利用这种能力来允许*任何对象*的依赖性注入。支持的目的是用于在任何容器的控制之外创建的对象。domain对象通常属于这一类，因为它们通常是使用new操作符以编程方式创建，或者由一个ORM工具作为数据库查询的结果。 

 `@Configurable`注解将一个类标记为对Spring-driven configuration 是合格的。在最简单的情况下，它可以这样注解： 

```
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable
public class Account {
    // ...
}
```

当以这种方式作为标记接口时，Spring将配置带注释类型的新实例 （Account），使用与完全限定类名（`com.xyz.myapp.domain.Account`）相同的name的 bean定义（通常是prototype-scoped ）。因为bean的默认名称是其类型的完全限定名称，声明prototype定义的一种方便的方法是简单地忽略`id`属性： 

```
<bean class="com.xyz.myapp.domain.Account" scope="prototype">
    <property name="fundsTransferService" ref="fundsTransferService"/>
</bean>
```

如果您想显式地指定要使用的prototype  bean定义的名称，你可以直接在注释中这样做： 

```
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable("account")
public class Account {
    // ...
}
```

Spring现在将寻找一个名为“account”的bean定义，并将其用作配置新`Account` 实例的定义。 

您还可以使用自动连接来避免必须指定专门的bean定义。 要让Spring应用自动装配，使用 `@Configurable` 注解的`autowire`  属性：指定 `@Configurable(autowire=Autowire.BY_TYPE)` 或者`@Configurable(autowire=Autowire.BY_NAME`  分别通过类型和name。作为替代方案，自从Spring 2.5，最好对于 `@Configurable ` beans 指定显式，注解驱动的依赖注入，使用 `@Autowired` or `@Inject`  在field或方法级别上（see [Annotation-based container configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-annotation-config) for further details ）。

最后你可以用`dependencyCheck` 属性，在新创建和配置的对象上检查对象引用（例如：`@Configurable(autowire=Autowire.BY_NAME,dependencyCheck=true)` ）。如果这个属性被set true，Spring将在配置之后验证所有已经设置好了的属性（不是原语或集合）。 

当然，使用注释在其本身并没有什么作用。 是在`spring-aspects.jar`  的`AnnotationBeanConfigurerAspect`  对注解起作用。aspect 本质上说，“从一个带有 `@Configurable`的新对象的初始化返回后，根据注释的属性配置新的Spring创建的对象 “。在这种context中，初始化指的是新实例化的对象（例如，用new操作符实例化的对象），以及正在反序列化的`Serializable` 对象（例如，通过 [readResolve()](https://docs.oracle.com/javase/6/docs/api/java/io/Serializable.html)）。 

**上面一段的关键词是“本质上”。 在大多数情况下，“从一个新对象的初始化返回之后”的确切语义将会很好...在这种情况下，“初始化”意味着在对象被构造之后将注入依赖项 ——这意味着依赖关系将无法在类的构造函数体中使用。 如果你想要在构造函数执行之前注入依赖项，因此可以在构造函数的主体中使用 ，然后你需要在`@Configurable` 声明上定义`@Configurable(preConstruction=true)`**

**You can find out more information about the language semantics of the various pointcut types in AspectJ [in this appendix](https://www.eclipse.org/aspectj/doc/next/progguide/semantics-joinPoints.html) of the [AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/next/progguide/index.html).** 

为了使其工作，带注释的类型必须与AspectJ weaver一起编织 ——你可以使用构建时Ant或Maven任务来完成这项工作 （see for example the [AspectJ Development Environment Guide](https://www.eclipse.org/aspectj/doc/released/devguide/antTasks.html) ）或使用load-time weaving （see [Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) ）。`AnnotationBeanConfigurerAspect` 本身需要由Spring配置（为了获得用于配置新对象的bean factory的引用）。如果您使用的是基于Java的配置 ：

```
@Configuration
@EnableSpringConfigured
public class AppConfig {

}
```

如果xml，Spring context定义了 [`context` namespace](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-schemas-context) 名称空间：

```
<context:spring-configured/>
```

`@Configurable` 对象的实例在aspect被配置之前被创建，结果是消息发送到调试日志，并且没有对象的配置发生。一个例子可能是Spring配置中的bean，它在Spring初始化时创建domain对象 。在这种情况你可以用"depends-on" bean attribute 手动指定bean依赖于aspect配置。

```
<bean id="myService"
        class="com.xzy.myapp.service.MyService"
        depends-on="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect">

    <!-- ... -->

</bean>
```

不要通过bean configurer aspect 激活`@Configurable` 除非你真的想在运行时依赖它的语义 。特别是，确保不要在常规容器里的beans的类上添加`@Configurable` ：否则，你会得到双重初始化，一次通过容器，一次通过aspect。 

##### Unit testing @Configurable objects

 `@Configurable` 支持的一个目的是没有与硬编码查找相关的困难，domain objects的独立单元测试。如果 `@Configurable` 类型没有被AspectJ 织入，在单元测试期间，注释没有效果。你可以简单地在测试对象中设置mock or stub property属性引用，然后正常进行。如果`@Configurable` 被AspectJ 织入，你也可以在容器外正常 unit test ，但是你会每次构建`@Configurable`  对象时看到warning信息表明它还没有被Spring配置。 

##### Working with multiple application contexts

`AnnotationBeanConfigurerAspect`  用于实现 `@Configurable`  支持，是 一个单例AspectJ  aspect。singleton aspect 的scope和 `static`  成员的scope相同，也就是说，每个类加载器都有一个aspect实例，它定义了类型。 这意味着，如果您在同一个类加载器层次结构中定义多个应用程序上下文，您需要考虑在何处定义 `@EnableSpringConfigured` bean以及何处放置 `spring-aspects.jar` 到classpath。 

考虑一个典型的Spring web应用程序配置，它具有一个共享的parent应用程序上下文，定义公共业务服务和支持它们所需的一切 ，每个servlet的一个子应用程序上下文包含特定于该servlet的定义。所有这些上下文都将在相同的类加载器层次结构中共存， `AnnotationBeanConfigurerAspect` 只能引用其中一个。这种情况我们建议`@EnableSpringConfigured`定义在shared (parent) application context ：这可能定义了您希望将其注入领域对象的service。结果是您不能用对在子（servlet特定）上下文中定义的bean的引用来配置领域对象 ，通过使用@Configurable机制（可能不是你想要做的事 ）。

在同一个容器中部署多个web应用程序时，保证每个web应用使用它自己的类加载器加载`spring-aspects.jar`  里的types（例如，by placing `spring-aspects.jar` in `'WEB-INF/lib'` ）。如果`pring-aspects.jar` 只被添加到容器范围classpath（因此被父容器加载），所有的web应用程序都将共享同一个切面实例，这可能不是您想要的。

####  5.8.2. Other Spring aspects for AspectJ

除了 `@Configurable`  切面，`spring-aspects.jar` 包含了一个AspectJ aspect ，对 `@Transactional`  注解的类型或方法驱动Spring’s transaction management 。这主要是为那些希望在Spring容器之外使用Spring框架的事务支持的用户准备的。 

解释 `@Transactional` 注解的aspect是`AnnotationTransactionAspect`。当使用这个方面时，您必须注解实现类（和/或该类内的方法），而不是该类实现的接口（如果有的话）。 AspectJ遵循Java的规则，即接口上的注解不是继承的。 

一个类上的`@Transactional`注解指定了该类中任何public操作的执行的默认事务语义。 

一个方法上的`@Transactional` 覆盖类给出的默认事务语义 。任何可见性的方法都可以被注解，包括私有方法。 直接注释非public方法是获取这些方法执行的事务界定的唯一方法。

自Spring 4.2，`spring-aspects`  提供了一个类似的aspect，提供了和 `javax.transaction.Transactional` 一样的特性。更多信息查看 `JtaAnnotationTransactionAspect`  。

对于AspectJ 编程者，想使用Spring配置和事务管理支持，但不想（或不能）使用注解，`spring-aspects.jar`  也包含 `abstract` aspects，你可以扩展他提供自己的pointcut定义。看`AbstractBeanConfigurerAspect` and `AbstractTransactionAspect`  aspects 更多信息。作为一个例子，下面的摘录展示了如何编写一个方面来配置在领域模型中定义的所有对象实例，使用与完全限定类名匹配的原型bean定义： 

```
public aspect DomainObjectConfiguration extends AbstractBeanConfigurerAspect {

    public DomainObjectConfiguration() {
        setBeanWiringInfoResolver(new ClassNameBeanWiringInfoResolver());
    }

    // the creation of a new bean (any object in the domain model)
    protected pointcut beanCreation(Object beanInstance) :
        initialization(new(..)) &&
        SystemArchitecture.inDomainModel() &&
        this(beanInstance);

}
```

#### 5.8.3. Configuring AspectJ aspects using Spring IoC

当Spring应用使用AspectJ aspects ，使用Spring来配置这些方面是很自然的事情。 AspectJ运行时本身负责aspect的创建， 通过Spring来配置AspectJ创建方面的方法依赖于由aspect使用的AspectJ实例化模型（the `per-xxx` clause ）。

大多数AspectJ切面是单例切面的。这些方面的配置非常简单： 简单地创建一个引用aspect类型的bean定义，包含bean属性'factory-method="aspectOf"' 。这确保了Spring通过询问AspectJ而不是试图创建实例本身来获得方面实例。 例如：

```
<bean id="profiler" class="com.xyz.profiler.Profiler"
        factory-method="aspectOf">

    <property name="profilingStrategy" ref="jamonProfilingStrategy"/>
</bean>
```

Non-singleton aspects 更难配置：然而这是可能的，通过创建prototype bean定义，使用来自`spring-aspects.jar`  的`@Configurable`  支持，来配置切面实例，只要他们在AspectJ runtime 被创建。

如果你有些 @AspectJ aspects 想用 AspectJ  编织（例如，为domain模型类型使用加载时编织 ）并且其他@AspectJ aspects 你想通过Spring AOP使用，这些aspects都用Spring配置，那么你需要告诉Spring AOP @AspectJ autoproxying support  哪个配置里定义的@AspectJ aspects  的子集应该被用于autoproxying 。你可以使用一个或多个`<include/>`  在 `<aop:aspectj-autoproxy/>`  声明内。每个 `<include/>` 指定一个name pattern，只有具有至少一种模式匹配的名称的bean才会用于Spring AOP autoproxy 配置： 

```
<aop:aspectj-autoproxy>
    <aop:include name="thisBean"/>
    <aop:include name="thatBean"/>
</aop:aspectj-autoproxy>
```

不要被`<aop:aspectj-autoproxy/>` 误导：用这个导致Spring AOP代理的创建 。@AspectJ style of aspect declaration 只是在这里用， AspectJ  runtime不涉及。

#### 5.8.4. Load-time weaving with AspectJ in the Spring Framework

Load-time weaving (LTW) 指的是把AspectJ aspects 织入到应用class文件中，当它们被加载到Java virtual machine (JVM) 的时候的一个过程。本节的重点是在Spring框架的特定上下文中配置和使用LTW： 不过，本节并不是对LTW的介绍。 关于LTW的详细信息，并使用AspectJ配置LTW （Spring没涉及），see the [LTW section of the AspectJ Development Environment Guide](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html). 

Spring框架给AspectJ LTW带来的增值是对编织过程的更细粒度的控制。  'Vanilla' AspectJ LTW 使用Java (5+)  代理实现的，在启动JVM时，通过指定VM参数来打开它。因此，它是一个jvm范围的设置， 但通常有点太粗糙了。支持spring的LTW使您能够在每个类加载器的基础上切换LTW，很明显，它更细粒度，在“单jvm多应用程序”环境中更有意义 （例如在典型的应用程序服务器环境中发现的 ）。

另外，在 [in certain environments](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw-environments) ，这种支持支持加载时编织，而不需要对应用服务器的启动脚本进行任何修改，这将需要添加 `-javaagent:path/to/aspectjweaver.jar`  或`-javaagent:path/to/org.springframework.instrument-{version}.jar` （以前叫spring-agent.jar ）。开发人员只需修改一个或多个构成应用程序上下文的文件，以支持加载时编织，而不是依赖于通常负责部署配置的管理员，比如启动脚本。 

既然sales pitch完了，让我们先来看看使用Spring的AspectJ LTW的一个简单示例，接下来的示例中介绍了关于元素的详细细节。对于一个完整的示例，请参阅 [Petclinic sample application](https://github.com/spring-projects/spring-petclinic) 。

#####  A first example

让我们假设您是一个应用程序开发人员，他的任务是诊断系统中某些性能问题的原因。 不是打开一个分析工具，我们要做的是切换一个简单的剖析aspect，这将使我们能够快速获得一些性能指标 ，这样我们就可以在之后立即应用一个更细粒度的分析工具到那个特定的区域。 

这里展示的示例使用XML样式配置，也可以用 @AspectJ with [Java Configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-java) 配置。特别是`@EnableLoadTimeWeaving` 注解可以用来代替`<context:load-time-weaver/>` (see [below](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw-spring) for details)。

这是性能分析切面。没有什么太花哨，只是一个quick-and-dirty 基于时间的分析器使用 @AspectJ-style 声明：

```
package foo;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.util.StopWatch;
import org.springframework.core.annotation.Order;

@Aspect
public class ProfilingAspect {

    @Around("methodsToBeProfiled()")
    public Object profile(ProceedingJoinPoint pjp) throws Throwable {
        StopWatch sw = new StopWatch(getClass().getSimpleName());
        try {
            sw.start(pjp.getSignature().getName());
            return pjp.proceed();
        } finally {
            sw.stop();
            System.out.println(sw.prettyPrint());
        }
    }

    @Pointcut("execution(public * foo..*.*(..))")
    public void methodsToBeProfiled(){}
}
```

我们也需要创建 `META-INF/aop.xml` ，为了通知AspectJ weaver我们想要将我们的剖析aspect编织进我们的类中。 文件约定是标准AspectJ约定。

```
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>

    <weaver>
        <!-- only weave classes in our application-specific packages -->
        <include within="foo.*"/>
    </weaver>

    <aspects>
        <!-- weave in just this aspect -->
        <aspect name="foo.ProfilingAspect"/>
    </aspects>

</aspectj>
```

现在到了Spring-特定配置的部分。我们需要配置一个`LoadTimeWeaver` （后面会解释，现在相信即可）。这个 load-time weaver 是基本组件，负责织入一个或多个`META-INF/aop.xml`  文件里的aspect 配置 ，好处是它不需要太多的配置， 如下所示（还有一些您可以指定的选项，但是这些选项稍后会详细介绍 ）：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- a service object; we will be profiling its methods -->
    <bean id="entitlementCalculationService"
            class="foo.StubEntitlementCalculationService"/>

    <!-- this switches on the load-time weaving -->
    <context:load-time-weaver/>
</beans>
```

所有需要的artifacts都就位了——the aspect, the `META-INF/aop.xml` file, and the Spring configuration ，让我们用`main(..)` 方法创建一个简单驱动类实践演示LTW  。

```java
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

    public static void main(String[] args) {

        ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService
            = (EntitlementCalculationService) ctx.getBean("entitlementCalculationService");

        // the profiling aspect is 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```

还有最后一件事要做。 本节的介绍说，可以有选择地在每个类加载器的基础上对LTW进行切换，这是真的。但是，仅在这个例子中，我们将使用Java代理人（提供Spring）来切换LTW。 这是我们将用来运行上面的`Main`类的命令行： 

```
java -javaagent:C:/projects/foo/lib/global/spring-instrument.jar foo.Main
```

`-javaagent` 是一个标志，指定和启用 [agents to instrument programs running on the JVM](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html)。Spring框架附带了这样的代理， `InstrumentationSavingAgent`在`spring-instrument.jar`  包，这是在上面的例子中作为-javaagent参数的值提供的。 

主程序执行的输出如下所示。 我引入了 `Thread.sleep(..)` 到 `calculateEntitlement()`  实现，所以分析器能获取到一些东西而不是0ms，01234ms不是引入aop的开销。

```
Calculating entitlement

StopWatch 'ProfilingAspect': running time (millis) = 1234
------ ----- ----------------------------
ms     %     Task name
------ ----- ----------------------------
01234  100%  calculateEntitlement
```

因为LTW是使用成熟的AspectJ进行的，我们不仅仅局限于为beans提供advice；下面 `Main` 程序的轻微变化导致结果相同。

```
package foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Main {

    public static void main(String[] args) {

        new ClassPathXmlApplicationContext("beans.xml", Main.class);

        EntitlementCalculationService entitlementCalculationService =
            new StubEntitlementCalculationService();

        // the profiling aspect will be 'woven' around this method execution
        entitlementCalculationService.calculateEntitlement();
    }
}
```

注意，在上面的程序中我们只是简单地引导Spring容器，然后在Spring外new了一个`StubEntitlementCalculationService`实例——profiling advice还是被织入了。

无可否认，这个例子过于简单...然而，在上面的示例中介绍了Spring LTW支持的基础知识，本节的其余部分将详细解释每种配置和用法背后的“原因” 。

**本例的 `ProfilingAspect` 可能很简单，但很有用。这是开发人员在开发过程中可以使用的一个很好的例子（当然） ，然后很容易地将应用程序的构建排除在UAT或生产环境中。** 

#####  Aspects

在LTW中使用的aspects必须是AspectJ aspects。它们可以用AspectJ语言本身编写或者你可以用@AspectJ风格来写你的方面 这意味着您的aspects同时是有效的AspectJ和Spring AOP aspects 。  此外，编译好的aspect类需要在类路径中可用。

##### 'META-INF/aop.xml'

AspectJ LTW infrastructure 使用一个或者多个`META-INF/aop.xml` 文件来配置，是在classpath上（要么直接，要么在jar）

该文件的结构和内容在主要的AspectJ参考文档中详细介绍，and the interested reader is[referred to that resource](https://www.eclipse.org/aspectj/doc/released/devguide/ltw-configuration.html)。（我很欣赏这部分很简短，但是aop。xml文件是百分之百的AspectJ——没有特定于spring的信息或语义，因此没有额外的价值，因此我可以贡献任何一个结果 ）。所以，与其重新讨论AspectJ开发人员编写的令人满意的部分，我只是在指导。） 

##### Required libraries (JARS)

至少您需要以下库来使用Spring Framework对AspectJ LTW的支持 

- `spring-aop.jar` (version 2.5 or later, plus all mandatory dependencies)
- `aspectjweaver.jar` (version 1.6.8 or later)

If you are using the [Spring-provided agent to enable instrumentation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw-environment-generic), you will also need: 

- `spring-instrument.jar`

##### Spring configuration

Spring LTW支持的关键组件是`LoadTimeWeaver`接口（在`org.springframework.instrument.classloading`  包），Spring有很多它的实现。一个`LoadTimeWeaver`负责在运行时添加一个或者多个`java.lang.instrument.ClassFileTransformers` 到`ClassLoader` ，打开了到所有关注应用程序的大门，其中一个恰好是切面的LTW。 

**如果您不熟悉运行时类文件转换的想法，读`java.lang.instrument`  api doc for continuing。这并不是一项艰巨的任务，因为 —文档很小。当您阅读本节时，关键的接口和类至少会在您前面列出。** 

对特定application context配置 `LoadTimeWeaver` 很简单。（请注意，您几乎肯定要将`ApplicationContext`用作Spring容器——通常情况下，`BeanFactory`将不够用，因为LTW支持使用`BeanFactoryPostProcessors`处理器 ）

要启用Spring框架的LTW支持 ，配置 `LoadTimeWeaver` ，使用`@EnableLoadTimeWeaving` 注解做：

```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {

}
```

或者，如果您喜欢基于XML的配置，使用 `<context:load-time-weaver/>` 。注意，元素是在`context`  名称空间中定义的。 

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver/>

</beans>
```

上面的配置将自动为您定义和注册许多特定于ltw特有的基础设施bean，such as a `LoadTimeWeaver` and an `AspectJWeavingEnabler`。默认 `LoadTimeWeaver`  是 `DefaultContextLoadTimeWeaver` 类，它试图修饰自动检测的`LoadTimeWeaver `：将“自动检测”的LoadTimeWeaver的确切类型取决于您的运行时环境（在下表中总结 ）。

*Table 13. DefaultContextLoadTimeWeaver LoadTimeWeavers*

| Runtime Environment                                          | `LoadTimeWeaver` implementation |
| ------------------------------------------------------------ | ------------------------------- |
| Running in Oracle’s [WebLogic](http://www.oracle.com/technetwork/middleware/weblogic/overview/index-085209.html) | `WebLogicLoadTimeWeaver`        |
| Running in Oracle’s [GlassFish](http://glassfish.dev.java.net/) | `GlassFishLoadTimeWeaver`       |
| Running in [Apache Tomcat](https://tomcat.apache.org/)       | `TomcatLoadTimeWeaver`          |
| Running in Red Hat’s [JBoss AS](http://www.jboss.org/jbossas/) or [WildFly](http://www.wildfly.org/) | `JBossLoadTimeWeaver`           |
| Running in IBM’s [WebSphere](https://www-01.ibm.com/software/webservers/appserv/was/) | `WebSphereLoadTimeWeaver`       |
| JVM started with Spring `InstrumentationSavingAgent` *(java -javaagent:path/to/spring-instrument.jar)* | `InstrumentationLoadTimeWeaver` |
| Fallback, expecting the underlying ClassLoader to follow common conventions (e.g. applicable to `TomcatInstrumentableClassLoader` and [Resin](http://www.caucho.com/)) | `ReflectiveLoadTimeWeaver`      |

注意，这些只是在使用`DefaultContextLoadTimeWeaver`时自动检测到的`loadtimeweaver `：当然也可以指定自己想用的`LoadTimeWeaver`  实现。

实现`LoadTimeWeavingConfigurer` 接口，重新 `getLoadTimeWeaver()` 方法，通过Java配置指定`LoadTimeWeaver`  ：

```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig implements LoadTimeWeavingConfigurer {

    @Override
    public LoadTimeWeaver getLoadTimeWeaver() {
        return new ReflectiveLoadTimeWeaver();
    }
}
```

如果你使用xml，指定`<context:load-time-weaver/>`  的 `weaver-class`属性为全限定类名。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:load-time-weaver
            weaver-class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>

</beans>
```

配置注册的`LoadTimeWeaver`  可以稍后从Spring 容器重新得到，使用name  `loadTimeWeaver` 。记住 `LoadTimeWeaver`  只是一个对 Spring’s LTW infrastructure的机制，添加一个或者多个 `ClassFileTransformers`。实际做LTW的 `ClassFileTransformer`  是`ClassPreProcessorAgentAdapter`  （来自 `org.aspectj.weaver.loadtime`  包）。看类级别的`ClassPreProcessorAgentAdapter`  javadoc  for further details ，因为编织实际上是如何影响的，超出了本节的范围。 

剩下要讨论的配置的最后一个属性：`aspectjWeaving` 属性（或xml的`aspectj-weaving`  ）。这是一个简单的属性，它控制LTW是否启用；就这么简单。它接受下面总结的三个可能值中的一个，如果属性不存在，默认值将是 `autodetect` 。 

*Table 14. AspectJ weaving attribute values*

| Annotation Value | XML Value    | Explanation                                                  |
| ---------------- | ------------ | ------------------------------------------------------------ |
| `ENABLED`        | `on`         | AspectJ weaving is on, and aspects will be woven at load-time as appropriate. |
| `DISABLED`       | `off`        | LTW is off… no aspect will be woven at load-time.            |
| `AUTODETECT`     | `autodetect` | If the Spring LTW infrastructure can find at least one `META-INF/aop.xml` file, then AspectJ weaving is on, else it is off. This is the default value. |

#####  Environment-specific configuration

最后一节包含了在应用程序服务器和web容器等环境中使用Spring LTW支持时需要的任何附加设置和配置。 

###### Tomcat 

历史的， [Apache Tomcat](https://tomcat.apache.org/)的默认类加载器不支持class transformation ，这就是Spring提供了一个增强的实现来满足这种需求的原因。 叫`TomcatInstrumentableClassLoader`，works on Tomcat 6.0+ 。

不要在Tomcat 8.0+定义 `TomcatInstrumentableClassLoader`  了。而是让Spring自动使用Tomcat的新的 `InstrumentableClassLoader` ，通过 `TomcatLoadTimeWeaver` 策略。

如果你仍然需要 `TomcatInstrumentableClassLoader` ，可以为每个web application 单独注册：



- 复制`org.springframework.instrument.tomcat.jar` 到*$CATALINA_HOME*/lib ， *$CATALINA_HOME* 是安装目录。

- 指示Tomcat使用自定义类装入器 （而不是默认的）通过编辑web application context文件。

  ```
  <Context path="/myWebApp" docBase="/my/webApp/location">
      <Loader
          loaderClass="org.springframework.instrument.classloading.tomcat.TomcatInstrumentableClassLoader"/>
  </Context>
  ```

Apache Tomcat (6.0+) 支持几个context locations ：

-  *$CATALINA_HOME/conf/server.xml* 
- 默认context配置*$CATALINA_HOME/conf/context.xml*  影响全部部署的应用
- 可以在服务器端部署的每个web应用程序配置 *$CATALINA_HOME/conf/[enginename]/[hostname]/[webapp]-context.xml*  或者嵌入到web应用程序存档中 *META-INF/context.xml* 

为了提高效率，建议使用嵌入的每个web应用程序配置样式，因为它只会影响使用定制类装入器的应用程序，并且不需要对服务器配置进行任何更改。 See the Tomcat 6.0.x [documentation](https://tomcat.apache.org/tomcat-6.0-doc/config/context.html) for more details about available context locations. 

或者，考虑使用spring提供的通用VM代理 ，要在Tomcat的启动脚本中指定（见上文）。 这将使所有部署的web应用程序都可以使用插装，无论它们在什么类加载器发生。 

###### WebLogic, WebSphere, Resin, GlassFish, JBoss 

。。。

###### Generic Java applications 

在不支持或不支持现有LoadTimeWeaver实现的环境中，需要class instrumentation，JDK代理可以是唯一的解决方案。这样的例子，Spring提供`InstrumentationLoadTimeWeaver`，需要一个 Spring-specific (but very general) VM agent ，`org.springframework.instrument-{version}.jar` (previously named `spring-agent.jar`)。

要使用它，您必须使用Spring agent启动虚拟机，通过提供以下JVM选项： 

注意，这需要修改VM启动脚本，这可能会阻止您在application server环境中使用它（取决于您的操作策略）。另外，JDK代理将会对整个VM进行测试，这可能会被证明是昂贵的。 

出于性能方面的原因，建议只有在目标环境（如Jetty）没有（或不支持）专用LTW时才使用这种配置。 

### 5.9. Further Resources

More information on AspectJ can be found on the [AspectJ website](https://www.eclipse.org/aspectj). 

The book *Eclipse AspectJ* by Adrian Colyer et. al. (Addison-Wesley, 2005) provides a comprehensive introduction and reference for the AspectJ language. 

The book *AspectJ in Action, Second Edition* by Ramnivas Laddad (Manning, 2009) comes highly recommended; the focus of the book is on AspectJ, but a lot of general AOP themes are explored (in some depth). 

## 6. Spring AOP APIs

###  6.1. Introduction

上章讲了使用 @AspectJ and schema-based aspect描述Spring对AOP的支持。在本章中，我们将讨论底层Spring AOP api和在Spring 1.2应用程序中通常使用的AOP支持。 对于新的应用程序，我们建议使用前一章中描述的Spring 2.0和后来的AOP支持，但是当使用现有的应用程序，或者在阅读书籍和文章时，您可能会遇到Spring 1.2风格的例子 。Spring 5仍然向后兼容Spring 1.2，在这一章中描述的一切都在Spring 5中得到了完全的支持。 

###  6.2. Pointcut API in Spring

让我们来看看Spring如何处理关键的切入点概念。 

#### 6.2.1. Concepts

Spring的切入点模型支持切入点重用，独立于通知类型。使用相同的切入点来针对不同的advice是可能的。 `org.springframework.aop.Pointcut` 接口是核心接口，用于target advices 到特定类和方法。完整接口如下：

```
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}
```

将`Pointcut`接口分割为两个部分，这允许重用类和方法匹配部件和细粒度成分操作（例如和另一个方法的matcher执行"union" ）。

`ClassFilter` 接口用来将切入点限制在给定的目标类集合中 。如果 `matches()`  返回总是true，所有目标类被匹配：

```
public interface ClassFilter {

    boolean matches(Class clazz);
}
```

`MethodMatcher`  接口通常是更重要的。如下：

```
public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class targetClass, Object[] args);
}
```

`matches(Method, Class)`  方法用于测试这个切入点是否匹配目标类上给定的方法。这个评估可以在创建AOP代理时执行，来避免对每种方法调用进行测试。如果两个参数的matches方法对给定的方法返回true，`isRuntime()`  对 MethodMatcher 返回true，三个参数的matches会在每一个方法调用上被调用。这使得切入点能够在目标通知执行之前立即查看传递给方法调用的参数。 

大多MethodMatchers 是static的，意味着它们的`isRuntime()`  是false。在这个例子，三个参数的matches  方法永远都不会被调用。

**如果可能，尝试使pointcut保持静态， 允许AOP框架在创建AOP代理时缓存pointcut评估的结果。** 

#### 6.2.2. Operations on pointcuts

Spring支持的切入点操作： notably, *union* and *intersection*. 

- Union 意味着pointcut匹配的任意一个方法
- Intersection 意味着两个pointcut都匹配到的方法
- Union 通常更有用
- pointcuts可以在*org.springframework.aop.support.Pointcuts* 类使用静态方法组合，或者使用相同包的*ComposablePointcut*  类。然而，使用AspectJ 切点表达式通常是一个更简单的方法。

#### 6.2.3. AspectJ expression pointcuts

自2.0，Spring使用的最重要的pointcut类型是`org.springframework.aop.aspectj.AspectJExpressionPointcut`。这是使用AspectJ提供库的pointcut，来转换AspectJ切点表达式字符串。

请参阅前一章，讨论支持的AspectJ切入点原语。 

#### 6.2.4. Convenience pointcut implementations

Spring提供了几个方便的pointcut实现。 一些开箱即用；其他的目的是在特定于应用程序的切入点中被子类继承。

##### Static pointcuts

静态切入点是基于方法和目标类的，不能考虑方法的参数。静态切入点对于大多数用法来说是足够的，也是最好的。 Spring只对静态切入点进行一次评估是可能的，当一个方法被首次调用时： 不需要在每次方法调用时再次评估切入点。 

让我们考虑Spring中包含的静态切入点实现。 

###### Regular expression pointcuts 

指定静态切入点的一个明显方法是正则表达式。除了Spring之外，还有几个AOP框架使这成为可能。 `org.springframework.aop.support.JdkRegexpMethodPointcut`是一个通用正则表达式切入点，使用JDK中的正则表达式支持。

使用`JdkRegexpMethodPointcut` 类，你可以提供 a list of pattern Strings。如果这些是一个匹配，切入点将被评估为true 。（所以结果就是这些切入点的结合 ）

用法如下： 

```
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

Spring提供了一个方便的类，`RegexpMethodPointcutAdvisor`，运行我们也可以引用一个Advice（记住一个Advice 可以是一个拦截器， before advice, throws advice 等等）。在幕后，Spring会使用`JdkRegexpMethodPointcut`。使用`RegexpMethodPointcutAdvisor` 简化了wiring，正如一个bean同时封装了切入点和advice一样，如下所示：

```
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

*RegexpMethodPointcutAdvisor*可以被任何advice类型使用。

###### Attribute-driven pointcuts 

一种重要的静态切入点是元数据驱动的切入点。 它使用元数据属性的值： 一般来说,source级别的元数据。 

##### Dynamic pointcuts

动态pointcuts 比静态pointcuts 评估更昂贵。他们考虑了方法参数，以及静态信息。 这意味着必须对每种方法调用进行评估；结果不能被缓存，因为参数会有所不同。 

主要例子是`control flow` pointcut。

###### Control flow pointcuts 

Spring 的 control flow pointcuts 概念上类似于AspectJ *cflow* pointcuts ，虽然没那么强大（现在还没办法指定在另一个pointcut匹配的 join point 下执行一个pointcut ）。一个control flow pointcut 匹配当前调用堆栈。例如，如果一个在`com.mycompany.web` 包的方法或者一个 `SomeCaller`  类调用连接点，它可能会触发。使用 `org.springframework.aop.support.ControlFlowPointcut`指定Control flow pointcuts  。

**在运行时，控制流切入点评估要比其他动态切入点更昂贵。 在Java 1.4中，成本大约是其他动态切入点的5倍。** 

####  6.2.5. Pointcut superclasses

Spring提供了有用的切入点超类来帮助您实现自己的切入点。 

因为静态切入点是最有用的，你可能继承StaticMethodMatcherPointcut，如下。这只需要实现一个抽象方法 （虽然也可以重写其他方法覆盖默认行为）：

```
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }
}
```

动态切入点也有超类。 

您可以在Spring 1.0 RC2和以上的任何advice类型中使用定制切入点。

####  6.2.6. Custom pointcuts

因为Spring中pointcuts 是Java类，而不是语言特性（如AspectJ） ，声明自定义切入点是可能的，无论静态还动态。Spring中的自定义切入点可能是任意复杂的。 但是，如果可能的话，建议使用AspectJ切入点表达式语言。 

**Spring的后续版本可能为JAC提供的“语义切入点”提供支持：例如，“所有在目标对象中改变实例变量的方法”。** 

###  6.3. Advice API in Spring

现在让我们看看Spring AOP是如何处理advice的。 

####  6.3.1. Advice lifecycles

每个advice是一个Spring bean。一个advice实例可以跨所有被advised的对象共享，或者对每个被advised的对象都是唯一的。这个和*per-class* or *per-instance*对应。

Per-class advice 最常用。它适用于诸如事务advisors之类的通用建议。 这些不依赖于被代理的对象的状态或添加新的状态; 他们只是按照方法和参数行事。

Per-instance advice 适用于introductions ，来支持混合。在这种情况下，建议为proxied对象添加状态。 

在同一个Spring AOP代理可以混用shared and per-instance advice 。

#### 6.3.2. Advice types in Spring

Spring提供了几种开箱即用的advice类型，并且可以扩展支持任意 advice类型。让我们看一下基本概念和标准advice类型。

#####  Interception around advice

Spring里最基本的advice类型是 *interception around advice*。

Spring遵循AOP联盟接口，使用方法拦截来提供around advice。MethodInterceptors  实现around advice 应该实现下面的接口：

```
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

`invoke()`方法的参数`MethodInvocation`  暴露了被调用的方法；目标 join point ；the AOP proxy ；和方法的参数。 `invoke()` 方法应该返回调用的结果：join point的返回值。

简单的 `MethodInterceptor`  实现如下：

```
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```

**注意 MethodInvocation的`proceed()` 方法的调用。这把拦截链降到 join point。大多数拦截器都将调用这个方法，并返回返回值。然而，一个MethodInterceptor ，像任何的 around advice，可以返回一个不同的值或者抛出异常而不是调用方法处理。然而，你不想在没有正当理由的情况下这么做！** 

 **MethodInterceptors 对其他AOP Alliance-compliant AOP 实现提供互通。本节剩下讨论的其他advice 类型实现了common AOP概念，但是是用Spring-specific的方式。虽然使用最具体的advice类型是有好处的，坚持使用 MethodInterceptor around advice 如果您想要在另一个AOP框架中运行aspect。 注意，切入点目前还不能在框架之间进行互操作，AOP联盟目前还没有定义切入点接口。** 

##### Before advice

一个简单的advice 类型是*before advice*。它不需要一个`MethodInvocation` 对象，因为只在进入方法前被调用。

before advice 的主要优势是不需要调用 `proceed()` 方法，因此没有能力inadvertently failing to proceed down the interceptor chain。

 `MethodBeforeAdvice`  接口如下。（Spring API 设计allow for field before advice， 尽管通常的对象适用于field 拦截，Spring不太可能实现 它）。

```
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;
}
```

注意返回类似是`void`。Before advice可以在 join point执行前插入自定义行为，但不能改变返回值。如果一个 before advice抛出一个异常，它将终止拦截链的进一步执行。这个异常会传播到拦截链。如果它是未经检查的，活在在被调用方法的签名上，它将直接传递给客户端；否则，它将被AOP代理包装在一个未经检查的异常中。 

Spring before advice 的一个例子，它计算所有的方法调用： 

```
public class CountingBeforeAdvice implements MethodBeforeAdvice {

    private int count;

    public void before(Method m, Object[] args, Object target) throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

**任何 pointcut都能用Before advice。** 

##### Throws advice

*Throws advice*会被调用，如果join point返回后 join point 抛出异常。Spring提供了typed throws advice 。注意这意味着`org.springframework.aop.ThrowsAdvice` 接口不包含任何方法：它是一个标记接口，用来识别给定对象实现一个或多个typed throws advice方法。这些应该是： 

```
afterThrowing([Method, args, target], subclassOfThrowable)
```

只有最后一个参数是必需的。 方法签名可能有一个或四个参数，取决于advice方法是否对方法和参数感兴趣。以下类是throws advice的例子。

以下advice被调用如果抛出`RemoteException`  （包括子类）：

```
public class RemoteThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }
}
```

如果抛出 `ServletException` 调用以下advice。不像上面的advice，声明了4个参数，这样它就可以访问被调用的方法、方法参数和目标对象 ：

```
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

最后的例子说明了如何在单个类中使用这两个方法 ，同时处理 `RemoteException` and `ServletException`。任何数量的throws advice方法都可以在一个类中组合。 

```
public static class CombinedThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

如果throws-advice 方法自身抛出异常，它会覆盖原始的异常 （例如，改变抛给用户的异常）。覆盖异常通常是 RuntimeException ；这与任何方法签名兼容 。然而，如果throws-advice抛出受检异常，它必须匹配目标方法声明的异常并且因此在某种程度上与特定的目标方法签名相结合。不要抛出一个未声明的检查异常，它与目标方法的签名不兼容！ 

any pointcut可以用Throws advice 

##### After Returning advice

Spring的 after returning advice 必须实现 `org.springframework.aop.AfterReturningAdvice`接口，如下：

```
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable;
}
```

一个 after returning advice必须访问返回值（但不能修改），调用方法，方法参数和target。

以下after returning advice 计数所有成功没有抛出异常的方法调用：

```
public class CountingAfterReturningAdvice implements AfterReturningAdvice {

    private int count;

    public void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

这个 advice不会改变执行路径。如果它抛出异常，会抛到拦截链而不是返回值。

**any pointcut 可以用 returning advice 。**

##### Introduction advice

Spring把introduction advice作为一种特殊种类的 interception advice 。

Introduction  需要一个`IntroductionAdvisor`和一个 `IntroductionInterceptor` ，实现以下接口：

```
public interface IntroductionInterceptor extends MethodInterceptor {

    boolean implementsInterface(Class intf);
}
```

`invoke()`  方法继承自AOP Alliance `MethodInterceptor` 接口必须实现introduction ：就是说，如果调用方法在一个 introduced 接口上， introduction interceptor 负责方法调用——他不能调用 `proceed()` 。

Introduction advice 不能被任何advice使用，因为它只适用于类，而不是方法级别。你只能通过 `IntroductionAdvisor` 使用introduction advice ，有如下方法：

```
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

    ClassFilter getClassFilter();

    void validateInterfaces() throws IllegalArgumentException;
}

public interface IntroductionInfo {

    Class[] getInterfaces();
}
```

这里没有 `MethodMatcher` ，因此没有 `Pointcut` 和introduction advice 关联。只有类过滤是符合逻辑的。 

 `getInterfaces()` 方法返回被advisor introduce的接口。

 `validateInterfaces()`  方法内部使用，查看引入的接口是否可以由配置的 `IntroductionInterceptor`实现。 

让我们看一个来自Spring test套件的简单示例。让我们假设我们想要向一个或多个对象引入以下接口： 

```
public interface Lockable {
    void lock();
    void unlock();
    boolean locked();
}
```

This illustrates a *mixin*. 我们希望能转换advised objects 为Lockable ，不管是什么类型的对象，并且调用lock和unlock方法。如果我们调用lock()  方法，我们希望所有setter方法抛出 `LockedException` 。因此，我们可以添加一个aspect，它提供了使对象不可变的能力，并且使这些对象并不知道：这是aop一个很好的例子。

首席，我们需要一个 `IntroductionInterceptor`  做繁重的工作。在此例，我们继承`org.springframework.aop.support.DelegatingIntroductionInterceptor` 便利类。我们可以直接实现IntroductionInterceptor，但大多情况下使用 `DelegatingIntroductionInterceptor` 是最好的。

 `DelegatingIntroductionInterceptor` 被设计用来委托一个introduction 到被 introduced 接口的具体实现，隐藏拦截的使用来做到。委托可以用构造函数参数被设置为任何对象；默认委托（当使用无参构造函数时）是这样的。在下面的例子中 ，委托是`DelegatingIntroductionInterceptor`的子类 `LockMixin` 。给定一个委托（默认情况下它自身）， 一个 `DelegatingIntroductionInterceptor` 实例查找由委托实现的所有接口 （而不是IntroductionInterceptor ），并将支持对其中任何一个的introductions。像 `LockMixin`  这样的子类调用`suppressInterface(Class intf)` 方法来suppress 不应该被暴露的接口是可能的。然而，无论一个`IntroductionInterceptor`准备支持多少接口， 使用的 `IntroductionAdvisor` 会控制哪个接口被实际暴露。一个introduced接口将隐藏任何相同目标接口的任何实现。

`LockMixin`  继承`DelegatingIntroductionInterceptor`  ，且实现 `Lockable`  自身。superclass 自动picks up ，Lockable可为introduction支持，所以我们不需要指定他。我们可以以这种方式引入任意数量的接口。 

注意实例变量 `locked` 的使用。这实际上增加了目标对象中持有的额外状态。 

```
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

    private boolean locked;

    public void lock() {
        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

    public boolean locked() {
        return this.locked;
    }

    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
            throw new LockedException();
        }
        return super.invoke(invocation);
    }

}
```

通常没有必要覆盖`invoke()`方法： `DelegatingIntroductionInterceptor`  实现——如果引入了方法，就调用委托方法，否则朝join point执行——通常是足够的。在目前的情况，我们需要添加一个检查：在locked mode不能调用任何setter method。

 introduction advisor 需要的很简单。所有要做的就是保持一个 `LockMixin` 实例，并指定 introduced  接口——在这个例子就是Lockable。一个更复杂的例子可能会引用 introduction interceptor  （会被定义成prototype ）：在此例，没有`LockMixin`相关配置，所以我们简单地`new`出来。

```
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

    public LockMixinAdvisor() {
        super(new LockMixin(), Lockable.class);
    }
}
```

我们可以很简单地应用这个advisor ：不需要配置。（然而，是必要的：使用`IntroductionInterceptor` 不带有 *IntroductionAdvisor*  ）。像往常一样与介绍，advisor 必须是 per-instance，因为它是有状态的 。我们需要一个 `LockMixinAdvisor`的不同的实例，因此每个advised对象 都有`LockMixin`。advisor包含建议对象的状态的一部分。

我们可以以编程的方式应用这个advisor，使用 `Advised.addAdvisor()` 方法，或（推荐的） XML  方式，像其他的adviser一样。所有代理创建选择在下面讨论，包括"auto proxy creators,"  正确处理introductions和有状态的mixin。 

### 6.4. Advisor API in Spring

在Spring，一个Advisor是一个aspect，只包含一个和切点表达式关联的advice对象。

除了介绍的特殊情况外，任何advisor 都可以和advice一起使用。

`org.springframework.aop.support.DefaultPointcutAdvisor` 是最常用的advisor  类。例如，可以和 `MethodInterceptor`, `BeforeAdvice` or `ThrowsAdvice`一起用。

在Spring里相同 AOP proxy 可以混合advice和advisor类型。例如，你可以在一个代理配置使用 interception around advice，throws advice and before advice ：Spring将自动创建必要的拦截器链。 

### 6.5. Using the ProxyFactoryBean to create AOP proxies

如果你对你的业务对象在用spring IoC container（ApplicationContext or BeanFactory ）-并且你应该这样！-你将想使用Spring’s AOP FactoryBeans的其中之一。（记住a factory bean 引入了一个间接层，使它能够创建不同类型的对象 ）。

 **Spring AOP 也支持了使用factory beans。**

创建AOP代理的基本方式是使用*org.springframework.aop.framework.ProxyFactoryBean*。它在 pointcuts和将执行的advice上提供了完全控制，以及它们的排序。然而，如果你不需要这样的控制，有一些更简单的选择。 

####  6.5.1. Basics

 `ProxyFactoryBean`，像其他的spring `FactoryBean` 实现，引入了一个间接的层次 。如果你定义一个`name`是`foo`的`ProxyFactoryBean` ，引用`foo`的对象不是`ProxyFactoryBean` 本事，而是`ProxyFactoryBean`'s `getObject()`方法的实现。这个方法将创建一个包装目标对象的AOP代理。 

使用`ProxyFactoryBean` 或另外一个 IoC-aware类来创建AOP代理的一个最大好处是，意味着advices and pointcuts 也被IoC管理。这是个强大特性，是一些其他AOP框架难以实现的方法得以实现。例如，一个advice本身可能引用应用程序对象（除了target，在任何AOP框架中都应该可用 ），受益于依赖注入所提供的所有可插入性。 

####  6.5.2. JavaBean properties

和大多数spring提供的`FactoryBean` 实现一样，`ProxyFactoryBean`  类自身是一个JavaBean。它的属性用于：

- 指定你想代理的target
- 指定用不用CGLIB（see below and also [JDK- and CGLIB-based proxies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-pfb-proxy-types)）

一些关键属性继承于`org.springframework.aop.framework.ProxyConfig`  （spring里一切AOP代理工厂的superclass）。这些关键属性包括：

- `proxyTargetClass`: 如果目标类要被代理是true，而不是目标类的接口。如果属性被设置为true， CGLIB 代理会被创建（but see also [JDK- and CGLIB-based proxies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-pfb-proxy-types) ）
- `optimize`:  控制是否把aggressive optimizations apply到CGLIB创建的代理。不应该轻率地使用这种设置，除非您完全理解相关AOP代理如何处理优化 。目前仅用于CGLIB代理;它对JDK动态代理没有影响。 
- `frozen`:  如果一个proxy设置是forzen，然后对配置的更改不再被允许 。这是一个小优化，如果你不想在代理被创建后调用者操纵代理（通过 `Advised` 接口）。默认值是false，所以比如添加额外advice的变化是允许的。
- `exposeProxy`: 决定是否把当前代理暴露在 `ThreadLocal`以致target可以访问到。如果target需要得到代理并且`exposeProxy` 属性被设置为true，target可以使用`AopContext.currentProxy()` 方法。

其他特定于 `ProxyFactoryBean` 的属性包括：

- `proxyInterfaces`:  接口名字的String的数组。如果没被提供，目标类的CGLIB代理将被使用（but see also [JDK- and CGLIB-based proxies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-pfb-proxy-types) ）。
- `interceptorNames`: `Advisor`, 拦截器和其他advice的name的String数组。顺序很重要，以先到先得的原则。 *：*也就是说，列表中的第一个拦截器将是第一个能够拦截调用的拦截器。 

这些名称是当前工厂的bean名称，包括来自祖先工厂的bean名称。您不能在这里提到bean引用，因为这样做会导致ProxyFactoryBean忽略建议的单例设置。 

你可以对一个拦截器名字追加星号（*）。这导致应用里所有以星号前面部分开头的advisor beans被应用。使用这个特性的一个例子： [Using 'global' advisors](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-global-advisors). 

- singleton: 是否工厂应该返回一个single对象，不管 `getObject()` 方法的调用频率如何。几个`FactoryBean` 实现提供了这个方法。默认值是true。如果你想使用有状态的advice-例如，for stateful mixins ——使用prototype advices，singleton 值是false。

#### 6.5.3. JDK- and CGLIB-based proxies

本节作为明确文档，在 对于一个特定target对象（要被代理）`ProxyFactoryBean`  怎么选择创建一个JDK还是CGLIB-baed的代理。

**ProxyFactoryBean在创建JDK或基于cglib的代理方面的行为在版本1.2和2.0之间发生了变化。 `ProxyFactoryBean`  现在对于自动检测接口作为`TransactionProxyFactoryBean` 类展现相同的语义。**

如果要被代理的目标对象的类（以下简称为target类）没有实现任何接口，那么就会创建一个基于cglib的代理。 这是最简单的情况，因为JDK代理是基于接口的，并且没有接口意味着JDK代理是不可能的。 简单地插入目标bean，并通过 `interceptorNames`属性指定拦截器的列表。注意CGLIB-based 代理将被创建即使`ProxyFactoryBean`  的`proxyTargetClass`  属性被设置为false（显然，这毫无意义，最好从bean定义中删除，因为它最多是多余的，最糟糕的是令人困惑）。

如果目标类实现一个（多个）接口，然后所创建的代理类型取决于`ProxyFactoryBean`的配置。 

如果 `ProxyFactoryBean`  的`proxyTargetClass`  属性被设置为true，CGLIB-based 代理会被创建。这是有道理的，而且符合 least surprise原则。 即使`ProxyFactoryBean`  的`proxyInterfaces`  属性已经被设置为一个或多个全限定接口名，实际上 `proxyTargetClass` 属性被设置为true，导致CGLIB-based代理生效。

如果  `ProxyFactoryBean`  的`proxyInterfaces` 属性被设置为一个或多个全限定接口名，然后JDK-based代理被创建。被创建的代理将实现指定在`proxyInterfaces` 属性的所有接口；如果目标类恰好实现了比指定在`proxyInterfaces` 的更多的接口，这一切都很好，但是这些额外的接口将不会由返回的代理来实现。 

如果`ProxyFactoryBean`  的`proxyInterfaces`  没有被设置，但是目标类确实实现了一个（或多个）接口，然后`ProxyFactoryBean`会自动检测到目标类确实实现了至少一个接口的事实，并且一个JDK-based代理被创建。实际上被代理的接口将是目标类实现的所有接口；实际上，这与简单地提供目标类实现的`ProxyInterface`属性的每个接口的列表是一样的。 当然，它的工作要少得多，而且不容易出现拼写错误。 

#### 6.5.4. Proxying interfaces

让我们看一个简单的`ProxyFactoryBean`的例子。 这个例子包括: 

- 将被代理的目标bean。下面的 "personTarget" bean定义
- 一个Advisor  和一个Interceptor  用来提供advice
- 一个AOP proxy bean定义，指定目标对象（personTarget bean），和要代理的接口以及要使用的advice。

```
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name" value="Tony"/>
    <property name="age" value="51"/>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person"
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>

    <property name="target" ref="personTarget"/>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

注意`interceptorNames` 属性取了String的list：在当前工厂中拦截器或advice的bean名称。Advisors, interceptors, before, after returning and throws advice 对象可以被使用。advisors的顺序是很重要的。 

**您可能想知道为什么这个列表不包含bean引用。 这样做的原因是，如果`ProxyFactoryBean`的singleton属性被设置为false，那么它必定能够返回独立的代理实例。 如果任何一个advisor本身就是一个prototype，需要返回一个独立的实例，因此，有必要从工厂获得prototype的实例 ；持有一个引用是不够的。** 

 上面的"person" bean定义可以被用来代替一个Person  实现，如下：

```
Person person = (Person) factory.getBean("person");
```

在同一个IoC上下文中的其他bean可以对其表示强类型依赖性，就像普通的Java对象一样： 

```
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person"/></property>
</bean>
```

例子的 `PersonUser` 类将会暴露type Person的属性。就其而言，AOP代理可以透明地使用，以代替一个 "real" Person的实现。然而，它的类将是一个动态代理类。可以将其转换为`Advised`接口（如下所述）。 

使用匿名内部bean来隐藏目标和代理之间的区别是可能的，如下所列。 只有`ProxyFactoryBean` 定义不同；advice被包含只为了完整性。

```
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name" value="Tony"/>
            <property name="age" value="51"/>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

这有一个优点，那就是只有一个type `Person`对象： 如果我们想要阻止应用程序上下文的用户获得对未建议对象的引用，这是很有用的，或者需要避免使用Spring IoC自动连接的任何歧义。还有一个好处就是ProxyFactoryBean定义是自包含的。然而，有时能够从工厂获得 un-advised target实际上可能是一个优势：例如，在某些测试场景中。 

#### 6.5.5. Proxying classes

如果您需要代理一个类，而不是一个或多个接口，该怎么办？ 

想象一下，在我们上面的例子中， 没有 `Person`接口：我们需要advise一个没有实现任何业务接口的`Person`类。在这种情况下，您可以配置Spring来使用CGLIB代理，而不是动态代理。 只需要设置ProxyFactoryBean  上的`proxyTargetClass` 属性为true。虽然最好是对接口进行编程，而不是类，在处理遗留代码时，为不实现接口的类提供建议的能力是非常有用的。 （一般来说，Spring并不是规范的。 虽然它可以很容易地应用好的实践，但是它避免了强制使用特定的方法 ）

如果你想，你可以强制在所有情况使用CGLIB ，即使你有接口。

CGLIB  代理通过在运行时生成目标类的子类工作。Spring配置这个生成的子类，将方法调用委托给原始目标：子类用于实现装饰模式，在advice中编织。 

CGLIB代理通常对用户是透明的。然而，有一些问题需要考虑： 

- `Final` 方法不能被advise，因为不能被override
- 不需要添加CGLIB到classpath。自Spring 3.2，CGLIB被重新包装，并包含在spring-core JAR中。 换句话说，CGLIB-based AOP 开箱即用和JDK动态代理一样。

在CGLIB代理和动态代理之间几乎没有性能差异。 自spring 1.0，动态代理的速度稍微快一些。然而，将来可能会改变。在这种情况下，性能不应该是决定性的考虑因素。

#### 6.5.6. Using 'global' advisors

通过把星号附加到interceptor name，所有advisor的bean name匹配星号前面部分的，都被添加到advisor链。如果你需要添加一个标志 'global' advisors 集合就派上了用场：

```
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```

###  6.6. Concise proxy definitions

特别是在定义事务性代理时，你可能会得到许多类似的代理定义。使用parent和子bean定义 ，和内部bean定义可以产生更简洁的代理定义。

首先一个parent，*template* bean为代理创建：

```
<bean id="txProxyTemplate" abstract="true"
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

这将永远不会被实例化，所以是不完整的。然后每个需要被创建的代理都只是一个子bean定义 ，将代理的目标包装为内部bean定义，因为目标永远不会被单独使用。 

```
<bean id="myService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MyServiceImpl">
        </bean>
    </property>
</bean>
```

当然，可以从父模板中覆盖属性，在这种情况下，事务传播设置：

```
<bean id="mySpecialService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MySpecialServiceImpl">
        </bean>
    </property>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="store*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

注意，在上面的例子中，我们通过使用*abstract* 属性显式地将父bean定义标记为抽象， 如 [previously](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions) 所述，所以它可能不会被实例化。应用程序上下文（但不是简单的bean工厂）在默认情况下会预先实例化所有的单例。 因此(至少对单例bean),如果你有一个(父)bean定义你只打算使用作为模板,这个定义指定了一个类,您必须确保设置abstract属性为true,否则应用程序上下文会试图pre-instantiate它。 

###  6.7. Creating AOP proxies programmatically with the ProxyFactory

使用Spring编程式创建AOP代理很容易。这使您能够不依赖Spring IoC使用Spring AOP。 

下面的清单显示了目标对象代理的创建， 有1个interceptor和一个advisor。目标对象实现的接口会被自动代理：

```
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addAdvice(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
```

第一步构建 `org.springframework.aop.framework.ProxyFactory`类型对象。你可以用目标对象创建这个，就像上面的例子一样 ，或者指定在备用构造器中被代理的接口。

你可以添加advices（带有interceptors 作为专门类型的advice）and/or advisors ，并在ProxyFactory的life中操纵它们。如果你添加一个 IntroductionInterceptionAroundAdvisor，您可以使代理实现额外的接口。 

在ProxyFactory上也有方便的方法 （继承自AdvisedSupport ）允许你添加另外的advice类型比如 before and throws advice 。AdvisedSupport是ProxyFactory and ProxyFactoryBean的父类。

在大多数应用程序中，将AOP代理创建与IoC框架集成是最佳实践。 我们建议您使用AOP将配置从Java代码中具体化，就像一般情况一样。 

### 6.8. Manipulating advised objects

然而你创建AOP代理，你可以用 `org.springframework.aop.framework.Advised` 接口操作他们。任何AOP代理都可以被转化成这个接口，不管它实现了任何接口。该接口包括以下方法： 

```
Advisor[] getAdvisors();

void addAdvice(Advice advice) throws AopConfigException;

void addAdvice(int pos, Advice advice) throws AopConfigException;

void addAdvisor(Advisor advisor) throws AopConfigException;

void addAdvisor(int pos, Advisor advisor) throws AopConfigException;

int indexOf(Advisor advisor);

boolean removeAdvisor(Advisor advisor) throws AopConfigException;

void removeAdvisor(int index) throws AopConfigException;

boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;

boolean isFrozen();
```

`getAdvisors()` 方法对任何advisor，interceptor或者其他被添加到工厂的advice类型会返回一个Advisor 。如果你添加一个Advisor，在这个index返回的advisor将是你添加的对象。如果你添加了interceptor或者其他advice类型Spring将把它封装在一个带有pointcut的advisor中，这个切入点总是返回true。 因此，如果你添加`MethodInterceptor`，对这个indext返回的advisor 会是`DefaultPointcutAdvisor`  ，返回你的 `MethodInterceptor` 和一个匹配所有类和方法的pointcut。

 `addAdvisor()` 方法用于添加任何Advisor。通常持有 pointcut and advice 的advisor会是generic `DefaultPointcutAdvisor`，用了被任何advice or pointcut使用。（不能用于 introductions）

默认下，即使在创建代理之后，也可以添加或删除advisor或拦截器。唯一的限制是不可能添加或删除introduction advisor，因为来自工厂的现有代理不会显示接口更改。 

一个简单的例子，将AOP代理转换为`Advised`接口，并检查和操作它的advice： 

```
Advised advised = (Advised) myObject;
Advisor[] advisors = advised.getAdvisors();
int oldAdvisorCount = advisors.length;
System.out.println(oldAdvisorCount + " advisors");

// Add an advice like an interceptor without a pointcut
// Will match all proxied methods
// Can use for interceptors, before, after returning or throws advice
advised.addAdvice(new DebugInterceptor());

// Add selective advice using a pointcut
advised.addAdvisor(new DefaultPointcutAdvisor(mySpecialPointcut, myAdvice));

assertEquals("Added two advisors", oldAdvisorCount + 2, advised.getAdvisors().length);
```

在生产中修改业务对象的建议是否可取（没有双关的意思）是值得怀疑的，尽管毫无疑问，是合法使用案例。然而，这在开发中很有用：例如,在测试。 我有时发现，能够以拦截器或其他建议的形式添加测试代码非常有用，进入我想要测试的方法调用。 （例如，建议可以进入为该方法创建的事务中：例如，在为回滚进行标记之前 ，运行SQL检查数据库是否正确更新 ）

根据你创建代理的方式，你通常可以设置一个 `frozen` 标志 ，在这种情况下`Advised` `isFrozen()` 返回true，任何通过添加或删除来修改advice的尝试会导致`AopConfigException`。在某些情况下，冻结建议对象的状态的能力是有用的，例如，为了防止调用代码删除安全拦截器。它也可以在Spring 1.1中使用，以允许在不需要运行时通知修改的情况下进行积极的优化。 

###  6.9. Using the "auto-proxy" facility

到目前为止，我们已经考虑使用`ProxyFactoryBean`或类似的工厂bean显式地创建AOP代理。 

spring也允许使用"auto-proxy"  bean定义，可以自动代理选定bean定义。这是在"bean post processor" 基础结构上构建的，可以在容器加载时修改任何bean定义。

在这个模型中，您在XML bean定义文件中设置了一些特殊的bean定义来配置 auto proxy 基础结构。这允许你声明符合自动代理的目标：不需要使用 `ProxyFactoryBean` 。

有两种方法可以做到这一点： 

- 使用 auto-proxy creator引用当前context的特殊的beans
- 一个特殊的自动代理创建案例，值得单独考虑；source-level metadata attributes驱动的auto-proxy creation。

#### 6.9.1. Autoproxy bean definitions

`org.springframework.aop.framework.autoproxy` 包提供以下标准auto-proxy creators。

##### BeanNameAutoProxyCreator

`BeanNameAutoProxyCreator`类是一个`BeanPostProcessor`  ，为匹配name或通配符的beans自动创建AOP代理。

```
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```

和`ProxyFactoryBean`一样，有一个拦截器的属性，而不是一个拦截器列表，为prototype advisors 提供正确行为。名为 "interceptors" 可以是advisors or any advice type。

与一般自动代理一样，使用`BeanNameAutoProxyCreator` 要点是将相同的配置应用于多个对象，使用最小的配置。这在对多个对象的声明式事务处理很流行。

名称匹配的Bean定义，比如上例的"jdkMyBean" and "onlyJdk" 是普通的bean定义与目标类。AOP代理会自动通过 `BeanNameAutoProxyCreator`创建。同样的advice也适用于所有匹配的bean。注意，如果使用advisors（而不是上面例子中的拦截器），切入点可能会以不同的方式应用于不同的bean。 

##### DefaultAdvisorAutoProxyCreator

一个更通用的，非常强大的auto proxy creator 是`DefaultAdvisorAutoProxyCreator`。它会自定义应用当前context里合适的advisors，不需要在auto-proxy advisor’s bean定义中包含指定的bean names。和 `BeanNameAutoProxyCreator` 一样，它提供了相同的优点，即一致的配置和避免重复。

使用此机制包括: 

- 指定一个`DefaultAdvisorAutoProxyCreator`  定义
- 在相同或相关context指定一些Advisors。注意这些必须是Advisors，不能仅仅是interceptors或者其他advices。这是必要的因为必须有一个pointcut 来评估，检查每个建议对候选bean定义的资格。 

`DefaultAdvisorAutoProxyCreator`  会自动评估每个advisor包含的pointcut，去看哪个（如果有）advice应该被应用到每个业务对象（例如例子的"businessObject1" and "businessObject2"  ）。

这意味着任何数量的advisors都可以自动地应用到每个业务对象中。 如果任何advisors中的切入点都不匹配业务对象中的任何方法，对象不会被代理。随着bean定义被添加到新的业务对象中 ，如果有必要，将自动进行代理。

Autoproxying 通常有一个优势，使调用者或依赖得到一个un-advised对象是不可能的。在application context调用getBean("businessObject1") 将返回一个AOP代理，而不是目标业务对象（前面的"inner bean" idiom 也提供个这个优点）。

```
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```

`DefaultAdvisorAutoProxyCreator` 很有用，如果你想将相同的advice始终应用于许多业务对象。 一旦基础设施定义就位，您可以简单地添加新的业务对象，而不包括特定的代理配置。您还可以很容易地删除额外的advice——例如 ，跟踪或性能监视aspect——对配置进行最小的更改。

 DefaultAdvisorAutoProxyCreator 提供了过滤器支持（使用命名约定，这样只对特定的顾问进行评估，允许在同一工厂使用不同配置的、不同配置的顾问自动创建器 ） 和排序。Advisor可以实现`org.springframework.core.Ordered` 接口保证正确排序。上个例子使用的TransactionAttributeSourceAdvisor 有一个可配置的排序值；默认设置是没有排序的。

###  6.10. Using TargetSources

spring提供*TargetSource*概念，在`org.springframework.aop.TargetSource` 接口被解释。这个接口负责返回实现join point的"target object" 。`TargetSource` 实现在每一次AOP代理处理一个方法调用时，被要求返回一个target instance 。

使用spring AOP不需要直接操作TargetSources ，但他提供了种强大的支持pooling、热插拔和其他复杂目标的方法 。例如，一个pooling TargetSource 对每次调用返回不同的target instance，使用一个pool来管理实例。

如果你没有指定TargetSource，一个默认的包装本地对象的实现被使用。对每次调用返回相同的target（如你所期望）。

让我们看一下Spring提供的标准target sources，以及如何使用它们。 

**当使用自定义target source，你的target通常需要是prototype而不是singleton bean定义。这允许Spring在需要时创建一个新的目标实例。** 

####  6.10.1. Hot swappable target sources

`org.springframework.aop.target.HotSwappableTargetSource` 的存在，允许一个AOP代理的target 在允许调用者保持对它的引用的同时进行切换。

改变target source’s target立刻生效。`HotSwappableTargetSource` 是线程安全的。

你可以通过HotSwappableTargetSource  上的 `swap()`  方法改变target，如下：

```
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```

The XML definitions required look as follows: 

```
<bean id="initialTarget" class="mycompany.OldTarget"/>

<bean id="swapper" class="org.springframework.aop.target.HotSwappableTargetSource">
    <constructor-arg ref="initialTarget"/>
</bean>

<bean id="swappable" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="swapper"/>
</bean>
```

`swap()`  调用改变了swappable bean的target。对那个bean有引用的客户端将不会意识到这个变化，但是会立刻hitting新的target。

虽然这个例子没有加advice——并且使用`TargetSource` 不必要添加advice——当然`TargetSource`  和任意类型的advice一起使用。

####  6.10.2. Pooling target sources

使用一个pooling target source 为stateless session EJBs 提供相似的编程模型，在EJBs存在一个包含相同对象的pool，带有释放池中对象的方法调用。

Spring pooling and SLSB pooling 的一个关键区别是Spring pooling 可以应用到任意POJO。使用spring通常以无侵入性的方式应用service。

spring为Commons Pool 2.2 提供开箱即用的支持，这提供了一个相当有效的池实现。使用这个特性需要 commons-pool Jar 。继承`org.springframework.aop.target.AbstractPoolingTargetSource` 支持任意其他的pooling API也是可以的。

**Commons Pool 1.5+ 也是支持的从spring4.2被弃用。**

示例配置如下所示： 

```
<bean id="businessObjectTarget" class="com.mycompany.MyBusinessObject"
        scope="prototype">
    ... properties omitted
</bean>

<bean id="poolTargetSource" class="org.springframework.aop.target.CommonsPool2TargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
    <property name="maxSize" value="25"/>
</bean>

<bean id="businessObject" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="poolTargetSource"/>
    <property name="interceptorNames" value="myInterceptor"/>
</bean>
```

注意例子的 "businessObjectTarget"  target object -必须是prototype 。这允许 `PoolingTargetSource`实现来创建target新实例来增长池。see `AbstractPoolingTargetSource`   和具体实现类的javadocs关于它的属性： "maxSize" 是最基本的，通常保证要给出。

在这个例子， "myInterceptor" 是一个interceptor ，需要定义在相同IoC context。然而，没必要指定interceptors 使用pooling。如果你想只用pooling，不用其他advice，不要设置interceptorNames 属性。

配置spring把任何pooled object转换成`org.springframework.aop.target.PoolingConfig` 接口是可以的，它通过一个introduction暴露了配置和当前池大小的信息。你需要定义advisor 像这样：

```
<bean id="poolConfigAdvisor" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="poolTargetSource"/>
    <property name="targetMethod" value="getPoolingConfigMixin"/>
</bean>
```

调用 `AbstractPoolingTargetSource` 类上的便利方法获得advisor，因此使用MethodInvokingFactoryBean 。advisor（"poolConfigAdvisor" ）name必须在暴露pooled对象的ProxyFactoryBean 中的interceptors names 的list。

The cast will look as follows: 

```
PoolingConfig conf = (PoolingConfig) beanFactory.getBean("businessObject");
System.out.println("Max pool size is " + conf.getMaxSize());
```

Pooling stateless service objects 通常不必要。我们不相信他会是默认选择，因为大多数无状态对象是线程安全的，并且如果资源被缓存，实例池是有问题的。

使用auto-proxying Simpler pooling is available 。设置TargetSources 被任意 auto-proxy creator 使用是可以的。

#### 6.10.3. Prototype target sources

设置"prototype" target source 类似于 pooling TargetSource 。在这种情况下，将在每个方法调用上创建目标的新实例。 虽然在现代JVM中创建新对象的成本并不高 ，连接新对象（满足其IoC依赖性）的成本可能更昂贵。 因此，你不应该在没有充分理由的情况下使用这种方法。 

要做到，你可以修改`poolTargetSource` 定义如下，（为了清晰，改了名字）：

```
<bean id="prototypeTargetSource" class="org.springframework.aop.target.PrototypeTargetSource">
    <property name="targetBeanName" ref="businessObjectTarget"/>
</bean>
```

只有一个属性：target bean  name。TargetSource 实现使用了继承确保一致的命名。与the pooling target source一样， target bean  必须是一个prototype bean 定义。

####  6.10.4. ThreadLocal target sources

`ThreadLocal` target sources 是有用的，如果你要为每个要来的请求（per thread that is ）创建一个对象。`ThreadLocal` 概念提供JDK-wide设施来透明地将资源存储在线程旁边 。设置`ThreadLocalTargetSource`几乎与其他类型的目标源所解释的一样： 

```
<bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
</bean>
```

ThreadLocals 带来严重问题（可能导致内存泄漏 ）当在多线程和多类加载器环境中错误地使用它们时 。应该总是考虑在其他类中包装ThreadLocal，并且从不直接使用ThreadLocal本身（当然，除了包装器类之外）。 而且，应该始终记住正确set and unset  （后者只需要调用ThreadLocal.set（null））线程本地的资源。 在任何情况下都应该进行Unsetting  ，因为not Unsetting 可能会导致有问题的行为。 Spring的ThreadLocal支持对您来说是这样的，并且应该始终考虑使用ThreadLocal，而不需要其他适当的处理代码。 

###  6.11. Defining new Advice types

Spring AOP  被设计是可扩展的。虽然拦截实现策略目前是内部使用，但是有可能会支持除了out-of-the-box interception around advice, before, throws advice and after returning advice 之外的任意advice类型。

 `org.springframework.aop.framework.adapter`  包是一个SPI包，允许添加自定义advice类型而不改变core framework。一个`Advice` 的唯一约束是必须实现`org.aopalliance.aop.Advice` 标记接口。

Please refer to the `org.springframework.aop.framework.adapter` javadocs for further information. 

## 7. Null-safety

虽然Java的类型系统不允许表达 null-safety ，spring在`org.springframework.lang` 包提供了nullability of APIs and fields: 

- [`@NonNull`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/lang/NonNull.html)  注解在参数，返回值和field不能为null（在有 `@NonNullApi` and `@NonNullFields`上不需要在参数和返回值加）
- [`@Nullable`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/lang/Nullable.html) 注解在参数，返回值或field不为null
- [`@NonNullApi`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html) 在包级别声明参数和返回值的non-null行为
- [`@NonNullFields`](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/lang/NonNullFields.html) 包级别声明field的non-null行为

spring自己就用这些注解，但是它们也可以在任何Spring的Java项目中使用，以声明 null-safe APIs和optionally null-safe fields。 泛型类型参数，可变参数和数组元素的nullability 至今还未支持，但应该在要发布的版本中有，最新的信息 see [SPR-15942](https://jira.spring.io/browse/SPR-15942)  。

像 Reactor or Spring Data 库利用这个特性提供null-safe APIs 。

### 7.1. Use cases

除了对Spring Framework API nullability 提供显示声明外，这些注解也可以被IDE使用，提供有用的关于null-safety 提醒避免运行时 `NullPointerException` 。

They are also used to make Spring API null-safe in Kotlin projects since Kotlin natively supports [null-safety](https://kotlinlang.org/docs/reference/null-safety.html). More details are available in [Kotlin support documentation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/languages.html#kotlin-null-safety). 

### 7.2. JSR 305 meta-annotations

Spring注解是用 [JSR 305](https://jcp.org/en/jsr/detail?id=305) 的元注释 （一个休眠但广泛传播的JSR ）。JSR 305元注释允许像IDEA或Kotlin这样的工具供应商以一种通用的方式提供空安全支持，而不需要对Spring注释进行硬编码支持。 

在项目类路径中添加JSR 305依赖项是不必要的，也不建议使用Spring nulsafe API。 只有像基于spring的库这样的项目在代码库中使用空安全注释才会添加`com.google.code.findbugs:jsr305:3.0.2` with `compileOnly` Gradle configuration or Maven `provided` scope to avoid compile warnings. 

##  8. Data Buffers and Codecs

### 8.1. Introduction

`DataBuffer` 接口在byte buffers 上定义了一个抽象。引入它且不使用`java.nio.ByteBuffer` 的主要原因是Netty。Netty 不使用 `ByteBuffer` ，而是提供 `ByteBuf`  作为替代。Spring的`DataBuffer`是ByteBuf的一个简单的抽象，它也可以用于非netty平台（即Servlet 3.1+）。 

###  8.2. `DataBufferFactory`

 `DataBufferFactory`  提供分配新 data buffers 以及封装存在数据的功能。 `allocate`方法分配一个 new data buffer ，有默认的或给定的容量 。虽然 `DataBuffer`  实现按需增加或缩减，提前给出容量更有效，如果知道容量的话。 `wrap`  方法装饰一个存在的 `ByteBuffer` or byte array 。包装不涉及分配： 他只是用一个`DataBuffer`  实现来包装给定数据。

 `DataBufferFactory` 有2个实现： `NettyDataBufferFactory` 意味着在Netty平台使用，比如Reactor Netty 。另一个实现， `DefaultDataBufferFactory`，在其他平台使用，比如Servlet 3.1+ servers 。

###  8.3. The `DataBuffer` interface

`DataBuffer` 接口类似 `ByteBuffer` ，但提供一些优势。和 Netty’s `ByteBuf`类似，`DataBuffer` 抽象提供独立的读写位置。不同于JDK的`ByteBuffer`, 只是为读和写暴露一个位置，以及一个`flip()`  操作在这两种I/O之间切换。通常，以下不变条件适用：

```
0 <= read position <= write position <= capacity
```

当从 `DataBuffer` 读bytes，read position 自动更新与从buffer读到的数据量一致。类似地，当写bytes到 `DataBuffer` ，write position 更新为写到buffer的数据量一致。并且，写的时候，`DataBuffer` 容量是自动扩展的，就像`StringBuilder`,`ArrayList`, 和类似类型。

除了上述的读写能力， `DataBuffer` 也有方法可以把a (slice of a) buffer 看作`ByteBuffer`, `InputStream`, or `OutputStream`。另外，提供了决定给定byte的索引的方法。

两个 `DataBuffer`实现：`NettyDataBuffer`  ，意味着在Netty平台使用，比如Reactor Netty 。另一个实现， `DefaultDataBuffer`，在其他平台使用，比如Servlet 3.1+ servers 。

####  8.3.1. `PooledDataBuffer`

`PooledDataBuffer` 是 `DataBuffer` 的扩展添加了引用计数方法。`retain`  方法引用增加就+1。`release` 方法count-1，当count为0，释放buffer的内存。两个方法和*reference counting*关联，这个机制解释如下。

注意`DataBufferUtils`  提供有用的utils方法来释放和保留pooled data buffers 。这些方法把一个普通的`DataBuffer` 作为参数，但是只调用`retain` or `release` 如果passed data buffer是 `PooledDataBuffer` 的实例。

##### Reference Counting

引用计数在Java中不是一种常见的技术；在其他编程语言Object C and C++ 更常见。就其本身而言，引用计数并不复杂 ：它主要涉及到跟踪应用于对象的引用数量。 一个 `PooledDataBuffer` 的引用计数从1开始，调用 `retain` 增加，调用 `release` 减少。只要buffer的引用计数大于0，buffer就不会被释放。当数字减少到0时，实例将被释放 。在实践中，这意味着缓冲所捕获的保留内存将被返回到内存池，准备用于未来的分配。 

通常，访问*DataBuffer*的最后一个组件负责释放它。在spring，有两种类型的组件可以释放缓冲区： decoders and transports。Decoders 负责转换buffers 的流成为其他类型（see [Codecs](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#codecs) below  ），transports 负责通过网络边界发送buffer，特别是作为HTTP信息。这意味着你分配data buffers为了把它们放进外出的HTTP信息（例如客户端请求或服务器端响应 ），它们不需要被释放。这条规则的另一个结果是如果你分配的数据缓冲区不会出现在body中，例如，由于抛出异常，您必须自己释放它们。 下面的代码片段显示了一个典型的DataBuffer使用场景，在处理抛出异常的方法时： 

```
DataBufferFactory factory = ...
DataBuffer buffer = factory.allocateBuffer(); 1
boolean release = true; 2
try {
    writeDataToBuffer(buffer); 3
    putBufferInHttpBody(buffer);
    release = false; 4
}
finally {
    if (release) {
        DataBufferUtils.release(buffer); 5
    }
}

private void writeDataToBuffer(DataBuffer buffer) throws IOException { 3
    ...
}
```

1. 分配new buffer 
2. 表明分配的buffer是否应该被释放的flag
3. 示例方法加载数据到buffer。注意方法可以抛出`IOException`，因此`finally` 块用来释放buffer是需要的。
4. 如果没有exception  发生，切换 `release` flag to `false` ，缓冲区将被释放，作为将HTTP主体发送到整个网络的一部分。 
5. 发生异常，flag仍然是rue，buffer在这里被释放。

#### 8.3.2. DataBufferUtils

`DataBufferUtils` 包含各种util方法操作data buffers 。它包含方法从`InputStream` or NIO `Channel` 读取`DataBuffer`  对象的 `Flux`  ，和方法写data buffer `Flux` 到`OutputStream` or `Channel` 。`DataBufferUtils`  也暴露`retain` and `release`  方法操作一个plain  `DataBuffer`  实例（那样转换到 `PooledDataBuffer` 就不需要了）。

另外，`DataBufferUtils` 暴露了`compose`，把data buffers的流合并为一个。比如，这个方法可以用来把整个HTTP body转化成 single buffer （从a `String`, or `InputStream` ）。这在处理旧的阻塞api时特别有用。 但是请注意，这将整个body置于内存，因此，使用比纯流解决方案更多的内存。 

### Codecs

`org.springframework.core.codec` 包 包含2个主要抽象把字节流转换成对象流，或者反过来。 `Encoder`是策略接口把对象流编码到data buffers 的输出流。 `Decoder`做相反的事情：把data buffers流转换成 objects的流。注意decoder 实例需要考虑 [reference counting](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#databuffer-reference-counting) 。

Spring提供了大量的默认编解码器，能转换from/to `String`, `ByteBuffer`, byte arrays,  以及支持 marshalling libraries比如JAXB和Jackson（with [Jackson 2.9+ support for non-blocking parsing](https://github.com/FasterXML/jackson-core/issues/57) ）。在Spring WebFlux context，codecs  用于转换request body 为 `@RequestMapping` 参数，或者把返回类型转换成response body 返回给客户端。默认codecs 在`WebFluxConfigurationSupport`  配置，可以继承`configureHttpMessageCodecs` 重写来改变。For more information about using codecs in WebFlux, see [this section](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-codecs). 

##  9. Appendix

###  9.1. XML Schemas

附录的这一部分列出了与核心容器相关的XML模式。 

####  9.1.1. The util schema

#### 9.1.2. The aop schema

#### 9.1.3. The context schema

##### ` <property-placeholder/>`

##### `<annotation-config/>`

#####  `<component-scan/>`

This element is detailed in [Annotation-based container configuration](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-annotation-config).

##### `<load-time-weaver/>`

This element is detailed in [Load-time weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-aj-ltw).

##### `<spring-configured/>`

This element is detailed in [Using AspectJ to dependency inject domain objects with Spring](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-atconfigurable).

##### `<mbean-export/>`

This element is detailed in [Configuring annotation based MBean export](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/integration.html#jmx-context-mbeanexport).

#### 9.1.4. The beans schema

### 9.2. XML Schema Authoring

####  9.2.1. Introduction

Since version 2.0, Spring has featured a mechanism for schema-based extensions to the basic Spring XML format for defining and configuring beans. This section is devoted to detailing how you would go about writing your own custom XML bean definition parsers and integrating such parsers into the Spring IoC container.

To facilitate the authoring of configuration files using a schema-aware XML editor, Spring’s extensible XML configuration mechanism is based on XML Schema. If you are not familiar with Spring’s current XML configuration extensions that come with the standard Spring distribution, please first read the appendix entitled[[xsd-config\]](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-config).

Creating new XML configuration extensions can be done by following these (relatively) simple steps:

- [Authoring](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-custom-schema) an XML schema to describe your custom element(s).
- [Coding](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-custom-namespacehandler) a custom `NamespaceHandler` implementation (this is an easy step, don’t worry).
- [Coding](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-custom-parser) one or more `BeanDefinitionParser` implementations (this is where the real work is done).
- [Registering](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-custom-registration) the above artifacts with Spring (this too is an easy step).

What follows is a description of each of these steps. For the example, we will create an XML extension (a custom XML element) that allows us to configure objects of the type `SimpleDateFormat` (from the `java.text` package) in an easy manner. When we are done, we will be able to define bean definitions of type `SimpleDateFormat` like this:

```
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

*(Don’t worry about the fact that this example is very simple; much more detailed examples follow afterwards. The intent in this first simple example is to walk you through the basic steps involved.)*

#### 9.2.2. Authoring the schema

Creating an XML configuration extension for use with Spring’s IoC container starts with authoring an XML Schema to describe the extension. What follows is the schema we’ll use to configure `SimpleDateFormat` objects.

```
<!-- myns.xsd (inside package org/springframework/samples/xml) -->

<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.mycompany.com/schema/myns"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:beans="http://www.springframework.org/schema/beans"
        targetNamespace="http://www.mycompany.com/schema/myns"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="dateformat">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType">
                    <xsd:attribute name="lenient" type="xsd:boolean"/>
                    <xsd:attribute name="pattern" type="xsd:string" use="required"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>
```

(The emphasized line contains an extension base for all tags that will be identifiable (meaning they have an `id` attribute that will be used as the bean identifier in the container). We are able to use this attribute because we imported the Spring-provided`'beans'` namespace.)

The above schema will be used to configure `SimpleDateFormat` objects, directly in an XML application context file using the `<myns:dateformat/>` element.

```
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

Note that after we’ve created the infrastructure classes, the above snippet of XML will essentially be exactly the same as the following XML snippet. In other words, we’re just creating a bean in the container, identified by the name `'dateFormat'` of type`SimpleDateFormat`, with a couple of properties set.

```
<bean id="dateFormat" class="java.text.SimpleDateFormat">
    <constructor-arg value="yyyy-HH-dd HH:mm"/>
    <property name="lenient" value="true"/>
</bean>
```

|      | The schema-based approach to creating configuration format allows for tight integration with an IDE that has a schema-aware XML editor. Using a properly authored schema, you can use autocompletion to have a user choose between several configuration options defined in the enumeration. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 9.2.3. Coding a NamespaceHandler

In addition to the schema, we need a `NamespaceHandler` that will parse all elements of this specific namespace Spring encounters while parsing configuration files. The `NamespaceHandler` should in our case take care of the parsing of the `myns:dateformat` element.

The `NamespaceHandler` interface is pretty simple in that it features just three methods:

- `init()` - allows for initialization of the `NamespaceHandler` and will be called by Spring before the handler is used
- `BeanDefinition parse(Element, ParserContext)` - called when Spring encounters a top-level element (not nested inside a bean definition or a different namespace). This method can register bean definitions itself and/or return a bean definition.
- `BeanDefinitionHolder decorate(Node, BeanDefinitionHolder, ParserContext)` - called when Spring encounters an attribute or nested element of a different namespace. The decoration of one or more bean definitions is used for example with the[out-of-the-box scopes Spring supports](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes). We’ll start by highlighting a simple example, without using decoration, after which we will show decoration in a somewhat more advanced example.

Although it is perfectly possible to code your own `NamespaceHandler` for the entire namespace (and hence provide code that parses each and every element in the namespace), it is often the case that each top-level XML element in a Spring XML configuration file results in a single bean definition (as in our case, where a single `<myns:dateformat/>` element results in a single `SimpleDateFormat` bean definition). Spring features a number of convenience classes that support this scenario. In this example, we’ll make use the `NamespaceHandlerSupport` class:

```
package org.springframework.samples.xml;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class MyNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        registerBeanDefinitionParser("dateformat", new SimpleDateFormatBeanDefinitionParser());
    }

}
```

The observant reader will notice that there isn’t actually a whole lot of parsing logic in this class. Indeed… the `NamespaceHandlerSupport` class has a built in notion of delegation. It supports the registration of any number of `BeanDefinitionParser` instances, to which it will delegate to when it needs to parse an element in its namespace. This clean separation of concerns allows a `NamespaceHandler` to handle the orchestration of the parsing of *all* of the custom elements in its namespace, while delegating to `BeanDefinitionParsers` to do the grunt work of the XML parsing; this means that each `BeanDefinitionParser` will contain just the logic for parsing a single custom element, as we can see in the next step

#### 9.2.4. BeanDefinitionParser

A `BeanDefinitionParser` will be used if the `NamespaceHandler` encounters an XML element of the type that has been mapped to the specific bean definition parser (which is `'dateformat'` in this case). In other words, the `BeanDefinitionParser` is responsible for parsing *one* distinct top-level XML element defined in the schema. In the parser, we’ll have access to the XML element (and thus its subelements too) so that we can parse our custom XML content, as can be seen in the following example:

```
package org.springframework.samples.xml;

import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser;
import org.springframework.util.StringUtils;
import org.w3c.dom.Element;

import java.text.SimpleDateFormat;

public class SimpleDateFormatBeanDefinitionParser extends AbstractSingleBeanDefinitionParser { 

    protected Class getBeanClass(Element element) {
        return SimpleDateFormat.class; 
    }

    protected void doParse(Element element, BeanDefinitionBuilder bean) {
        // this will never be null since the schema explicitly requires that a value be supplied
        String pattern = element.getAttribute("pattern");
        bean.addConstructorArg(pattern);

        // this however is an optional property
        String lenient = element.getAttribute("lenient");
        if (StringUtils.hasText(lenient)) {
            bean.addPropertyValue("lenient", Boolean.valueOf(lenient));
        }
    }

}
```

|      | We use the Spring-provided `AbstractSingleBeanDefinitionParser` to handle a lot of the basic grunt work of creating a *single* `BeanDefinition`. |
| ---- | ------------------------------------------------------------ |
|      | We supply the `AbstractSingleBeanDefinitionParser` superclass with the type that our single `BeanDefinition` will represent. |

In this simple case, this is all that we need to do. The creation of our single `BeanDefinition` is handled by the `AbstractSingleBeanDefinitionParser` superclass, as is the extraction and setting of the bean definition’s unique identifier.

#### 9.2.5. Registering the handler and the schema

The coding is finished! All that remains to be done is to somehow make the Spring XML parsing infrastructure aware of our custom element; we do this by registering our custom `namespaceHandler` and custom XSD file in two special purpose properties files. These properties files are both placed in a `'META-INF'` directory in your application, and can, for example, be distributed alongside your binary classes in a JAR file. The Spring XML parsing infrastructure will automatically pick up your new extension by consuming these special properties files, the formats of which are detailed below.

##### 'META-INF/spring.handlers'

The properties file called `'spring.handlers'` contains a mapping of XML Schema URIs to namespace handler classes. So for our example, we need to write the following:

```
http\://www.mycompany.com/schema/myns=org.springframework.samples.xml.MyNamespaceHandler
```

*(The ':' character is a valid delimiter in the Java properties format, and so the ':' character in the URI needs to be escaped with a backslash.)*

The first part (the key) of the key-value pair is the URI associated with your custom namespace extension, and needs to *match exactly* the value of the `'targetNamespace'` attribute as specified in your custom XSD schema.

##### 'META-INF/spring.schemas'

The properties file called `'spring.schemas'` contains a mapping of XML Schema locations (referred to along with the schema declaration in XML files that use the schema as part of the `'xsi:schemaLocation'` attribute) to *classpath* resources. This file is needed to prevent Spring from absolutely having to use a default `EntityResolver` that requires Internet access to retrieve the schema file. If you specify the mapping in this properties file, Spring will search for the schema on the classpath (in this case`'myns.xsd'` in the `'org.springframework.samples.xml'` package):

```
http\://www.mycompany.com/schema/myns/myns.xsd=org/springframework/samples/xml/myns.xsd
```

The upshot of this is that you are encouraged to deploy your XSD file(s) right alongside the `NamespaceHandler` and `BeanDefinitionParser` classes on the classpath.

#### 9.2.6. Using a custom extension in your Spring XML configuration

Using a custom extension that you yourself have implemented is no different from using one of the 'custom' extensions that Spring provides straight out of the box. Find below an example of using the custom `<dateformat/>` element developed in the previous steps in a Spring XML configuration file.

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:myns="http://www.mycompany.com/schema/myns"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.mycompany.com/schema/myns http://www.mycompany.com/schema/myns/myns.xsd">

    <!-- as a top-level bean -->
    <myns:dateformat id="defaultDateFormat" pattern="yyyy-MM-dd HH:mm" lenient="true"/>

    <bean id="jobDetailTemplate" abstract="true">
        <property name="dateFormat">
            <!-- as an inner bean -->
            <myns:dateformat pattern="HH:mm MM-dd-yyyy"/>
        </property>
    </bean>

</beans>
```

#### 9.2.7. Meatier examples

Find below some much meatier examples of custom XML extensions.

##### Nesting custom tags within custom tags

This example illustrates how you might go about writing the various artifacts required to satisfy a target of the following configuration:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:foo="http://www.foo.com/schema/component"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.foo.com/schema/component http://www.foo.com/schema/component/component.xsd">

    <foo:component id="bionic-family" name="Bionic-1">
        <foo:component name="Mother-1">
            <foo:component name="Karate-1"/>
            <foo:component name="Sport-1"/>
        </foo:component>
        <foo:component name="Rock-1"/>
    </foo:component>

</beans>
```

The above configuration actually nests custom extensions within each other. The class that is actually configured by the above `<foo:component/>` element is the `Component` class (shown directly below). Notice how the `Component` class does *not* expose a setter method for the `'components'` property; this makes it hard (or rather impossible) to configure a bean definition for the `Component` class using setter injection.

```
package com.foo;

import java.util.ArrayList;
import java.util.List;

public class Component {

    private String name;
    private List<Component> components = new ArrayList<Component> ();

    // mmm, there is no setter method for the 'components'
    public void addComponent(Component component) {
        this.components.add(component);
    }

    public List<Component> getComponents() {
        return components;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```

The typical solution to this issue is to create a custom `FactoryBean` that exposes a setter property for the `'components'`property.

```
package com.foo;

import org.springframework.beans.factory.FactoryBean;

import java.util.List;

public class ComponentFactoryBean implements FactoryBean<Component> {

    private Component parent;
    private List<Component> children;

    public void setParent(Component parent) {
        this.parent = parent;
    }

    public void setChildren(List<Component> children) {
        this.children = children;
    }

    public Component getObject() throws Exception {
        if (this.children != null && this.children.size() > 0) {
            for (Component child : children) {
                this.parent.addComponent(child);
            }
        }
        return this.parent;
    }

    public Class<Component> getObjectType() {
        return Component.class;
    }

    public boolean isSingleton() {
        return true;
    }

}
```

This is all very well, and does work nicely, but exposes a lot of Spring plumbing to the end user. What we are going to do is write a custom extension that hides away all of this Spring plumbing. If we stick to [the steps described previously](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#xsd-custom-introduction), we’ll start off by creating the XSD schema to define the structure of our custom tag.

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.com/schema/component"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.com/schema/component"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:element name="component">
        <xsd:complexType>
            <xsd:choice minOccurs="0" maxOccurs="unbounded">
                <xsd:element ref="component"/>
            </xsd:choice>
            <xsd:attribute name="id" type="xsd:ID"/>
            <xsd:attribute name="name" use="required" type="xsd:string"/>
        </xsd:complexType>
    </xsd:element>

</xsd:schema>
```

We’ll then create a custom `NamespaceHandler`.

```
package com.foo;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class ComponentNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        registerBeanDefinitionParser("component", new ComponentBeanDefinitionParser());
    }

}
```

Next up is the custom `BeanDefinitionParser`. Remember that what we are creating is a `BeanDefinition` describing a `ComponentFactoryBean`.

```
package com.foo;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.ManagedList;
import org.springframework.beans.factory.xml.AbstractBeanDefinitionParser;
import org.springframework.beans.factory.xml.ParserContext;
import org.springframework.util.xml.DomUtils;
import org.w3c.dom.Element;

import java.util.List;

public class ComponentBeanDefinitionParser extends AbstractBeanDefinitionParser {

    protected AbstractBeanDefinition parseInternal(Element element, ParserContext parserContext) {
        return parseComponentElement(element);
    }

    private static AbstractBeanDefinition parseComponentElement(Element element) {
        BeanDefinitionBuilder factory = BeanDefinitionBuilder.rootBeanDefinition(ComponentFactoryBean.class);
        factory.addPropertyValue("parent", parseComponent(element));

        List<Element> childElements = DomUtils.getChildElementsByTagName(element, "component");
        if (childElements != null && childElements.size() > 0) {
            parseChildComponents(childElements, factory);
        }

        return factory.getBeanDefinition();
    }

    private static BeanDefinition parseComponent(Element element) {
        BeanDefinitionBuilder component = BeanDefinitionBuilder.rootBeanDefinition(Component.class);
        component.addPropertyValue("name", element.getAttribute("name"));
        return component.getBeanDefinition();
    }

    private static void parseChildComponents(List<Element> childElements, BeanDefinitionBuilder factory) {
        ManagedList<BeanDefinition> children = new ManagedList<BeanDefinition>(childElements.size());
        for (Element element : childElements) {
            children.add(parseComponentElement(element));
        }
        factory.addPropertyValue("children", children);
    }

}
```

Lastly, the various artifacts need to be registered with the Spring XML infrastructure.

```
# in 'META-INF/spring.handlers'
http\://www.foo.com/schema/component=com.foo.ComponentNamespaceHandler
```

```
# in 'META-INF/spring.schemas'
http\://www.foo.com/schema/component/component.xsd=com/foo/component.xsd
```

##### Custom attributes on 'normal' elements

Writing your own custom parser and the associated artifacts isn’t hard, but sometimes it is not the right thing to do. Consider the scenario where you need to add metadata to already existing bean definitions. In this case you certainly don’t want to have to go off and write your own entire custom extension; rather you just want to add an additional attribute to the existing bean definition element.

By way of another example, let’s say that the service class that you are defining a bean definition for a service object that will (unknown to it) be accessing a clustered [JCache](https://jcp.org/en/jsr/detail?id=107), and you want to ensure that the named JCache instance is eagerly started within the surrounding cluster:

```
<bean id="checkingAccountService" class="com.foo.DefaultCheckingAccountService"
        jcache:cache-name="checking.account">
    <!-- other dependencies here... -->
</bean>
```

What we are going to do here is create another `BeanDefinition` when the `'jcache:cache-name'` attribute is parsed; this `BeanDefinition` will then initialize the named JCache for us. We will also modify the existing `BeanDefinition` for the`'checkingAccountService'` so that it will have a dependency on this new JCache-initializing `BeanDefinition`.

```
package com.foo;

public class JCacheInitializer {

    private String name;

    public JCacheInitializer(String name) {
        this.name = name;
    }

    public void initialize() {
        // lots of JCache API calls to initialize the named cache...
    }

}
```

Now onto the custom extension. Firstly, the authoring of the XSD schema describing the custom attribute (quite easy in this case).

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<xsd:schema xmlns="http://www.foo.com/schema/jcache"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.foo.com/schema/jcache"
        elementFormDefault="qualified">

    <xsd:attribute name="cache-name" type="xsd:string"/>

</xsd:schema>
```

Next, the associated `NamespaceHandler`.

```
package com.foo;

import org.springframework.beans.factory.xml.NamespaceHandlerSupport;

public class JCacheNamespaceHandler extends NamespaceHandlerSupport {

    public void init() {
        super.registerBeanDefinitionDecoratorForAttribute("cache-name",
            new JCacheInitializingBeanDefinitionDecorator());
    }

}
```

Next, the parser. Note that in this case, because we are going to be parsing an XML attribute, we write a `BeanDefinitionDecorator` rather than a `BeanDefinitionParser`.

```
package com.foo;

import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.BeanDefinitionDecorator;
import org.springframework.beans.factory.xml.ParserContext;
import org.w3c.dom.Attr;
import org.w3c.dom.Node;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class JCacheInitializingBeanDefinitionDecorator implements BeanDefinitionDecorator {

    private static final String[] EMPTY_STRING_ARRAY = new String[0];

    public BeanDefinitionHolder decorate(Node source, BeanDefinitionHolder holder,
            ParserContext ctx) {
        String initializerBeanName = registerJCacheInitializer(source, ctx);
        createDependencyOnJCacheInitializer(holder, initializerBeanName);
        return holder;
    }

    private void createDependencyOnJCacheInitializer(BeanDefinitionHolder holder,
            String initializerBeanName) {
        AbstractBeanDefinition definition = ((AbstractBeanDefinition) holder.getBeanDefinition());
        String[] dependsOn = definition.getDependsOn();
        if (dependsOn == null) {
            dependsOn = new String[]{initializerBeanName};
        } else {
            List dependencies = new ArrayList(Arrays.asList(dependsOn));
            dependencies.add(initializerBeanName);
            dependsOn = (String[]) dependencies.toArray(EMPTY_STRING_ARRAY);
        }
        definition.setDependsOn(dependsOn);
    }

    private String registerJCacheInitializer(Node source, ParserContext ctx) {
        String cacheName = ((Attr) source).getValue();
        String beanName = cacheName + "-initializer";
        if (!ctx.getRegistry().containsBeanDefinition(beanName)) {
            BeanDefinitionBuilder initializer = BeanDefinitionBuilder.rootBeanDefinition(JCacheInitializer.class);
            initializer.addConstructorArg(cacheName);
            ctx.getRegistry().registerBeanDefinition(beanName, initializer.getBeanDefinition());
        }
        return beanName;
    }

}
```

Lastly, the various artifacts need to be registered with the Spring XML infrastructure.

```
# in 'META-INF/spring.handlers'
http\://www.foo.com/schema/jcache=com.foo.JCacheNamespaceHandler
# in 'META-INF/spring.schemas'
http\://www.foo.com/schema/jcache/jcache.xsd=com/foo/jcache.xsd
```

