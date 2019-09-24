## **springboot集成kaptcha验证码**

1.  引入pom文件依赖

   ```xml
   <dependency>
       <groupId>com.github.penggle</groupId>
       <artifactId>kaptcha</artifactId>
       <version>2.3.2</version>
   </dependency>
   ```

2. Resources下新建kaptcha.properties

   ```yml
   ##### Kaptcha Information
   kaptcha.width=150
   kaptcha.height=42
   kaptcha.border=no
   kaptcha.textproducer.font.size=40
   kaptcha.textproducer.char.space=10
   kaptcha.textproducer.font.names=\u4EFF\u5B8B,\u5FAE\u8F6F\u96C5\u9ED1
   kaptcha.textproducer.char.string=1234567890
   kaptcha.textproducer.char.length=4
   kaptcha.background.clear.from=92,189,170
   kaptcha.background.clear.to=255,255,255
   ```

   创建KaptchaConfig类修改成从属性文件中读取

   ````java
   @Component
   public class KaptchaConfig {
       private static Properties props = new Properties();
   
       @Bean
       public DefaultKaptcha defaultKaptcha() throws Exception {
           // 创建DefaultKaptcha对象
           DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
   
           // 读取配置文件
           try {
               props.load(KaptchaConfig.class.getClassLoader()
                       .getResourceAsStream("kaptcha.properties"));
           }catch (Exception e) {
               e.printStackTrace();
           }
   
           // 将Properties文件设到DefaultKaptcha对象中
           defaultKaptcha.setConfig(new Config(props));
           return defaultKaptcha;
       }
       
   }
   ````

   



