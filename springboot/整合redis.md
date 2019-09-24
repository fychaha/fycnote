##  springboot整合redis

1. 在pom文件中添加依赖(这里使用stater来整合)

   ```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. 在appliction.yml文件中添加下面的配置(下列属性节点在spring下)

   ```yml
   redis:
     #单机
     # Redis服务器地址
     host: 127.0.0.1
     # Redis服务器连接端口
     port: 6379
     # 集群
     #多个节点以逗号分隔
     #cluster:
      # nodes: 192.168.0.78:6379,192.168.0.78:6380,192.168.0.78:6381,192.168.0.78:6382,192.168.0.78:6383,192.168.0.78:6384
     # Redis服务器连接密码（默认为空）
     password: 123456
     # Redis数据库索引（默认为0）
     database: 0
     #连接池
     pool:
       # 连接池最大连接数（使用负值表示没有限制）
       max-active: 8
       # 连接池最大阻塞等待时间（使用负值表示没有限制）
       max-wait: -1
       # 连接池中的最大空闲连接
       max-idle: 8
       # 连接池中的最小空闲连接
       min-idle: 0
       # 连接超时时间（毫秒）
       timeout: 0
   ```

3. 测试类(存入两个值的方法)

   ```java
   SpringBootTest
   @RunWith(SpringRunner.class)
   public class RedisTest {
   	
       @Autowired
       StringRedisTemplate stringRedisTemplate;
   
       @Test
       public void test1(){
          
           //stringRedisTemplate.opsForValue().set("name", "zhangsan");
           
           stringRedisTemplate.boundValueOps("name").set("lisi");
       }
   }
   ```

4. 实例

   ````java
   @Service
   public class StudentServiceImpl implements StudentService {
       @Resource
       private StudentDao stduentDao;
       @Resource
       private RedisTemplate redisTemplate;
       @Override
       public List<Student> findAll() {
           List<Student> studentList=null;
           ObjectMapper mapper=new ObjectMapper();
           String strStudent= (String) redisTemplate.boundValueOps("student.tabstudent.findall").get();
           if(StringUtils.isEmpty(strStudent)){
               studentList=stduentDao.findAll();
               //序列化
               String test1= null;
               try {
           //在redis缓存中找不到，才去数据库里查询，将查到的对象转化成json字符串,
           //然后把字符串存入redis缓存中
                   test1 = mapper.writeValueAsString(studentList);
                   redisTemplate.boundValueOps("student.tabstudent.findall").set(test1);
               } catch (JsonProcessingException e) {
                   e.printStackTrace();
               }
   
           }else{
               try {
   //反序列化
                   //将在redis中查到的json格式的字符串转成对象
                   studentList=mapper.readValue(strStudent, new TypeReference<List<Student>>() {
                       /**/});
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
          // redisTemplate.opsForValue().get("student.tabstudent.findall")
           return studentList;
       }
   
     }
   ````

   