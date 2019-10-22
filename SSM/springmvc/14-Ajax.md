# $.ajax() 和 SpringMVC

## 一、ajax() 方法

```js
$.ajax({
    url: ...,
    method: ...,
    contentType: ...,
    data: ...,
    dataType: ...,
    success : function(result) { ... }
    error: function(XMLHttpRequest, textStatus, errorThrown) {...}
    complete: function(XMLHttpRequest, textStatus) {...}
});
```

| 属性名 | 类型 | 说明 |
| :- | :- | :- |
| url         | string   | 所请求的 URL |
| method      | string   | 请求方式 |
| contentType | string   | 所发送的参数数据的格式/类型<br>默认值为： `application/x-www-form-urlencoded` |
| data        | string   | 所发送的请求的参数数据 | |
| dataType    | string   |（预期的）服务器返回的数据格式/类型。使用时，通常就是 `json`。 <br>注意，一不小心很容易错写成 `application/json`，<br>从而导致程序总是进入 error 部分。 |
| success     | function | 成功收到服务器回的数据之后执行的回调函数 |
| error       | function | 收到服务器返回的错误信息之后执行的回调函数 |

## 二、常见的三个工具方法

```js
var str = $.param(obj);         // 对象转请求参数字符串
var str = JSON.stringify(obj);  // 对象转json格式字符串
var obj = JSON.parse(json_str); // json格式字符串转对象
```

## 三、`ajax()` 发送普通请求的参数

这种情况下，和表单 `<form>` 的提交传参没有任何区别。Spring MVC 在后台都是通过 `@RequestParam` 进行参数绑定，获得 get/post 请求的请求参数。

无论是 get 类型的请求，还是 post 类型的请求，`ajax()` 方法的 `data` 属性需要的是一个请求参数字符串。

不过，get 类型的请求可以不必放在 `data` 属性部分，而是直接拼接在 `uri` 之后。

```js
$.ajax({
  ...
  data: 'empno=20&ename=tom',
  ...
});
```

原则上，此时 `data` 属性是需要一个请求参数字符串的。不过，`ajax()` 方法在此处有一个简化：你可以直接给 `data` 指定一个对象，由 `ajax()` 方法它自己去将对象转成请求参数字符串，而不需要我们手动来调用 `$.param()` 方法进行转换。



### `ajax()` 发送简单类型数组的一个坑

类似于表单元素 checkbox 的那种情况，有时候，你需要通过 $.ajax() 向后台传递同一个 key 的多个 value 。

```javascript
var nums = [1, 2, 3];
var str = $.param({"xxx": nums});
console.info(str);  // 注意此处的输出！

$.ajax({
  ...
  data: str,
  ...
});
```

这种情况，本质上和普通的请求中的 checkbox 有很大（但又很不起眼）的不同，jQuery 会在请求参数字符串的 key 的名字中加上 `%5b%5d`，其实就是 `[]` 。

因此，在 SpringMVC 的 `@RequestParam` 中指明的请求参数并不是 `xxx`，而应该是 `xxx[]` 。

```java
public void demo(@RequestParam("xxx[]") Integer[] prodNums) {
  ...
}
```

### 2. ajax 发送 json 对象

通过 ajax 发送请求参数字符串<small>（所谓的发送 js 对象，本质上还是在其内部转换成了请求参数字符串）</small>，SpringMVC 在接受参数时，各个参数是『散』在方法的形参中的。

对于简单的情况，借助高级的参数绑定功能，SpringMVC 也可以将各个参数『收拢』到一个 JavaBean 中。

```javascript
var emp = {
  empno: 20,
  ename: 'tom'
};

var str = $.param(emp); // empno=20&ename=tom

$.ajax({
  ...
  data: str,  // emp
  ...
});
```

```java
public void demo(Integer empno, String ename) {
}

public void demo(Employee emp) {	// 注意，此处不需要 @RequestParam 注解
}
```

### 3. ajax 发送 `application/json` 参数类型的请求

`contentType` 没有赋值时，其默认值是 `application/x-www-form-urlencoded` 表示是普通的 get/post 请求。

如果，我们将 `contentType` 赋值为 `application/json` 表示向后台发起请求时，是将一个 JSON 格式的字符串携带在了 Request 的 body 部分，需要 Spring MVC 通过 `@RequestBody` 进行参数绑定，获取并解析出这个 JSON 格式字符串。

例如，向后台传递一个对象的数组：

```javascript
var emp1 = { empno: 20, ename: 'tom' };
var emp2 = { empno: 19, ename: 'ben' };
var arr = [emp1, emp2];

var str = JSON.stringify(arr); // [{"empno":20,"ename":"tom"},{"empno":21,"ename":"jerry"}]

$.ajax({
  ...
  data: str,
  contentType: "application/json", // “额外”说明，这次发送的数据是一个 json 格式字符串，而不是参数字符串
  ...                              
});
```

```java
public void add(@RequestBody Employee[] emps) {
}
```

## 四、ajax 方法的 error 参数


当 http 响应的状态码不是 `200` 的时候，就会执行 error function 。

> `NOTE`：一个常见的 Bug 是 `data` 属性值本应是 `json`，但是不小心错写成 `applicaiton/json`。
>
> 这种情况下，ajax 方法总是会进入 error 部分。因为预期的返回的数据类型（`application/json`）与实际类型（`json`）并不一致，也算是 error 。

一般 error 函数返回的参数有三个：`function(XMLHttpRequest, textStatus, errorThrown)`。我们关注的是第一个 `XMLHttpRequest` 。

从第一个参数 `XMLHttpRequest` 中我们可以获得服务端返回的错误相关的信息：

- `XMLHttpRequest.status`
  - 返回的 HTTP 状态码，例如 `404`、`500` 等错误代码。
- `XMLHttpRequest.statusText` 
  - 对应状态码的错误信息。比如 `404` 错误信息是 `not found`；`500` 是 `Internal Server Error` 。
- `XMLHttpRequest.responseText`
  - 服务器响应返回的文本信息，即，Response 的 body 的内容。






