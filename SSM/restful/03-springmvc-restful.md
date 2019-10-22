# Spring MVC 对 Restful 的支持



## HTTP 请求和响应的 content-type 

HTTP 请求和响应信息中的 `header` 部分都有一个 `content-type` 属性，它的值标志着本次请求-响应的 Body 部分的 **文本数据的类型** 。

理论上来说：

- Request 的 `content-type` 应该与 Request 的 body 中的内容的『格式』相呼应。
- Response 的 `content-type` 应该与 Response 的 body 中的内容的『格式』相呼应。
- 逻辑上，Response 的 `content-type` 应该与 Request 的 `accept` 相呼应。



| 请求               | content-type                        | Body 内容        |
| ------------------ | ----------------------------------- | ---------------- |
| 普通 get 请求      | 缺失                                | 无               |
| 普通 post 请求     | `application/x-www-form-urlencoded` | 请求参数字符串   |
| 文件上传 post 请求 | `multipart/form-data`               | 文件的二进制内容 |

- `application/json` 形式的请求（通常是 post 请求形式），无法通过 `<form>` 表单发起。只能通过 `$.ajax()` 方法发起，并且需要手动指定其 `contentType` 属性值为 `application/json` 。此时，HTTP 的 Body 中的内容即为一个 JSON 格式字符串。

```js
$.ajax({
    url: '...',
    method: 'post',
    contentType: 'application/json',
    data: JSON.stringify(obj),
    ...
    success: function (result) {
        ...
    }
});
```



> <small>补注，可以想象得到 `$.ajax()` 方法的 `contentType` 属性可以指定请求的 Body 的内容类型为 json 格式字符串，自然也可以指定为其它类型，例如 `application/xml` （不过，一般情况下没有这些需求）。</small>





## HttpMessageConverter

HttpMessageConverter 是为 `@RequestBody` 和 `@ResponseBody `『服务』的！<small>它与 `@RequestParam` 和 `@RequestPart` 无关。</small>

- `@RequestBody` 需要利用 HttpMessageConverter 来进行 **参数绑定** ；

- `@ResponseBody` 需要利用 HttpMessageConverter 来进行 **返回值类型转换** 。



> <small>再次强调，此处进行 **参数绑定** 和 **返回值类型转换** 的前提是标注 @RequestBody 和 @ResponseBody 注解。</small>

HttpMessageConverter 有很多种，Spring MVC 针对于不同的场景会使用不同的 Converter 来进行参数绑定和返回值类型转换。什么时候使用 *A-Converter*，什么时候使用 *B-Converter* 取决于 Request 和 Response 的 `Content-Type` 。

SpringMVC 中已经默认提供了相当多的转换器：

- **`StringHttpMessageConverter`** :
    - 用于 body 数据与 String 类型的相互转换
    - 请求类型为 `*/*` 时，被用于参数绑定
    - 响应类型为 `text/pain` 时，被用于返回值类型转换
- **`MappingJackson2HttpMessageConverter`**:
    - 用于 body 数据<small>（逻辑上，它应该是一个JSON格式字符串）</small>与对象的相互转换
    - 请求类型为 `application/json` 时，被用于参数绑定
    - 响应类型为 `application/json` 时，被用于返回值类型转换
- MappingJackson2XmlHttpMessageConverter
    - 用于 body 数据<small>（逻辑上，它应该是一个XML格式字符串）</small>与对象的相互转换
    - 请求类型为 `application/xml` 时，被用于参数绑定
    - 响应类型为 `application/xml` 时，被用于返回值类型转换
    - <small>实际上用不到它。此处只是用它来说明还有其它的 HttpMessageConvert 来处理其他的 `Content-type` 的参数绑定和返回值类型转换。</small>



 `补注`：逻辑上，Response 的 `Content-Type` 应该和 Request 的 `Accept` 是一致的。



另外，StringHttpMessageConverter 有个小坑，它默认使用的是 `iso-8859-1` 编码，因此不支持中日韩文。如果涉及该问题，需要在配置中指定采用 UTF-8 编码，不少 Restful api 返回 JSON 数据中文乱码问题就源自于此。

```xml
<mvc:annotation-driven>
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/plain;charset=UTF-8</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

MappingJackson2HttpMessageConverter 默认是使用的是 UTF-8 编码。不需要额外手动指定。



## @RequestBody

从本质上讲，从前台传到后台 Java 代码中的 Http 请求参数有且仅有两种情况：

- 查询参数字符串（Query String）。形如：`username=tom&password=123`
- JSON 格式字符串。形如：`{'username':'tom', 'password':123}`

如果响应处理方法的某个参数标注了 `@RequestBody` 注解，Spring MVC 根据 Request 的 header 中的 `content-type` 属性的值来适配合适的 HttpMessageConverter 进行参数绑定。 `



> Spring MVC 并没有自动依赖 Jackson 包，因此使用它，需要在 pom 中添加 Jackson 的包。不过 Spring MVC 中有 Jackson Converter 的配置，因此，不需要手动进行配置。如果想使用除 Jackson 之外的包（例如 fastjson），那么在引入包之外，还需要在项目的配置文件中进行配置。



## @ResponseBody

如果响应处理方法标注了 `@ResponseBody` 注解，Spring MVC 根据 Response 的 header 的 `content-type`（它的理论上应该是等同于 Request 的 header 中的 `accept` 属性值），适配合适的 HttpMessageConverter 将请求处理方法的返回值转换成对应的文本字符串，再『塞』进 Response 的 body 中。



## @ResponseStatus

任何一个 HTTP 响应信息都包含 `行-头-体` 三部分。

其中 `响应行` 中最重要的信息就是响应的状态码。200 和 404 就是两个常见的状态码。

在 Servlet 编程领域，本质上来说，每次返回数据给客户端浏览器时，我们应该指定正确的符合逻辑的 HTTP 响应状态码。

```java
response.setStatus(200);
```

Spring MVC 通过 `@ResponseStatus` 简化了手动调用方法设置状态码的过程。

`@ResponseStatus` 注解通常标注于自定义的异常类上。当 Controller 中的方法抛出这个异常时，Spring MVC（背后是 ResponseStatusExceptionResolver 在干活）会根据这个注解的属性设置请求响应码（及对应的响应信息）。

```java
@ResponseStatus(value = HttpStatus.FORBIDDEN, reason = "用户名和密码不匹配!")
public class UserNameNotMatchPasswordException extends RuntimeException{
}
```

没有抛出异常，返回状态码自然是 200 。

另外，不要讲该注解标注在了 Controller 的方法上。这样该方法会执行成功，但前端会看到对应的异常页面。



## ResponseEntity\<T> 类型

Spring MVC 返回 JSON 格式的响应信息时，本质上，需要干两件事情：

1. 设置响应状态码和对应的信息（`@ResponseStatus`，设置 Response 的响应行）
2. 将 Controller 方法返回的对象转换成 JSON 格式字符串，后发回（`@ResponseBody`，设置 Response 的响应体）。

*一般情况下，我们不会/不需要手动去设置响应头*

**`@ResponseEntity`** 相当于一个人去干 @ResponseStatus 和 @ResponseBody 两个人干的活。

```java
@RequestMapping("/test1")
public ResponseEntity<List<Department>> test1(){
   List<Department> list = ...; 
   return ResponseEntity.ok(list); // 等同于
// return ResponseEntity.status(HttpStatus.OK).body(dept);
}

@RequestMapping("/test2")
public ResponseEntity<String> test2(){
    return ResponseEntity.badRequest().body("..."); // 等同于
//  return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(...)
}
```

注意，`@ResponseEntity` 和 `@ResponseStatus` 一起使用是无效的。另外，标注了 `@ResponseEntity`  注解，就不需要再标注 `@ResponseBody` 注解 。


『完』

