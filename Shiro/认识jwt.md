##   JWT 的组成

一个 JWT 实际上就是一个字符串，它由三部分组成：头部、荷载 与 签名。

这个 JWT 的标准形式为 `头部.荷载.签名`

例如：

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0b20iLCJleHAiOjE1NjY5MjA4MzIsImlhdCI6MTU2NjkxNzIzMiwianRpIjoiMGM4ODRhNzAtOTdkNy00MTczLTgyYzItMTcyNzA2ZmMyZDU4In0.IKO9rBpLz-u2m2gA1S2gR-8CFn1Z1qs-AZvW55A1SoY
```



### 头部（Header）

JWT 都有一个头部，头部用于描述关于该 JWT 的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个 JSON 对象。

```js
{
  "typ": "JWT",
  "alg": "HS256"
}
```

它表示当前信息是一个 `JWT`，且是被 `HS256` 算法加密过的。

当然，一个 JWT 的头部信息真正的样子并不是上面这个样子，它会被 Base64 算法编码。

上述 JWT 的头部信息被编码后长成的是 `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9` 这个样子。

### 荷载（Payload）部分

JWT 的荷载是 JWT 的最重要部分，它就代表着 JWT 的信息内容。JWT 的荷载部分和头部部分一样，其具体内容也是一个 JSON 格式字符串。例如：

```js
{
    "iss": "John Wu JWT",
    "iat": 1441593502,
    "exp": 1441594722,
    "aud": "www.example.com",
    "sub": "jrocket@example.com",
    "from_user": "B",
    "target_user": "A"
}
```

荷载部分中有些字段是 JWT 的标准字段（例如上例的前五个），除此之外，你可以向 JWT 中添加对你有用的任意的信息（例如上例的后两个）。

和头部部分一样，真正的荷载部分也是要经过 Base64 算法编码的。上例真正长的是这个样子：

```
eyJpc3MiOiJKb2huIFd1IEpXVCIsImlhdCI6MTQ0MTU5MzUwMiwiZXhwIjoxNDQxNTk0NzIyLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJqcm9ja2V0QGV4YW1wbGUuY29tIiwiZnJvbV91c2VyIjoiQiIsInRhcmdldF91c2VyIjoiQSJ9
```

### 签名（Signature）部分

签名部分是用前面的头部和荷载部分的内容生成的。

将上面的两个编码后的字符串都用句号 `.` 连接在一起（头部在前，荷载在后），再使用 `HS256` 算法进行加密。

之所以是 HS256 算法，是因为要与头部中所生成的加密算法呼应。

在加密过程中，需要提供一个秘钥（secret）， 例如，以 `mystar` 作为秘钥，上述的头部和荷载部分生成的签名就是：

```
rSWamyAYwuHCo7IFAgd1oRpSP7nzL7BF5t7ItqpKViM
```

最终，上例中的整个 JWT 的内容就是：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcm9tX3VzZXIiOiJCIiwidGFyZ2V0X3VzZXIiOiJBIn0.rSWamyAYwuHCo7IFAgd1oRpSP7nzL7BF5t7ItqpKViM
```

一旦服务端生成并发回这个 JWT 字符串之后，后续用户的请求就应该带上这个 JWT，以证明自己成功登陆过 ：

```
http://.../xxx.do?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcm9tX3VzZXIiOiJCIiwidGFyZ2V0X3VzZXIiOiJBIn0.rSWamyAYwuHCo7IFAgd1oRpSP7nzL7BF5t7ItqpKViM
```



添加过滤器

```java
// 添加jwt过滤器
        Map<String, Filter> filterMap = new HashMap<>();
        filterMap.put("anon", new AnonymousFilter());
        filterMap.put("jwt", new JwtFilter());
        shiroFilter.setFilters(filterMap);
```

设置拦截器

```java
   // 拦截器
        Map<String, String> filterRuleMap = new LinkedHashMap<>();
        filterRuleMap.put("/login", "anon");
        filterRuleMap.put("/**", "jwt");
        shiroFilter.setFilterChainDefinitionMap(filterRuleMap);
```



JwtFilter 编写

```java
public class JwtFilter extends AccessControlFilter {

            /**
             * @return 返回true时，Shiro 会放过请求，允许访问URL。此时不考虑onAccessDenied方法。
             *         返回false时，Shiro 才会根据 onAccessDenied 的返回值考虑是否放过请求。
             */
            @Override
            protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
                log.debug("isAccessAllowed 方法被调用");
                return false;
            }
            @Override
            protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
                log.debug("onAccessDenied 方法被调用");
                HttpServletRequest request = (HttpServletRequest) servletRequest;
                String jwt = request.getHeader("Authorization");
                log.info("请求的 Header 中藏有 jwtToken {}", jwt);
                JwtToken jwtToken = new JwtToken(jwt);
                try {
                    // 委托 realm 进行登录认证
                    // 兜兜转转，最后调用的就是 JwtRealm 中的 doGetAuthenticationInfo 方法
                    getSubject(servletRequest, servletResponse).login(jwtToken);
                } catch (Exception e) {
                    e.printStackTrace();
                    onLoginFail(servletResponse);
                    return false;
                }
                return true;
            }
            // 登录失败时默认返回 401 状态码
            private void onLoginFail(ServletResponse response) throws IOException {
                HttpServletResponse httpResponse = (HttpServletResponse) response;
                httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        httpResponse.getWriter().write("login error");
    }
}
```

