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

#### 1.2.7. Exceptions

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-dispatcher-exceptions) 

如果在请求映射期间发生异常，或者从请求处理程序中抛出，比如`@Controller`，`DispatcherServlet`  委托给一个`HandlerExceptionResolver` beans链来处理异常和提供替代处理，通常是一个错误响应。 

下表列出可用的`HandlerExceptionResolver`实现：

*Table 2. HandlerExceptionResolver implementations*

##### Chain of resolvers

你可以构成一个exception resolver chain ，通过声明多个 `HandlerExceptionResolver`beans在spring 配置并设置 `order` 属性如果需要的话。order属性越大，exception resolver被定位越晚。

`HandlerExceptionResolver` 的contract 指定了他可以返回的类型：

- 指向一个错误视图的`ModelAndView` 
- 空的`ModelAndView`  ，如果exception 在resolver 内被处理
- `null` ，如果异常还没解决，对后面的resolvers 去尝试；如果异常在最终被保留，允许冒泡到 Servlet container 

 [MVC Config](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config)自动声明了默认Spring MVC异常的内置解决方案，用于`@ResponseStatus`标注的异常，以及支持`@ExceptionHandler`方法。 你可以自定义list或替换它。

##### Container error page

如果异常没有被任何`HandlerExceptionResolver` 处理，因此只能传播，或者如果响应状态被设置为错误状态（即4xx、5xx）， Servlet containers可以渲染默认的错误HTML页面。为了定制容器的默认错误页面，您可以在`web.xml`中声明一个错误页面映射： 

```
<error-page>
    <location>/error</location>
</error-page>
```

如上给出的，当一个异常bubbles up ，或者response有一个error status ，Servlet container 做一个ERROR dispatch 到配置的URL（例如/error）。这然后被`DispatcherServlet`处理，可能将它映射到一个`@Controller`，它可以被实现，用一个模型返回一个错误视图名，或者呈现一个JSON响应，如下所示： 

```java
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```

Servlet API 在Java没有提供一个创建错误页面映射的方式。然而你可以用`WebApplicationInitializer`  和a minimal `web.xml`。

#### 1.2.8. View Resolution

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-viewresolution) 

Spring MVC定义了`ViewResolver` and `View`  接口使你可以在浏览器渲染model，而不需要把你绑定到一个特定的视图技术。`ViewResolver` 在视图names和实际视图之间提供了一个映射。 `View`处理数据的准备工作在将其移交给特定的视图技术之前。

下面在 `ViewResolver`层次提供更多细节：

*Table 3. ViewResolver implementations*

| ViewResolver                     | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `AbstractCachingViewResolver`    | Sub-classes of `AbstractCachingViewResolver` cache view instances that they resolve. Caching improves performance of certain view technologies. It’s possible to turn off the cache by setting the `cache` property to `false`. Furthermore, if you must refresh a certain view at runtime (for example when a FreeMarker template is modified), you can use the `removeFromCache(String viewName, Locale loc)` method. |
| `XmlViewResolver`                | Implementation of `ViewResolver` that accepts a configuration file written in XML with the same DTD as Spring’s XML bean factories. The default configuration file is`/WEB-INF/views.xml`. |
| `ResourceBundleViewResolver`     | Implementation of `ViewResolver` that uses bean definitions in a `ResourceBundle`, specified by the bundle base name, and for each view it is supposed to resolve, it uses the value of the property `[viewname].(class)` as the view class and the value of the property `[viewname].url` as the view url. Examples can be found in the chapter on [View Technologies](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-view). |
| `UrlBasedViewResolver`           | Simple implementation of the `ViewResolver` interface that effects the direct resolution of logical view names to URLs, without an explicit mapping definition. This is appropriate if your logical names match the names of your view resources in a straightforward manner, without the need for arbitrary mappings. |
| `InternalResourceViewResolver`   | Convenient subclass of `UrlBasedViewResolver` that supports `InternalResourceView` (in effect, Servlets and JSPs) and subclasses such as `JstlView` and `TilesView`. You can specify the view class for all views generated by this resolver by using `setViewClass(..)`. See the `UrlBasedViewResolver`javadocs for details. |
| `FreeMarkerViewResolver`         | Convenient subclass of `UrlBasedViewResolver` that supports `FreeMarkerView` and custom subclasses of them. |
| `ContentNegotiatingViewResolver` | Implementation of the `ViewResolver` interface that resolves a view based on the request file name or `Accept` header. See [Content negotiation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multiple-representations). |

##### Handling

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-viewresolution-handling)

你可以通过声明一个或多个resolver beans来chain view resolvers，并且，如果需要的话设置`order` 属性来指定顺序。记住，order属性越大，view resolver 在chain被定位越晚。

`ViewResolver` 的contract指定了它可以返回null表明view找不到。然而在JSPs, and `InternalResourceViewResolver`的情况下，发现JSP存不存在的唯一方式就是通过`RequestDispatcher`执行一个dispatch。因此一个`InternalResourceViewResolver`  必须总是在所有的resolvers中被配置成最后一个。

配置 view resolution和向spring配置添加 `ViewResolver` beans一样简单。 [MVC Config](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config) 对[View Resolvers](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-view-resolvers) 以及对添加没有控制器逻辑的HTML模板呈现 的logic-less [View Controllers](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-view-controller) 提供了专用API。

##### Redirecting

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-redirecting-redirect-prefix) 

在view name的特殊`redirect:` 前缀运行执行重定向。`UrlBasedViewResolver` (and sub-classes)  认识到这是一个需要重定向的指令。余下的视图名是重定向URL。 

 net effect和controller返回一个 `RedirectView`一样，但现在controller自身可以简单在 logical view names方面操作。 logical view names 比如`redirect:/myapp/some/resource` 会重定向到相对于当前Servlet context的路径，而 `redirect:http://myhost.com/some/arbitrary/path` 重定向一个绝对的URL。

注意如果一个controller方法被 `@ResponseStatus`注解，注解值是优先`RedirectView`设置的response status。

#####  Forwarding

也可以对view names用一个特殊的`forward:`前缀，最终被 `UrlBasedViewResolver` and subclasses 解析。这创建一个 `InternalResourceView` 做一个`RequestDispatcher.forward()`。因此，这个前缀对`InternalResourceViewResolver` and `InternalResourceView` (for JSPs) 是没用的，但是使用另外的 view technology会有用，但是仍然希望强制一个资源的转发由servlet/jsp引擎来处理。 注意你也可以使用chain multiple view resolvers 来做。

##### Content negotiation

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-multiple-representations) 

[ContentNegotiatingViewResolver](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/web/servlet/view/ContentNegotiatingViewResolver.html) 自身不解析视图，而是委托给其他视图解析器，并选择与客户端请求的表示类似的视图。 这个表现可以由 `Accept` header或a query parameter决定，比如，`"/path?format=pdf"`。

`ContentNegotiatingViewResolver`选择一个合适的 `View` 来处理请求，通过把request media type(s)和 与 `ViewResolvers` 关联的`View`  支持的media type (also known as `Content-Type`) 进行比对。在list中的第一个有兼容的 `Content-Type`  的`View`  返回给客户端表征。如果兼容视图不能被`ViewResolver`链提供，然后通过`DefaultViews` 指定的views list会被咨询。后一种选择适合于单例视图，它可以呈现当前资源的适当表示，而不考虑逻辑视图名。 `Accept` header 可能包含通配符，例如`text/*`，在这种情况下，其内容类型为text/xml的视图是兼容的。 

See [View Resolvers](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-view-resolvers) under [MVC Config](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config) for configuration details.

#### 1.2.9. Locale

spring架构的大多部分支持国际化，就像 Spring web MVC。`DispatcherServlet` 可以让你自动使用客户端locale解析message。这是 `LocaleResolver`对象做到的。

请求来时，`DispatcherServlet` 查找一个 locale resolver ，如果找到就会设置它为 locale。使用`RequestContext.getLocale()` 得到 locale 。

除了自动 locale resolution ，你也可以把一个interceptor  附加到handler mapping （see [Interception](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-handlermapping-interceptor) for more information on handler mapping interceptors ）在特定情况下来改变locale，例如，基于request中的参数。

Locale resolvers and interceptors 定义在`org.springframework.web.servlet.i18n` 包以普通的方式定义在application context。下面是Spring中包含的locale resolvers的选择。 

#####  TimeZone

除了获得客户端的locale之外，知道它们的time zone也很有用。`LocaleContextResolver`接口扩展 `LocaleResolver` 运行resolvers提供一个更丰富的`LocaleContext`，可能包含time zone信息。

当可用时，用户的`TimeZone` 可以用`RequestContext.getTimeZone()`  获得。time zone信息自动被Spring’s `ConversionService` 注册的`Converter` and `Formatter` objects 使用。

##### Header resolver

这个 locale resolver检查客户端发送请求的`accept-language` header。通常这个header field 包含客户端操作系统locale。*注意，这个解析器不支持timezone信息。* 

#####  Cookie resolver

这个locale resolver检查一个存在在客户端的 `Cookie` 来看 `Locale` or `TimeZone` 有没有被指定。如果指定了，它使用指定的细节。使用locale resolver的属性，你可以指定cookie的名称以及最大年龄。下面是一个定义`CookieLocaleResolver`的例子。 

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

*Table 4. CookieLocaleResolver properties*

| Property     | Default                   | Description                                                  |
| ------------ | ------------------------- | ------------------------------------------------------------ |
| cookieName   | classname + LOCALE        | The name of the cookie                                       |
| cookieMaxAge | Servlet container default | The maximum time a cookie will stay persistent on the client. If -1 is specified, the cookie will not be persisted; it will only be available until the client shuts down their browser. |
| cookiePath   | /                         | Limits the visibility of the cookie to a certain part of your site. When cookiePath is specified, the cookie will only be visible to that path and the paths below it. |

#####  Session resolver

`SessionLocaleResolver` 允许你从和request关联的session接收`Locale` and `TimeZone` 。和 `CookieLocaleResolver`对比，这个策略储存在 Servlet container’s `HttpSession` 的locally chosen locale settings 。因此，这些设置对于每个会话都是临时的，每个session中断就丢失。

注意和额外的session管理机制比如Spring Session project 是没有直接关系的。这个`SessionLocaleResolver`会简单评估和修改在当前`HttpServletRequest`对应的`HttpSession` 属性。

##### Locale interceptor

你可以把 `LocaleChangeInterceptor` 添加到其中之一的 handler mappings （see [[mvc-handlermapping\]](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-handlermapping) ）来启用改变locale。它会检测请求的参数来改变locale。它调用 `LocaleResolver` 上的`setLocale()`  。下例展示了所有 `*.view` 资源的调用包含`siteLanguage` 参数会改变locale。所以，例如，一个请求URL， `http://www.sf.net/home.view?siteLanguage=nl`  ，会把语言改变为Dutch 。

####  1.2.10. Themes

您可以应用Spring Web MVC框架主题来设置应用程序的整体外观和感觉 ，从而提高用户体验。主题是静态资源的集合，通常是样式表和图像，它们会影响应用程序的视觉风格。 

#####  Define a theme

在应用使用主题，必须设置一个`org.springframework.ui.context.ThemeSource` 接口实现。`WebApplicationContext` 接口扩展了`ThemeSource`  但是把职责委托给专门的实现。默认委托给`org.springframework.ui.context.support.ResourceBundleThemeSource`  实现，它从classpath加载属性文件。为了使用自定义`ThemeSource`  实现或者配置`ResourceBundleThemeSource`的base name prefix ，你可以在application context注册一个bean，使用保留name `themeSource`。web application context 自动检测bean并使用。

当使用 `ResourceBundleThemeSource`，一个主题定义在属性文件。属性文件列出构成主题的资源。这是一个例子: 

```
styleSheet=/themes/cool/style.css
background=/themes/cool/img/coolBg.jpg
```

 properties 的key是从视图代码中引用主题元素的name。对于JSP，使用`spring:theme`  自定义tag来做，他和 `spring:message` tag很像。以下JSP fragment使用了先前定义的theme：

```
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code='background'/>">
        ...
    </body>
</html>
```

默认， `ResourceBundleThemeSource` 使用empty base name prefix 。因此，属性文件是从类路径的根部加载的。因此你可以把 `cool.properties` theme definition放在classpath，例如，in `/WEB-INF/classes` ，`ResourceBundleThemeSource`  使用标准java资源包加载机制，允许主题的完全国际化。 例如，我们可以有 `/WEB-INF/classes/cool_nl.properties` 引用一个带有荷兰文本的特殊背景图像 。

##### Resolve themes

在定义主题之后，如前一节中，您将决定使用哪个主题。 `DispatcherServlet`  查找一个`themeResolver` bean找出使用哪个`ThemeResolver` 实现。一个主题解析器的工作方式与`LocaleResolver`的工作方式大致相同。 它检测到用于特定请求的主题，也可以改变请求的主题。 下面的主题解决方案是由Spring提供的： 

*Table 5. ThemeResolver implementations*

| Class                  | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `FixedThemeResolver`   | Selects a fixed theme, set using the `defaultThemeName` property. |
| `SessionThemeResolver` | The theme is maintained in the user’s HTTP session. It only needs to be set once for each session, but is not persisted between sessions. |
| `CookieThemeResolver`  | The selected theme is stored in a cookie on the client.      |

spring也提供`ThemeChangeInterceptor` 允许根据每个带有简单参数的请求改变主题。

#### 1.2.11. Multipart resolver

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-multipart)

 `MultipartResolver`  来自`org.springframework.web.multipart`包，是转化multipart requests包括文件上传的策略。有一个实现基于 [*Commons FileUpload*](https://jakarta.apache.org/commons/fileupload) 另一个基于Servlet 3.0 multipart request parsing。

启用multipart处理，你需要在`DispatcherServlet`  声明 `MultipartResolver` bean，使用name "multipartResolver"。 `DispatcherServlet` 检测它并应用到请求。当POST带有"multipart/form-data"  的 content-type 被接受，resolver 把content 转化并包装当前`HttpServletRequest` 成为`MultipartHttpServletRequest` ，来提供对已解决部分的访问，并将其作为请求参数公开。

#####  Apache FileUpload

使用Apache Commons FileUpload ，简单地配置bean类型为`CommonsMultipartResolver` ，name是`multipartResolver`。当然你要把`commons-fileupload` 依赖添加到classpath。

##### Servlet 3.0

Servlet 3.0 multipart parsing 需要通过Servlet container 配置来启用：

- Java里，在Servlet注册时设置`MultipartConfigElement` 
- 在 `web.xml`，添加`"<multipart-config>"` 到Servlet声明

```
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```

一旦Servlet 3.0配置就位了，简单地添加`StandardServletMultipartResolver` bean类型使用name `multipartResolver`。

### 1.3. Filters

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-filters)

The `spring-web` module provides some useful filters.

#### 1.3.1. HTTP PUT Form

浏览器只能通过HTTP GET or HTTP POST提交数据但是非浏览器还能用HTTP PUT and PATCH。Servlet API 要求`ServletRequest.getParameter*()` 方法支持表单字段只能通过HTTP POST访问。 

`spring-web` 模块提供`HttpPutFormContentFilter` 拦截content type是 `application/x-www-form-urlencoded`的 HTTP PUT and PATCH 请求，从request body读取form data，包装`ServletRequest` 来使表单数据通过ServletRequest.getParameter *()的方法可用。

#### 1.3.2. Forwarded Headers

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-filters-forwarded-headers) 

请求通过代理，比如负载均衡器，host, port, and scheme 可能会变，向需要创建资源链接的应用程序提出挑战因为这些链接应该反映从客户端角度看的原始请求的host, port, and scheme。

[RFC 7239](https://tools.ietf.org/html/rfc7239)对代理定义了"Forwarded" HTTP header ，用来提供原始请求信息。也有其他non-standard headers再用，比如 "X-Forwarded-Host", "X-Forwarded-Port", and "X-Forwarded-Proto"。

`ForwardedHeaderFilter` 检测，提取，和使用来自 "Forwarded" header, or from "X-Forwarded-Host", "X-Forwarded-Port", and "X-Forwarded-Proto"的信息。它包装请求以覆盖它的主机、端口和scheme，并且还“隐藏”转发的头部以进行后续处理。 

注意如Section 8 of RFC 7239解释的，使用forwarded headers有安全考虑。在应用程序级别，很难确定转发的标头是否可信。 这就是为什么应该正确地配置上游网络，以便从外部过滤掉不受信任的转发头。 

没有代理且不需要使用forwarded headers的应用可以配置 `ForwardedHeaderFilter` 来移除和忽略这些headers。

#### 1.3.3. Shallow ETag

`ShallowEtagHeaderFilter` 过滤器创建一个"shallow" ETag 通过caching写入response的content，并从之计算MD5 hash。下次客户端发送时，也一样做，将计算值与If-None-Match请求标头进行比较，如果两者是相等的，则返回304（未修改）。 

这种策略节省了网络带宽，而不是CPU，因为每个请求都必须计算完整的响应。上面描述的控制器级别的其他策略可以避免计算。 See [HTTP Caching](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-caching). 

这个过滤器有一个`writeWeakETag`  参数来配置filter 写Weak ETags ，像这样：`W/"02a2d595e6ed9a0b24f027f2b63b134d6"`, as defined in [RFC 7232 Section 2.3](https://tools.ietf.org/html/rfc7232#section-2.3)。

####  1.3.4. CORS

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-filters-cors)

Spring MVC 通过controller注解对CORS配置提供细粒度支持。然而，当一起使用Spring Security建议依赖内置的`CorsFilter` 一样被排序在Spring Security’s chain of filters之前。

See the section on [CORS](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors) and the [CORS Filter](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors-filter) for more details。

### 1.4. Annotated Controllers

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-controller) 

Spring MVC 提供基于注解的编程模型在 `@Controller` and `@RestController` 组件使用注解来表达request mappings, request input, exception handling 等等。带注释的Controller具有灵活的方法签名，不必扩展基类，也不必实现特定的接口。 

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

在这个特定的例子中，这个方法接受一个模型并返回一个视图名作为一个字符串，但是有许多其他的选项，并且在这一章中进一步解释。 

**Guides and tutorials on [spring.io](https://spring.io/guides) use the annotation-based programming model described in this section.** 

####  1.4.1. Declaration

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-controller)

To enable auto-detection of such `@Controller` beans, you can add component scanning to your Java configuration:

```
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

The XML configuration equivalent:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```

`@RestController` 是一个 [composed annotation](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-meta-annotations)，自身用`@Controller` and `@ResponseBody` 注解表示每个方法继承了type-level `@ResponseBody`注解，直接写入 response body 。

#####  AOP proxies

一些情况下，controller 运行时可能需要被AOP代理装饰。一个例子是直接在controller使用`@Transactional` 。在这种情况下，对于controller来说，我们建议使用基于类的代理 。这通常是controller的默认选择。然而如果一个controller必须实现不是Spring Context callback的接口（`InitializingBean`, `*Aware`等），你需要显示配置class-based 代理。例如对于`<tx:annotation-driven/>`，变成`<tx:annotation-driven proxy-target-class="true"/>`。

#### 1.4.2. Request Mapping

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping)

匹配请求到方法，通过URL, HTTP method, request parameters, headers, and media types匹配。可在类和方法上。

`@RequestMapping`还有HTTP方法特定的快捷方式 ：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping``@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

以上是 [Custom Annotations](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-composed) ，开箱即用，同样，在类级别上仍然需要`@RequestMapping`来表达共享映射。 

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

##### URI patterns

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-uri-templates) 

你可以使用glob patterns and wildcards映射请求：

- `?` matches one character
- `*` matches zero or more characters within a path segment
- `**` match zero or more path segments

你也可以声明URI变量，用 `@PathVariable`访问：

```
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

URI变量可被定义在类/方法级别：

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI变量自动转换为合适类型或者`TypeMismatchException`。简单类型，`int`, `long`, `Date`默认支持，您可以为任何其他数据类型注册支持。 See [Type Conversion](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-typeconversion) and [DataBinder](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-initbinder). 

URI变量可以显式地命名——例如. `@PathVariable("customId")` ，但是，如果名称是相同的，并且您的代码是用调试信息编译的，或者在Java 8上使用-参数编译器标志，那么您可以将这些细节保留下来。 

`{varName:regex}` 语法声明正则表达式——例如，给定URL`"/spring-web-3.0.5 .jar"`，下面的方法提取名称、版本和文件扩展名： 

```
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI path patterns 也可以有嵌入的`${…}` placeholders ，在startup时，通过 `PropertyPlaceHolderConfigurer`在local, system, environment, and other property sources解析。这可以用来举例说明基于某些外部配置的基本URL。 

**spring mvc 使用`PathMatcher` contract 和 `spring-core`  的`AntPathMatcher` 实现来进行 URI path matching。**

##### Pattern comparison

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-pattern-comparison)

当多个patterns 匹配URL，必须被比较找到最好的匹配。这是`AntPathMatcher.getPatternComparator(String path)` 做的，找到一个更具体的patterns 。

一个pattern如果URI变量更少那么就更不具体。单通配符计数为1，2个通配符计数为2。给出相同的分数，较长的被选择。分数和长度都一样，更多URI变量的被选择。

默认匹配pattern `/**`  排除在打分外，总是排最后一个。并且前缀pattern比如 `/public/**`比其他没有两个通配符的被认为更不具体。

For the full details see `AntPatternComparator` in `AntPathMatcher` and also keep mind that the `PathMatcher` implementation used can be customized. See [Path Matching](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-path-matching) in the configuration section. 

##### Suffix match

##### Suffix match and RFD

Check [CVE-2015-5211](https://pivotal.io/security/cve-2015-5211) for additional recommendations related to RFD.

##### Consumable media types

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-consumes)

You can narrow the request mapping based on the `Content-Type` of the request:

```
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

The consumes attribute also supports negation expressions — e.g. `!text/plain` means any content type other than "text/plain". 

您可以在类级别声明共享消费属性。与大多数其他请求映射属性不同的是，当在类级别使用时，方法级消耗属性将覆盖类级别声明。 

MediaType为常用的媒体类型提供常量，例如`APPLICATION_JSON_VALUE`, `APPLICATION_XML_VALUE`. 

##### Producible media types

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-produces)

You can narrow the request mapping based on the `Accept` request header and the list of content types that a controller method produces:

```
@GetMapping(path = "/pets/{petId}", produces = "application/json;charset=UTF-8")
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

媒体类型可以指定一个字符集。 支持否定表达： e.g. `!text/plain` means any content type other than "text/plain". 

**对于JSON内容类型，即使RFC7159清楚地声明“没有为这个注册定义charset参数”，也应该指定UTF-8 charset，因为有些浏览器需要它来正确地解释UTF-8特殊字符。** 

##### Parameters, headers

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-params-and-headers)

您可以根据请求参数条件缩小请求映射。 您可以测试请求参数`（“myParam”）`的存在 ,或者不存在(`"!myParam"`) ，或指定值 (`"myParam=myValue"`): 

```
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue")
public void findPet(@PathVariable String petId) {
    // ...
}
```

您还可以使用相同的请求头条件： 

```
@GetMapping(path = "/pets", headers = "myHeader=myValue")
public void findPet(@PathVariable String petId) {
    // ...
}
```

你可以用header 条件匹配`Content-Type` and `Accept`  ，但最好使用use [consumes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-consumes) and [produces](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-produces) 。

##### HTTP HEAD, OPTIONS

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-head-options)

`@GetMapping` 和`@RequestMapping(method=HttpMethod.GET)`，支持HTTP头部透明地进行请求映射。Controller方法不需要改变。一个response wrapper，被应用在`javax.servlet.http.HttpServlet`，确保`"Content-Length"` header 被设置写入的字节数，而不需要实际写入响应。 

`@GetMapping` 和 `@RequestMapping(method=HttpMethod.GET)` ，隐式映射到并支持HTTP HEAD。 一个HTTP HEAD请求被处理，就好像它是HTTP GET一样，但是不是写body，而是计算字节数和 "Content-Length" header set。 

默认下，HTTP OPTIONS通过设置"Allow"响应头到列出在所有带有URL pattern的`@RequestMapping` 方法HTTP methods的list。

对于`@RequestMapping`没有HTTP方法声明， "Allow"头被设置为"GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS" 。Controller方法应该总是声明支持的HTTP方法，例如使用变体—— `@GetMapping`, `@PostMapping`, etc。

`@RequestMapping`方法可以被显式映射到HTTP HEAD and HTTP OPTIONS ，但一般情况没必要。

##### Custom Annotations

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#mvc-ann-requestmapping-head-options)

spring mvc对 request mapping 支持 [composed annotations](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans-meta-annotations) 。

#### 1.4.3. Handler Methods

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-methods)

`@RequestMapping`  handler方法有一个灵活的签名，可以从一系列受支持的Controller方法参数和返回值中进行选择。 

##### Method Arguments

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-arguments)

下表列出支持的controller method arguments。任何参数都不支持Reactive types。 

JDK 8的 `java.util.Optional` 作为方法参数被支持，结合注释有一个必须的属性，比如`@RequestParam`, `@RequestHeader`, etc，等价于`required=false`。

| Controller method argument                                   | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `WebRequest`, `NativeWebRequest`                             | Generic access to request parameters, request & session attributes, without direct use of the Servlet API. |
| `javax.servlet.ServletRequest`, `javax.servlet.ServletResponse` | Choose any specific request or response type — e.g. `ServletRequest`, `HttpServletRequest`, or Spring’s `MultipartRequest`, `MultipartHttpServletRequest`. |
| `javax.servlet.http.HttpSession`                             | Enforces the presence of a session. As a consequence, such an argument is never `null`. **Note:** Session access is not thread-safe. Consider setting the`RequestMappingHandlerAdapter`'s "synchronizeOnSession" flag to "true" if multiple requests are allowed to access a session concurrently. |
| `javax.servlet.http.PushBuilder`                             | Servlet 4.0 push builder API for programmatic HTTP/2 resource pushes. Note that per Servlet spec, the injected `PushBuilder` instance can be null if the client does not support that HTTP/2 feature. |
| `java.security.Principal`                                    | Currently authenticated user; possibly a specific `Principal` implementation class if known. |
| `HttpMethod`                                                 | The HTTP method of the request.                              |
| `java.util.Locale`                                           | The current request locale, determined by the most specific `LocaleResolver`available, in effect, the configured `LocaleResolver`/`LocaleContextResolver`. |
| `java.util.TimeZone` + `java.time.ZoneId`                    | The time zone associated with the current request, as determined by a `LocaleContextResolver`. |
| `java.io.InputStream`, `java.io.Reader`                      | For access to the raw request body as exposed by the Servlet API. |
| `java.io.OutputStream`, `java.io.Writer`                     | For access to the raw response body as exposed by the Servlet API. |
| `@PathVariable`                                              | For access to URI template variables. See [URI patterns](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-uri-templates). |
| `@MatrixVariable`                                            | For access to name-value pairs in URI path segments. See [Matrix variables](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-matrix-variables). |
| `@RequestParam`                                              | For access to Servlet request parameters. Parameter values are converted to the declared method argument type. See [@RequestParam](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestparam).Note that use of `@RequestParam` is optional, e.g. to set its attributes. See "Any other argument" further below in this table. |
| `@RequestHeader`                                             | For access to request headers. Header values are converted to the declared method argument type. See [@RequestHeader](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestheader). |
| `@CookieValue`                                               | For access to cookies. Cookies values are converted to the declared method argument type. See [@CookieValue](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-cookievalue). |
| `@RequestBody`                                               | For access to the HTTP request body. Body content is converted to the declared method argument type using `HttpMessageConverter`s. See [@RequestBody](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestbody). |
| `HttpEntity<B>`                                              | For access to request headers and body. The body is converted with `HttpMessageConverter`s. See [HttpEntity](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-httpentity). |
| `@RequestPart`                                               | For access to a part in a "multipart/form-data" request. See [Multipart](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-multipart-forms). |
| `java.util.Map`, `org.springframework.ui.Model`, `org.springframework.ui.ModelMap` | For access to the model that is used in HTML controllers and exposed to templates as part of view rendering. |
| `RedirectAttributes`                                         | Specify attributes to use in case of a redirect — i.e. to be appended to the query string, and/or flash attributes to be stored temporarily until the request after redirect. See [Redirect attributes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-redirecting-passing-data) and [Flash attributes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-flash-attributes). |
| `@ModelAttribute`                                            | For access to an existing attribute in the model (instantiated if not present) with data binding and validation applied. See [@ModelAttribute](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-modelattrib-method-args) as well as [Model](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-modelattrib-methods) and [DataBinder](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-initbinder).Note that use of `@ModelAttribute` is optional, e.g. to set its attributes. See "Any other argument" further below in this table. |
| `Errors`, `BindingResult`                                    | For access to errors from validation and data binding for a command object (i.e. `@ModelAttribute` argument), or errors from the validation of an `@RequestBody` or`@RequestPart` arguments; an `Errors`, or `BindingResult` argument must be declared immediately after the validated method argument. |
| `SessionStatus` + class-level `@SessionAttributes`           | For marking form processing complete which triggers cleanup of session attributes declared through a class-level `@SessionAttributes` annotation. See[@SessionAttributes](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-sessionattributes) for more details. |
| `UriComponentsBuilder`                                       | For preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping also taking into account `Forwarded` and `X-Forwarded-*` headers. See [URI Links](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-uri-building). |
| `@SessionAttribute`                                          | For access to any session attribute; in contrast to model attributes stored in the session as a result of a class-level `@SessionAttributes` declaration. See[@SessionAttribute](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-sessionattribute) for more details. |
| `@RequestAttribute`                                          | For access to request attributes. See [@RequestAttribute](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-requestattrib) for more details. |
| Any other argument                                           | If a method argument is not matched to any of the above, by default it is resolved as an `@RequestParam` if it is a simple type, as determined by[BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-), or as an `@ModelAttribute` otherwise. |

##### Return Values

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-return-types)

下面的表格显示了受支持的controller方法返回值。 Reactive types对所有的返回值都可以支持。



| Controller method return value                               | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@ResponseBody`                                              | The return value is converted through `HttpMessageConverter`s and written to the response. See [@ResponseBody](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-responsebody). |
| `HttpEntity<B>`, `ResponseEntity<B>`                         | The return value specifies the full response including HTTP headers and body be converted through `HttpMessageConverter`s and written to the response. See [ResponseEntity](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-responseentity). |
| `HttpHeaders`                                                | For returning a response with headers and no body.           |
| `String`                                                     | A view name to be resolved with `ViewResolver`'s and used together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (see above). |
| `View`                                                       | A `View` instance to use for rendering together with the implicit model — determined through command objects and `@ModelAttribute` methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (see above). |
| `java.util.Map`, `org.springframework.ui.Model`              | Attributes to be added to the implicit model with the view name implicitly determined through a `RequestToViewNameTranslator`. |
| `@ModelAttribute`                                            | An attribute to be added to the model with the view name implicitly determined through a `RequestToViewNameTranslator`.Note that `@ModelAttribute` is optional. See "Any other return value" further below in this table. |
| `ModelAndView` object                                        | The view and model attributes to use, and optionally a response status. |
| `void`                                                       | A method with a `void` return type (or `null` return value) is considered to have fully handled the response if it also has a `ServletResponse`, or an `OutputStream` argument, or an `@ResponseStatus` annotation. The same is true also if the controller has made a positive ETag or lastModified timestamp check (see [Controllers](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-caching-etag-lastmodified) for details).If none of the above is true, a `void` return type may also indicate "no response body" for REST controllers, or default view name selection for HTML controllers. |
| `DeferredResult<V>`                                          | Produce any of the above return values asynchronously from any thread — e.g. possibly as a result of some event or callback. See [Async Requests](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async) and[`DeferredResult`](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async-deferredresult). |
| `Callable<V>`                                                | Produce any of the above return values asynchronously in a Spring MVC managed thread. See [Async Requests](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async) and [`Callable`](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async-callable). |
| `ListenableFuture<V>`,`java.util.concurrent.CompletionStage<V>`,`java.util.concurrent.CompletableFuture<V>` | Alternative to `DeferredResult` as a convenience for example when an underlying service returns one of those. |
| `ResponseBodyEmitter`, `SseEmitter`                          | Emit a stream of objects asynchronously to be written to the response with`HttpMessageConverter`'s; also supported as the body of a `ResponseEntity`. See [Async Requests](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async) and [HTTP Streaming](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async-http-streaming). |
| `StreamingResponseBody`                                      | Write to the response `OutputStream` asynchronously; also supported as the body of a `ResponseEntity`. See [Async Requests](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async) and [HTTP Streaming](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async-http-streaming). |
| Reactive types — Reactor, RxJava, or others via `ReactiveAdapterRegistry` | Alternative to `DeferredResult` with multi-value streams (e.g. `Flux`, `Observable`) collected to a `List`.For streaming scenarios — e.g. `text/event-stream`, `application/json+stream` —  `SseEmitter` and `ResponseBodyEmitter` are used instead, where `ServletOutputStream` blocking I/O is performed on a Spring MVC managed thread and back pressure applied against the completion of each write.See [Async Requests](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async) and [Reactive types](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-async-reactive-types). |
| Any other return value                                       | If a return value is not matched to any of the above, by default it is treated as a view name, if it is `String` or `void` (default view name selection via`RequestToViewNameTranslator` applies); or as a model attribute to be added to the model, unless it is a simple type, as determined by[BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-) in which case it remains unresolved. |

##### Type Conversion

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-typeconversion)

一些被注解的Controller方法参数表明了String-based的请求输入——例如 `@RequestParam`, `@RequestHeader`, `@PathVariable`, `@MatrixVariable`, and `@CookieValue`，如果将参数声明为除字符串以外的其他东西，则需要类型转换。 

对于这种情况，类型转换是根据配置的转换器自动应用的。 默认情况下，支持诸如int、long、Date等简单类型。类型转换可以通过`WebDataBinder`进行定制 ，see [DataBinder](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-initbinder) ，或通过`FormattingConversionService`注册`Formatters`， see [Spring Field Formatting](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#format)。

##### Matrix variables

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-matrix-variables)

[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3) 讨论了在path片段上的键值对。在spring mvc我们称之为"matrix variables"，基于 Tim Berners-Lee 提出的 an ["old post"](https://www.w3.org/DesignIssues/MatrixURIs.html) ，但它们也可以被称为URI path parameters 。

Matrix变量可以出现在任何path片段，每个变量由分号分隔，多个值由逗号分隔。例如，`"/cars;color=red,green;year=2012"`。多个值也可以通过多个重复变量名来指定，比如`"color=red;color=green;color=blue"`。

如果一个URL期望包含 matrix variables，controller方法的request mapping，必须使用一个URI variable来mask variable content并且确保请求能够独立于matrix variable order and presence而成功匹配。下面是例子：

```
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

考虑到所有的路径段可能包含矩阵变量，有时你需要消除歧义，matrix变量期望在哪个path片段，例如：

```
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

一个matrix变量可以定义成optional和指定默认值：

```
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

得到所有matrix variables，使用`MultiValueMap`：

```
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

注意，您需要启用矩阵变量。 在MVC Java config ， 你需要通过 [Path Matching](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-path-matching)设置`UrlPathHelper` 的`removeSemicolonContent=false` 。在 MVC XML namespace ，使用`<mvc:annotation-driven enable-matrix-variables="true"/>`。

##### @RequestParam

[Same in Spring WebFlux](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-ann-requestparam) 

使用`@RequestParam`  把Servlet request parameters 绑定到controller方法参数。

下面的代码片段显示了用法： 

```
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

方法参数使用整个注解默认是required的，但是你可以设置为optional通过`required=false`或者用`java.util.Optional`包装来声明参数。

如果目标方法参数类型不是字符串，类型转换就会自动应用。 See [Type Conversion](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-ann-typeconversion). 

当 `@RequestParam` 被声明为 `Map<String, String>` 或`MultiValueMap<String, String>` 参数，map填充了所有的请求参数。 

注意 `@RequestParam` 的使用是optional，例如设置它的属性。默认任何参数是简单值类型，如 [BeanUtils#isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.0.7.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-) 决定，并没有被任何其他的参数解析器解决，就像用@requestparam注释一样。 