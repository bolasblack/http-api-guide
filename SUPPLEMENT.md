# 补充内容

这里是一些补充性质的内容，在需要实现某些功能时可以做一个参考，不一定有相应的标准可以遵循。

## 目录

* [User-Agent](#user-agent)
* [两步验证](#两步验证)

## User-Agent

请求头中的 `User-Agent` 可以帮助服务端收集设备信息，建议格式：

* iOS

        iOS/iOS版本号 (设备型号; 是否越狱<unjailbroken, jailbroken>; 网络类型<Wi-Fi, Cellular, Unknown>; 语言) CFBundleIdentifier/CFBundleVersion

* Android

        Android/Android版本号 (设备型号; ROM版本号; 是否root<unrooted, rooted>; 网络类型; 语言) PackageName/PackageVersion

* Web 应用的 User-Agent 由浏览器设定

示例：

    User-Agent: iOS/6.1.2 (iPhone 5; jailbroken; Wi-Fi; zh-CN) com.bundle.id/3.2

    User-Agent: Android/4.2 (MI-ONE Plus; MIUI-2.3.6f; unrooted; GPRS; zh-TW) com.bundle.id/2.1

Android 的网络类型获取可以参考文档：[http://developer.android.com/reference/android/telephony/TelephonyManager.html](http://developer.android.com/reference/android/telephony/TelephonyManager.html)

## 两步验证

如果只是打算简单实现，建议使用 [TOTP](http://tools.ietf.org/html/rfc6238)([Wikipedia](http://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)) 协议，可以兼容 [Google Authenticator](https://code.google.com/p/google-authenticator/) 。

相关资料：

* 这里是一份 TOTP 协议中密码生成算法的简单说明：[http://jacob.jkrall.net/totp/](http://jacob.jkrall.net/totp/)
* [What are the advantages of TOTP over HOTP?](http://crypto.stackexchange.com/questions/2220/what-are-the-advantages-of-totp-over-hotp)
