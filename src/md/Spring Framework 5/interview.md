# Spring 整体

## Spring Framework 中有多少个模块，它们分别是什么？

**Spring 核心容器**

- Spring Core

- Spring Beans

  核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。BeanFactory 使用控制反转 （IOC）模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。

- Spring Context

- SpEL (Spring Expression Language)

或者说，这块就是 Spring IoC 。

**数据访问**

- JDBC (Java DataBase Connectivity)
- ORM (Object Relational Mapping)
- Transaction

**Web**

- WebMVC

  MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

- WebFlux

- WebSocket

**AOP**

- AOP

  通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。

  Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

- Aspects 该模块为与 AspectJ 的集成提供支持。

**其它**

- JMS (Java Messaging Service)

- Test

## 使用 Spring 框架能带来哪些好处？

- **DI** ：**Dependency Injection(DI)** 方法，使得构造器和 JavaBean、properties 文件中的依赖关系一目了然。
- **轻量级**
- **面向切面编程(AOP)**
- **集成主流框架**
- 模块化
- **便捷的测试**
- **Web 框架**
- **事务管理**：
- **异常处理**：Spring 提供一个方便的 API ，将特定技术的异常(由JDBC, Hibernate, 或 JDO 抛出)转化为一致的、Unchecked 异常。

## Spring 框架中都用到了哪些设计模式？

- 代理模式 — 在 AOP 和 remoting 中被用的比较多。
- 单例模式 — 在 Spring 配置文件中定义的 Bean 默认为单例模式。
- 模板方法 — 用来解决代码重复的问题。比如 [RestTemplate](http://howtodoinjava.com/2015/02/20/spring-restful-client-resttemplate-example/)、JmsTemplate、JdbcTemplate 。
- 前端控制器 — Spring提供了 DispatcherServlet 来对请求进行分发。
- 依赖注入 — 贯穿于 BeanFactory / ApplicationContext 接口的核心理念。
- 工厂模式 — BeanFactory 用来创建对象的实例。



# Spring IoC

> 下面，我们会将分成 IoC 和 Bean 两部分来分享 Spring 容器的内容。
>
> - IoC ，侧重在于容器。
> - Bean ，侧重在于被容器管理的 Bean 。

## 什么是 Spring IoC 容器？

Spring 框架的核心是 Spring IoC 容器。容器创建 Bean 对象，将它们装配在一起，配置它们并管理它们的完整生命周期。

- Spring 容器使用**依赖注入**来管理组成应用程序的 Bean 对象。
- 容器通过读取提供的**配置元数据** Bean Definition 来接收对象进行实例化，配置和组装的指令。
- 该配置元数据 Bean Definition 可以通过 XML，Java 注解或 Java Config 代码**提供**。

![1565513887194](E:\studydyup\notes\src\pic\1565513887194.png)

## 什么是依赖注入？

在依赖注入中，你不必主动、手动创建对象，但必须描述如何创建它们。

- 你不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。
- 然后，再由 IoC 容器将它们装配在一起。

另外，依赖注入的英文缩写是 Dependency Injection ，简称 DI 。

## IoC 和 DI 有什么区别？

IoC更宽泛，DI更具体。

## 可以通过多少种方式完成依赖注入？

通常，依赖注入可以通过**三种**方式完成，即：

> 上面一个问题的三种方式的英文，下面是三种方式的中文。

- 接口注入
- 构造函数注入
- setter 注入

目前，在 Spring Framework 中，仅使用构造函数和 setter 注入这**两种**方式。



- 实际场景下，setting 注入使用的更多。



## Spring 中有多少种 IoC 容器？

Spring 提供了两种( 不是“个” ) IoC 容器，分别是 BeanFactory、ApplicationContext 。

**BeanFactory**

> BeanFactory 在 `spring-beans` 项目提供。

BeanFactory ，就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象。

**ApplicationContext**

> ApplicationContext 在 `spring-context` 项目提供。

ApplicationContext 接口扩展了 BeanFactory 接口，它在 BeanFactory 基础上提供了一些额外的功能。内置如下功能：

- MessageSource ：管理 message ，实现国际化等功能。
- ApplicationEventPublisher ：事件发布。
- ResourcePatternResolver ：多资源加载。
- EnvironmentCapable ：系统 Environment（profile + Properties）相关。
- Lifecycle ：管理生命周期。
- Closable ：关闭，释放资源
- InitializingBean：自定义初始化。
- BeanNameAware：设置 beanName 的 Aware 接口。

另外，ApplicationContext 会自动初始化非懒加载的 Bean 对象们。

| BeanFactory                | ApplicationContext       |
| -------------------------- | ------------------------ |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

另外，BeanFactory 也被称为**低级**容器，而 ApplicationContext 被称为**高级**容器。

## 请介绍下常用的 BeanFactory 容器？

BeanFactory 最常用的是 XmlBeanFactory 。它可以根据 XML 文件中定义的内容，创建相应的 Bean。

## 请介绍下常用的 ApplicationContext 容器？

- 1、ClassPathXmlApplicationContext ：从 ClassPath 的 XML 配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。示例代码如下：

  ```
  ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);
  ```

- 2、FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。示例代码如下：

  ```
  ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
  ```

- 3、XmlWebApplicationContext ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。

当然，目前我们更多的是使用 Spring Boot 为主，所以使用的是第四种 ApplicationContext 容器，ConfigServletWebServerApplicationContext 。

## 列举一些 IoC 的一些好处？

- 它将最小化应用程序中的代码量。
- 它以最小的影响和最少的侵入机制促进松耦合。
- 它支持即时的实例化和延迟加载 Bean 对象。
- 它将使您的应用程序易于测试，因为它不需要单元测试用例中的任何单例或 JNDI 查找机制。

## ⭐简述 Spring IoC 的实现机制？

简单来说，Spring 中的 IoC 的实现原理，就是**工厂模式**加**反射机制**。代码如下：

```java
interface Fruit {

     public abstract void eat();
     
}
class Apple implements Fruit {

    public void eat(){
        System.out.println("Apple");
    }
    
}
class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {

    public static Fruit getInstance(String className) {
        Fruit f = null;
        try {
            f = (Fruit) Class.forName(className).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
    
}

class Client {

    public static void main(String[] args) {
        Fruit f = Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f != null){
            f.eat();
        }
    }
    
}
```

- Fruit 接口，有 Apple 和 Orange 两个实现类。
- Factory 工厂，通过反射机制，创建 `className` 对应的 Fruit 对象。
- Client 通过 Factory 工厂，获得对应的 Fruit 对象。
- 😈 实际情况下，Spring IoC 比这个复杂很多很多，例如单例 Bean 对象，Bean 的属性注入，相互依赖的 Bean 的处理，以及等等。



### 广义的 IOC

- IoC(Inversion of Control) 控制反转，即“不用打电话过来，我们会打给你”。

两种实现： 依赖查找（DL）和依赖注入（DI）。



###  Spring 的 IoC

Spring 的 IoC 设计支持以下功能：

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调某些方法（但是需要实现 Spring 接口，略有侵入）

**其中，最重要的就是依赖注入，从 XML 的配置上说， 即 ref 标签。对应 Spring RuntimeBeanReference 对象。**

对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入。

那么， Spring 如何设计容器的呢？

- BeanFactory
- ApplicationContext



BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。

ApplicationContext 可以称之为 **“高级容器”**。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。所以你看他的名字，已经不是 BeanFactory 之类的工厂了，而是 “应用上下文”， 代表着整个大容器的所有功能。该接口定义了一个 refresh 方法，此方法是所有阅读 Spring 源码的人的最熟悉的方法，用于刷新整个容器，即重新加载/刷新所有的 bean。

下图是 ClassPathXmlApplicationContext 的构造过程，**实际就是 Spring IoC 的初始化过程**。



![img](http://static.iocoder.cn/7789456cebf7cfaab215b5b46c46f57c)

1. 用户构造 ClassPathXmlApplicationContext（简称 CPAC）
2. CPAC 首先访问了 “抽象高级容器” 的 final 的 refresh 方法，这个方法是模板方法。所以要回调子类（低级容器）的 refreshBeanFactory 方法，这个方法的作用是使用低级容器加载所有 BeanDefinition 和 Properties 到容器中。
3. 低级容器加载成功后，高级容器开始处理一些回调，例如 Bean 后置处理器。回调 setBeanFactory 方法。或者注册监听器等，发布事件，实例化单例 Bean 等等功能，这些功能，随着 Spring 的不断升级，功能越来越多，很多人在这里迷失了方向 ：）。

简单说就是：

1. 低级容器 加载配置文件（从 XML，数据库，Applet），并解析成 BeanDefinition 到低级容器中。
2. 加载成功后，高级容器启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。

好，当我们创建好容器，就会使用 getBean 方法，获取 Bean，而 getBean 的流程如下：

![img](http://static.iocoder.cn/97f12ec77b4136b606d291be00e9dd7c)

假设 ： 当 Bean_A 依赖着 Bean_B，而这个 Bean_A 在加载的时候，其配置的 ref = “Bean_B” 在解析的时候只是一个占位符，被放入了 Bean_A 的属性集合中，当调用 getBean 时，需要真正 Bean_B 注入到 Bean_A 内部时，就需要从容器中获取这个 Bean_B，因此产生了递归。

为什么不是在加载的时候，就直接注入呢？因为加载的顺序不同，很可能 Bean_A 依赖的 Bean_B 还没有加载好，也就无法从容器中获取，你不能要求用户把 Bean 的加载顺序排列好，这是不人道的。

所以，Spring 将其分为了 2 个步骤：

1. 加载所有的 Bean 配置成 BeanDefinition 到容器中，如果 Bean 有依赖关系，则使用占位符暂时代替。
2. 然后，在调用 getBean 的时候，进行真正的依赖注入，即如果碰到了属性是 ref 的（占位符），那么就从容器里获取这个 Bean，然后注入到实例中 —— 称之为依赖注入。

可以看到，依赖注入实际上，只需要 “低级容器” 就可以实现。

###  总结

说了这么多，不知道你有没有理解Spring IoC？ 这里小结一下：IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

a. 加载配置文件，解析成 BeanDefinition 放在 Map 里。

b. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

# Spring Bean

## 什么是 Spring Bean ？

- Bean 由 Spring IoC 容器实例化，配置，装配和管理。
- Bean 是基于用户提供给 IoC 容器的配置元数据 Bean Definition 创建。

## Spring 有哪些配置方式

1、XML 配置文件。

2、注解配置。

3、Java Config 配置。

Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

- `@Bean` 注解扮演与 `<bean />` 元素相同的角色。
- `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

## Spring 支持几种 Bean Scope ？

Spring Bean 支持 5 种 Scope ，分别如下：

- Singleton - 每个 Spring IoC 容器仅有一个单 Bean 实例。**默认**
- Prototype - 每次请求都会产生一个新的实例。
- Request - 每一次 HTTP 请求都会产生一个新的 Bean 实例，并且该 Bean 仅在当前 HTTP 请求内有效。
- Session - 每一个的 Session 都会产生一个新的 Bean 实例，同时该 Bean 仅在当前 HTTP Session 内有效。
- Application - 每一个 Web Application 都会产生一个新的 Bean ，同时该 Bean 仅在当前 Web Application 内有效。

仅当用户使用支持 Web 的 ApplicationContext 时，**最后三个才可用**。

不错呢，还是那句话，这个题目简单了解下即可，实际常用的只有 Singleton 和 Prototype 两种级别，甚至说，只有 Singleton 级别。😈



## ⭐Spring Bean 在容器的生命周期是什么样的？



Spring Bean 的**初始化**流程如下：

- 实例化 Bean 对象

  - Spring 容器根据配置中的 Bean Definition(定义)中**实例化** Bean 对象。

    > Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。

  - Spring 使用依赖注入**填充**所有属性，如 Bean 中所定义的配置。

- Aware 相关的属性，注入到 Bean 对象
  - 如果 Bean 实现 **BeanNameAware** 接口，则工厂通过传递 Bean 的 beanName 来调用 `#setBeanName(String name)` 方法。
  - 如果 Bean 实现 **BeanFactoryAware** 接口，工厂通过传递自身的实例来调用 `#setBeanFactory(BeanFactory beanFactory)` 方法。
- 调用相应的方法，进一步初始化 Bean 对象
  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则调用 `#preProcessBeforeInitialization(Object bean, String beanName)` 方法。
  - 如果 Bean 实现 **InitializingBean** 接口，则会调用 `#afterPropertiesSet()` 方法。
  - 如果为 Bean 指定了 **init** 方法（例如 `<bean />` 的 `init-method` 属性），那么将调用该方法。
  - 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则将调用 `#postProcessAfterInitialization(Object bean, String beanName)` 方法。
- Spring Bean 的**销毁**流程如下：
  - 如果 Bean 实现 **DisposableBean** 接口，当 spring 容器关闭时，会调用 `#destroy()` 方法。
  - 如果为 bean 指定了 **destroy** 方法（例如 `<bean />` 的 `destroy-method` 属性），那么将调用该方法。



整体如下图：

![bean生命周期](http://static2.iocoder.cn/images/Spring/2018-12-24/03.jpg)

![bean生命周期](http://static2.iocoder.cn/images/Spring/2018-12-24/08.png)

## 解释什么叫延迟加载？

默认情况下，容器启动之后会将所有作用域为**单例**的 Bean 都创建好，但是有的业务场景我们并不需要它提前都创建好。此时，我们可以在Bean 中设置 `lzay-init = "true"` 。

- 这样，当容器启动之后，作用域为单例的 Bean ，就不在创建。
- 而是在获得该 Bean 时，才真正在创建加载。

## Spring 框架中的单例 Bean 是线程安全的么？

Spring 框架并没有对[单例](http://howtodoinjava.com/2012/10/22/singleton-design-pattern-in-java/) Bean 进行任何多线程的封装处理。

- 关于单例 Bean 的[线程安全](http://howtodoinjava.com/2014/06/02/what-is-thread-safety/)和并发问题，需要开发者自行去搞定。
- 并且，单例的线程安全问题，也不是 Spring 应该去关心的。Spring 应该做的是，提供根据配置，创建单例 Bean 或多例 Bean 的功能。

当然，但实际上，大部分的 Spring Bean 并没有可变的状态(比如Serview 类和 DAO 类)，所以在某种程度上说 Spring 的单例 Bean 是线程安全的。

如果你的 Bean 有多种状态的话，就需要自行保证线程安全。最浅显的解决办法，就是将多态 Bean 的作用域( Scope )由 Singleton 变更为 Prototype 。

## ⭐IoC源码解析

![æ´ä½æµç¨](http://static.iocoder.cn/38419d23d29c83a4758f73f85281e076)



## ⭐Spring Bean 怎么解决循环依赖的问题？





# Spring 注解

这块内容，实际写在 [「Spring Bean」](http://svip.iocoder.cn/Spring/Interview/#) 中比较合适，考虑到后续的问题，都是关于注解的，所以单独起一个大的章节。

## 什么是基于注解的容器配置？

不使用 XML 来描述 Bean 装配，开发人员通过在相关的类，方法或字段声明上使用**注解**将配置移动到组件类本身。它可以作为 XML 设置的替代方案。例如：

Spring 的 Java 配置是通过使用 `@Bean` 和 `@Configuration` 来实现。

- `@Bean` 注解，扮演与 `<bean />` 元素相同的角色。
- `@Configuration` 注解的类，允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。



## 如何在 Spring 中启动注解装配？

默认情况下，Spring 容器中未打开注解装配。因此，要使用基于注解装配，我们必须通过配置 <context：annotation-config />

当然，如果胖友是使用 Spring Boot ，默认情况下已经开启。

## @Component, @Controller, @Repository, @Service 有何区别？

- `@Component` ：它将 Java 类标记为 Bean 。它是任何 Spring 管理组件的**通用**构造型。
- `@Controller` ：它将一个类标记为 Spring Web MVC **控制器**。
- `@Service` ：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在**服务层**类中使用 @Service 而不是 `@Component` ，因为它以更好的方式指定了意图。
- `@Repository` ：这个注解是具有类似用途和功能的 `@Component` 注解的特化。它为 **DAO** 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException 。

## @Required 注解有什么用？

`@Required` 注解，应用于 Bean 属性 setter 方法。

- 此注解仅指示必须在配置时使用 Bean 定义中的显式属性值或使用自动装配填充受影响的 Bean 属性。
- 如果尚未填充受影响的 Bean 属性，则容器将抛出 BeanInitializationException 异常。



## @Autowired 注解有什么用？

`@Autowired` 注解，可以更准确地控制应该在何处以及如何进行自动装配。

- 此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 Bean。
- 默认情况下，它是类型驱动的注入。



## @Qualifier 注解有什么用？



当你创建多个**相同类型**的 Bean ，并希望仅使用属性装配**其中一个** Bean 时，您可以使用 `@Qualifier` 注解和 `@Autowired` 通过指定 ID 应该装配哪个**确切的** Bean 来消除歧义。

例如，应用中有两个类型为 Employee 的 Bean ID 为 `"emp1"` 和 `"emp2"` ，此处，我们希望 EmployeeAccount Bean 注入 `"emp1"` 对应的 Bean 对象。代码如下：

```java
public class EmployeeAccount {

    @Autowired
    @Qualifier(emp1)
    private Employee emp;

}
```

# Spring AOP

Spring AOP 的面试题中，大多数都是概念题，主要是对切面的理解。概念点主要有：

- AOP
- Aspect
- JoinPoint
- PointCut
- Advice
- Target
- AOP Proxy
- Weaving

## 什么是 AOP ？

AOP(Aspect-Oriented Programming)，即**面向切面编程**, 它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成， 提供了与 OOP 不同的抽象软件结构的视角。

- 在 OOP 中，以类( Class )作为基本单元
- 在 AOP 中，以**切面( Aspect )**作为基本单元。



## 什么是 Aspect ？

Aspect 由 **PointCut** 和 **Advice** 组成。

- 它既包含了横切逻辑的定义，也包括了连接点的定义。
- Spring AOP 就是负责实施切面的框架，它将切面所定义的横切逻辑编织到切面所指定的连接点中。

AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:

1. 如何通过 PointCut 和 Advice 定位到特定的 **JoinPoint** 上。
2. 如何在 Advice 中编写切面代码。



**可以简单地认为, 使用 @Aspect 注解的类就是切面**



![æµç¨å¾](http://static2.iocoder.cn/images/Spring/2018-12-24/04.jpg)

## 什么是 JoinPoint ?

JoinPoint ，**切点**，程序运行中的一些时间点, 例如：

- 一个方法的执行。
- 或者是一个异常的处理。

在 Spring AOP 中，JoinPoint 总是方法的执行点。



## 什么是 PointCut ？

PointCut ，**匹配** JoinPoint 的谓词(a predicate that matches join points)。

> 简单来说，PointCut 是匹配 JoinPoint 的条件。

- Advice 是和特定的 PointCut 关联的，并且在 PointCut 相匹配的 JoinPoint 中执行。即 `Advice => PointCut => JoinPoint` 。
- 在 Spring 中, 所有的方法都可以认为是 JoinPoint ，但是我们并不希望在所有的方法上都添加 Advice 。**而 PointCut 的作用**，就是提供一组规则(使用 AspectJ PointCut expression language 来描述) 来匹配 JoinPoint ，给满足规则的 JoinPoint 添加 Advice 。



## 关于 JoinPoint 和 PointCut 的区别

JoinPoint 和 PointCut 本质上就是**两个不同纬度上**的东西。

- 在 Spring AOP 中，所有的方法执行都是 JoinPoint 。而 PointCut 是一个描述信息，它修饰的是 JoinPoint ，通过 PointCut ，我们就可以确定哪些 JoinPoint 可以被织入 Advice 。
- Advice 是在 JoinPoint 上执行的，而 PointCut 规定了哪些 JoinPoint 可以执行哪些 Advice 。

或者，我们在换一种说法：

1. 首先，Advice 通过 PointCut 查询需要被织入的 JoinPoint 。
2. 然后，Advice 在查询到 JoinPoint 上执行逻辑。



## 什么是 Advice ？



Advice ，**通知**。

- 特定 JoinPoint 处的 Aspect 所采取的动作称为 Advice 。
- Spring AOP 使用一个 Advice 作为拦截器，在 JoinPoint “周围”维护一系列的**拦截器**。



**有哪些类型的 Advice？**

- Before - 这些类型的 Advice 在 JoinPoint 方法之前执行，并使用 `@Before` 注解标记进行配置。
- After Returning - 这些类型的 Advice 在连接点方法正常执行后执行，并使用 `@AfterReturning` 注解标记进行配置。
- After Throwing - 这些类型的 Advice 仅在 JoinPoint 方法通过抛出异常退出并使用 `@AfterThrowing` 注解标记配置时执行。
- After Finally - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 `@After` 注解标记进行配置。
- Around - 这些类型的 Advice 在连接点之前和之后执行，并使用 `@Around` 注解标记进行配置。



## 什么是 Target ？

Target ，织入 Advice 的**目标对象**。目标对象也被称为 **Advised Object** 。

- 因为 Spring AOP 使用运行时代理的方式来实现 Aspect ，因此 Advised Object 总是一个代理对象(Proxied Object) 。
- **注意, Advised Object 指的不是原来的对象，而是织入 Advice 后所产生的代理对象**。
- Advice + Target Object = Advised Object = Proxy 。



## AOP 有哪些实现方式？



实现 AOP 的技术，主要分为两大类：

- ① **静态代理** - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强。

  - 编译时编织（特殊编译器实现）

  - 类加载时编织（特殊的类加载器实现）。

    > 例如，SkyWalking 基于 Java Agent 机制，配置上 ByteBuddy 库，实现类加载时编织时增强，从而实现链路追踪的透明埋点。
    >
    > 感兴趣的胖友，可以看看 [《SkyWalking 源码分析之 JavaAgent 工具 ByteBuddy 的应用》](http://www.kailing.pub/article/index/arcid/178.html) 。

- ② **动态代理** - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。目前 Spring 中使用了两种动态代理库：

  - JDK 动态代理
  - CGLIB

那么 Spring 什么时候使用 JDK 动态代理，什么时候使用 CGLIB 呢？

```
// From 《Spring 源码深度解析》P172
// Spring AOP 部分使用 JDK 动态代理或者 CGLIB 来为目标对象创建代理。（建议尽量使用 JDK 的动态代理）
// 如果被代理的目标对象实现了至少一个接口，则会使用 JDK 动态代理。所有该目标类型实现的接口都讲被代理。
// 若该目标对象没有实现任何接口，则创建一个 CGLIB 代理。
// 如果你希望强制使用 CGLIB 代理，（例如希望代理目标对象的所有方法，而不只是实现自接口的方法）那也可以。但是需要考虑以下两个方法：
//      1> 无法通知(advise) Final 方法，因为它们不能被覆盖。
//      2> 你需要将 CGLIB 二进制发型包放在 classpath 下面。
// 为什么 Spring 默认使用 JDK 的动态代理呢？笔者猜测原因如下：
//      1> 使用 JDK 原生支持，减少三方依赖
//      2> JDK8 开始后，JDK 代理的性能差距 CGLIB 的性能不会太多。可参见：https://www.cnblogs.com/haiq/p/4304615.html
```



## Spring AOP and AspectJ AOP 有什么区别？

- 代理方式不同
  - Spring AOP 基于动态代理方式实现。
  - AspectJ AOP 基于静态代理方式实现。
- PointCut 支持力度不同
  - Spring AOP **仅**支持方法级别的 PointCut 。
  - AspectJ AOP 提供了完全的 AOP 支持，它还支持属性级别的 PointCut 。



## Spring 如何使用 AOP 切面？

在 Spring AOP 中，有两种方式配置 AOP 切面：

- 基于 **XML** 方式的切面实现。
- 基于 **注解** 方式的切面实现。

目前，主流喜欢使用 **注解** 方式。胖友可以看看 [《彻底征服 Spring AOP 之实战篇》](https://segmentfault.com/a/1190000007469982) 。



# Spring Transaction

## 什么是事务？

事务就是对一系列的数据库操作（比如插入多条数据）进行统一的提交或回滚操作，如果插入成功，那么一起成功，如果中间有一条出现异常，那么回滚之前的所有操作。

这样可以防止出现脏数据，防止数据库数据出现问题。



## 事务的特性指的是？

指的是 **ACID** ，如下图所示：

![äºå¡çç¹æ§](http://static2.iocoder.cn/images/Spring/2018-12-24/06.png)

1. **原子性** Atomicity ：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
2. **一致性** Consistency ：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设[约束](https://zh.wikipedia.org/wiki/数据完整性)、[触发器](https://zh.wikipedia.org/wiki/触发器_(数据库))、[级联回滚](https://zh.wikipedia.org/w/index.php?title=级联回滚&action=edit&redlink=1)等。
3. **隔离性** Isolation ：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
4. **持久性** Durability ：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。



## 列举 Spring 支持的事务管理类型？

目前 Spring 提供两种类型的事务管理：

- **声明式**事务：通过使用注解或基于 XML 的配置事务，从而事务管理与业务代码分离。
- **编程式**事务：通过编码的方式实现事务管理，需要在代码中显式的调用事务的获得、提交、回滚。它为您提供极大的灵活性，但维护起来非常困难。

## Spring 事务如何和不同的数据持久层框架做集成？



① 首先，我们先明确下，这里数据持久层框架，指的是 Spring JDBC、Hibernate、Spring JPA、MyBatis 等等。

② 然后，Spring 事务的管理，是通过 `org.springframework.transaction.PlatformTransactionManager` 进行管理，定义如下：

```java
// PlatformTransactionManager.java

public interface PlatformTransactionManager {

    // 根据事务定义 TransactionDefinition ，获得 TransactionStatus 。 
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

    // 根据情况，提交事务
    void commit(TransactionStatus status) throws TransactionException;
    
    // 根据情况，回滚事务
    void rollback(TransactionStatus status) throws TransactionException;
    
}
```

- ```
  #getTransaction(TransactionDefinition definition)
  ```

   

  方法，根据事务定义 TransactionDefinition ，获得 TransactionStatus 。

  - 为什么不是创建事务呢？因为如果当前如果已经有事务，则不会进行创建，一般来说会跟当前线程进行绑定。如果不存在事务，则进行创建。
  - 为什么返回的是 TransactionStatus 对象？在 TransactionStatus 中，不仅仅包含事务属性，还包含事务的其它信息，例如是否只读、是否为新创建的事务等等。😈 下面，也会详细解析 TransactionStatus 。
  - 事务 TransactionDefinition 是什么？😈 下面，也会详细解析 TransactionStatus 。

- ```
  #commit(TransactionStatus status)
  ```

   

  方法，根据 TransactionStatus 情况，提交事务。

  - 为什么根据 TransactionStatus 情况，进行提交？例如说，带

    ```
    @Transactional
    ```

     

    注解的的 A 方法，会调用

     

    ```
    @Transactional
    ```

     

    注解的的 B 方法。

    - 在 B 方法结束调用后，会执行 `PlatformTransactionManager#commit(TransactionStatus status)` 方法，此处事务**是不能**、**也不会**提交的。
    - 而是在 A 方法结束调用后，执行 `PlatformTransactionManager#commit(TransactionStatus status)` 方法，提交事务。

- ```
  #rollback(TransactionStatus status)
  ```

   

  方法，根据 TransactionStatus 情况，回滚事务。

  - 为什么根据 TransactionStatus 情况，进行回滚？原因同 `#commit(TransactionStatus status)` 方法。

③ 再之后，PlatformTransactionManager 有**抽象子**类 `org.springframework.transaction.support.AbstractPlatformTransactionManager` ，基于 [模板方法模式](https://blog.csdn.net/carson_ho/article/details/54910518) ，实现事务整体逻辑的骨架，而抽象 `#doCommit(DefaultTransactionStatus status)`、`#doRollback(DefaultTransactionStatus status)` 等等方法，交由子类类来实现。



④ 最后，不同的数据持久层框架，会有其对应的 PlatformTransactionManager 实现类，如下图所示：



- 所有的实现类，都基于 AbstractPlatformTransactionManager 这个骨架类。
- HibernateTransactionManager ，和 Hibernate5 的事务管理做集成。
- DataSourceTransactionManager ，和 JDBC 的事务管理做集成。所以，它也适用于 MyBatis、Spring JDBC 等等。
- JpaTransactionManager ，和 JPA 的事务管理做集成。



如下，是一个比较常见的 XML 方式来配置的事务管理器，使用的是 DataSourceTransactionManager 。代码如下：

```
<!-- 事务管理器 --><bean id="transactionManager"class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    <!-- 数据源 -->    <property name="dataSource" ref="dataSource" /></bean>
```

- 正如上文所说，它适用于 MyBatis、Spring JDBC 等等。



## 为什么在 Spring 事务中不能切换数据源？



做过 Spring 多数据源的胖友，都会有个惨痛的经历，为什么在开启事务的 Service 层的方法中，无法切换数据源呢？因为，在 Spring 的事务管理中，**所使用的数据库连接会和当前线程所绑定**，即使我们设置了另外一个数据源，使用的还是当前的数据源连接。



另外，多个数据源且需要事务的场景，本身会带来**多事务一致性**的问题，暂时没有特别好的解决方案。



所以一般一个应用，推荐除了读写分离所带来的多数据源，其它情况下，建议只有一个数据源。并且，随着微服务日益身形，一个服务对应一个 DB 是比较常见的架构选择。



## @Transactional 注解有哪些属性？如何使用？

| 属性                   | 类型                               | 描述                                   |
| ---------------------- | ---------------------------------- | -------------------------------------- |
| value                  | String                             | 可选的限定描述符，指定使用的事务管理器 |
| propagation            | enum: Propagation                  | 可选的事务传播行为设置                 |
| isolation              | enum: Isolation                    | 可选的事务隔离级别设置                 |
| readOnly               | boolean                            | 读写或只读事务，默认读写               |
| timeout                | int (in seconds granularity)       | 事务超时时间设置                       |
| rollbackFor            | Class对象数组，必须继承自Throwable | 导致事务回滚的异常类数组               |
| rollbackForClassName   | 类名数组，必须继承自Throwable      | 导致事务回滚的异常类名字数组           |
| noRollbackFor          | Class对象数组，必须继承自Throwable | 不会导致事务回滚的异常类数组           |
| noRollbackForClassName | 类名数组，必须继承自Throwable      | 不会导致事务回滚的异常类名字数组       |



- 一般情况下，我们直接使用 `@Transactional` 的所有属性默认值即可。



- `@Transactional` 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 `public` 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。
- 虽然 `@Transactional` 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， **@Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的**。如果你在 `protected`、`private` 或者默认可见性的方法上使用 `@Transactional` 注解，这将被忽略，也不会抛出任何异常。**这一点，非常需要注意**。



## 什么是事务的隔离级别？分成哪些隔离级别？

```
// TransactionDefinition.java

/**
 * 【Spring 独有】使用后端数据库默认的隔离级别
 *
 * MySQL 默认采用的 REPEATABLE_READ隔离级别
 * Oracle 默认采用的 READ_COMMITTED隔离级别
 */
int ISOLATION_DEFAULT = -1;

/**
 * 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
 */
int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;

/**
 * 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
 */
int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
/**
 * 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
 */
int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
/**
 * 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。
 *
 * 但是这将严重影响程序的性能。通常情况下也不会用到该级别。
 */
int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;
```



## 什么是事务的传播级别？分成哪些传播级别？

## 什么是事务的超时属性？

## 什么是事务的只读属性？

## 什么是事务的回滚规则？

回滚规则，定义了哪些异常会导致事务回滚而哪些不会。

- 默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）。
- 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

注意，事务的回滚规则，并不是数据库事务规范中的名词，**而是 Spring 自身所定义的**。



## 使用 Spring 事务有什么优点？

1. 通过 PlatformTransactionManager ，为不同的数据层持久框架提供统一的 API ，无需关心到底是原生 JDBC、Spring JDBC、JPA、Hibernate 还是 MyBatis 。
2. 通过使用声明式事务，使业务代码和事务管理的逻辑分离，更加清晰。

从倾向上来说，艿艿比较喜欢**注解** + 声明式事务。\



# Spring Data Access





## Spring 数据数据访问层有哪些异常？

通过使用 Spring 数据数据访问层，它统一了各个数据持久层框架的不同异常，统一进行提供 `org.springframework.dao.DataAccessException` 异常及其子类。如下图所示：

![æµç¨å¾](http://static2.iocoder.cn/images/Spring/2018-12-24/09.jpg)



# Spring MVC

## Spring MVC 框架有什么用？

Spring Web MVC 框架提供”模型-视图-控制器”( Model-View-Controller )架构和随时可用的组件，用于开发灵活且松散耦合的 Web 应用程序。

MVC 模式有助于分离应用程序的不同方面，如输入逻辑，业务逻辑和 UI 逻辑，同时在所有这些元素之间提供松散耦合。



## 介绍下 Spring MVC 的核心组件？



Spring MVC 一共有九大核心组件，分别是：

- MultipartResolver
- LocaleResolver
- ThemeResolver
- HandlerMapping
- HandlerAdapter
- HandlerExceptionResolver
- RequestToViewNameTranslator
- ViewResolver
- FlashMapManager



虽然很多，但是在前后端分离的架构中，在描述一下 DispatcherServlet 的工作流程问题中，我们会明白，最关键的只有 HandlerMapping + HandlerAdapter + HandlerExceptionResolver 。



## ⭐描述一下 DispatcherServlet 的工作流程？

![DispatcherServlet çå·¥ä½æµç¨](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15300766829012.jpg)

① **发送请求**

用户向服务器发送 HTTP 请求，请求被 Spring MVC 的调度控制器 DispatcherServlet 捕获。

② **映射处理器**

DispatcherServlet 根据请求 URL ，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 **Handler** 对象以及 Handler 对象对应的**拦截器**），最后以 HandlerExecutionChain 对象的形式返回。

- 即 HandlerExecutionChain 中，包含对应的 **Handler** 对象和**拦截器**们。

  > 此处，对应的方法如下：
  >
  > ```
  > > // HandlerMapping.java> > @Nullable> HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;>
  > ```

。。。



**但是 Spring MVC 的流程真的一定是酱紫么**？

既然这么问，答案当然不是。对于目前主流的架构，前后端已经进行分离了，所以 Spring MVC 只负责 **M**odel 和 **C**ontroller 两块，而将 **V**iew 移交给了前端。所以，在上图中的步骤 ⑤ 和 ⑥ 两步，已经不在需要。

那么变成什么样了呢？在步骤 ③ 中，如果 Handler(Controller) 执行完后，如果判断方法有 `@ResponseBody` 注解，则直接将结果写回给用户( 浏览器 )。

但是 HTTP 是不支持返回 Java POJO 对象的，所以需要将结果使用 [HttpMessageConverter](http://svip.iocoder.cn/Spring-MVC/HandlerAdapter-5-HttpMessageConverter/) 进行转换后，才能返回。例如说，大家所熟悉的 [FastJsonHttpMessageConverter](https://github.com/alibaba/fastjson/wiki/在-Spring-中集成-Fastjson) ，将 POJO 转换成 JSON 字符串返回。



嘻嘻，再来补充两个图，这真的是 Spring MVC 非常关键的问题，所以要用心理解。



![æµç¨ç¤ºæå¾](http://static2.iocoder.cn/images/Spring/2022-02-21/01.png)

![ä»£ç åºåå¾](http://static2.iocoder.cn/images/Spring/2022-02-21/02.png)

![ãæµç¨ç¤ºæå¾ã](http://static2.iocoder.cn/images/Spring/2022-02-21/03.png)



## @RestController 和 @Controller 有什么区别？

`@RestController` 注解，在 `@Controller` 基础上，增加了 `@ResponseBody` 注解，更加适合目前前后端分离的架构下，提供 Restful API ，返回例如 JSON 数据格式。当然，返回什么样的数据格式，根据客户端的 `"ACCEPT"` 请求头来决定。

## @RequestMapping 注解有什么用？

`@RequestMapping` 注解，用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：

- 类级别：映射请求的 URL。
- 方法级别：映射 URL 以及 HTTP 请求方法。

## @RequestMapping 和 @GetMapping 注解的不同之处在哪里？

- `@RequestMapping` 可注解在类和方法上；`@GetMapping` 仅可注册在方法上。
- `@RequestMapping` 可进行 GET、POST、PUT、DELETE 等请求方法；`@GetMapping` 是 `@RequestMapping` 的 GET 请求方法的特例，目的是为了提高清晰度。



## 返回 JSON 格式使用什么注解？

可以使用 **@ResponseBody** 注解，或者使用包含 `@ResponseBody` 注解的 **@RestController** 注解。

当然，还是需要配合相应的支持 JSON 格式化的 HttpMessageConverter 实现类。例如，Spring MVC 默认使用 MappingJackson2HttpMessageConverter 。



## 介绍一下 WebApplicationContext ？



WebApplicationContext 是实现ApplicationContext接口的子类，专门为 WEB 应用准备的。

- 它允许从相对于 Web 根目录的路径中**加载配置文件**，**完成初始化 Spring MVC 组件的工作**。
- 从 WebApplicationContext 中，可以获取 ServletContext 引用，整个 Web 应用上下文对象将作为属性放置在 ServletContext 中，以便 Web 应用环境可以访问 Spring 上下文。



## Spring MVC 的异常处理？



- Spring MVC 提供了异常解析器 HandlerExceptionResolver 接口，将处理器( `handler` )执行时发生的异常，解析( 转换 )成对应的 ModelAndView 结果

- 也就是说，如果异常被解析成功，则会返回 ModelAndView 对象。
- 详细的源码解析，见 [《精尽 Spring MVC 源码解析 —— HandlerExceptionResolver 组件》](http://svip.iocoder.cn/Spring-MVC/HandlerExceptionResolver/) 。
- 一般情况下，我们使用 `@ExceptionHandler` 注解来实现过异常的处理，可以先看看 [《Spring 异常处理 ExceptionHandler 的使用》](https://www.jianshu.com/p/12e1a752974d) 。

## Spring MVC 有什么优点？

1. 使用真的真的真的非常**方便**，无论是添加 HTTP 请求方法映射的方法，还是不同数据格式的响应。
2. 提供**拦截器机制**，可以方便的对请求进行拦截处理。
3. 提供**异常机制**，可以方便的对异常做统一处理。
4. 可以任意使用各种**视图**技术，而不仅仅局限于 JSP ，例如 Freemarker、Thymeleaf 等等。
5. 不依赖于 Servlet API (目标虽是如此，但是在实现的时候确实是依赖于 Servlet 的，当然仅仅依赖 Servlet ，而不依赖 Filter、Listener )。



## Spring MVC 怎样设定重定向和转发 ？



## Spring MVC 的 Controller 是不是单例？

绝绝绝大多数情况下，Controller 是**单例**。

那么，Controller 里一般不建议存在**共享的变量**。实际场景下，艿艿也没碰到需要使用共享变量的情况。



## Spring MVC 和 Struts2 的异同？

1. 入口

   不同

   - Spring MVC 的入门是一个 Servlet **控制器**。
   - Struts2 入门是一个 Filter **过滤器**。

2. 配置映射

   不同，

   - Spring MVC 是基于**方法**开发，传递参数是通过**方法形参**，一般设置为**单例**。
   - Struts2 是基于**类**开发，传递参数是通过**类的属性**，只能设计为**多例**。

- 视图

  不同

  - Spring MVC 通过参数解析器是将 Request 对象内容进行解析成方法形参，将响应数据和页面封装成 **ModelAndView** 对象，最后又将模型数据通过 **Request** 对象传输到页面。其中，如果视图使用 JSP 时，默认使用 **JSTL**。
  - Struts2 采用**值栈**存储请求和响应的数据，通过 **OGNL** 存取数据。



## ⭐详细介绍下 Spring MVC 拦截器？

`org.springframework.web.servlet.HandlerInterceptor` ，拦截器接口。代码如下：



```java
// HandlerInterceptor.java

/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行之前
 */
default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
	return true;
}

/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行成功之后
 */
default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
		@Nullable ModelAndView modelAndView) throws Exception {
}

/**
 * 拦截处理器，在 {@link HandlerAdapter#handle(HttpServletRequest, HttpServletResponse, Object)} 执行完之后，无论成功还是失败
 *
 * 并且，只有该处理器 {@link #preHandle(HttpServletRequest, HttpServletResponse, Object)} 执行成功之后，才会被执行
 */
default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
		@Nullable Exception ex) throws Exception {
}
```

- 一共有三个方法，分别为：

  - `#preHandle(...)` 方法，调用 Controller 方法之**前**执行。

  - `#postHandle(...)` 方法，调用 Controller 方法之**后**执行。

  - ```
    #afterCompletion(...)
    ```

     

    方法，处理完 Controller 方法返回结果之

    后

    执行。

    - 例如，页面渲染后。
    - **当然，要注意，无论调用 Controller 方法是否成功，都会执行**。

- 举个例子：

  - 当俩个拦截器都实现放行操作时，执行顺序为 `preHandle[1] => preHandle[2] => postHandle[2] => postHandle[1] => afterCompletion[2] => afterCompletion[1]` 。
  - 当第一个拦截器 `#preHandle(...)` 方法返回 `false` ，也就是对其进行拦截时，第二个拦截器是完全不执行的，第一个拦截器只执行 `#preHandle(...)` 部分。
  - 当第一个拦截器 `#preHandle(...)` 方法返回 `true` ，第二个拦截器 `#preHandle(...)` 返回 `false` ，执行顺序为 `preHandle[1] => preHandle[2] => afterCompletion[1]` 。

- 总结来说：

  - `#preHandle(...)` 方法，按拦截器定义**顺序**调用。若任一拦截器返回 `false` ，则 Controller 方法不再调用。
  - `#postHandle(...)` 和 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用。
  - `#postHandler(...)` 方法，在调用 Controller 方法之**后**执行。
  - `#afterCompletion(...)` 方法，只有该拦截器在 `#preHandle(...)` 方法返回 `true` 时，才能够被调用，且一定会被调用。为什么“且一定会被调用”呢？即使 `#afterCompletion(...)` 方法，按拦截器定义**逆序**调用时，前面的拦截器发生异常，后面的拦截器还能够调用，**即无视异常**。



## Spring MVC 的拦截器可以做哪些事情？

拦截器能做的事情非常非常非常多，例如：

- 记录访问日志。
- 记录异常日志。
- 需要登陆的请求操作，拦截未登陆的用户。
- …



## HttpMessageConverter 在 Spring REST 中代表什么?

HttpMessageConverter 是一种[策略接口](http://www.java67.com/2014/12/strategy-pattern-in-java-with-example.html) ，它指定了一个转换器，它可以转换 HTTP 请求和响应。Spring REST 用这个接口转换 HTTP 响应到多种格式，例如：JSON 或 XML 。

每个 HttpMessageConverter 实现都有一种或几种相关联的MIME协议。Spring 使用 `"Accept"` 的标头来确定客户端所期待的内容类型。

然后，它将尝试找到一个注册的 HTTPMessageConverter ，它能够处理特定的内容类型，并使用它将响应转换成这种格式，然后再将其发送给客户端。

如果胖友对 HttpMessageConverter 不了解，可以看看 [《Spring 中 HttpMessageConverter 详解》](https://leokongwq.github.io/2017/06/14/spring-MessageConverter.html) 。



## 如何创建 HttpMessageConverter 的自定义实现来支持一种新的请求/响应？

我们仅需要创建自定义的 AbstractHttpMessageConverter 的实现，并使用 WebMvcConfigurerAdapter 的 `#extendMessageConverters(List<HttpMessageConverter<?>> converters)` 方法注中册它，该方法可以生成一种新的请求 / 响应类型。

具体的示例，可以学习 [《在 Spring 中集成 Fastjson》](https://github.com/alibaba/fastjson/wiki/在-Spring-中集成-Fastjson) 文章。



- Spring 在创建 bean 的时候并不是等它完全完成，而是在创建过程中将创建中的 bean 的 ObjectFactory 提前曝光（即加入到 `singletonFactories` 缓存中）。
- 这样，一旦下一个 bean 创建的时候需要依赖 bean ，则直接使用 ObjectFactory 的 `#getObject()` 方法来获取了，(**但是它还不是很完美（没有进行属性填充和初始化），但是对于其他依赖它的对象而言已经足够了（可以根据对象引用定位到堆中对象），能够被认出来了**)。

- 首先 A 完成初始化第一步并将自己提前曝光出来（通过 ObjectFactory 将自己提前曝光），在初始化的时候，发现自己依赖对象 B，此时就会去尝试 get(B)，这个时候发现 B 还没有被创建出来
- 然后 B 就走创建流程，在 B 初始化的时候，同样发现自己依赖 C，C 也没有被创建出来
- 这个时候 C 又开始初始化进程，但是在初始化的过程中发现自己依赖 A，于是尝试 get(A)，这个时候由于 A 已经添加至缓存中（一般都是添加至三级缓存 `singletonFactories` ），通过 ObjectFactory 提前曝光，所以可以通过 `ObjectFactory#getObject()` 方法来拿到 A 对象，C 拿到 A 对象后顺利完成初始化，然后将自己添加到一级缓存中
- 回到 B ，B 也可以拿到 C 对象，完成初始化，A 可以顺利拿到 B 完成初始化。到这里整个链路就已经完成了初始化过程了。



