# XSS

跨站脚本攻击（Cross-Site Script），为了与层叠样式表 CSS 更好的区分，通常称之为 XSS。
XSS 指的是攻击者恶意向 Web 页面中插入恶意攻击脚本。

## XSS 类型

XSS 可以分为三类：

1. 存储型
2. 反射型
3. DOM 型

### 存储型 XSS

注入的脚本永久存储在目标服务器上。
然后，当浏览器发送数据请求时，受害者从服务器检索该恶意脚本。

现有一论坛网站，攻击者在评论回复 `<script>alert('you has ben hack');</script>`.
如果没对内容里的 html 进行转义，那么其他用户访问时，该脚本会被执行。

### 反射型 XSS

当用户被诱骗点击一个恶意链接，或者提交一个表单，或者浏览恶意网站时，注入脚本进入易受攻击的网站。
WEB 服务器会将注入的脚本反映回用户浏览器，例如错误信息，搜索结果或其它响应，其中包括作为请求的一部分发送到服务器的数据。
浏览器执行代码是因为它假定响应来自用户已经与之交互的“受信任”服务器。

攻击 URL 示例：
`https://baidu.com?q=<script>alert('you has ben hack');</script>`

### DOM 型 XSS

本质上 DOM 型 XSS 是反射型中的特殊一种。
只不过 DOM 型没有对文档内容进行修改，只对 DOM 环境进行了修改。

```html
<!-- ORIGIN SITE DOCUMENT -->
<html>
  <title>Welcome!</title>
  Hi
  <script>
    var pos = document.URL.indexOf("name=") + 5;
    document.write(document.URL.substring(pos, document.URL.length));
  </script>
  <br />
  Welcome to our system …
</html>
```

用于攻击的 URL:
`http://www.vulnerable.site/welcome.html?name=<script>alert(document.cookie)</script>`

DOM 型的特点

1. 恶意程序脚本在任何时候不会嵌入到处于自然状态下的 HTML 页面.
2. 这个攻击只有在浏览器没有修改 URL 字符时起作用。

## XSS 防御

综上可知，在以下 2 种情况下，容易发生 XSS 攻击：

1. 数据从一个不可靠的链接进入到一个 Web 应用程序。
2. 没有过滤掉恶意代码的动态内容被发送给 Web 用户。

针对以上两点就能避免 XSS 攻击。
