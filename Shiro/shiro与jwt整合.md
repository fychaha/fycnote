# Shiro 整合 JWT 实现无状态 Web 服务

## Subject 工厂

无状态的 Web 服务（Restful），意味着我们不会使用 Shiro 的 Session 功能，更用不上 SessionDAO 。因此，严谨的做法可以将这两个功能关闭掉。

```java
public class JwtDefaultSubjectFactory extends DefaultWebSubjectFactory {

    @Override
    public Subject createSubject(SubjectContext context) {
        // 不创建 session
        context.setSessionCreationEnabled(false);
        return super.createSubject(context);
    }
}
```

通过调用 `context.setSessionCreationEnabled(false)` 表示不创建会话；如果之后调用 `subject.getSession()` 将抛出 `DisabledSessionException` 异常。 

## JwtFilter

类似于 FormAuthenticationFilter，但是根据当前请求上下文信息每次请求时都要登录的认证过滤器。

AccessControlFilter 有两个方法：

- isAccessAllowed()
- onAccessDenied()

如果 `isAccessAllowed()` 返回 true，Shiro 将放过请求，允许访问 URL<small>（这时 Shiro 不考虑 onAccessDenied() 方法）</small>；

如果 `isAccessAllowed()` 返回 false，那么 Shiro 再来考虑 `onAccessDenied()` 方法。`onAccessDenied()` 方法返回 true，则表示认证通过，返回 fase，则表示认证不通过。

```java
@Slf4j
public class JwtFilter extends AccessControlFilter {

    /**
     * @return 返回结果是false的时候才会执行下面的onAccessDenied方法
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        log.warn("isAccessAllowed 方法被调用");
        return false;
    }

    /**
     * 返回结果为true表明登录通过
     */
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        log.warn("onAccessDenied 方法被调用");

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        String jwt = request.getHeader("Authorization");
        log.info("请求的 Header 中藏有 jwtToken {}", jwt);
        JwtToken jwtToken = new JwtToken(jwt);
        try {
            // 委托 realm 进行登录认证
            getSubject(servletRequest, servletResponse).login(jwtToken);
        } catch (Exception e) {
            e.printStackTrace();
            onLoginFail(servletResponse);
            return false;
        }

        return true;
    }

    //登录失败时默认返回 401 状态码
    private void onLoginFail(ServletResponse response) throws IOException {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        httpResponse.getWriter().write("login error");
    }
}
```

和客户端约定好，要求客户端发送的请求，在请求头中传递 jwtToken，然后生成 JWTToken 对象，再交由 Realm 进行认证。

## JwtToken

```java
public class JwtToken implements AuthenticationToken {

    private String token;

    public JwtToken (String  token) {
        this.token = token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```

用户身份和凭证都是 token 。

## JwtRealm

用于认证的 Realm 。

```java
public class CustomRealm extends AuthorizingRealm {

    private static final Set<String> tomRoleNameSet = new HashSet<>();
    private static final Set<String> tomPermissionNameSet = new HashSet<>();
    private static final Set<String> jerryRoleNameSet = new HashSet<>();
    private static final Set<String> jerryPermissionNameSet = new HashSet<>();

    static {
        tomRoleNameSet.add("admin");
        jerryRoleNameSet.add("user");

        tomPermissionNameSet.add("user:insert");
        tomPermissionNameSet.add("user:update");
        tomPermissionNameSet.add("user:delete");
        tomPermissionNameSet.add("user:query");

        jerryPermissionNameSet.add("user:query");
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = (String) principals.getPrimaryPrincipal();
        SimpleAuthorizationInfo info =  new SimpleAuthorizationInfo();
        if (username.equals("tom")) {
            info.addRoles(tomRoleNameSet);
            info.addStringPermissions(tomPermissionNameSet);
        } else if (username.equals("jerry")) {
            info.addRoles(jerryRoleNameSet);
            info.addStringPermissions(jerryPermissionNameSet);
        }

        return info;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String) token.getPrincipal();
        if (username == null)
            throw new UnknownAccountException("用户名不能为空");

        SimpleAuthenticationInfo info = null;

        if (username.equals("tom"))
            return new SimpleAuthenticationInfo("tom", "123", CustomRealm.class.getName());
        else if (username.equals("jerry"))
            return new SimpleAuthenticationInfo("jerry", "123", CustomRealm.class.getName());
        else
            return null;
    }
}
```

## ShiroConfig

由于这里需要去设置 SecurityManager，所以，这里不使用 Shiro 的 starter 进行整合。 

在 spring boot 整合 shiro 的基本配置的基础上，此处有三处不同的地方：

```java
 @Bean
public SubjectFactory subjectFactory() {
    return new JwtDefaultSubjectFactory();
}

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

@Bean
public ShiroFilterFactoryBean shiroFilterFactoryBean() {
    ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
    shiroFilter.setSecurityManager(securityManager());
    shiroFilter.setLoginUrl("/unauthenticated");
    shiroFilter.setUnauthorizedUrl("/unauthorized");

    // 添加jwt过滤器
    Map<String, Filter> filterMap = new HashMap<>();
    filterMap.put("anon", new AnonymousFilter());
    filterMap.put("jwt", new JwtFilter());
    filterMap.put("logout", new LogoutFilter());
    shiroFilter.setFilters(filterMap);

    // 拦截器
    Map<String, String> filterRuleMap = new LinkedHashMap<>();
    filterRuleMap.put("/login", "anon");
    filterRuleMap.put("/logout", "logout");
    filterRuleMap.put("/**", "jwt");
    shiroFilter.setFilterChainDefinitionMap(filterRuleMap);

    return shiroFilter;
}
```

