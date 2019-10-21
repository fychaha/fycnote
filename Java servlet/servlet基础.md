# serlvet基础

1. ### 概念

    Servlet（Server Applet），全称Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。

2. ### servlet的工作模式

   - 客户端发起请求至服务器。
   - 服务器启动并调用servlet，servletg根据客户端请求生成响应内容并将其传给服务器。
   - 服务器将相应返回给客户端。

3. ### servlet api 概览

   1.javax.servlet   其中包含定义servlet和servlet容器之间契约的类和接口。

   2.javax.servlet.http   其中包含定义HTTP Servlet 和Servlet容器之间的关系。

   3.javax.servlet.annotation   其中包含标注servlet，Filter,Listener的标注。它还为被标注元件定义元数据。

   4.javax.servlet.descriptor，其中包含提供程序化登录Web应用程序的配置信息的类型。

4. ### servlet原理

   Servlet 技术的核心是 Servlet 接口，它是所有 Servlet 类必须直接或间接实现的接口。

   ```java
   public class AServlet implements Servlet { ... }
   public class BServlet extends GenericServlet { ... }
   public class CServlet extends HttpServlet { ... }
   ```

   *注意，Servlet 容器有且不仅只有 Tomcat 一种。*

   Servlet 接口定义了 Servlet 类和 Servlet 容器（例如 Tomcat）之间的契约。这个契约归结起来就是，**Tomcat 将 Servlet 类载入内存，并由 Tomcat 调用 Servlet 对象的具体的方法。**<small>这些方法所需的参数也是由 Tomcat 准备并传入的</small>

   在 web 项目运行期间，每个 Servlet 类最多只能有一个对象。

   用户请求致使 Servlet 容器调用 Servlet 的 `service()` 方法，并传入一个 `ServletRequest` 实例和一个 `ServletResponse` 实例。

   - ServletRequest 中封装了当前的 HTTP 请求，因此，Servlet 开发人员不必解析和操作原始的 HTTP 数据。
   - ServletResponse 表示对当前用户的 HTTP 响应，它使得将响应发回给用户变得十分容易。

   对于每一个 WebApp，Servlet 容器还会创建一个 `ServletContext` 实例。这个实例中封装了上下文（WebApp）的环境详情。每个 WebApp 只有一个 ServletContext 实例。

   每个 Servlet 实例也都有一个封装 Servlet 配置的 `ServletCongfig` 。

   简而言之，一个 WebApp 在运行时，有：

   - 1 个 ServletContext 实例 
   - N 个 Servlet 实例 <small>（取决于 Servlet 类的数量）</small>
   - N 个 ServletConfig 实例 <small>（取决于 Servlet 类的数量）</small>
   - 任意个 HTTPRequest / HTTPResponse 实例 <small>（取决于用户请求的次数）</small>

   ### Servlet / GenericServlet / HttpServlet

   - Servlet 是个接口，是 Servlet 体系中的最顶层。
   - GenericServlet 是 Servlet 的实现类，它是一个抽象类。
     - HttpServlet 是 GenericServlet 的子类。

5. ### servlet接口

   init、service 和 destroy 方法是 Servlet 的生命周期方法。

   - `init()` 方法在该 Servlet 第一次被请求时，被 Servlet 容器调用。调用该方法时，容器会传入一个 ServletConfig 对象。
   - `service()` 方法在每次用户发起请求时，被容器调用。调用该方法时，容器会传入代表用户请求和相应的 HTTPRequest 对象和 HTTPResponse 对象。
   - `destroy()` 方法在销毁 Servlet 时，被容器调用。一般发生在卸载WebApp或关闭容器时。

   Servlet 中另外两个方法是非生命周期方法。

   - `getServletInfo()`，这个方法返回一个用于描述 Servlet 的字符串。
   - `getServletConfig()`，这个方法用于返回由 Servlet 传给 `init()` 方法的 ServletConfig 对象。

6. ### HTTPServlet

   尽管 GenericServlet 抽象类为我们提供了方便，但它仍不是最常用的办法。HttpServlet 类扩展了 GenericServlet 类，另外它还使用了 ServletRequest/ServletResponse 的子类：HTTPServletRequest/HTTPServletResponse。

   HTTPServlet 有两个特性是 GenericServlet 所不具备的：

   不用覆盖 `service()` 方法，而覆盖 `doGet()` 或者 `doPost()` 方法。
   使用 HttpServletRequest/HttpServletResponse，而非 ServletRequest/ServletResponse。

7. ###  ServletResponse 和 HTTPServletResponse

   Tomcat 在调用 Servlet 的 `service()` 方法前，容器首先会创建一个 ServletResponse 对象，并将它作为第二个参数传给 `service()` 方法。

   ServletResponse 隐藏了向浏览器发送响应的复杂过程。

   在 ServletResponse 所有的方法中，最常用的方法之一是 `getWriter()` 方法，它返回一个可以向客户端发送文本的 `java.io.PrintWriter` 对象。默认情况下，PrintWriter 对象使用 ISO-8859-1 编码。

   注意，有另一个向浏览器发送数据的方法叫 `getOutputStream()`，但这个方法是用于发送二进制数据的。因此大多数情况下使用的是 `getWriter()`，而非 `getOutPutStream()`。不要调用错了方法。

   在向客户端发送响应时，大多数时候是将它作为 HTML 发送。在发送任何 HTML 标签前，应该先调用 `setContentType()` 方法，设置响应的内容类型，并将“text/html”作为参数传入。这是告诉浏览器，所发送给它的数据内容是 HTML 格式内容。

   如果你的 Servlet 是继承自 HttpServlet，那么 tomcat 传入 doGet 和 doPost 方法的就是 HttpServletResponse 对象，<small>而非 ServletResponse 对象。</small>

   HTTPServletResponse 实现并扩展了 ServletResponse 接口。

   HTTPServletResponse 扩展的常用方法有：

   - void addCookie ( Cookie cookie )
   - void sendRedirect ( String location )