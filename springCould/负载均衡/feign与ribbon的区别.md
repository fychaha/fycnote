# 负载均衡之feign与ribbon的比较

####  	常规的微服务有两种类型：一种是基于dubbo的微服务架构、另外一种是基于Spring Cloud的微服务架构。从概念上来讲，Dubbo和Spring Cloud并不能放在一起对比，因为Dubbo仅仅是一个RPC框架，实现Java程序的远程调用，实施服务化的中间件则需要自己开发；而Spring Cloud则是实施微服务的一系列套件，包括：服务注册与发现、断路器、服务状态监控、配置管理、智能路由、一次性令牌、全局锁、分布式会话管理、集群状态管理等。

## 关于做负载均衡与远程服务调动这块是用feign还是ribbon与restTemplate？

#### ribbon的负载均衡写法：

注入restTemplatebean，并加上 @LoadBalanced注解添加负载均衡功能。

````java
 	@Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate(); //默认使用URLConnection
    }
````

自己构建http请求，模拟http请求然后使用RestTemplate发送给其他服务

````java
@RestController
@RequestMapping("/hello")
public class HelloController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/{name}")
    public Member hello(@PathVariable String name){
        return  restTemplate.getForObject("http://sayHello/hello/"+name,String.class);
    }
}
````

#### 使用feign来实现远程调用和负载均衡写法：

编写接口，加上注解指定要调用的服务名@FeignClient(name= "spring-cloud-producer") ，此接口中的方法和远程服务(服务时生产者)中contoller中的方法名和参数需保持一致。

`````java
@FeignClient(name= "spring-cloud-producer")  //name:远程服务名，及spring.application.name配置的名称,就是指定要去哪个生产者去取服务
public interface HelloRemote {
    @RequestMapping(value = "/hello")
    public String hello(@RequestParam(value = "name") String name);
}
`````

之后消费者端就像在调用自己的方法一样使用这些api。

feign的特点：

- feign本身里面就包含有了ribbon
- feign自身是一个声明式的伪http客户端，写起来更加思路清晰和方便
- fegin是一个采用基于接口的注解的编程方式，更加简便