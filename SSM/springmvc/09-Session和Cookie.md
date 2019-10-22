# SpringMVC 操作 Session 和 Cookie

## 一、操作 Session

### 方式一：方法中声明/使用 HttpServletRequest 参数

这种情况下，Spring MVC 在调用请求处理方法时，会传入本次请求的 Request 对象，后续可以像普通同的 Servlet 代码一样操作 Session 。

```java
@RequestMapping("/...")
public String demo1(HttpServletRequest request, ...) {
  HttpSession session = request.getSession();
  session.setAttribute(..., ...);
  ...
}
```

### 方式二：方法中直接声明/使用  HttpSession 参数

该方式是 方式一 的改进版。直接要求 SpringMVC “帮”我们去调用 request.getSession() 后再传入到请求处理方法中。

```java
@RequestMapping("/...")
public String login(HttpSession session, ...) {
    ....
}
```




## 二、操作 Cookie

在 SpringMVC 中操作 Cookie 和操作 Session 非常类似，你可以要求 SpringMVC 将当前请求的 Response 对象传入你的方法中，而后对其添加 Cookie ：

```java
@RequestMapping("...")
public String demo1(HttpServletResponse resp) {
  Cookie cookie = new Cookie("name", "hello world");
  cookie.setMaxAge(60);
  resp.addCookie(cookie);
}
```

SpringMVC 提供了一个 @CookieValue 注解来简化获取获得客户端传入的 Cookie 数据：

```java
@RequestMapping("...")
public String demo2(@CookieValue("name") String value) {
  System.out.println(value);
  ...
}
```
