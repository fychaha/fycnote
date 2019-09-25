## Springboot 整合shiro

#### 不使用stater

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring</artifactId>
       <version>1.4.0</version>
   </dependency>
   ```

2. 编写配置类

   ```java
   @Configuration
   public class ShiroConfig {
       //注入需要的realm
       @Bean
       public UserRealm userRealm(){
           return  new UserRealm();
       }
       //注入Default
       @Bean
       public SecurityManager securityManager() {
           DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
           manager.setRealm(realm());
           // 关闭 ShiroDAO 功能
           DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
           DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
           // 不需要将 Shiro Session 中的东西存到任何地方（包括 Http Session 中）
           defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
           subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
           manager.setSubjectDAO(subjectDAO);
           // 禁止 Subject 的 getSession 方法
           manager.setSubjectFactory(subjectFactory());
           return manager;
       }
       //注入ShiroFilterFactoryBean
       @Bean
       public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager){
          ShiroFilterFactoryBean shiroFilterFactoryBean=new ShiroFilterFactoryBean();
           //设置安全管理器
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           shiroFilter.setLoginUrl("/unauthenticated");//未认证的请求直接跳转登录页面
           shiroFilter.setUnauthorizedUrl("/unauthorized");//未授权的请求跳转去授权
         
          return shiroFilterFactoryBean;
       }
   }
   ```

3. Realm 的编写(以jwtRealm为例)

   ```java
   @Slf4j
   public class JwtRealm extends AuthorizingRealm {
       @Autowired
       private UserRepository userRepository;
       @Autowired
       private UserRoleRepository roleRepository;
       @Autowired
       private RolePermissionRepository permissionRepository;
       @Override
       public boolean supports(AuthenticationToken token) {
           /*
            * 逻辑上表示，JwtRealm 是专门用来验证 JwtToken 合法性的。
            * 它不负责验证 UsernamePasswordToken 等其他 Token 的合法性。
            */
           return token instanceof JwtToken;
       }
       /**
        * 授权
        */
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
           String jwt = (String) principals.getPrimaryPrincipal();
           if (jwt == null)
               throw new NullPointerException("jwt token 不允许为空");
           JwtUtil jwtUtil = new JwtUtil();
           if (!jwtUtil.isVerify(jwt))
               throw new UnknownAccountException();
           String username = (String) jwtUtil.decode(jwt).get("username");
           Set<String> roleNameSet = new HashSet<>();
           Set<String> permissionNameSet = new HashSet<>();
           UserRoleExample roleExample = new UserRoleExample();
           roleExample.or().andUsernameEqualTo(username);
           List<UserRole> roles = roleRepository.selectByExample(roleExample);
           // user 的所有的角色名
           for (UserRole role : roles)
               roleNameSet.add(role.getRoleName());
           // user 的所有的权限名
           for (String roleName : roleNameSet) {
               RolePermissionExample permissionExample = new RolePermissionExample();
               permissionExample.or().andRoleNameEqualTo(roleName);
               List<RolePermission> permissions = permissionRepository.selectByExample(permissionExample);
               for (RolePermission permission : permissions) {
                   permissionNameSet.add(permission.getPermission());
               }
           }
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           info.setRoles(roleNameSet);
           info.setStringPermissions(permissionNameSet);
           return info;
       }
       /**
        * 认证
        */
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
           String jwt = (String) token.getPrincipal();
           if (jwt == null)
               throw new NullPointerException("jwt token 不允许为空");
           JwtUtil jwtUtil = new JwtUtil();
           if (!jwtUtil.isVerify(jwt))
               throw new UnknownAccountException();
           String username = (String) jwtUtil.decode(jwt).get("username");
           // 去数据库判断 username 是否存在
           UserExample example = new UserExample();
           example.or().andUsernameEqualTo(username);
           List<User> users = userRepository.selectByExample(example);
           if (users.size() > 1) {
               log.warn("数据库中名为 [{}] 的用户不止一个", username);
               throw new RuntimeException("数据库数据错误");
           }
           else if (users.size() == 0) {
               log.warn("数据库中不存在名为 [{}] 的用户", username);
               throw new UnknownAccountException("该用户不存在");
           }
           else {
               log.info("用户 [{}] 正在使用 token", username);
           }
           return new SimpleAuthenticationInfo(jwt, jwt, "JwtRealm");
       }
   }
   ```

   



