# CSRF

跨站请求伪造（Cross-site Request forgery)，缩写 CSRF 或 XSRF ，也被称之为 one-click attack 或 session riding。
攻击者伪造用户身份向 Server 发起请求。

## 攻击原理

1. Site A 存在 XSRF 漏洞，Site B 为攻击者站点，User C 为 Site A 合法用户。
2. User C 正常登录 Site A，Site A 在 browser 存下 cookie，
3. User C 在未退出登录 Site A 的情况下，无意访问 Site B。
4. Site B 存在恶意的请求 Site A 的请求。例如 `<img src="https://example.com/withdraw?for=Alice&amount=10000000000" />`.
5. Site B 向 Site A 发送的请求会携带 Cookie。

CSRF 攻击重点：

1. 用户访问可信任 Site A，并产生了相关的 cookie;
2. 用户在访问 Site A 时没有退出，同时访问了危险站点 Site B。

## 防御手段

大多数用户对推出登录并无概念，认为只要关闭了浏览器就是推出登录，因此 CSRF 的攻击成功率很高且用户无感。

主要的防御手段：

1. 验证码
2. 检查 http header `Referer`. 判断请求来自于合法站点。
3. 增加校验 Token
4. Same-Site Cookies 和 Cookies Secure
5. JWT

### 验证码

验证码是对抗 CSRF 最简洁有效的方式。
出于用户体验，不能所有的操作都加上验证码校验

### Referer 检查

可以通过 `Referer` 检查请求来源属于白名单页面，从而判断用户是否瘦到 CSRF 的攻击。

**但是，并非任何时候 Server 都能获得 Referer。**

例如：

1. 浏览器直接输入 URL
2. HTTPS 转 HTTP

### Anti CSRF Token

CSRF 攻击成功的本质原因是**重要操作的所有参数都被攻击者猜到**。

对于重要接口加上一个足够安全的随机数作为 Token

```
https://domain.com/api/doSomething?param1=123&token=random(seed)
```

或是，在 Form 表单中加入一个隐藏的 `<input/>` 作为随机数

```html
<form>
  <input type="hidden" name="token" value="random(seed)" />
</form>
```

CSRF Token 注意事项：

Token 最好存在 session 中，并且每次提交后都需要产生新的 Token。

如果 Token 存在 Cookie 中，如果一个用户同时打开几个相同的页面，当某个页面消耗掉 Token 时。
其它页面的表单也会失效。

Token 如果放在 URL 中，则容易因为 Referer 泄漏。

由于存在代理服务器篡改 HTTP headers 的可能，该方案也不能独立防御攻击。

如果 Session 的实现方案基于 cookies 。
server 在 session 存储 User Info 可能性很高，拿到 cookies 中 sessionId 也就能拿到用户身份信息。

### Cookies Same-Site 和 Secure 和 HttpOnly

```
Set-Cookie: value=abc123; path=/; SameSite=Strict; Secure=true; HttpOnly=true;
```

设置 Cookies 仅在 https 和同站点可访问（`SameSite=Strict` ）是当前最有效的防范 CSRF 攻击的做法。

加上 HttpOnly 防御 xss 之后的 cookie 劫持，使得防御手段更加有效。

但是 same-site 的兼容性

### JWT

在 JWT 中仅存储必要的用户信息。
然后 Server 基于 JWT 做用户验证，也能有效的防御 CSRF。

但是，JWT 一般都有有效时间，最好避免在 URL 上携带，存储在 cookie 中，从而导致 Token 泄漏。
在 Token 泄露的这一段时间里该用户都存在的极大的风险。
