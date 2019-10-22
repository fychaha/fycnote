# 使用 MockMVC 测试 Controller

对模块进行集成测试时，希望能够通过输入 URL 对 Controller 进行测试。MockMvc 实现了对 Http 请求的模拟，能够直接使用网络的形式，转换到 Controller 的调用，这样可以使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。

MockMVC 测试步骤：

| # |说明|
|:-|:-|
| 1 | MockMvcBuilder 构造 MockMvc 的构造器 |
| 2 | mockMvc 调用 perform，执行一个 RequestBuilder 请求，调用 controller 的业务处理逻辑 |
| 3 | perform 返回 ResultActions，返回操作结果，通过 ResultActions，提供了统一的验证方式 |
| 4 | 使用 StatusResultMatchers 对请求结果进行验证 |
| 5 | 使用 ContentResultMatchers 对请求返回的内容进行验证 |

spring mvc 测试框架提供了两种方式，**独立安装** 和 **集成 Web 环境测试**（此种方式并不会集成真正的 web 环境，而是通过相应的 Mock API 进行模拟测试，无须启动服务器。推荐使用第二种。）

构建集成 Web 环境的 MockMVC，直接使用静态工厂 MockMvcBuilders 创建即可：

- `MockMvcBuilders.webAppContextSetup(WebApplicationContext context)`
  > 指定 WebApplicationContext，将会从该上下文获取相应的控制器并得到相应的MockMvc；

```java
@Autowired
private WebApplicationContext wac;  

protected MockMvc mockMvc;

@Before() // 这个方法在每个方法执行之前都会执行一遍
public void setup() {
    mockMvc = MockMvcBuilders.webAppContextSetup(wac).build(); // 初始化MockMvc对象
}
```

集成 Web 环境测试情况下的使用步骤：

| # |说明|
|:-|:-|
| 1 | `mockMvc.perform()` 执行一个请求；|
| 2 | `MockMvcRequestBuilders.get("/user/1")` 构造一个请求 |
| 3 | `ResultActions.andExpect` 添加执行完成后的断言 |
| 4 | `ResultActions.andDo` 添加一个结果处理器，表示要对结果做点什么事情。<br>比如，此处使用 MockMvcResultHandlers.print() 输出整个响应结果信息。|
| 5 | `ResultActions.andReturn` 表示执行完成后返回相应的结果。|

- perform
  > 执行一个 RequestBuilder 请求，会自动执行 Spring MVC 的流程并映射到相应的控制器执行处理；
- andExpect
  > 添加 ResultMatcher 验证规则，验证控制器执行完成后结果是否正确；
- andDo
  > 添加 ResultHandler 结果处理器，比如调试时打印结果到控制台；
- andReturn
  > 最后返回相应的 MvcResult；然后进行自定义验证/进行下一步的异步处理；

```java
@Test
public void demo() throws Exception {

    String responseString = mockMvc.perform(
        get("/api/employee")         // 请求的url,请求的方法是get
            .contentType(MediaType.APPLICATION_FORM_URLENCODED)  // 数据的格式
            .param("pcode", "root")) // 添加参数
        .andExpect(status().isOk())  // 返回的状态是200
        .andDo(print())              // 打印出请求和相应的内容
        .andReturn().getResponse().getContentAsString();   // 将相应的数据转换为字符串
    System.out.println("--------返回的json = " + responseString);
}
```


MockMvcRequestBuilders 除了有 get() 方法外，还有：

|方法| 说明 |
| :- | :- |
| `post()` | 发起 Post 请求 |
| `put()` | 发起 Put 请求 |
| `delete()` | 发起 Delete 请求 |
| ... | 其它 |


ResultActions

调用 MockMvc.perform(RequestBuilder requestBuilder) 后将得到 ResultActions，通过 ResultActions 完成如下三件事：

1. `ResultActions andExpect(ResultMatcher matcher)` 
   > 添加验证断言来判断执行请求后的结果是否是预期的
2. `ResultActions andDo(ResultHandler handler)` 
   > 添加结果处理器，用于对验证成功后执行的动作，如输出下请求/结果信息用于调试
3. `MvcResult andReturn()` 
   >  返回验证成功后的 MvcResult；用于自定义验证/下一步的异步处理；(主要是拿到结果进一步做自定义断言)

```java
String requestBody = "{\"id\":1, \"name\":\"zhang\"}";  
mockMvc.perform(post("/user")  
        .contentType(MediaType.APPLICATION_JSON).content(requestBody)  
        .accept(MediaType.APPLICATION_JSON)) // 执行请求
        .andExpect(content().contentType(MediaType.APPLICATION_JSON)) // 验证响应contentType
        .andExpect(jsonPath("$.id").value(1)); // 使用Json path验证JSON 请参考http://goessner.net/articles/JsonPath/  

String errorBody = "{\"id\":1, \"name\":zhang}";  
MvcResult result = mockMvc.perform(post("/user")  
        .contentType(MediaType.APPLICATION_JSON).content(errorBody)  
        .accept(MediaType.APPLICATION_JSON)) //执行请求  
        .andExpect(status().isBadRequest()) //400错误请求  
        .andReturn();
```