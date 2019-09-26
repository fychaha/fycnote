# Euraka整合（案例）

案例中有三个角色：服务注册中心、服务提供者、服务消费者，流程是首先启动注册中心，服务提供者生产服务并注册到服务中心中，消费者从服务中心中获取服务并执行。

我们假设服务提供者有一个hello方法，可以根据传入的参数，提供输出“hello xxx，this is first messge”的服务

#### 服务注册中心项目

1. 引入依赖

   ```xml
   <dependencies>
   	<dependency>
   		<groupId>org.springframework.cloud</groupId>
   		<artifactId>spring-cloud-starter</artifactId>
   	</dependency>
   	<dependency>
   		<groupId>org.springframework.cloud</groupId>
   		<artifactId>spring-cloud-starter-eureka-server</artifactId>
   	</dependency>
   	<dependency>
   		<groupId>org.springframework.boot</groupId>
   		<artifactId>spring-boot-starter-test</artifactId>
   		<scope>test</scope>
   	</dependency>
   </dependencies>
   ```

2.  添加启动代码中添加`@EnableEurekaServer`注解

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class SpringCloudEurekaApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(SpringCloudEurekaApplication.class, args);
   	}
   }
   ```

3. 配置文件
   在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，在`application.properties`添加以下配置：

   ```xml
   spring.application.name=spring-cloud-eureka
   
   server.port=8000
   eureka.client.register-with-eureka=false    <!--表示禁止将自己注册到Eureka Server ,默认为true -->
   eureka.client.fetch-registry=false   <!--表示是否从Eureka Server获取注册信息，默认为true -->
   <!-- 设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。-->
   eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
   ```

### 服务提供项目       (这个模块产生服务，并将服务注册到注册中心)

1. 引入依赖同上

2. 配置文件

   ```xml
   spring.application.name=spring-cloud-producer <!--表明这个项目是服务的生产者 -->
   server.port=9000
   eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
   ```

3. 启动类
   启动类中添加`@EnableDiscoveryClient`注解。
   添加`@EnableDiscoveryClient`注解后，项目就具有了服务注册的功能。启动工程后，就可以在注册中心的页面看到SPRING-CLOUD-PRODUCER服务。

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient  //让注册中心扫描到自己
   public class ProducerApplication {
   
   	public static void main(String[] args) {
   		SpringApplication.run(ProducerApplication.class, args);
   	}
   }
   ```

4. controller类
   提供服务的编写( hello 服务)

   ```java
   	@RestController
   public class HelloController {
   	
       @RequestMapping("/hello")
       public String index(@RequestParam String name) {
           return "hello "+name+"，this is first messge";
       }
   }
   ```

   到此服务提供者配置就完成了。

### 服务调用项目(这个模块消费服务，去注册中心要取服务)

1. 引入依赖同上

2. 配置文件

   ```xml
   sspring.application.name=spring-cloud-consumer  
   server.port=9001
   eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
   ```

3. 启动类

   启动类添加`@EnableDiscoveryClient`和`@EnableFeignClients`注解。

   ```java
   @SpringBootApplication 
   @EnableDiscoveryClient  
   @EnableFeignClients  //启用feign进行远程调用,表明它可以去注册中心调用服务
   public class ConsumerApplication {
   	public static void main(String[] args) {
   		SpringApplication.run(ConsumerApplication.class, args);
   	}
   
   }
   ```

   Feign是一个声明式Web Service客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

4. feign调用实现

   ```java
   @FeignClient(name= "spring-cloud-producer")  //name:远程服务名，及spring.application.name配置的名称
   public interface HelloRemote {
       @RequestMapping(value = "/hello")
       public String hello(@RequestParam(value = "name") String name);
   }
   ```

   此接口中的方法和远程服务(服务时生产者)中contoller中的方法名和参数需保持一致。

5. web层调用远程服务

   将HelloRemote注入到controller层，像普通方法一样去调用即可。

   ```java
   @RestController
   public class ConsumerController {
       @Autowired
       HelloRemote HelloRemote;
       @RequestMapping("/hello/{name}")
       public String index(@PathVariable("name") String name) {
           return HelloRemote.hello(name);
       }
   
   }
   ```

   到此，最简单的一个服务注册与调用的例子就完成了。