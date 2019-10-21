## OAuth2示例

1. 引入依赖

   ````xml
   <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-security</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
           </dependency>
           <!-- https://mvnrepository.com/artifact/org.springframework.security.oauth/spring-security-oauth2 -->
           <dependency>
               <groupId>org.springframework.security.oauth</groupId>
               <artifactId>spring-security-oauth2</artifactId>
               <version>2.3.6.RELEASE</version>
           </dependency>
       </dependencies>
   ````

2. 添加配置文件

   `````properties
   spring:
     redis:
       database: 0
       host: 127.0.0.1
       port: 6379
       password:
       jedis:
         pool:
           max-active: 8
           max-idle: 8
           max-wait: -1ms
           min-idle: 0
   `````

3. 配置授权服务器

   ````java
   
   @Configuration
   @EnableAuthorizationServer//开启授权服务器
   public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
       @Autowired
       AuthenticationManager authenticationManager;
       @Autowired
       RedisConnectionFactory redisConnectionFactory;
       @Autowired
       UserDetailsService userDetailsService;
       @Bean
       PasswordEncoder passwordEncoders(){
           return  new BCryptPasswordEncoder();
       }
   
       @Override
       public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
           clients.inMemory()
                   .withClient("password")
                   .authorizedGrantTypes("password","refresh_token")
                   .accessTokenValiditySeconds(1800)
                   .resourceIds("rid")
                   .scopes("all")
                   .secret("$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq");//加密密码，明文为123
       }
   
       @Override
       public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
           endpoints.tokenStore(new RedisTokenStore(redisConnectionFactory))
                   .authenticationManager(authenticationManager)
                   .userDetailsService(userDetailsService);
       }
   
       @Override
       public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
           security.allowFormAuthenticationForClients();//支持client_id 和client_secret做登录认证。
       }
   }
   
   ````

4. 配置资源服务

   `````java
   @Configuration
   @EnableResourceServer//开启资源服务配置
   public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
       @Override
       public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
           resources.resourceId("rid").stateless(true);//配置资源id
       }
   
       @Override
       public void configure(HttpSecurity http) throws Exception {//详见spring security
           http.authorizeRequests()
                   .antMatchers("/admin/**").hasRole("admin")
                   .antMatchers("/user/**").hasRole("user")
                   .anyRequest().authenticated();
       }
   }
   
   `````

5. 配置security

   ````java
   @Configuration
   public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       @Bean	//这两个bean将注入授权服务配置类中使用
       public AuthenticationManager authenticationManagerBean() throws Exception {
           return super.authenticationManagerBean();
       }
   
       @Override
       @Bean
       protected UserDetailsService userDetailsService() {
           return super.userDetailsService();
       }
   
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           auth.inMemoryAuthentication()
                   .withUser("admin")
                   .password("")
                   .roles("admin")
                   .and()
                   .withUser("user")
                   .password("$2a$10$RMuFXGQ5AtH4wOvkUqyvuecpqUSeoxZYqilXzbz50dceRsga.WYiq")
                   .roles("user");
       }
   
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.antMatcher("/oauth/**").authorizeRequests()
                   .antMatchers("/oauth/**").permitAll()
                   .and()
                   .csrf()
                   .disable();
       }
   }
   ````

6. 创建简单请求地址

   ````java
   
   @RestController
   public class HelloController {
       @GetMapping("/admin/hello")
       public String admin() {
           return "Hello admin!";
       }
       @GetMapping("/user/hello")
       public String user() {
           return "Hello user!";
       }
       @GetMapping("/hello")
       public String hello() {
           return "hello";
       }
   ````

7. 测试

   发送post请求

   ````bash
   curl -d "username=sang&password=123&grant_type=password&client_id=client1&scope=all&client_secret=123" http://localhost:8080/oauth/token
    或者postman 发送post请求
    http://localhost:8080/oauth/token?username=sang&password=123&grant_type=password&client_id=client1&scope=all&client_secret=123
   ````

   返回结果

   ````json
    {
        "access_token": "4b92bad1-5e56-4e34-8991-884991c692ce",//你的令牌
        "token_type": "bearer",
        "refresh_token": "1a80182b-9227-43b8-ac43-80cfbb7b69cd",//刷新令牌
        "expires_in": 1799,
        "scope": "all"
    }
   ````

   携带token发送请求

   ````bash
   http://localhost:8080/user/hello?access_token=4b92bad1-5e56-4e34-8991-884991c692ce
    页面上显示 Hello user!
   ````

   刷新token

   ``````bash
   curl -d "grant_type=refresh_token&refresh_token=37724209-ce23-4a1c-89c2-1aef7cfe10b8&client_id=client1&client_secret=123" http://localhost:8080/oauth/token
      或者使用postman发送请求
      http://localhost:8080/oauth/token?grant_type=refresh_token&refresh_token=1a80182b-9227-43b8-ac43-80cfbb7b69cd&client_id=client1&client_secret=123
   ``````

   结果会返回新的 令牌access_Token

   ````json
   {
       "access_token": "dec040f3-8ab7-43ed-8148-257788ae3d0b",
       "token_type": "bearer",
       "refresh_token": "1a80182b-9227-43b8-ac43-80cfbb7b69cd",
       "expires_in": 1799,
       "scope": "all"
   }
   ````

8. 看看redis中的数据

   ````bash
   key *
   ````

   