# HTTP 接口设计指北

* 文章主要目的是为设计接口骨架时提供建议
* 内容会经常更新
* **只是建议**

## 目录

* [URL](#url)
* [空字段](#空字段)
* [国际化](#国际化)
* [请求方法](#请求方法)
* [状态码](#状态码)
* [错误处理](#错误处理)
* [身份验证](#身份验证)
* [超文本驱动和资源发现](#超文本驱动和资源发现)
* [分页](#分页)
* [数据缓存](#数据缓存)
* [并发控制](#并发控制)
* [跨域](#跨域)
* [更细节的接口设计指南](#更细节的接口设计指南)

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

1. language：这部分是 [ISO 639](http://www.loc.gov/standards/iso639-2/php/code_list.php)([Wikipedia](http://zh.wikipedia.org/wiki/ISO_639-1)) 规定的代码，比如中文是 zh。
2. script：表示变体，比如简体汉字是 zh-Hans ，繁体汉字是 zh-Hant 。
3. region：是 [ISO 3166-1][iso3166-1]([Wikipedia][iso3166-1_wiki]) 规定的地理区域，比如 zh-Hans-CN 就是中国大陆使用的简体中文。
4. variant：表示方言。
5. extension：表示扩展。
6. privateus：表示私有标识。

**有一点需要注意，任何合法的标签都必须经过IANA的认证，已通过认证的标签可以在[这个网页](http://www.iana.org/assignments/language-subtag-registry)查到。此外，网上还有一个非官方的[标签搜索引擎](http://people.w3.org/rishida/utils/subtags/)。**

相关资料：

* Android 文档：[http://developer.android.com/guide/topics/resources/providing-resources.html#LocaleQualifier](http://developer.android.com/guide/topics/resources/providing-resources.html#LocaleQualifier) ，顺便鄙视 iOS 文档在获取语言接口的相关文档里根本不提这个。
* 《语种名称代码》：[http://www.ruanyifeng.com/blog/2008/02/codes_for_language_names.html](http://www.ruanyifeng.com/blog/2008/02/codes_for_language_names.html)
* 《Language tags in HTML and XML》：[http://www.w3.org/International/articles/language-tags/](http://www.w3.org/International/articles/language-tags/)

### 时区

客户端请求服务器时，如果对时间有特殊要求（如某段时间每天的统计信息），则可以参考 [IETF 相关草案](http://tools.ietf.org/html/draft-sharhalakis-httptz-05) 增加请求头 `Timezone: Asia/Shanghai` 。

时区的名称可以参考 [tz datebase](http://www.iana.org/time-zones)([Wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)) 。

如果客户端请求时没有指定相应的时区，则服务端默认使用 [UTC](http://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6) 时间返回相应数据。

PS 考虑到存在[夏时制](https://en.wikipedia.org/wiki/Daylight_saving_time)这种东西，所以不推荐客户端在请求时使用 Offset 。

### 时间格式

时间格式遵循 [ISO 8601](https://www.iso.org/obp/ui/#iso:std:iso:8601:ed-3:v1:en)([Wikipedia](https://en.wikipedia.org/wiki/ISO_8601)) 建议的格式：

* 日期 `2014-07-09`
* 时间 `14:31:22+0800`
* 具体时间 `2007-11-06T16:34:41Z`
* 持续时间 `P1Y3M5DT6H7M30S` （表示在一年三个月五天六小时七分三十秒内）
* 时间区间 `2007-03-01T13:00:00Z/2008-05-11T15:30:00Z` 、 `2007-03-01T13:00:00Z/P1Y2M10DT2H30M` 、 `P1Y2M10DT2H30M/2008-05-11T15:30:00Z`
* 重复时间 `R3/2004-05-06T13:00:00+08/P0Y6M5DT3H0M0S` （表示从2004年5月6日北京时间下午1点起，在半年零5天3小时内，重复3次）

相关资料：

* [What's the difference between ISO 8601 and RFC 3339 Date Formats?](http://stackoverflow.com/questions/522251/whats-the-difference-between-iso-8601-and-rfc-3339-date-formats)
* [JSON风格指南 - Google 风格指南（中文版）](https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md#%E5%B1%9E%E6%80%A7%E5%80%BC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)

### 货币名称

货币名称可以参考 [ISO 4217](javascript:;)([Wikipedia](http://en.wikipedia.org/wiki/ISO_4217)) 中的约定，标准为货币名称规定了三个字母的货币代码，其中的前两个字母是 [ISO 3166-1][iso3166-1]([Wikipedia][iso3166-1_wiki]) 中定义的双字母国家代码，第三个字母通常是货币的首字母。在货币上使用这些代码消除了货币名称（比如 dollar ）或符号（比如 $ ）的歧义。

相关资料：

* 《RESTful Web Services Cookbook 中文版》 3.9 节《如何在表述中使用可移植的数据格式》

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
    * 如果是替换了资源，则返回 `200 OK`
    * 完成请求后需要返回被修改的资源详细信息
* `PATCH` 用于局部更新资源
    * 完成请求后返回状态码 `200 OK`
    * 完成请求后需要返回被修改的资源详细信息
* `DELETE` 用于删除某个资源
    * 完成请求后返回状态码 `204 No Content`

## 状态码

* [维基百科上的《 HTTP 状态码》词条](http://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)
* [RFC2616](http://tools.ietf.org/html/rfc2616) - HTTP 协议本体
* [RFC4918](http://tools.ietf.org/html/rfc4918) - 422 状态码的来源
* [RFC5789](http://tools.ietf.org/html/rfc5789) - PATCH 方法的定义
* [RFC6585](http://tools.ietf.org/html/rfc6585) - 新增的四个 HTTP 状态码，[中文版](http://www.oschina.net/news/28660/new-http-status-codes)
* [Do I need to use http redirect code 302 or 307? - Stack Overflow](http://stackoverflow.com/questions/2467664/do-i-need-to-use-http-redirect-code-302-or-307)
* [400 vs 422 response to POST of data](http://stackoverflow.com/questions/16133923/400-vs-422-response-to-post-of-data)

### 请求成功

* 200 **OK** : 请求执行成功并返回相应数据，如 `GET` 成功
* 201 **Created** : 对象创建成功并返回相应资源数据，如 `POST` 成功；创建完成后响应头中应该携带头标 `Location` ，指向新建资源的地址
* 204 **No Content** : 请求执行成功，不返回相应资源数据，如 `PATCH` ， `DELETE` 成功

### 重定向

**重定向的新地址都需要在响应头 `Location` 中返回**

* 301 **Moved Permanently** : 被请求的资源已永久移动到新位置
* 302 **Found** : 请求的资源现在临时从不同的 URI 响应请求
* 303 **See Other** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该使用 `GET` 方法进行请求
* 307 **Temporary Redirect** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该保持原有的请求方法进行请求

### 客户端出错

* 400 **Bad Request** : 请求体包含语法错误
* 401 **Unauthorized** : 需要验证用户身份，如果服务器就算是身份验证后也不允许客户访问资源，应该响应 `403 Forbidden`
* 403 **Forbidden** : 服务器拒绝执行
* 404 **Not Found** : 找不到目标资源
* 405 **Method Not Allowed** : 不允许执行目标方法，响应中应该带有 `Allow` 头，内容为对该资源有效的 HTTP 方法
* 406 **Not Acceptable** : 服务器不支持客户端请求的内容格式（比如客户端请求 JSON 格式的数据，但服务器只能提供 XML 格式的数据）
* 409 **Conflict** : 被请求的资源的当前状态之间存在冲突
* 410 **Gone** : 被请求的资源已被删除
* 412 **Precondition Failed** : 服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。主要使用场景在于实现[并发控制](#并发控制)
* 413 **Request Entity Too Large** : `POST` 或者 `PUT` 请求的消息实体过大
* 415 **Unsupported Media Type** : 服务器不支持请求中提交的数据的格式
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

部分接口需要通过某种身份验证方式才能请求成功（这些接口**应该**在文档中标注出来），身份验证支持两种方式：

* 支持 [HTTP 基本认证](http://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)
* 支持通过登录接口使用账号密码获取 JSON Web Token ，在请求接口时使用 `Authorization: Bearer #{token}` 头标或者 `token` 参数的值的方式进行验证。
    * [JSON Web Token 规范](https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25)
    * [Json Web Tokens: Introduction](http://angular-tips.com/blog/2014/05/json-web-tokens-introduction/)
    * [Json Web Tokens: Examples](http://angular-tips.com/blog/2014/05/json-web-tokens-examples/)
    * [Cookies vs Tokens. Getting auth right with Angular.JS](https://auth0.com/blog/2014/01/07/angularjs-authentication-with-cookies-vs-token/)

## 超文本驱动和资源发现

REST 服务的要求之一，客户端不再需要将某些接口的 URI 硬编码在代码中，唯一需要存储的只是 API 的 HOST 地址，能够非常有效的降低客户端与服务端之间的耦合，服务端对 URI 的任何改动都不会影响到客户端的稳定。

目前只有几种方案差强人意：

* [JSON HAL 草案](http://tools.ietf.org/html/draft-kelly-json-hal-06) ，示例可以参考 [JSON HAL 作者自己的介绍](http://stateless.co/hal_specification.html)
* [GitHub API 使用的方案](https://developer.github.com/v3/#hypermedia) ，应该是一种 JSON HAL 的变体
* [JSON API 方案](http://jsonapi.org/) ，另外一种类似 JSON HAL 的方案，不过某些方面（比如甚至也考虑到了 URL ）考虑的比 JSON HAL 更为具体

目前来看应该是合并 JSON API 和 JSON HAL 两个方案的做法，各取所长，能够得到一个相对理想的方案

## 分页

请求某个资源集合时，可以通过指定 `count` 参数来指定每页的资源数量，通过 `page` 参数指定页码，或根据 `last_cursor` 参数指定上一页最后一个资源的标识符。

如果没有传递 `count` 参数或者 `count` 参数的值为空，则使用默认值 20 ， `count` 参数的最大上限为 100 。

如何同时传递了 `last_cursor` 和 `page` 参数，则使用 `page` 。

分页的相关信息会包含在 [Link Header](http://tools.ietf.org/html/rfc5988) 和 `X-Total-Count` 中。

如果是第一页或者是最后一页时，不会返回 `previous` 和 `next` 的 Link 。

```http
HTTP/1.1 200 OK
X-Total-Count: 542
Link: <http://api.example.com/#{RESOURCE_URI}?last_cursor=&count=100>; rel="first",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=200&count=100>; rel="last"
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=90&count=100>; rel="previous",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=120&count=100>; rel="next",

[
  ...
]
```

相关资料：

* [RFC5005 第3节 《Paged Feeds》](http://tools.ietf.org/html/rfc5005#section-3)
* [RFC5988 6.2.2节 《Initial Registry Contents》](http://tools.ietf.org/html/rfc5988#section-6.2.2)

## 数据缓存

大部分接口应该在响应头中携带 `Last-Modified` 和 `ETag` 信息，客户端可以在随后请求这些资源的时候，在请求头中使用 `If-Modified-Since` 或者 `If-None-Match` 两个头来确认资源是否经过修改。

```bash
$ curl -i http://api.example.com/#{RESOURCE_URI}
HTTP/1.1 200 OK
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT

$ curl -i http://api.example.com/#{RESOURCE_URI} -H "If-Modified-Since: Thu, 05 Jul 2012 15:31:30 GMT"
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT

$ curl -i http://api.example.com/#{RESOURCE_URI} -H 'If-None-Match: "644b5b0155e6404a9cc4bd9d8b1ae730"'
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
```

## 并发控制

不严谨的实现，或者缺少并发控制的 `PUT` 和 `PATCH` 请求可能导致 “更新丢失”。这个时候可以使用 `Last-Modified` 和/或 `ETag` 头来实现条件请求，支持乐观并发控制。

下文只考虑使用 `PUT` 和 `PATCH` 方法更新资源的情况。

* 客户端发起的请求如果没有包含 `If-Unmodified-Since` 或者 `If-Match` 头，那就返回状态码 `403 Forbidden` ，在响应正文中解释为何返回该状态码
* 客户端发起的请求提供的 `If-Unmodified-Since` 或者 `If-Match` 头与服务器记录的实际修改时间和 `ETag` 值不匹配的时候，返回状态码 `412 Precondition Failed`
* 客户端发起的请求提供的条件符合实际值，那就更新资源，响应 `200 OK` 或者 `204 No Content` ，并且包含更新过的 `Last-Modified` 和/或 `ETag` 头，同时包含 `Content-Location` 头，其值为更新后的资源 URI

相关资料：

* 《RESTful Web Services Cookbook 中文版》 10.4 节 《如何在服务器端实现条件 PUT 请求》

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
Access-Control-Expose-Headers: ETag, Link, X-Total-Count
Access-Control-Allow-Credentials: true
```

预检请求的响应示例：

```bash
$ curl -i https://api.example.com -H "Origin: http://example.com" -X OPTIONS
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-Total-Count
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
    "X-Total-Count": 542,
    "Link": [
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=0&count=100", "rel": "first"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=90&count=100", "rel": "prev"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=120&count=100", "rel": "next"},
      {"href": "http://api.example.com/#{RESOURCE_URI}?cursor=200&count=100", "rel": "last"}
    ]
  },
  "data": // data
})
```

## 更细节的接口设计指南

这里还有一些其他参考资料：

* 推荐参考文档 [HTTP API Design Guide](https://github.com/interagent/http-api-design/) 来设计 REST 风格的 API ，我基本同意这个文档上的所有建议，除了以下两点：
    * [Use consistent path formats](https://github.com/interagent/http-api-design/#use-consistent-path-formats)
        还是不建议将动作写在 URL 中，像文档中的情况，可以将这个行为抽象成一个事务资源 `POST /runs/:run_id/stop-logs` 或者 `POST /runs/:run_id/stoppers` 来解决
    * [Paginate with Ranges](https://github.com/interagent/http-api-design/#paginate-with-ranges)
        确实是一个巧妙的设计，但似乎并不符合 `Content-Range` 的设计意图，而且有可能和需要使用到 `Content-Range` 的正常场景冲突（虽然几乎不可能），所以不推荐
* [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
* [Thoughts on RESTful API Design](http://restful-api-design.readthedocs.org/en/latest/)
* [The RESTful CookBook](http://restcookbook.com/)

[iso3166-1]: javascript:;
[iso3166-1_wiki]: http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
