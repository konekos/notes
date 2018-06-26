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