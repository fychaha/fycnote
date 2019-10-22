## Spring

### 什么是Spring？

Spring是个包含一系列功能的集合，IOC与AOP是spring的核心。

### springMVC的流程

1. 客户端发送请求，DispactherServlet拦截器拦截请求并交给HandlerMapping.
2. HandlerMapping收到请求url后进行映射寻找对应的处理器。
3. 把寻找到的处理器对象教给DispactherServlet，由它来调用（处理器）controller
4. 处理器执行我们编写好的业务代码，将返回值ModeAndView交给DispactherServlet。
5. DispactherServlet拿到ModelAndView对象后。转交给视图解析器。
6. 视图解析器对收到的对象进行解析，解析出的view对象（视图），，返回DispactherServlet。
7. DispactherServlet对view进行渲染。将模型数据填充入视图中。
8. 渲染结束后将结果返回给客户端浏览器，呈现画面。

### 如何解决循环依赖？

- 构造器参数循环依赖:spring容器会把每一个正在创建的Bean标识符放在一个“当前创建Bean池”中，这个标识符在创建过程中将一直保持。一旦有Bean在创建过程中发现自己已经正在“当前创建Bean池”中，将会抛出BeanCurrentlyInCreationException来表示发生了循环依赖；对于创建完毕的Bean将从池中清除掉。
- setter循环依赖：setter注入方式，通过Spring容器提前暴露刚完成构造器注入但未完成其他步骤的bean来完成。

### Bean的作用域

- single：单例模式，spring Ioc容器只会存在一个共享的bean实例，无论多少个bean引用他，始终只想同一个对象。
- prootype：原型模式，每次通过spring容器获取prototype定义的bean时，容器都将创建一个新的bean实例，每个bean实例都有自己的属性和状态。
- request：在一次http请求中，容器会返回该bean的同一实例。而对不同的http请求则产生新的bean，而且该bean实例仅在当前session中有效。
- session：
- global session：

#### IOC（DI）

控制反转：原来是自己主动去new一个对象，现在由容器工具配置文件创建实例让自己用。

依赖注入：在运行过程当中当你需要这个对象时才给你实例化并注入，不需要管什么时候注入，只需要写好成员变量和set方法就行。