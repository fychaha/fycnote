# DispatcherServlet

Spring Web 的模型 - 视图 - 控制器（MVC）框架是围绕 DispatcherServlet 设计的，它处理所有的 HTTP 请求和响应。

以下是对应于到 DispatcherServlet 的传入 HTTP 请求的事件顺序：

- 在接收到 HTTP 请求后，DispatcherServlet 会查询 HandlerMapping，并通过 Adapter 调用相应的 Controller 。
- Controller 接受请求并根据使用的 GET 或 POST 方法调用相应的服务方法。 服务方法将基于定义的业务逻辑设置模型数据，并将视图名称返回给 DispatcherServlet 。
- DispatcherServlet 将从 ViewResolver 获取请求的定义视图。当视图完成，DispatcherServlet 将模型数据传递到最终的视图，并在浏览器上呈现。

## 一、必需的配置

需要通过使用 web.xml 文件中的 URL 映射来映射希望 DispatcherServlet 处理的请求。

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

  <!-- 配置 SpringMVC 前端控制器 -->
  <servlet>
    <servlet-name>HelloWeb</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
    <servlet-name>HelloWeb</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
</web-app>
```

## 二、加载配置文件的两个时机

SpringMVC 有两次加载 Spring 配置文件的时机：

首先在 SpringMVC 项目启动时，会依据 web.xml 配置文件中所配置的监听器：`ContextLoaderListener` 去加载对应位置下的 Spring 配置文件。

```xml
<web-app...>
  ...
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      classpath:spring/spring-dao.xml,
      classpath:spring/spring-service.xml
    </param-value>
  </context-param>
  ...
</web-app>
```

无论该监听器有没有配置，那么 Spring MVC 都会继续进入第二个加载配置文件时机，根据 DispatcherServlet 的初始化配置（init-param）加载对应位置的 Spring 配置文件：

```xml
<web-app>
  ...
  <servlet>
    <servlet-name>...</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-web.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    ...
  </servlet-mapping>
</web-app>
```

如果 `<init-param>` 没有配置，那么默认相当于是：

```xml
<init-param>
  <param-name>contextConfigLocation</param-name>
	<param-value>WebContent/WEB-INF/[servlet-name]-servlet.xml</param-value>
</init-param>
```

这里的 Spring 配置文件的路径如果是以 `classpath:` 开始，则表示配置文件是在 classpath 路径下。否则，就是在项目的工程根目录（`webapp`）下。

SpringMVC 首先加载的是 `<context-param>` 配置的内容，而并不会去初始化 servlet。只有进行了网站的跳转，经过了 DispatcherServlet 的导航的时候，才会初始化 servlet，从而加载 `<init-param>` 中的内容。

一般而言，`<context-param>` 配置的 Spring 配置文件，习惯性叫 applicationContext，或 webApplicationContext，表示全局性 Spring 配置。`<init-param>` 配置的 Spring 配置文件可以叫 spring-mvc，表示 spirng-webmvc （web 层）相关的 Spring 配置。


现在来看看 `HelloWeb-servlet.xml` 文件的必需配置，放在 Web 应用程序的 `WebContent/WEB-INF` 目录中：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:mvc="http://www.springframework.org/schema/mvc"
   xsi:schemaLocation="
      http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd
      http://www.springframework.org/schema/mvc
      http://www.springframework.org/schema/mvc/spring-mvc.xsd">

  <mvc:annotation-driven/>

  <context:component-scan base-package="com.xja.web.controller" />

</beans>
```

## 三、定义控制器

`DispatcherServlet` 将请求委派给控制器以执行特定于其的功能。 `@Controller` 注释指示特定类充当控制器的角色。`@RequestMapping` 注释用于将 URL 映射到整个类或特定处理程序方法。

```java
@Controller
public class HelloController {

  @RequestMapping ( value = "/hello", method = RequestMethod.GET )
  public String printHello ( ModelMap model ) {
    model.addAttribute("message", "Hello Spring MVC Framework!");
    return "/WEB-INF/jsp/hello.jsp";
  }
}
```

`@Controller` 注释将类定义为 ***Spring MVC*** 控制器。
`@RequestMapping` 的第一个用法表示此控制器上的所有处理方法都与 `/hello` 路径相关。
`@RequestMapping(method = RequestMethod.GET)` 用于声明 `printHello()` 方法作为控制器的默认服务方法来处理 HTTP GET请求。可以定义另一个方法来处理同一URL的任何 POST 请求。

`value` 属性指示处理程序方法映射到的 URL，`method` 属性定义处理 HTTP GET 请求的服务方法。关于以上定义的控制器，需要注意以下几点：

- 在服务方法中定义所需的业务逻辑。可以根据需要在此方法内调用其他方法。
- 基于定义的业务逻辑，将在此方法中创建一个模型。可以设置不同的模型属性，这些属性将被视图访问以呈现最终结果。此示例创建且有属性 “message” 的模型。
- 定义的服务方法可以返回一个 String，它包含要用于渲染模型的视图的名称。此示例将 “hello” 返回为逻辑视图名称。

## 四、创建 JSP 视图

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/WEB-INF/jsp/" />
  <property name="suffix" value=".jsp" />
</bean>
```

Spring MVC 支持许多类型的视图用于不同的表示技术。包括 - JSP，HTML，PDF，Excel 工作表，XML，Velocity 模板，XSLT，JSON，Atom 和 RSS 源，JasperReports 等。这里使用的是 JSP 模板，并在 `/WEB-INF/hello/hello.jsp` 中写一个简单的 hello 视图：

```xml
<html>
  <head>
    <title>Hello Spring MVC</title>
  </head>
  <body>
    <h2>${message}</h2>
  </body>
</html>
```

这里 `${message}` 是在 Controller 中设置的属性。可以在视图中显示多个属性。
