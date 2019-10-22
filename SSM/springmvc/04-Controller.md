# Controller 的编写和配置

`@Controller` 注解和 `@RequestMapping` 注解是 SpringMVC 最重要的两个注解。

使用基于注解的控制器的优点如下：

- 一个 Controller 类可以处理多个动作，而实现了一个 `Controller` 接口的控制器只能处理一个动作。
- 基于 Controller 注解的控制器的请求映射不需要写在配置文件中。使用 `@RequestMapping` 注解类型，可以对一个方法进行请求处理。


## 一、Controller 注解类型

Spring 使用扫描机制来找到应用程序中所有基于注解的控制器类。为了保证 Spring 能找到你的控制器，必须完成两件事：

```xml
<beans
  ...
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
    ...
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    ... ">
```

```xml
<context:component-scan base-package="..." />
```

注意，不要让 Spring 扫描一个太广泛的包，这会包含无意义的行为。

## 二、RequestMapping 注解类型

`@RequestMapping` 注解类型的作用如同起名字所暗示：映射一个请求和一个方法。可以使用它注解一个方法或类。

被 `@RequestMapping` 注解的方法将成为一个 ***请求处理方法*** ，在接收到URL请求时被调用。

```java
@RequestMapping(value="/hello", method = {RequestMethod.GET, RequestMethod.POST})
public String printHello(ModelMap model) {
    System.out.println("Hello World");
    model.addAttribute("message", "Hello Spring MVC Framework!");
    return "hello";
}
```

`value` 属性是 `@RequestMapping` 的默认属性，唯一时可省略属性名。

`method` 属性用来指示该方法仅处理哪些 HTTP 方法。若 `method` 属性只有一个值时，则无须花括号。若没有指定 `method` 属性值，则请求方法可处理任意 HTTP 方法。

此外，如果用 `@RequestMapping` 注解一个控制器类，那么，所有的方法都将映射为 ***相对于*** 类级别的请求。


## 三、编写请求方法

每个请求处理方法的参数和返回值 ***既灵活又严格*** 。

最为常见的参数类型有：

- HttpServletRequest、HttpServletResponse、HttpSession
- Map、Model、ModelMap
- 表单对象
- 带指定注解的参数

最为常见的参数类型有：

> - ModelAndView
> - Model、View
> - String
> - （被当作 View 的模型对象的）任意类型


## 四、请求参数和路径变量


Spring MVC 提供了一个更简单的方法来获取Get请求参数：通过使用 @RequestParam 注解。

```java
@RequestMapping("/hehe/{id}")
public String printGoodbye(ModelMap model, @PathVariable int id) {
  System.out.println(id);
  return "hello";
}
```

此处需要注意的是，你的拦截规则是拦截所有请求，还是拦截特定后缀（无法拦截并触发该方法的执行）？！
