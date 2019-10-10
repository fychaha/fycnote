## 使用RestTemplate和Ribbon消费服务并实现负载均衡

我们可以使用RestTemplate来访问第三方rest接口，从而消费服务。

举例说明，新建一个RestTestController类,获取百度网页的html代码

```java
@RestController
public class RestTestController {
    @GetMapping("/test")
    public String testRest(){ 
        RestTemplate restTemplate = new RestTemplate();//这里restTemplate通常由spring注入
        return restTemplate.getForObject("http://www.baidu.com/",String.class)
    }
}

```



#### 使用Ribbon实现负载均衡

在RestTemplate的基础上加上Ribbon，注入RestTemplate的bean，在bean上加上注解@Loadbalance

```java
  	@Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate(); //默认使用URLConnection
    }
```

此时这个template就有了负载均衡的功能。

接下来开启两个服务名为sayHello，功能相同的服务，改变他们两个的端口为8000，和9000

```java
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
```

这里访问的服务名sayHello会去注册中心匹配，列出所有该名的服务，然后根据负载均衡策略调用服务。

## 注意

#### 负载均衡应该是在服务消费者那块启用，服务提供者在注册中心注册服务，消费者则在注册中心列出的服务群中根据自己的负载均衡策略决定调用哪个服务。