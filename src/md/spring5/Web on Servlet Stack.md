# Web on Servlet Stack

This part of the documentation covers support for Servlet stack, web applications built on the Servlet API and deployed to Servlet containers. Individual chapters include [Spring MVC](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc), [View Technologies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-view), [CORS Support](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors), and [WebSocket Support](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#websocket). For reactive stack, web applications, go to [Web on Reactive Stack](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#spring-web-reactive). 

## 1. Spring Web MVC

### 1.1. Introduction

Spring Web MVC is the original web framework built on the Servlet API and included in the Spring Framework from the very beginning.   The formal name "Spring Web MVC" comes from the name of its source module [spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc) but it is more commonly known as "Spring MVC". 

Parallel to Spring Web MVC, Spring Framework 5.0 introduced a reactive stack, web framework whose name Spring WebFlux is also based on its source module [spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux).  

For baseline information and compatibility with Servlet container and Java EE version ranges please visit the Spring Framework[Wiki](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions). 

###  1.2. DispatcherServlet

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-dispatcher-handler) 

Spring MVC 围绕前端控制器模式设计 ，中心的`DispatcherServlet`, 为请求处理提供一个共享的算法，而实际工作是由可配置的委托组件执行的。*：*这个模型是灵活的，并且支持不同的工作流。 

 `DispatcherServlet`，像任何`Servlet `，需要按照Servlet规范使用Java配置或web.xml声明和映射。 反过来，DispatcherServlet使用Spring配置来发现它需要的委托组件，它需要请求映射、视图分辨率、异常处理 [and more。](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-servlet-special-bean-types) 

下面是注册和初始化`DispatcherServlet`的Java配置示例。这个类是由Servlet容器自动检测的 (see [Servlet Config](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-container-config)) ：

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

**除了直接使用ServletContext API之外， 你可以继承`AbstractAnnotationConfigDispatcherServletInitializer` 复写特定方法（see example under [Context Hierarchy](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-servlet-context-hierarchy) ）。**

下面是注册和初始化`DispatcherServlet`的`web.xml`配置 ：

```
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

**Spring Boot引导一个不同的初始化顺序。而不是连接到Servlet容器的生命周期，Spring Boot 使用Spring configuration 引导本身和被嵌入的Servlet容器。`Filter` and `Servlet`  声明在Spring configuration 被检测并且在Servlet容器中注册。For more details check the [Spring Boot docs](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-embedded-container).** 

####  1.2.1. Context Hierarchy

`DispatcherServlet` 用自己的配置期待一个`WebApplicationContext`，plain  `ApplicationContext`的扩展。`WebApplicationContext`  有一个链接到`ServletContext`  和与它关联的 `Servlet`  。他也被绑定到`ServletContext`  ，这样应用可以在`RequestContextUtils`  上使用静态方法来查找 `WebApplicationContext` 如果它们需要连接到其上的话。

对于大多数应用只有一个 `WebApplicationContext` ，是简单有效的。也可以有一个context hierarchy  ，在这一个root `WebApplicationContext` 在多个`DispatcherServlet` (or other `Servlet`) 实例上被共享，每个都有自己的子`WebApplicationContext`配置。 See [Additional Capabilities of the ApplicationContext](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#context-introduction) for more on the context hierarchy feature. 

root  `WebApplicationContext` 通常包含基础设施beans比如需要在多个`Servlet`实例共享的data repositories and business services。这些bean实际上是继承的，可以被覆盖 （例如，重声明）在Servlet-specific, child `WebApplicationContext`  ，它特别包含了给定 `Servlet` 的local bean：

![](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/images/mvc-context-hierarchy.png)

下面是带有`WebApplicationContext`层次结构的示例配置：

```
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```

**如果不需要 application context hierarchy  ，应用可以从`getRootConfigClasses()` 得到所有配置，从 `getServletConfigClasses()`获得null。**

等价的`web.xml` ：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

如果不需要application context hierarchy，应用只需要配置 "root" context  ，把 `contextConfigLocation` Servlet  属性留空。

####  1.2.2. Special Bean Types

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-special-bean-types) 

`DispatcherServlet`  把处理请求委托个特殊的beans并呈现合适的响应。"special beans" 我们是说 Spring-managed ，实现WebFlux framework contracts的Object实例。这些通常都有内置的contracts，但是您可以定制它们的属性，扩展或替换它们。 

下表列出 `DispatcherHandler`自动扫描的special beans：

| Bean type                                                    | Explanation                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [HandlerMapping](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-handlermapping) | Map a request to a handler along with a list of [interceptors](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-handlermapping-interceptor) for pre- and post-processing. The mapping is based on some criteria the details of which vary by `HandlerMapping` implementation.The two main `HandlerMapping` implementations are `RequestMappingHandlerMapping` which supports `@RequestMapping` annotated methods and `SimpleUrlHandlerMapping` which maintains explicit registrations of URI path patterns to handlers. |
| HandlerAdapter                                               | Help the `DispatcherServlet` to invoke a handler mapped to a request regardless of how the handler is actually invoked. For example, invoking an annotated controller requires resolving annotations. The main purpose of a `HandlerAdapter`is to shield the `DispatcherServlet` from such details. |
| [HandlerExceptionResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers) | Strategy to resolve exceptions possibly mapping them to handlers, or to HTML error views, or other. See [Exceptions](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers). |
| [ViewResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-viewresolver) | Resolve logical String-based view names returned from a handler to an actual `View` to render to the response with. See [View Resolution](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-viewresolver) and [View Technologies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-view). |
| [LocaleResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-localeresolver), [LocaleContextResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-timezone) | Resolve the `Locale` a client is using and possibly their time zone, in order to be able to offer internationalized views. See [Locale](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-localeresolver). |
| [ThemeResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-themeresolver) | Resolve themes your web application can use, for example, to offer personalized layouts. See [Themes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-themeresolver). |
| [MultipartResolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multipart) | Abstraction for parsing a multi-part request (e.g. browser form file upload) with the help of some multipart parsing library. See [Multipart resolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multipart). |
| [FlashMapManager](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-flash-attributes) | Store and retrieve the "input" and the "output" `FlashMap` that can be used to pass attributes from one request to another, usually across a redirect. See [Flash attributes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-flash-attributes). |

####  1.2.3. Web MVC Config

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-framework-config) 

应用可以声明在 [Special Bean Types](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-servlet-special-bean-types)列出的需要处理requests的 infrastructure beans 。`DispatcherServlet` 对 `WebApplicationContext` 来检查每个 special bean 。如果没有匹配bean类型，它返回到 [DispatcherServlet.properties](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/resources/org/springframework/web/servlet/DispatcherServlet.properties)列出的默认类型。

在大多数情况下， [MVC Config](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config) 是最好的起点。 它在Java或XML中声明所需的bean，并提供一个更高级别的配置回调API来定制它。 

**Spring Boot依赖MVC Java config 来配置Spring MVC ，并提供许多额外的便捷选择。**

#### 1.2.4. Servlet Config

在一个 Servlet 3.0+ 环境，你可以选择编程式配置 Servlet container 作为替代，或者结合`web.xml`文件。下面是一个注册`DispatcherServlet`的示例：

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

`WebApplicationInitializer`  是springmvc提供的接口，确保您的实现被检测到并自动用于初始化任何Servlet 3容器 。一个 `WebApplicationInitializer`的抽象实现基类`AbstractDispatcherServletInitializer`  使注册`DispatcherServlet` 更容易，简单复写方法指定 servlet mapping和 `DispatcherServlet` 配置的位置。

use Java-based Spring configuration 是推荐的：

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

如果使用 XML-based Spring configuration ，你应该直接从 `AbstractDispatcherServletInitializer`扩展：

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

`AbstractDispatcherServletInitializer` 也提供便利方式添加`Filter` 实例并自动映射到 `DispatcherServlet` ：

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

每个filter有一个基于具体类型的name，并自动匹配到 `DispatcherServlet`。

` AbstractDispatcherServletInitializer`的 `isAsyncSupported`  protected method 对在 `DispatcherServlet`和所有匹配的filters上启用异步支持提供了a single place。默认是true。

最后，如果您需要进一步定制`DispatcherServlet`本身 ，重写`createDispatcherServlet`  方法。

#### 1.2.5. Processing

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-dispatcher-handler-sequence) 

`DispatcherServlet` 处理请求如下：

- `WebApplicationContext`在请求中被搜索并绑定，作为控制器和流程中的其他元素可以使用的属性。 他被默认绑定在key `DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`下。
-  locale resolver 被绑定在request上，在处理请求时启用进程中的元素来解析locale（呈现视图，准备数据，等等） 。If you do not need locale resolving, you do not need it. 
- theme resolver 被绑定在request，让视图等元素决定使用哪个主题。 If you do not use themes, you can ignore it. 
- 如果你指定一个multipart file resolver ，请求被检查multiparts；如果找到了multiparts，request被包装到`MultipartHttpServletRequest`  由过程中的其他元素进一步处理。 See [Multipart resolver](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multipart) for further information about multipart handling. 
- 一个合适的handler 被查找。如果找到一个handler，关联handler的执行链（preprocessors, postprocessors, and controllers）被执行来准备model or rendering。或者，对于带注释的控制器，响应可以被呈现（在HandlerAdapter中）而不是返回一个视图。 
- 如果model被返回，视图被呈现。如果没有model返回，（也许是因为preprocessor or postprocessor可能因为安全原因打断了请求），没有视图被呈现，因为请求可能已经被完成了。

`HandlerExceptionResolver`  beans声明在`WebApplicationContext` ，用于解决在请求处理过程中抛出的异常。这些异常的解析器允许定制逻辑来处理异常。 See [Exceptions](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers) for more details. 

spring `DispatcherServlet`  也支持 *last-modification-date* 的返回，是由Servlet API指出的。确定特定请求的最后修改日期的过程非常简单： `DispatcherServlet` 查找一个合适的handler mapping 并且测试被找到的处理程序是否实现了*LastModified*接口 。如果实现了， `LastModified`  接口的 `long getLastModified(request)`  方法的值返回到客户端。

你可以自定义个体`DispatcherServlet`实例，通过添加Servlet initialization parameters （`init-param` elements ）到`web.xml`的Servlet声明。请参阅下面的表，以获得受支持的参数列表。 

*Table 1. DispatcherServlet initialization parameters*

| Parameter                        | Explanation                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `contextClass`                   | Class that implements `WebApplicationContext`, which instantiates the context used by this Servlet. By default, the `XmlWebApplicationContext` is used. |
| `contextConfigLocation`          | String that is passed to the context instance (specified by `contextClass`) to indicate where context(s) can be found. The string consists potentially of multiple strings (using a comma as a delimiter) to support multiple contexts. In case of multiple context locations with beans that are defined twice, the latest location takes precedence. |
| `namespace`                      | Namespace of the `WebApplicationContext`. Defaults to `[servlet-name]-servlet`. |
| `throwExceptionIfNoHandlerFound` | Whether to throw a `NoHandlerFoundException` when no handler was found for a request. The exception can then be caught with a `HandlerExceptionResolver`, e.g. via an`@ExceptionHandler` controller method, and handled as any others.By default this is set to "false", in which case the `DispatcherServlet` sets the response status to 404 (NOT_FOUND) without raising an exception.Note that if [default servlet handling](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-default-servlet-handler) is also configured, then unresolved requests are always forwarded to the default servlet and a 404 would never be raised. |

####  1.2.6. Interception

所有`HandlerMapping` 实现支持handler interceptors ，在你想把特定功能应用到request很有用，比如checking for a principal 。 Interceptors 必须实现`org.springframework.web.servlet`  包的 `HandlerInterceptor` ，有3个方法提供足够的灵活性做各种pre-processing and post-processing: 

- `preHandle(..)` — *before* the actual handler is executed
- `postHandle(..)` — *after* the handler is executed
- `afterCompletion(..)` — *after the complete request has finished*

 `preHandle(..)` 方法返回boolean 值。你可以用这个方法打断和继续执行链的处理。当返回true，handler execution chain 会继续；返回false，`DispatcherServlet`  认为 interceptor自身已经处理了请求（举个例子，给出一个合适的视图 ），不再继续执行其他interceptors 和在 execution chain的具体handler。

See [Interceptors](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-interceptors) in the section on MVC configuration for examples of how to configure interceptors. 你还可以通过个体 `HandlerMapping` 实现上的setter来注册他们。

注意 `postHandle`  对 `@ResponseBody` and `ResponseEntity` methods 是没用的，响应是在HandlerAdapter和postHandle之前编写和提交的。 这意味着对响应进行任何更改都太晚了，比如添加额外的header。对于这样的场景，您可以实现`ResponseBodyAdvice`，或者将它声明为[Controller Advice](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice) bean，或者直接在`RequestMappingHandlerAdapter`上配置它。 

