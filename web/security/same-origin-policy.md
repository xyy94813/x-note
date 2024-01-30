# 同源策略（Same Origin Policy，SOP）

同源策略是指在 Web 浏览器中，允许某个网页脚本访问另一个网页的数据，但前提是这两个网页必须有相同的 URI、主机名和端口号，一旦两个网站满足上述条件，这两个网站就被认定为具有相同来源。

## 为什么需要同源策略

同源策略可防止某个网页上的恶意脚本通过该页面的文档对象模型访问另一网页上的敏感数据。
同源策略对 Web 应用程序具有特殊意义，因为 Web 应用程序广泛依赖于 HTTP cookie 来维持用户会话，所以必须将不相关网站严格分隔，以防止丢失数据泄露。

**而且浏览器请求常常默认携带 cookie 返回给 Server。**

假设：
A 网站是一家银行，用户登录以后，又去浏览其他网站。
如果其他网站可以读取 A 网站的 Cookie，那么 Cookie 包含的隐私（比如存款总额）就会泄漏。

Cookie 往往用来保存用户的登录状态，如果用户没有退出登录（用户往往不清楚自己是否退出登录），其他网站就可以冒充用户，为所欲为。
因为浏览器同时还规定，提交表单不受同源政策的限制。

Same Origin Policy 可以有效的避免 [CSRF(XSRF)](/fe/XSRF.md)

## 源

如果两个 URL 的 protocol、port (如果有指定的话)和 host 都相同的话，则这两个 URL 是同源。

判断跟 `https://example.com/path1` 同源

| URL                             | Result                      |
| ------------------------------- | --------------------------- |
| `https://example.com/path2`     | same origin                 |
| `https://example.com/path1/a`   | same origin                 |
| `http://example.com/path1`      | fail by protocol difference |
| `https://example.com:444/path1` | fail by port difference     |
| `https://www.example.com/path1` | fail by host differnece     |

> 根据浏览器的实现 `https://example.com:443/a` 和 `https://example.com/a` **有可能**不同源

## Same Origin Policy 的限制

同源策略控制不同源之间的交互，例如在使用 XMLHttpRequest 或 `<img>` 标签时则会受到同源策略的约束。这些交互通常分为三类：

- 跨域写操作（Cross-origin writes）一般是被允许的。例如链接（links），重定向以及表单提交。特定少数的 HTTP 请求需要添加 preflight。
- 跨域资源嵌入（Cross-origin embedding）一般是被允许（后面会举例说明）。
- 跨域读操作（Cross-origin reads）一般是不被允许的，但常可以通过内嵌资源来巧妙的进行读取访问。
  例如，你可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法，或 availability of an embedded resource.

以下是可能嵌入跨源的资源的一些示例：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能被同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入 CSS。由于 CSS 的松散的语法规则，CSS 的跨域需要一个设置正确的 HTTP 头部 `Content-Type`。
  不同浏览器有不同的限制： IE, Firefox, Chrome, Safari (CVE-2010-0051) 和 Opera。
- 通过 `<img>` 展示的图片。支持的图片格式包括 PNG,JPEG,GIF,BMP,SVG,...
- 通过 `<video>` 和 `<audio>` 播放的多媒体资源。
- 通过 `<object>`、 `<embed>` 和 `<applet>` 嵌入的插件。
- 通过 `@font-face` 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
- 通过 `<iframe>` 载入的任何资源。站点可以使用 X-Frame-Options 消息头来阻止这种形式的跨域交互。

## 解决方案

解决 SOP 的常见思路

1. 跨域资源共享（CORS)
2. 反向代理
3. JSONP
4. iframe
5. postMessage

### CORS

CORS（Cross-origin resource sharing，跨域资源共享）CORS 是 HTTP 的一部分，它允许服务端来指定哪些主机可以从这个服务端加载资源。

- `Access-Control-Allow-Credentials`
- `Access-Control-Allow-Headers`
- `Access-Control-Allow-Methods`
- `Access-Control-Allow-Origin`
- `Access-Control-Expose-Headers`
- `Access-Control-Max-Age`
- `Access-Control-Request-Headers`
- `Access-Control-Request-Method`

`Access-Control-Allow-Origin` 响应头指定了该响应的资源是否被允许与给定的 origin 共享。

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Origin: <origin>
```

假设 `https://example.com/path1` 认为这个请求可以接受，就在 Access-Control-Allow-Origin 头部中回发相同的源信息，如：

```
Access-Control-Allow-Origin: https://www.example.com, http://example.com
Vary: Origin
```

此时 `https://www.example.com`, `http://example.com` 能够正常通过 XMLHttpRequest 或 fetch 请求 https://example.com。

如果服务器未使用 `*`，而是指定了一个域，那么为了向客户端表明服务器的返回会根据 Origin 请求头而有所不同，必须在 `Vary` 响应头中包含 Origin。

#### CORS 的缺点

服务器端需要维护一份 orign list 对于 DEV 不够友好。

### 反向代理

正向代理隐藏真实客户端，反向代理隐藏真实服务端。

通过搭建一个反向代理服务器，使得文档资源与真实服务之间处于同一域之下。

常见的 Nginx 和 Caddy 支持反向代理。Webpack Dev Server 也支持反响代理。

如下面一个 Nginx 代理服务器配置

```
#proxy 服务器
server {
    listen       81;
    server_name  www.example1.com;

    location / {
        proxy_pass   http://www.example2.com:8080;  #反向代理
        proxy_cookie_domain www.example2.com www.example1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用 webpack-dev-server 等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.example1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
```

#### 反向代理 的缺陷

代理服务器带来了额外的成本

- 维护代理服务器需要额外的成本
- 每一次的请求都需要经过代理服务器

必须为每一种应该用卡法一个反向代理服务器。

针对每一次代理，代理服务器就必须打开两个连接，对于高并发会成为瓶颈

### JSONP

JSONP 利用 `<script />` 未限制第三方请求和回调的方式实现跨域请求。

```html
<!-- https://example.com/index.html -->
<script>
  function showUserInfo(userInfo) {
    // do something
  }
</script>
<script src="https://www.example.com/api/userInfo?userId=1&callback=showUserInfo"></script>
```

```js
// in express server
res.send(`${req.query[callback]}(${JSON.stringfy({ name: 'alex', age: 18 })})`);
```

#### JSONP 的安全问题

##### JSON 劫持

**JSON（JSON Hijacking）**，最开始提出这个概念大概是在 2008 年国外有安全研究人员提到这个 JSONP 带来的风险。
这个问题属于 CSRF（ Cross-site request forgery 跨站请求伪造）攻击范畴。

```html
<script>
  function wooyun(v) {
    alert(v.username);
  }
</script>
<script src="http://js.login.360.cn/?o=sso&m=info&func=wooyun"></script>
```

当被攻击者在登陆 360 网站的情况下访问了该网页时，那么用户的隐私数据可能被攻击者劫持。

###### 防御手段

- 验证 JSON 调用的来源（ Referer ）
- token 防御

验证 JSON 调用的来源（ Referer ）。
这个方案是主要利用了 `<script>` 远程加载 JSON 文件时会发送 `Referer` ，在网站输出 JSON 数据时判断 `Referer` 是不是白名单就可以进行防御.

但需要注意 Referer 过滤不严谨以及空 Referer 的问题

```html
<script src="http://www.attack.com/attack.htm?360.cn"></script>
```

在很多情况下，开发者在部署过滤 Referer 来源时，忽视了一个空 Referer 的过滤。
一般情况下浏览器直接访问某 URL 是不带 Referer 的，所以很多防御部署是允许空 Referer 的。

使用 `<iframe>` 调用 javscript 伪协议来实现空 Referer 调用 JSON 文件。

```html
<iframe
  src="javascript:'<script>function JSON(o){alert(o.userinfo.userid);}</script><script src=http://www.qq.com/login.php?calback=JSON></script>'"
></iframe>
```

通过随机 token 来防御，如：`http://r.qzone.qq.com/cgi-bin/tfriend/friend_show_qqfriends.cgi?uin=[QQ号码]&g_tk=[随机token]` 来输出 JSON ，同样这个方案也是效的，但是同样可能出现防御实现的不严谨问题。

如这个 token 可以暴力

##### Callback 可自定义导致的安全问题

在输出 JSON 时，没有严格定义好 Content-Type（ Content-Type: application/json ）
而且没有镦 callback 这个输出点没有进行过滤

直接在浏览器输入以下 URL

```
https://www.example.com/api/userInfo?callback=<script>alert(/xss/)</script>
https://www.example.com/api/userInfo/x.html?callback=<script>alert(/xss/)</script>
```

#### JSONP 防御

1. 严格安全的实现 CSRF 方式调用 JSON 文件：限制 `Referer` 、部署一次性 Token 等。
2. 严格安装 JSON 格式标准输出 `Content-Type` 及编码（ Content-Type : application/json; charset=utf-8 ）。
3. 严格过滤 callback 函数名及 JSON 里数据的输出。
4. 严格限制对 JSONP 输出 callback 函数名的长度(如防御上面 flash 输出的方法)。
5. 其他一些比较“猥琐”的方法：如在 Callback 输出之前加入其他字符(如：/\*\*/、回车换行)这样不影响 JSON 文件加载，又能一定程度预防其他文件格式的输出。
   还比如 Gmail 早起使用 AJAX 的方式获取 JSON ，听过在输出 JSON 之前加入 while(1); 这样的代码来防止 JS 远程调用

### iframe

满足某些限制条件的情况下，页面是可以修改它的源。脚本可以将 document.domain 的值设置为其当前域或其当前域的父域。
如果将其设置为其当前域的父域，则这个较短的父域将用于后续源检查。

#### document.domain + iframe 跨域

此方案仅限主域相同，子域不同的跨域应用场景。

实现原理：两个页面都通过 js 强制设置 document.domain 为基础主域，就实现了同域。

```html
<!-- http://child.example.com/a.html -->
<iframe id="iframe" src="http://child.example.com/b.html"></iframe>
<script>
  document.domain = 'example.com';
</script>
```

```html
<!-- http://child.example.com/b.html -->
<script>
  document.domain = 'example.com';
  // document.domain = window.parent.document.domain;
</script>
```

#### iframe 的缺陷

iframe 已被 HTML5 废弃。

HTML5 基于以下原因废弃 iframe：

1. iframe 会阻塞主页面的 onload 事件
2. iframe 和主页面共享连接池，而浏览器对相同域的连接有限制，所以会影响页面的并行加载。

如果 iframe 中的域名因为过期而被恶意攻击者抢注，或者第三方被黑客攻破，iframe 中的内容被替换掉了，从而利用用户浏览器中的安全漏洞下载安装木马、恶意勒索软件等等？？？？

### postMessage

postMessage 是 HTML5 XMLHttpRequest Level 2 中的 API，且是为数不多可以跨域操作的 window 属性之一，它可用于解决以下方面的问题：

1. 页面和其打开的新窗口的数据传递
2. 多窗口之间消息传递
3. 页面与嵌套的 iframe 消息传递

```html
<!-- http://www.example1.com/path -->
<iframe src="http://www.example2.com/path"></iframe>
<script>
  const iframe = document.getElementById('iframe');
  iframe.onload = function () {
    const data = {
      name: 'aym',
    };
    // 向domain2传送跨域数据
    iframe.contentWindow.postMessage(
      JSON.stringify(data),
      'http://www.example2.com',
    );
  };

  // 接受domain2返回数据
  window.addEventListener('message', function (e) {
    console.log('data from example2 ---> ' + e.data);
  });
</script>
```

```html
<!-- parent.html -->
<script>
  window.addEventListener(
    'message',
    function (e) {
      console.loga('data from example1 ---> ' + e.data);
      var data = JSON.parse(e.data);
      window.parent.postMessage(
        JSON.stringify(data),
        'http://www.example1.com',
      );
    },
    false,
  );
</script>
```

## 参考

[Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
[浏览器同源政策及其规避方法](https://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)
[JSONP 跨域资源共享的安全问题](https://xie1997.blog.csdn.net/article/details/87248938)
