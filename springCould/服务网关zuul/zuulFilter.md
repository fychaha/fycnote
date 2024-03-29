## zuulFilter过滤器

1. ## Zuul的核心(Filter)

   Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR”，整个生命周期可以用下图来表示。

   ![img](http://favorites.ren/assets/images/2018/springcloud/zuul-core.png)

   Zuul大部分功能都是通过过滤器来实现的，这些过滤器类型对应于请求的典型生命周期。

   - **PRE：** 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。

   - **ROUTING：**这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netfilx Ribbon请求微服务。

   - **POST：**这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

   - **ERROR：**在其他阶段发生错误时执行该过滤器。 除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。

     

   ### Zuul中默认实现的Filter

   | 类型  | 顺序 | 过滤器                  | 功能                       |
   | :---- | :--- | :---------------------- | :------------------------- |
   | pre   | -3   | ServletDetectionFilter  | 标记处理Servlet的类型      |
   | pre   | -2   | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
   | pre   | -1   | FormBodyWrapperFilter   | 包装请求体                 |
   | route | 1    | DebugFilter             | 标记调试标志               |
   | route | 5    | PreDecorationFilter     | 处理请求上下文供后续使用   |
   | route | 10   | RibbonRoutingFilter     | serviceId请求转发          |
   | route | 100  | SimpleHostRoutingFilter | url请求转发                |
   | route | 500  | SendForwardFilter       | forward请求转发            |
   | post  | 0    | SendErrorFilter         | 处理有错误的请求响应       |
   | post  | 1000 | SendResponseFilter      | 处理正常的请求响应         |

   **禁用指定的Filter**

   可以在application.yml中配置需要禁用的filter，格式：

   ```yml
   zuul:
   	FormBodyWrapperFilter:
   		pre:
   			disable: true
   ```

   ## 自定义Filter

   实现自定义Filter，需要继承ZuulFilter的类，并覆盖其中的4个方法。

   ```java
   public class MyFilter extends ZuulFilter {
       @Override
       String filterType() {
           return "pre"; //定义filter的类型，有pre、route、post、error四种
           /** pre :代表在网关路由之前进行过滤
           	route：代表在网关路由时进行过滤
           	post：路由后
           	error：
           
           
           **/
       }
   
       @Override
       int filterOrder() {
           return 10; //定义filter的顺序，数字越小表示顺序越高，越先执行
       }
   
       @Override
       boolean shouldFilter() {
           return true; //表示是否需要执行该filter，true表示执行，false表示不执行
           //相当于这个过滤器的开关。
       }
   
       @Override
       Object run() {
           return null; //filter需要执行的具体操作
       }
   }
   ```

   ## 自定义Filter示例(类单点登录)

     我们假设有这样一个场景，因为服务网关应对的是外部的所有请求，为了避免产生安全隐患，我们需要对请求做一定的限制，比如请求中含有Token便让请求继续往下走，如果请求不带Token就直接返回并给出提示。

   首先自定义一个Filter，在run()方法中验证参数是否含有Token。

   ```java
   	public class TokenFilter extends ZuulFilter {
   
       private final Logger logger = LoggerFactory.getLogger(TokenFilter.class);//日志
   
       @Override
       public String filterType() {
           return "pre"; // 可以在请求被路由之前调用
       }
   
       @Override
       public int filterOrder() {
           return 0; // filter执行顺序，通过数字指定 ,优先级为0，数字越大，优先级越低
       }
   
       @Override
       public boolean shouldFilter() {
           return true;// 是否执行该过滤器，此处为true，说明需要过滤
       }
   
       @Override
       public Object run() {
           RequestContext ctx = RequestContext.getCurrentContext();
           HttpServletRequest request = ctx.getRequest();
   
           logger.info("--->>> TokenFilter {},{}", request.getMethod(), request.getRequestURL().toString());
   
           String token = request.getParameter("token");// 获取请求的参数
   
           if (StringUtils.isNotBlank(token)) {
               ctx.setSendZuulResponse(true); //收到了携带的token，对请求进行路由
               ctx.setResponseStatusCode(200);
               ctx.set("isSuccess", true);
               return null;
           } else {
               ctx.setSendZuulResponse(false); //不对其进行路由
               ctx.setResponseStatusCode(400);
               ctx.setResponseBody("token is empty");
               ctx.set("isSuccess", false);
               return null;
           }
       }
   
   }
   ```

   

   

   ------

   

   将TokenFilter加入到请求拦截队列，在启动类中添加以下代码：

   ```java
   @Bean
   public TokenFilter tokenFilter() {
   	return new TokenFilter();
   }
   ```

   这样就将我们自定义好的Filter加入到了请求拦截中。

   ### 或者：

   为TokenFilter增加注解`@Component`

   ```java
   @Component
   public class TokenFilter extends ZuulFilter {
   }
   ```

   `@ComponentScan`会自动扫描包下所有`@Component`组件，转换为bean注入到容器中

