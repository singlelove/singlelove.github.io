---
title: '''oauth相关'''
date: 2019-08-16 10:39:51
tags:
---

- OpenId：只提供认证
- OAuth：认证+授权
- SSO：是多系统之间登录的一种解决方案，CAS和OAuth都可以是SSO的实现方式，但OAuth初衷是用于第三方认证和授权，并非用于SSO

# OAuth
## 授权码模式(authorization code)
1) 用户访问客户端，客户端重定向到认证服务器，携带redirection_uri
2) 用户选择是否给客户端授权
3) 授权后，认证服务器重定向到客户端指定的redirection_uri，附上code授权码
4) 客户端收到code，附上redirection_uri，由客户端的后台服务器向认证服务器申请令牌(token)，对用户(浏览器端)不可见，避免token泄露
5) 认证服务器核对授权码和redirection_uri，确认无误后向客户端发送访问令牌（access token）和更新令牌（refresh token）
6) 客户端携带access_token向资源服务器请求用户信息资源
7) 资源服务器验证access_token合法后返回用户信息给客户端

1)步骤携带参数
response_type：表示授权类型，必选项，此处的值固定为"code"
client_id：表示客户端的ID，必选项
redirection_uri：表示重定向URI，可选项
scope：表示申请的权限范围，可选项
state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。
> state参数可以避免CSRF攻击，防止攻击者骗受害者绑定攻击者的账号，参考[关于 OAuth2.0 安全性你应该要知道的一些事](https://www.chrisyue.com/security-issue-about-oauth-2-0-you-should-know.html)

3)步骤携带参数
code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。
> 为什么使用code比access_token安全，因为code有效期短，只能使用一次，一旦多次使用，就会令原本换到的access_token失效

4)步骤携带参数
grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
code：表示上一步获得的授权码，必选项。
redirection_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致(**避免redirection_uri篡改攻击，同时应该要求第三方服务注册时，提供redirection_uri**)
client_id：表示客户端ID，必选项。
> 建议增加secret字段，否则一旦第三方网站被DNS污染，导致攻击者拿到了code，就可以直接通过code换取access_token了，
因为client_id是公开的(在步骤1中就可以看到)，参考[关于 OAuth2.0 安全性你应该要知道的一些事](https://www.chrisyue.com/security-issue-about-oauth-2-0-you-should-know.html)
一文code与secret部分章节。

5)步骤携带参数
access_token：表示访问令牌，必选项。
token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

流程参考[阮一峰的理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
安全性问题参考[The OAuth 2.0 Authorization Framework 入门](http://blog.abandonzhang.me/2017/11/12/OAuth-2-0-Authorization-Framework/#%E5%AE%89%E5%85%A8%E6%80%A7%E8%80%83%E8%99%91%EF%BC%88Security-Considerations%EF%BC%89)