## springboot 集成rabbitmq

1. 引入依赖

   ```
   <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-amqp</artifactId>
   </dependency>
   ```

2. application.yml 中设置相关属性

   ```yml
   spring.rabbitmq.host=192.168.0.86
   spring.rabbitmq.port=5672
   spring.rabbitmq.username=admin
   spring.rabbitmq.password=123456
   ```

3. 队列queue配置

   ```
   @Configuration
   public class RabbitConfig {
       @Bean
       public Queue Queue() {
           return new Queue("hello");
       }
   }
   ```

   发送者

   ```
   @Component
   public class HelloSender {
   // rabbitTemplate是springboot 提供的默认实现
       @Autowired
       private AmqpTemplate rabbitTemplate;
       public void send() {
           String context = "hello " + new Date();
           System.out.println("Sender : " + context);
           this.rabbitTemplate.convertAndSend("hello", context);
       }
   }
   ```

   接收者

   ```
   @Component
   @RabbitListener(queues = "hello")
   public class HelloReceiver {
   
       @RabbitHandler
       public void process(String hello) {
           System.out.println("Receiver  : " + hello);
       }
   
   }
   ```

   测试

   ```
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class RabbitMqHelloTest {
   
       @Autowired
       private HelloSender helloSender;
   
       @Test
       public void hello() throws Exception {
           helloSender.send();
       }
   
   }
   
   //注意，发送者和接收者的queue name必须一致，不然不能接收
   ```

   