#RFC 6750不记名令牌

> 编号：OAPI-DESIGN-003
> 作者：刘海龙
> 微博：[http://weibo.com/liuhailong2008]
> 博客：[http://blog.csdn.net/stationxp]

##含义

oAuth使客户通过令牌而不是资源所有者的凭证来访问受保护资源，在RFC 6749中，访问令牌（access token）被定义为：颁发给客户端代表访问权限的字符串。

不记名令牌（bearer），顾名思义，除令牌本身外，不再验证其他权限。凭借访问令牌即可实现对受控资源的访问，因此，令牌的传输安全就极其重要。
令牌本身隐含的语义包括：范围、实效，也可以包含其他授权属性。在RFC 6750中强调了不记名，和身份相关的属性应该摒弃。

>bearer ：单词本意是递送人，持有人。
>典型应用：网盘共享文件，设置提取码，谁知道提取码都可以提取文件。

RFC 6750定义了采用传输层安全（TLS，RFC 5246）在HTTP上使用不记名令牌访问受保护资源。要求强制令牌传输过程中，强制使用 TLS。
不记名身份验证方案主要采用 使用WWW-Authenticate和Authorization的HTTP头部作为服务器身份验证。

>RFC 2617

##算法

```
b64token = 1 * (ALPHA / DIGIT/"-"/"."/"_"/"~"/"+"/"/") *"="
credentials = "Bearer" 1*SP b64token
```

如：`Bearer mF-9.B5f-4.1Jqm`

##发送令牌的方式

###授权请求头部字段

例如：
```
GET /resource HTTP/1.1
Host:resource.server.com
Authorization: Bearer mF-9.B5f-4.1Jqm
```

服务器必须支持此方法。

###表单编码的主体参数

例如：
```
POST /resource HTTP/1.1
Host:resource.server.com
Content-Type:application/x-www-form-urlencoded
access_token=mF-9.B5f-4.1Jqm
```

除非客户端不能使用第一种方法，才可以用这种方法，而且，必须严格按照要求，符合一定规范（比如：只能用POST方法，主体中只能使用ASCII）。
服务器可以不支持这种方法。

###URL查询参数

简单到懒得解释，如下：

```
http://resource.server.com/resource?access_token=mF-9.B5f-4.1Jqm
```

客户端发送请求时，应发送含有“no-store”选项的Cache-Control头。成功状态下，服务器响应应带有“private”选项的Cache-Control头。

这种方法不应使用。就当没有这种方法吧。


##错误信息的返回

没有权限访问资源时候，HTTP格式的错误返回消息中必须包含`HTTP "WWW-Authenticate"`响应头部字段。

>RFC 2617

>realm：领域。

scope不需要暴露给客户端。

例如：
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate:Bearer realm="admin"
```

其他错误消息的返回方法，参照RFC 6749 5.2。
例如：
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate:Bearer realm="user"
error="invalid_token",
error_description="The access token expired"
```

如果客户端在盲试，什么错误信息都不要给他 。

##不记名令牌响应的例子

```
HTTP/1.1 200
Content-Type:application/json;charset=UTF-8
Cache-Control:no-store
Pragma:no-cache
{
    "access_token":"blabla",
    "token_type":"Bearer",
    "expires_in":3600,
    "refresh_token":"blabla2"
}
```

##安全考量

###可能的威胁

- 伪造令牌／修改令牌
- 泄漏敏感信息
- 令牌重定向
- 令牌重放

###我们的盾牌

数字签名（防止令牌被修改）
消息认证码（MAC）

> TLS ,RFC 5246, RFC 2246

客户端必须验证TLS证书链。

>证书吊销列表（CRL，RFC5280）

Cookie是明文的，靠不住。只有二货才会把令牌保存在Cookie中。

>关于Cookie的安全考量，RFC 6265。

集群环境中，TLS不能只能覆盖早前段代理服务器，存在一段明文暴露的过程。

令牌有效期1小时，是个合理期限。

为了不把令牌发错人，客户端必须按照RFC 2818 3.1验证资源服务器身份。

###安全建议汇总

- 客户端验证证书链：防DNS劫持。
- 总是使用TLS。
- 不存在Cookie中，不在url中传。

## IANA考量

错误代码和说明的规范清单。

>RFC 6750。



