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

**你也可以singleton scope的beans间使用`<aop:scoped-proxy/>`,通过这个引用然后通过一个可序列化的中间代理，从而通过反序列化重新得到目标单例bean。  
当对prototype的bean声明`<aop:scoped-proxy/>`，共享代理上的每个方法调用，都会导致一个方法调用所导向的新的目标实例的创建。  
此外，scoped proxies不是唯一的方式，以一种生命周期安全的方式，访问shorter scopes的beans。你也可以简单地声明你的注入点（例如，constructor/setter参数，或者autowired field）作为```ObjectFactory<MyTargetBean>```，允许getObject（）调用在每次需要时按需检索当前实例——而不需要保留实例或单独存储它。  
作为一个扩展的变体，您可以声明`ObjectProvider<MyTargetBean>`，它提供了几个额外的访问形式，包括`getIfAvailable`和`getIfUnique`。  
JSR-330在此叫`Provider`，使用`Provider<MyTargetBean>`声明和一个对应的`get()`调用，用于每一次获取尝试。see Using JSR 330 Standard Annotations for more details on JSR-330 overall。**


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






















