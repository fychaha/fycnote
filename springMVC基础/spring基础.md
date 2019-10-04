## springmvc原理与工作流程

### SpringMVC主要包含一下组件

- DispatcherServlet-前端控制器
- HandlerMapping-处理器映射
- Controller-控制器
- ViewResolver-视图解析器
- View-视图

### Springmvc的请求流程

1. 首先请求到达dispatcherServlet,实际上是这个servlet把我们的请求拦截了下来。
2. 然后调用HandlerMapping(处理器映射器)，把请求交给它来处理。
3. HandlerMapping根据收到的请求url进行寻找对应的处理器(就是我们编写的Controller类),并生成该处理器的对象，然后返回给dispatcherServlet。
4. DispatcherServlet收到处理器对象后，将它再传给(HandlerAdapter)处理器适配器。
5. HandlerAdapter拿到后进行适配并调用对应的Controller(Handler)。
6. Controller执行完后，返回一个ModeAndView对象，此时HandlerAdapter拥有了一个ModelAndView对象。
7. HandlerAdapter把拿到的这个对象返回给DispatcherServlet。
8. DispatcherServlet拿到ModelAndView对象后选择合适的视图解析器(ViewResovler)。
9. 视图解析器对收到的 ModelAndView对象进行解析，解析出里面的(视图)View对象并返回给DispathcerServlet。
10. DispatcherServlet 对 View 进行渲染。即，将模型数据填充至视图中。
11. DispatcherServlet 将渲染后的结果返回/发送给客户端浏览器，呈现画面。

