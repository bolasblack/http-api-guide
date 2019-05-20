# 补充内容

这里是一些补充性质的内容，在需要实现某些功能时可以做一个参考，不一定有相应的标准可以遵循。

## 目录

* [扩充巴科斯范式](#user-content-扩充巴科斯范式-abnf)
* [User-Agent](#user-content-user-agent)
* [WWW-Authenticate 头](#user-content-www-authenticate-头)
* [两步验证](#user-content-两步验证)
* [同时操作多个资源](#user-content-同时操作多个资源)
* [超文本驱动](#user-content-超文本驱动)
* [错误处理](#user-content-错误处理)
* [分页](#user-content-分页)

## 扩充巴科斯范式 (ABNF)

这算是阅读规范的预备知识吧，但写在 README 里还是太占空间了，所以写在了这里。

规范里类似：

```abnf
header-field   = field-name ":" OWS field-value OWS

field-name     = token
field-value    = *( field-content / obs-fold )
field-content  = field-vchar [ 1*( SP / HTAB ) field-vchar ]
field-vchar    = VCHAR / obs-text

obs-fold       = CRLF 1*( SP / HTAB )
               ; obsolete line folding
               ; see Section 3.2.4
```

格式的内容叫做“扩充巴科斯范式”，是由 [RFC 5234](http://tools.ietf.org/html/rfc5234) ([Wikipedia](http://zh.wikipedia.org/wiki/%E6%89%A9%E5%85%85%E5%B7%B4%E7%A7%91%E6%96%AF%E8%8C%83%E5%BC%8F)) 定义用以描述一些内容的详细格式的定义语言。

## User-Agent

请求头中的 `User-Agent` 可以帮助服务端收集设备信息，但格式需要遵循 [RFC 7231](http://tools.ietf.org/html/rfc7231#section-5.5.3) 中的定义，下文是一些建议格式：

* iOS

        iOS/iOS版本号 (设备型号; 是否越狱<unjailbroken, jailbroken>; 网络类型<Wi-Fi, Cellular, Unknown>; 语言) CFBundleIdentifier/CFBundleVersion

* Android

        Android/Android版本号 (设备型号; ROM版本号; 是否root<unrooted, rooted>; 网络类型; 语言) PackageName/PackageVersion

* Web 应用的 User-Agent 由浏览器设定

示例：

    User-Agent: iOS/6.1.2 (iPhone 5; jailbroken; Wi-Fi; zh-CN) com.bundle.id/3.2

    User-Agent: Android/4.2 (MI-ONE Plus; MIUI-2.3.6f; unrooted; GPRS; zh-TW) com.bundle.id/2.1

Android 的网络类型获取可以参考文档：[http://developer.android.com/reference/android/telephony/TelephonyManager.html](http://developer.android.com/reference/android/telephony/TelephonyManager.html)

## WWW-Authenticate 头

如果是自定义的身份验证方式，比如要求请求时带上请求头 `Authentication: Token <token>`，那么一般在 token 验证失败返回 `401` 的 `WWW-Authenticate` 头可以是 `WWW-Authenticate: Token` ，当然也可以带上任意其他自定义信息。客户端在发现自己无法识别的信息时应该略过。

## 两步验证

如果只是打算简单实现，建议使用 [TOTP](http://tools.ietf.org/html/rfc6238)([Wikipedia](http://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)) 协议，可以兼容 [Google Authenticator](https://code.google.com/p/google-authenticator/) 。

关于如何在 API 中实现对两步验证的支持，可以参考 [GitHub 的文档](https://developer.github.com/v3/auth/#working-with-two-factor-authentication)。

相关资料：

* 这里是一份 TOTP 协议中密码生成算法的简单说明：[http://jacob.jkrall.net/totp/](http://jacob.jkrall.net/totp/)
* [What are the advantages of TOTP over HOTP?](http://crypto.stackexchange.com/questions/2220/what-are-the-advantages-of-totp-over-hotp)

## 同时操作多个资源

### 创建多个相同的资源

请求：

```http
POST /resources HTTP/1.1

[{
  "id": "1 也允许由客户端直接指定 ID ，比如 UUID",
  "name": "resource1",
  "property": "a"
}, {
  "name": "resource2",
  "property": "b"
}, {
  "name": "resource3",
  "property": "c"
}]
```

响应：

```http
HTTP/1.1 201 Created
Location: /resources/1,2,3

[{
  "id": "1",
  "name": "resource1",
  "property": "a"
}, {
  "id": "2",
  "name": "resource2",
  "property": "b"
}, {
  "id": "3",
  "name": "resource3",
  "property": "c"
}]
```

### 删除多个相同的资源

```http
// 请求
DELETE /resources/1,2,3 HTTP/1.1

// 响应
HTTP/1.1 204 No Content
```

### 修改多个相同的资源

请求：

```http
PATCH /resources/1,2,3 HTTP/1.1

Content-Type: application/json
{
  "property": "d"
}

// 也可以使用 JSON Patch
Content-Type: application/json-patch+json
{
  "op": "replace",
  "replace": "/property",
  "value": "d"
}
```

请求实体可以直接写一个 JSON 进行修改，也可以发送一个 [JSON Patch](http://tools.ietf.org/html/rfc6902) 进行修改。

响应：

```http
HTTP/1.1 204 No Content
```

## 超文本驱动

想法受启发于 [JSON API 方案](http://jsonapi.org/)，做法基本照搬，主要是把 `links` 相关内容放到了响应头里。

可以添加 `schema` 参数链接到目标数据的结构描述文档，比如 [JSON Schema](http://json-schema.org/) 、 [Schema.org](http://schema.org/) 等。

想法目前还不成熟，不建议投入使用。

```http
HTTP/1.1 200 OK
Link: <http://api.example.com/peoples/{posts.author}>; rel="url-template:author"; allow="COLLECTION,GET"; schema="...",
      <http://api.example.com/comments/{posts.comments}>; rel="url-template:comments"; allow="COLLECTION,CREATE,GET,DELETE"; schema="...",
      <http://api.example.com/todos/order>; rel="url-template:order"; allow="GET,PUT"; schema="..."

[{
  "id": "1",
  "title": "Rails is Omakase",
  "author": "9",
  "comments": [ "5", "12", "17", "20" ]
}]
```

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
          "code": "required"
        }
      ]
    }
    ```

所有的 `error` 哈希表都有 `resource`, `field`, `code` 字段，以便于定位错误，`code` 字段则用于表示错误类型：

* `invalid`: 某个字段的值非法，接口文档中会提供相应的信息
* `required`: 缺失某个必须的字段
* `not_exist`: 说明某个字段的值代表的资源不存在
* `already_exist`: 发送的资源中的某个字段的值和服务器中已有的某个资源冲突，常见于某些值全局唯一的字段，比如 @ 用的用户名（这个错误我有纠结，因为其实有 409 状态码可以表示，但是在修改某个资源时，很一般显然请求中不止是一种错误，如果是 409 的话，多种错误的场景就不合适了）

其他参考：

* [Microsoft REST API Guidelines - 7.10.2. Error condition responses](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#7102-error-condition-responses)
* [GitHub Developer - Client errors](https://developer.github.com/v3/#client-errors)

## 分页

请求某个资源集合时，可以通过指定 `count` 参数来指定每页的资源数量，通过 `page` 参数指定页码，或根据需求使用 `last_cursor` 参数指定上一页最后一个资源的标识符替代 `page` 参数。

如果没有传递 `count` 参数或者 `count` 参数的值为空，则使用默认值，建议在设计时设置一个最大值。

分页的相关信息可以包含在 [Link Header](http://tools.ietf.org/html/rfc5988) 和 `X-Pagination-Info` 中（ HTTP 头的语法格式可以参考 [ABNF List Extension: #rule](https://tools.ietf.org/html/rfc7230#section-7) ）。

如果是第一页或者是最后一页时，不返回 `previous` 和 `next` 的 Link 。

```http
HTTP/1.1 200 OK
X-Pagination-Info: count="542"
Link: <http://api.example.com/#{RESOURCE_URI}?last_cursor=&count=100>; rel="first",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=200&count=100>; rel="last",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=90&count=100>; rel="previous",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor=120&count=100>; rel="next",
      <http://api.example.com/#{RESOURCE_URI}?last_cursor={last_cursor}&count={count}>; rel="url-template:pagination"

[
  ...
]
```

相关资料：

* [RFC 5005 第3节 _Paged Feeds_](http://tools.ietf.org/html/rfc5005#section-3)
* [RFC 5988 6.2.2节 _Initial Registry Contents_](http://tools.ietf.org/html/rfc5988#section-6.2.2)

其他参考：

* [Microsoft REST API Guidelines - 9.8. Pagination](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#98-pagination)
