# 有关 RESTful API 良好设计的实践

## 有版本的 API 

在 url 中指定 API 的版本是个很好地做法。如果 API 变化比较大，可以把 API 设计为子域名。

> <p>https://api.example.com/v{n}/</p>


## 以资源为中心的 URL 设计 


『**资源**』 是 Restful API 的核心元素，所有的操作都是针对特定资源进行的。而资源就是 URL（Uniform Resoure Locator）表示的，所以简洁、清晰、结构化的 URL 设计是至关重要的。

Github 可以说是这方面的典范，下面我们就拿 repository 来说明。

```
/users/:username/repos

/users/:org/repos

/repos/:owner/:repo

/repos/:owner/:repo/tags

/repos/:owner/:repo/branches/:branch
```

我们可以看到几个特性：

- 资源分为单个文档和集合，尽量使用复数来表示资源，单个资源通过添加 id 或者 name 等来表示

- 一个资源可以有多个不同的 URL

- 资源可以嵌套，通过类似目录路径的方式来表示，以体现它们之间的关系

> `NOTE`：根据 RFC3986 定义，URL 是大小写敏感的。所以为了避免歧义，尽量使用小写字母。


## 使用正确的 Method

| VERB	| 描述 |
| GET	| 获取资源 |
| POST	| 创建资源 |
| PATCH	| 更新资源的部分属性。因为 PATCH 比较新，而且规范比较复杂，所以真正实现的比较少，一般都是用 POST 替代 <small>见最后</small>|
|PUT	| 替换资源，客户端需要提供新建资源的所有属性。如果新内容为空，要设置 Content-Length为 0，以区别错误信息 |
| DELETE	| 删除资源 |


```
GET /repos/:owner/:repo/issues

GET /repos/:owner/:repo/issues/:number

POST /repos/:owner/:repo/issues

PATCH /repos/:owner/:repo/issues/:number

DELETE /repos/:owner/:repo
```

> `NOTE`：更新和创建操作应该返回最新的资源，来通知用户资源的情况；删除资源一般不会返回内容。


反例：

```
/getAllCars
/createNewCar
/deleteAllRedCars
```


## 不符合 CRUD 的情况

在实际资源操作中，总会有一些不符合 CRUD（Create-Retrieve-Update-Delete） 的情况，一般有几种处理方法。

### 使用 POST

为需要的动作增加一个 endpoint（uri 末端），使用 POST 来执行动作，比如 `POST /.../resend` 表示重新发送邮件。

### 增加控制参数

添加动作相关的参数，通过修改参数来控制动作。

比如一个博客网站，会有把写好的文章“发布”的功能

- 可以用上面的 `POST /articles/{:id}/publish` 方法，

- 也可以在文章中增加 `published:boolean` 字段，发布的时候就是更新该字段 `PUT /articles/{:id}?published=true`

### 把动作转换成资源

（换一个思路）把动作转换成某种可以执行 CRUD 操作的资源， github 就是用了这种方法。

比如『喜欢』一个 gist，就增加一个 `/gists/:id/star` 子资源，然后对其进行操作：『喜欢』使用 `PUT /gists/:id/star`，『取消喜欢』`使用 DELETE /gists/:id/star` 。


### 过滤信息

如果记录数量很多，服务器不可能都将它们返回给用户。API 应该提供参数，过滤返回结果。
下面是一些常见的参数。

> `?limit=10`：指定返回记录的数量
>
> `?offset=10`：指定返回记录的开始位置。
> 
> `?page=2&per_page=100`：指定第几页，以及每页的记录数。
> 
> `?sortby=name&order=asc`：指定返回结果按照哪个属性排序，以及排序顺序。
> 
> `?producy_type=1`：指定筛选条件


## 选择合适的状态码

大体上来说：

- 2xx 表示成功；

- 4xx 表示由于客户端的原因，导致该请求失败（客户端做出修正后，在其请求有可能成功）；

- 5xx 表示由于服务端的原因导致该请求失败（客户端再次请求，仍有可能失败）。

只要 API 接口成功接到请求，就不能返回 200 以外的 HTTP 状态。

|操作|状态|返回码|信息|返回|
|:-|:-|:-|:-|:-|
| POST   | 成功 |201|CREATED|创建的数据|
| DELETE | 成功 |204|NO CONTENT|无|
| PUT    | 成功 |200|SUCCESS|对应的数据|
| PATCH  | 成功 |200|SUCCESS|对应的数据|
| GET    | 成功 |200|SUCCESS|对应的数据|
|        | 失败 |404|NOT FOUND|无|
| 任何请求 | 参数校验失败 | 400 | BAD REQUEST|无|
| 任何请求 | 没有通过身份认证 |401|NOT AUTHORIZED|无|
| 任何请求 | 权限不够 |403|FORBIDDEN|无|
| 任何请求 | 服务器内部错误 | 500 |Internal Server Error	 | 无 |


### 分页

自己实现分页机制，策略是：返回资源集合时，包含于分页有关的数据：

```javascript
{
  "page": 1,            # 当前是第几页
  "pages": 3,           # 总共多少页
  "per_page": 10,       # 每页多少数据
  "has_next": true,     # 是否有下一页数据
  "has_prev": false,    # 是否有前一页数据
  "total": 27           # 总共多少数据
}
```

当向 API 请求资源集合时，可选的分页参数为：

|参数	|含义|
|:-|:-|
|page	|当前是第几页，默认为1|
|per_page	|每页多少条记录，默认为系统默认值|

另外，系统内还设置一个 per_page_max 字段，用于标记系统允许的每页最大记录数，当 per_page 值大于 per_page_max 值时，每页记录条数为 per_page_max。

### 非 Restful API 的需求

页面级的 API，把当前页面中需要用到的所有数据通过一个接口一次性返回全部数据。例如：`api/v1/get-home-data`，返回首页用到的所有数据。这类 API 有一个非常不好的地址，只要业务需求变动，这个 API 就需要跟着变更。

自定义组合 API，把当前用户需要在第一时间内容加载的多个接口合并成一个请求发送到服务端，服务端根据请求内容，一次性把所有数据合并返回，相比于页面级 API，具备更高的灵活性，同时又能很容易的实现页面级的 API 功能。此时，

传入参数：
```javascript
data:[
    {url:'api1', type:'get', data:{...}},
    {url:'api2', type:'get', data:{...}},
    {url:'api3', type:'get', data:{...}},
    {url:'api4', type:'get', data:{...}}
]
```

返回数据
```javascript
{
  status:0,
     msg:'',
    data:[
        {status: 0, msg:'...', data:[...]},
        {status:-1, msg:'...', data:{...}},
        {status: 1, msg:'...', data:{...}},
        {status: 0, msg:'...', data:[...]},
    ]
}
```

## 其他

RESTful API 应具备良好的可读性，当 URL 中某一个片段（segment）由多个单词组成时，建议使用 `-` 来隔断单词，而不是使用 `_`


##  Spring MVC 中 PUT、PATCH 请求参数

RESTful API 要求服务端能响应：get / post / put / delete 请求，以对应增删改查四大功能。对此 Spring MVC 都能支持，只需要在请求处理方法头上加上：

```java
@RequestMapping(value="/...", method = RequestMethod.GET)
```

就能表示该方法仅针对于特定请求方式作出响应。

但是，出于某些原因，默认 Spring MVC 不会接受 PUT 请求和 POST 请求发送来参数！！！

> `@RequestParam` 实际上只支持 `application/x-www-form-urlencoded` 方式的 GET 和 POST 请求的参数传递。不支持 PUT、PATCH（也包括 DELETE）请求的参数的获取。
>
> （这不能怪 Spring MVC）`@RequestParam` 注解实际上最终利用的还是 Servlet 体系中的 `request.getParameter("");` 方法，而 `getParameter()` 方法本身就只支持 GET 和 POST 方法传参。
> 
> 看看 Servlet-api 你会发现实际上 PATCH 更惨，Servlet-api 中甚至都没有 `doPatch()` 方法，更谈不上获取请求参数了。

解决 PUT、PATCH 的参数传递的方案有两种：

### 方案一：使用 HttpPutFormContentFilter 

Spring MVC 从 3.1 开始提供了一个 Filter（过滤器）来解决这个传参问题。

```xml
<filter>	<!-- 默认 Spring MVC 无法接受 PUT 请求参数 -->
    <filter-name>httpPutFormContentFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpPutFormContentFilter</filter-name>
    <url-pattern>/*</url-pattern>   <!-- 因为匹配优先级问题，这里必须是 /* ，不能是 / -->
</filter-mapping>
```

这个过滤器会拦截所有的 PUT 和 PATCH 请求，将它们的底层处理方式转变为 POST 请求处理方式，这样这些 PUT 和 PATCH 请求的参数的获取变得和 POST 请求一样。

### 方法二：使用 application/json 曲线救国 

PUT 和 PATCH 的请求参数默认是以 `x-www-form-urlencoded` 的 `contentType` 来发送信息，即请求参数是放在 request 的 body 中的。在默认情况下，`@ReqeustParam` 是不回去获取它们在 body 中的请求参数的（除非采用方案一，挂羊头卖狗肉）。

方案二的本质就是索性不使用 `@RequestParam` 获取参数，而使用 `@RequestBody` 。

利用 `application/json` 方式传奇请求参数，将参数以 JSON 格式字符串的形式放在 request 的 body 中，在 Controller 中再配合 `@RequestBody` 进行参数绑定，从而获取参数。绕开 `@RequestParam` / `request.getParam()` 的限制。
