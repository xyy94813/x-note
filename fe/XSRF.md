# XSRF

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

1. 不在 cookies 里存储身份验证必须的信息。
2. 检查 http header `Referer`. 判断请求来自于合法站点。
3. 增加校验 Token
4. Same-Site Cookies 和 Cookies Secure

```
Set-Cookie: sess=abc123; path=/; SameSite; Secure=true;
```

设置 Cookies 仅在 htttps 和同站点可访问是当前最有效的防范 CSRF 攻击的做法。

增加校验 Token 是常规做法，如果站点的 Token 存储在 Cookies 中，仍旧无法单靠该方案防御 XSRF 攻击。

由于存在代理服务器篡改 HTTP headers 的可能，该方案也不能独立防御攻击。

不在 cookies 里存储身份验证必须的信息不太可靠。
server 在 session 存储 User Info 可能性很高，拿到 cookies 中 sessionId 也就能拿到用户身份信息。
Session 的实现方案还得基于非 cookies。
