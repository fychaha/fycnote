## Redis 整合springboot

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   <!-- 不要忘记加这个包 -->
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
   ```

   引入 `commons-pool2` 是因为 Lettuce 需要使用 `commons-pool2` 创建 Redis 连接池。

2. application配置

   ````properties
   # Redis 服务器地址
   spring.redis.host=localhost
   # Redis 服务器连接端口
   spring.redis.port=6379
   # Redis 数据库索引(默认为 0)
   spring.redis.database=0
   # Redis 服务器连接密码(默认为空)
   spring.redis.password=
   # 连接池最大连接数(使用负值表示没有限制) 默认 8
   spring.redis.lettuce.pool.max-active=8
   # 连接池最大阻塞等待时间(使用负值表示没有限制) 默认 -1
   spring.redis.lettuce.pool.max-wait=-1
   # 连接池中的最大空闲连接 默认 8
   spring.redis.lettuce.pool.max-idle=8
   # 连接池中的最小空闲连接 默认 0
   spring.redis.lettuce.pool.min-idle=0
   ````

3. 测试使用

   在单元测试中，注入 ***`RedisTemplate`*** 。String 是最常用的一种数据类型，普通的 key-value 存储都可以归为此类，value 其实不仅是 String 也可以是数字。

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class TestRedisTemplate {
       @Autowired
       private RedisTemplate redisTemplate;
       @Test
       public void testString() {
           redisTemplate.opsForValue().set("ben", "hello world");
           Assert.assertEquals("hello world", redisTemplate.opsForValue().get("ben"));
       }
   }
   ```

   在这个单元测试中，我们使用 redisTemplate 存储了一个字符串 *`"hello world"`* ，存储之后获取进行验证，多次进行 set 相同的 key，键对应的值会被覆盖。

   Spring Data Redis 针对 api 进行了重新归类与封装，将同一类型的操作封装为 **`Operation`** 接口：

   

   | 专有操作        | 说明                |
   | :-------------- | :------------------ |
   | ValueOperations | 简单 K-V 操作       |
   | ListOperations  | list 类型的数据操作 |
   | SetOperations   | set 类型数据操作    |
   | ZSetOperations  | zset 类型数据操作   |
   | HashOperations  | map 类型的数据操作  |

   ```java
   @Autowired
   private RedisTemplate<String, String> redisTemplate;
   
   @Test
   public void contextLoad() {
       assertNotNull(redisTemplate);
       ValueOperations operations1 = redisTemplate.opsForValue();
       ListOperations operations2 = redisTemplate.opsForList();
       SetOperations operations3 = redisTemplate.opsForSet();
       ZSetOperations operations4 = redisTemplate.opsForZSet();
       HashOperations operations5 = redisTemplate.opsForHash();
   }
   ```

   Spring Data Redis 通过 jedis / Lettuce 向 Redis 中存取数据时，会使用 Serializer 对键和值进行序列化和反序列化。

   Spring Data Redis 支持多种 Serializer ，可以通过配置，指定对某种数据结构的 key 和 value 使用特定 Serialiazer 进行序列化和反序列化。

   Spring Data Redis 默认使用 *`JdkSerializationRedisSerializer`*，（对于所有的数据结构）将 key 和 value 系列化为字节序列（*`byte[]`*），再调用 jedis / Lettuce 进行存储。

   

   # 各类型实践（重要操作）

   ### 实体

   先来看 Redis 对 Pojo 的支持，新建一个 Student 对象<small>（需要实现 Serializable 接口）</small>，放到缓存中，再取出来。

   即将对象序列化成json字符串，再存入缓存,取出时则相反。

   ```java
   @Test
   public void testObj() {
       Student user = new Student("tom", 20);
       ValueOperations<String, Student> operations = redisTemplate.opsForValue();
       operations.set("com.tom", user);
       Student u = operations.get("com.tom");
       System.out.println("user: " + u);
   }
   ```

   输出结果:

   ```
   user: Student{name='tom', age=20}
   ```

   理论上说，上述操作中本应该是将一个 String-String 的键值对存于 Redis 的 String 结构中，但是，实际山个，我们是强行将一个 String-Object 这样的键值对存入了 String 结构中。此时，RedisSerializer 就会起到一个编解码的作用。

   存入的时候，它将对象编码为其二进制形式的字符串；取出时，它将一个二进制形式的字符串解码为一个对象。

   

   ### 超时失效

   Redis 在存入每一个数据的时候都可以设置一个超时间，过了这个时间就会自动删除数据。

   新建一个 Student 对象，存入 Redis 的同时设置 100 毫秒后失效，设置一个线程暂停 1000 毫秒之后，判断数据是否存在并打印结果。

   ```java
   @Test
   public void testExpire() throws InterruptedException {
       Student user = new Student("tom", 20);
       ValueOperations<String, Student> operations = redisTemplate.opsForValue();
       operations.set("expire", user, 100, TimeUnit.MILLISECONDS);
   
       Thread.sleep(1000);
   
       boolean exists = redisTemplate.hasKey("expire");
       if (exists) {
           System.out.println("exists is true");
       } else {
           System.out.println("exists is false");
       }
   }
   ```

   输出结果:

   ```
   exists is false
   ```

   从结果可以看出，Reids 中已经不存在 Student 对象了，此数据已经过期，同时我们在这个测试的方法中使用了 `hasKey("expire")` 方法，可以判断 key 是否存在。

   ####  删除数据

   有些时候，我们需要对过期的缓存进行删除，下面来测试此场景的使用。首先 set 一个字符串“ityouknow”，紧接着删除此 key 的值，再进行判断。

   ```java
   redisTemplate.delete("key");
   ```

   

   

   

   

