# API 接口设计指北

文档深受受 [Github 接口文档](http://developer.github.com/v3/)以及文中所有提及的协议、标准和文章的影响，因此在顶部注明，感谢。

## URL

HOST 地址：

    http://api.example.com

所有 URI 都需要遵循 [RFC3986](http://tools.ietf.org/html/rfc3986) 的要求。

## 空字段

接口遵循“输入宽容，输出严格”原则，输出的数据结构中空字段的值一律为 `null`

## 国际化

### 语言名称

[RFC 5646](http://tools.ietf.org/html/rfc5646) 规定的语言的标签的格式如下：

```
language-script-region-variant-extension-privateuse
```

1. language：这部分是 [ISO 639](http://www.loc.gov/standards/iso639-2/php/code_list.php) [wikipedia](http://zh.wikipedia.org/wiki/ISO_639-1) 规定的代码，比如中文是 zh。
2. script：表示变体，比如简体汉字是 zh-Hans ，繁体汉字是 zh-Hant 。
3. region：是 [ISO 3166-1](javascript:;) [wikipedia](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) 规定的地理区域，比如 zh-Hans-CN 就是中国大陆使用的简体中文。
4. variant：表示方言。
5. extension：表示扩展。
6. privateus：表示私有标识。

**有一点需要注意，任何合法的标签都必须经过IANA的认证，已通过认证的标签可以在[这个网页](http://www.iana.org/assignments/language-subtag-registry)查到。此外，网上还有一个非官方的[标签搜索引擎](http://people.w3.org/rishida/utils/subtags/)。**

相关链接：

* Android 文档：[http://developer.android.com/guide/topics/resources/providing-resources.html#LocaleQualifier](http://developer.android.com/guide/topics/resources/providing-resources.html#LocaleQualifier) ，顺便鄙视 iOS 文档在获取语言接口的相关文档里根本不提这个。
* 《语种名称代码》：[http://www.ruanyifeng.com/blog/2008/02/codes_for_language_names.html](http://www.ruanyifeng.com/blog/2008/02/codes_for_language_names.html)
* 《Language tags in HTML and XML》：[http://www.w3.org/International/articles/language-tags/](http://www.w3.org/International/articles/language-tags/)

### 时区

客户端请求服务器时，如果对时间有特殊要求（如某段时间每天的统计信息），则可以参考 [IETF 相关草案](http://tools.ietf.org/html/draft-sharhalakis-httptz-05) 增加请求头 `Timezone: Asia/Shanghai` ，或者现在可能更常见的请求头（ GitHub 使用的） `Time-Zone: Asia/Shanghai` 。

时区的名称可以参考 [tz datebase](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 。

考虑到维护这样一份列表可能会产生一定的困扰，因此也可以使用请求头 `Time-Offset: +0800` 来替代（[来源](http://blogs.windwardreports.com/davidt/2010/04/can-we-please-get-a-time-zone-in-the-http-request-header.html?cid=6a0115711bf0ae970b01347fd28db8970c#comment-6a0115711bf0ae970b01347fd28db8970c-content)）

如果客户端请求时没有指定相应的时区，则服务端默认使用 [UTC](http://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6) 时间返回相应数据。

### 时间格式

时间格式遵循 [ISO 8601](http://zh.wikipedia.org/w/index.php?title=ISO_8601) 表示方法

    YYYY-MM-DDTHH:MM:SSZ

## 请求方法

* [维基百科](http://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE#.E8.AF.B7.E6.B1.82.E6.96.B9.E6.B3.95)
* [RFC2616](http://tools.ietf.org/html/rfc2616)
* 如果请求头中存在 `X-HTTP-Method-Override` 或参数中存在 `_method`（拥有更高权重），且值为 `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTION`, `HEAD` 之一，则视作相应的请求方式进行处理
* `GET`, `DELETE`, `HEAD` 方法，参数风格为标准的 `GET` 风格的参数，如 `url?a=1&b=2`
* `POST`, `PUT`, `PATCH`, `OPTION` 方法
    * 默认情况下请求实体会被视作标准 json 字符串进行处理，当然，依旧推荐设置头信息的 `Content-Type` 为 `application/json`
    * 在一些特殊接口中（会在文档中说明），可能允许 `Content-Type` 为 `application/x-www-form-urlencoded` 或者 `multipart/form-data` ，此时请求实体会被视作标准 `POST` 风格的参数进行处理

关于方法语义的说明：

* `OPTIONS` 用于获取资源支持的所有 HTTP 方法
* `HEAD` 用于只获取请求某个资源返回的头信息
* `GET` 用于从服务器获取某个资源的信息
    * 完成请求后返回状态码 `200 OK`
    * 完成请求后需要返回被请求的资源详细信息
* `POST` 用于创建新资源
    * 创建完成后返回状态码 `201 Created`
    * 完成请求后需要返回被创建的资源详细信息
* `PUT` 用于完整的替换资源或者创建指定身份的资源，比如创建 id 为 123 的某个资源
    * 如果是创建了资源，则返回 `201 Created`
    * 如果是替换了资源，则返回 `202 Accepted`
    * 完成请求后需要返回被修改的资源详细信息
* `PATCH` 用于局部更新资源
    * 完成请求后返回状态码 `202 Accepted`
    * 完成请求后需要返回被修改的资源详细信息
* `DELETE` 用于删除某个资源
    * 完成请求后返回状态码 `204 No Content`

## 状态码

* [维基百科](http://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
* [四个新的 HTTP 状态码](http://www.oschina.net/news/28660/new-http-status-codes)
* [RFC2616](http://tools.ietf.org/html/rfc2616)
* [RFC6585](http://tools.ietf.org/html/rfc6585)
* [Do I need to use http redirect code 302 or 307? - Stack Overflow](http://stackoverflow.com/questions/2467664/do-i-need-to-use-http-redirect-code-302-or-307)

### 请求成功

* 200 **OK** : 请求执行成功并返回相应数据，如 `GET` 成功
* 201 **Created** : 对象创建成功并返回相应资源数据，如 `POST` 成功；创建完成后响应头中应该携带头标 `Location` ，指向新建资源的地址
* 202 **Accepted** : 更新成功并且返回相应资源数据，如 `PUT` ， `PATCH` 成功
* 204 **No Content** : 请求执行成功，不返回相应资源数据，如 `PATCH` ， `DELETE` 成功

### 重定向

**重定向的新地址都需要在响应的 `Location` 头标中返回**

* 301 **Moved Permanently** : 被请求的资源已永久移动到新位置
* 302 **Found** : 请求的资源现在临时从不同的 URI 响应请求
* 303 **See Other** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该使用 `GET` 方法进行请求
* 307 **Temporary Redirect** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该保持原有的请求方法进行请求

### 客户端出错

* 400 **Bad Request** : 请求体包含语法错误
* 401 **Unauthorized** : 需要验证用户身份
* 403 **Forbidden** : 服务器拒绝执行
* 404 **Not Found** : 找不到目标资源
* 405 **Method Not Allowed** : 不允许执行目标行为
* 409 **Conflict** : 被请求的资源的当前状态之间存在冲突
* 410 **Gone** : 被请求的资源已被删除
* 413 **Request Entity Too Large** : 请求实体过大
* 415 **Unsupported Media Type** : 当前请求的方法和所请求的资源不支持请求中提交的实体的格式
* 422 **Unprocessable Entity** : 请求格式正确，但是由于含有语义错误，无法响应
* 428 **Precondition Required** : 要求先决条件，如果想要请求能成功必须满足一些预设的条件

### 服务端出错

* 500 **Internal Server Error** : 服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。
* 501 **Not Implemented** : 服务器不支持当前请求所需要的某个功能。
* 502 **Bad Gateway** : 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
* 503 **Service Unavailable** : 由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 `Retry-After` 头用以标明这个延迟时间（内容可以为数字，单位为秒；或者是一个 [HTTP 协议指定的时间格式](http://tools.ietf.org/html/rfc2616#section-3.3)）。如果没有给出这个 `Retry-After` 信息，那么客户端应当以处理 500 响应的方式处理它。

`501` 与 `405` 的区别是：`405` 是表示服务端不允许客户端这么做，`501` 是表示客户端或许可以这么做，但服务端还没有实现这个功能

## 错误处理

在调用接口的过程中，可能出现下列几种错误情况：

* 服务器维护中，`503` 状态码

    ```http
    HTTP/1.1 503 Service Unavailable
    Retry-After: 3600
    Content-Length: 41

    {"message": "Service In the maintenance"}
    ```

* 发送了无法转化的请求体，`400` 状态码

    ```http
    HTTP/1.1 400 Bad Request
    Content-Length: 35

    {"message": "Problems parsing JSON"}
    ```

* 服务到期（比如付费的增值服务等）， `403` 状态码

    ```http
    HTTP/1.1 403 Forbidden
    Content-Length: 29

    {"message": "Service expired"}
    ```

* 因为某些原因不允许访问（比如被 ban ），`403` 状态码

    ```http
    HTTP/1.1 403 Forbidden
    Content-Length: 29

    {"message": "Account blocked"}
    ```

* 权限不够，`403` 状态码

    ```http
    HTTP/1.1 403 Forbidden
    Content-Length: 31

    {"message": "Permission denied"}
    ```

* 需要修改的资源不存在， `404` 状态码

    ```http
    HTTP/1.1 404 Not Found
    Content-Length: 32

    {"message": "Resource not found"}
    ```

* 缺少了必要的头信息，`428` 状态码

    ```http
    HTTP/1.1 428 Precondition Required
    Content-Length: 35

    {"message": "Header User-Agent is required"}
    ```

* 发送了非法的资源，`422` 状态码

    ```http
    HTTP/1.1 422 Unprocessable Entity
    Content-Length: 149

    {
      "message": "Validation Failed",
      "errors": [
        {
          "resource": "Issue",
          "field": "title",
          "code": "missing_field"
        }
      ]
    }
    ```

所有的 `error` 哈希表都有 `resource`, `field`, `code` 字段，以便于定位错误，`code` 字段则用于表示错误类型：

* `missing`: 说明某个字段的值代表的资源不存在
* `invalid`: 某个字段的值非法，接口文档中会提供相应的信息
* `missing_field`: 缺失某个必须的字段
* `already_exist`: 发送的资源中的某个字段的值和服务器中已有的某个资源冲突，常见于某些值全局唯一的字段，比如 @ 用的用户名（这个错误我有纠结，因为其实有 409 状态码可以表示，但是在修改某个资源时，很一般显然请求中不止是一种错误，如果是 409 的话，多种错误的场景就不合适了）

## 身份验证

部分接口需要通过某种身份验证方式才能请求成功（这些接口会在文档中标注出来），身份验证支持 [HTTP 基本认证](http://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)，也支持通过登录接口使用账号密码换取 token ，在请求接口时使用 `Authorization: token #{token}` 头标或者 `token` 参数的值的方式进行验证。

## 超文本驱动和资源发现

REST 服务的要求之一，客户端不再需要将某些接口的 URI 硬编码在代码中，唯一需要存储的只是 API 的 HOST 地址，能够非常有效的降低客户端与服务端之间的耦合，服务端对 URI 的任何改动都不会影响到客户端的稳定。

目前有两种方案备选：

* [Web Linking](http://tools.ietf.org/html/rfc5988) ，示例可以参考*分页*一节。
* [JSON HAL 草案](http://tools.ietf.org/html/draft-kelly-json-hal-05) ，示例可以参考 [JSON HAL 作者自己的介绍](http://stateless.co/hal_specification.html)

提到 `Web Linking` 因为这类与资源无关的元信息不适合放在响应体中，提到 `JSON HAL` 是因为在很多时候一个资源会有一些关联资源，如： `post.user.name` ，在这类情况下 `user` 客户端在当前是无法知道资源的相关操作的，`JSON HAL` 在不具备方便的缓存工具的情况下比较好的规避了这个问题。

我赞同文章 《[Should RESTful API's Include Relationships](http://idbentley.com/blog/2013/03/14/should-restful-apis-include-relationships/)》 的说法并更加倾向于使用 `Web Linking` 。

客户端缓存所有的请求，在需要知道某个关联资源的相关操作的时候使用 `HEAD` 方法，URL 的格式遵从 [URI Template](http://tools.ietf.org/html/rfc6570) 的语法：

```http
GET http://api.example.com/users/33221

HTTP/1.1 200 OK
Link: <http://api.example.com/users/33221>; rel="self",
      <http://api.example.com/users/33221>; rel="edit",
      <http://api.example.com/team{/team}>; rel="team_url",
      <http://api.example.com/projects{/project}>; rel="project_url",
      <http://api.example.com/users/{owner}/{repo}>; rel="repository_url"

{
  "id": 596502,
  "name": "test user",
  "projects": [160418896, 160418897, 160418898, 160418899],
  "team": 94665
}
```

然而客户端有可能处于没有缓存的的状态中，因此接口可以适当地支持文章 《[Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)》 的 《[Auto loading related resource representations](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#autoloading)》 小节中提到的 `embed` 参数。

不过个人觉得文中的实现方式成本太高而且意义不大，因此进行了简化：

```http
GET http://api.example.com/users/33221?embed=projects,team

HTTP/1.1 200 OK
Link: <http://api.example.com/users/33221>; rel="self",
      <http://api.example.com/users/33221>; rel="edit",
      <http://api.example.com/team{/team}>; rel="team_url",
      <http://api.example.com/projects{/project}>; rel="project_url",
      <http://api.example.com/users/{owner}/{repo}>; rel="repository_url"

{
  "id": 596502,
  "name": "test user",
  "projects": [{
    "id": 160418896,
    "name": "project1",
    "description": "description of project1"
  }, {
    "id": 160418897,
    "name": "project2",
    "description": "description of project2"
  }, {
    "id": 160418898,
    "name": "project3",
    "description": "description of project3"
  }, {
    "id": 160418899,
    "name": "project4",
    "description": "description of project4"
  }],
  "team": {
    "id": 94665,
    "name": "my team",
    "members": [596502, 596503, 596504, 596505, 596506]
  }
}
```

## 分页

请求某个资源集合时，可以通过指定 `count` 参数来指定每页的资源数量，通过 `page` 参数指定页码。

如果没有传递 `count` 参数或者 `count` 参数的值为空，则使用默认值 20 ， `count` 参数的最大上限为 100 。

分页的相关信息会包含在 [Link Header](http://tools.ietf.org/html/rfc5988) 和 `X-Resource-Count` 中。

如果是第一页或者是最后一页时，不会返回 `prev` 和 `next` 的 Link 。

更多 `rel` 相关信息，可以参阅 [RFC5988 6.2.2节](http://tools.ietf.org/html/rfc5988#section-6.2.2) 。

```http
HTTP/1.1 200 OK
X-Resource-Count: 542
Link: <http://api.example.com/#{RESOURCE_URI}?cursor=&count=100>; rel="first",
      <http://api.example.com/#{RESOURCE_URI}?cursor=90&count=100>; rel="prev",
      <http://api.example.com/#{RESOURCE_URI}?cursor=120&count=100>; rel="next",
      <http://api.example.com/#{RESOURCE_URI}?cursor=200&count=100>; rel="last"

[
  ...
]
```

## 数据缓存

大部分接口都会在响应头中携带 `Last-Modified` 和 `ETag` 信息，你可以在随后请求这些资源的时候，在请求头中使用 `If-Modified-Since` 或者 `If-None-Match` 两个头来确认资源是否经过修改。

```bash
$ curl -i http://api.example.com/#{RESOURCE_URI}
HTTP/1.1 200 OK
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 200 OK

$ curl -i http://api.example.com/#{RESOURCE_URI} -H "If-Modified-Since: Thu, 05 Jul 2012 15:31:30 GMT"
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 304 Not Modified

$ curl -i http://api.example.com/#{RESOURCE_URI} -H 'If-None-Match: "644b5b0155e6404a9cc4bd9d8b1ae730"'
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 304 Not Modified
```

## User-Agent

请求头中的 `User-Agent` 头标是**必须**的，如果没有，则服务器会响应 `400` 状态码。

建议格式：

* iOS

        iOS/iOS版本号 (设备型号; 是否越狱<unjailbroken, jailbroken>; 网络类型<Wi-Fi, Cellular>; 语言) CFBundleIdentifier/CFBundleVersion

* Android

        Android/Android版本号 (设备型号; ROM版本号; 是否root<unrooted, rooted>; 网络类型; 语言) PackageName/版本号

* Web 应用的 User-Agent 由浏览器设定

示例：

    User-Agent: iOS/6.1.2 (iPhone 5; jailbroken; Wi-Fi; zh-CN) com.bundle.id/3.2

    User-Agent: Android/4.2 (MI-ONE Plus; MIUI-2.3.6f; unrooted; GPRS; zh-TW) com.bundle.id/2.1

Android 的网络类型获取可以参考文档：[http://developer.android.com/reference/android/telephony/TelephonyManager.html](http://developer.android.com/reference/android/telephony/TelephonyManager.html)

## 跨域

### CORS

接口支持[“跨域资源共享”（Cross Origin Resource Sharing, CORS）](http://www.w3.org/TR/cors)，[这里](http://enable-cors.org/)和[这里](http://code.google.com/p/html5security/wiki/CrossOriginRequestSecurity)和[这份中文资料](http://newhtml.net/using-cors/)有一些指导性的资料。

简单示例：

```bash
$ curl -i https://api.example.com -H "Origin: http://example.com"
HTTP/1.1 302 Found
```

```bash
$ curl -i https://api.example.com -H "Origin: http://example.com"
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Access-Control-Expose-Headers: ETag, Link, X-Resource-Count
Access-Control-Allow-Credentials: true
```

预检请求的响应示例：

```bash
$ curl -i https://api.example.com -H "Origin: http://example.com" -X OPTIONS
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-Resource-Count
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true
```

### JSON-P

如果在任何 `GET` 请求中带有参数 `callback` ，且值为非空字符串，那么接口将返回如下格式的数据

```bash
$ curl http://api.example.com/#{RESOURCE_URI}?callback=foo
```

```javascript
foo({
  "meta": {
    "status": 200,
    "X-Resource-Count": 542,
    "Link": [
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=0&count=100", "rel": "first"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=90&count=100", "rel": "prev"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=120&count=100", "rel": "next"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=200&count=100", "rel": "last"}
    ]
  },
  "data"; {
    // data
  }
})
```

