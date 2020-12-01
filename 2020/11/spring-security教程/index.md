# Spring Security教程


## 4 种授权模式：

### 1.授权码模式

将用户引导到授权服务器进行身份认证，授权服务器将发放的访问令牌传递给客户端。
授权码的客户端指接入 OAuth 的第三方应用。

## 会话管理

要使用会话管理功能，就应该同时配置 HttpSessionEventPublisher

```java
@Bean
public HttpSessionEventPublisher httpSessionEventPublisher(){
  return new HttpSessionEventPublisher();
}
```

并且要为自定义用户类覆写 hashCode 和 equals 两个方法

## CSRF 防御手段

在任何情况下，都应尽可能避免以 GET 方式提供涉及数据修改的 API

防御 CSRF 工具的方式主要有一下两种：

1. HTTP Referer：判断请求来源，拒绝其他站点的请求。可修改，不可靠。
2. CsrfToken 认证：添加一些并不存在于 cookie 的验证值，并在每个请求中验证。

## 使用 Spring Security 防御 CSRF 攻击

默认用一个 HttpSessionCsrfTokenRepository 将 CsrfToken 存储在 HttpSession 中，指定前端把 CsrfToken 放在名为`_csrf`的请求参数或名为`X-CSRF-TOKEN`的请求头字段里。校验时通过判断是否一致。

在单页应用中用 CookieCsrfTokenRe。它将 CsrfToken 值存储在用户的 cookie 内（要设置 httponly 为 false）。前端用 Js 读取，放在请求参数或请求头中。

现已默认使用 LazyCsrfTokenRepository 来包裹 HttpSessionCsrfTokenRepository

