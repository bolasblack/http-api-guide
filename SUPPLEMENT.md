# 补充内容

这里是一些补充性质的内容，在需要实现某些功能时可以做一个参考，不一定有相应的标准可以遵循。

## 目录

* [扩充巴科斯范式](#user-content-扩充巴科斯范式-abnf)
* [User-Agent](#user-content-user-agent)
* [WWW-Authenticate 头](#user-content-www-authenticate-头)
* [两步验证](#user-content-两步验证)
* [同时操作多个资源](#user-content-同时操作多个资源)
* [超文本驱动](#user-content-超文本驱动)

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

想法受启发于 [JSON API 方案](http://jsonapi.org/)，很多做法基本照搬，主要是把 `links` 相关内容放到了请求头里。

想法目前还不成熟，并不建议投入使用。

```http
HTTP/1.1 200 OK
Link: <http://api.example.com/peoples/{posts.author}>; rel="res:author"; allow="collection,get",
      <http://api.example.com/comments/{posts.comments}>; rel="res:comments"; allow="collection,create,get,delete",
      <http://api.example.com/todos/order>; rel="res:order"; allow="get,put"

[{
  "id": "1",
  "title": "Rails is Omakase",
  "author": "9",
  "comments": [ "5", "12", "17", "20" ]
}]
```
