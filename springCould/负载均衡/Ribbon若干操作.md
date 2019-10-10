### 不使用注册中心，直接指定服务地址，consumer直连provider

server:
  port: 8093  #服务端口号
spring:
  application:
    name: service-consumer #服务名称--调用的时候根据名称来调用该服务的方法

```properties
ribbon.eureka.enabled=false
```

eureka-hello(你所指定的服务名)         指定具体的Provider服务清单,多个用,隔开

```properties
eureka-hello.ribbon.listOfServers=localhost:8000,localhost:9000
```



### 自定义Ribbon负载均衡策略

修改配置文件

```yaml

#设置负载均衡策略 provider-member1 为调用的服务的名称,后面的组成部分是固定的。
provider-member1:
  ribbon:
      NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

配置类

````java
package edu.xja.config;

import com.netflix.loadbalancer.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


//@Configuration
public class RibbonConfig {

    @Bean
    public IRule ribbonRule(){
        //默认为轮询策略
        return new RandomRule(); //分配策略：随机选择一个server
//    return new BestAvailableRule(); //分配策略：选择一个最小的并发请求的server，逐个考察Server，如果Server被tripped了，则忽略
//    return new RoundRobinRule(); //分配策略：轮询选择，轮询index，选择index对应位置的server
//    return new WeightedResponseTimeRule(); //分配策略：根据响应时间分配一个weight(权重)，响应时间越长，weight越小，被选中的可能性越低
//    return new ZoneAvoidanceRule(); //分配策略：复合判断server所在区域的性能和server的可用性选择server
//    return new RetryRule(); //分配策略：对选定的负载均衡策略机上重试机制，在一个配置时间段内当选择server不成功，则一直尝试使用subRule的方式选择一个可用的server
    }

//    @Bean
//    public IPing ribbonPing() {
//        return new PingUrl();
//    }
//
//    @Bean
//    public ServerListSubsetFilter serverListFilter() {
//        ServerListSubsetFilter filter = new ServerListSubsetFilter();
//        return filter;
//    }

}

````

![1570696123566](C:\Users\admi\AppData\Roaming\Typora\typora-user-images\1570696123566.png)