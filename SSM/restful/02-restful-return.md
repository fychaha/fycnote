## Status codes

| 请求方式 | 描述|
| :- | :- |
| `GET`	| 访问成功返回 `200 OK` 状态码，并将所访问的一个或多个资源以 JSON 形式返回 |
| `POST` | 创建成功时返回 `201 Created` 状态码，并将新创建的资源以 JSON 形式返回 |
| `PUT`	| 修改成功时返回 `200 OK` 状态码，并将所修改的资源以 JSON 形式返回 |
| `DELETE`	| 删除成功时返回 `204 No Content` 状态码 |

| 状态码 | 描述 |
| :- | :- |
| `200 OK` | The `GET`, `PUT` or `DELETE` request was successful, the resource(s) itself is returned as JSON.|
| `204 No Content` | The server has successfully fulfilled the request and that there is no additional content to send in the response payload body.|
| `201 Created` | The `POST` request was successful and the resource is returned as JSON.|
| `304 Not Modified` | Indicates that the resource has not been modified since the last request.|
| `400 Bad Request` | 	A required attribute of the API request is missing, e.g., the title of an issue is not given.|
| `401 Unauthorized` | 	The user is not authenticated, a valid user token is necessary.|
| `403 Forbidden` | The request is not allowed, e.g., the user is not allowed to delete a project.|
| `404 Not Found` | 	A resource could not be accessed, e.g., an ID for a resource could not be found.|
| `405 Method Not Allowed`|	The request is not supported.|
| `409 Conflict` | A conflicting resource already exists, e.g., creating a project with a name that already exists.|
| `412` | Indicates the request was denied. May happen if the `If-Unmodified-Since` header is provided when trying to delete a resource, which was modified in between.|
| `422 Unprocessable` | The entity could not be processed.|
| `500 Server Error` | While handling the request something went wrong server-side.|


## Pagination

分页请求参数

| Parameter	    | Description|
| :-            | :- |
| `page`        | Page number (default: `1`) |
| `per_page`    | Number of items to list per page (default: `20`, max: `100`) |

分页信息响应返回头附加信息


| Header | Description |
| :- | :- | 
| `X-Total`	| The total number of items |
| `X-Total-Pages`	| The total number of pages |
| `X-Per-Page`	| The number of items per page |
| `X-Page`	| The index of the current page (starting at 1) |
| `X-Next-Page`	| The index of the next page |
| `X-Prev-Page`	| The index of the previous page |

## Data validation and error reporting

后台进行必要的参数校验而又未通过时，返回 `400` 响应码。

错误原因分为两类：

- 参数缺失
- 参数没有通过校验（`合法` 而又 `有效`）

`参数缺失`时，返回类似如下信息：

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "message": "\"xxx\" not given"
}
```

`参数没有通过校验`时，返回类似如下信息：

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "message": {
        "xxx": [
            "is too long (maximum is 255 characters)",
            "...",
            "..."
        ]
    }
}
```

通用格式为：

```json
{
    "message": {
        "<property-name>": [
            "<error-message>",
            "<error-message>",
            ...
        ],
        "<embed-entity>": {
            "<property-name>": [
                "<error-message>",
                "<error-message>",
                ...
            ],
        }
    }
}
```


## Unknown route

```json
HTTP/1.1 404 Not Found
Content-Type: application/json
{
    "error": "404 Not Found"
}
```
