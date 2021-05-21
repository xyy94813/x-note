# 点击劫持（Click Jacking）

**点击劫持本质上是一种视觉欺骗手段。**
攻击者使用一个透明不可见的 iframe 覆盖在一个网页上，然后诱使用户在不知情的情况下点击透明的 iframe 页面。

```html
<html>
  <body>
    <button>这是一个很安全的按钮</button>
    <iframe
      src="https://attact-site-url"
      style="position:fixed;opacity:0;"
      scrolling="no"
    ></iframe>
  </body>
</html>
```

## 攻击类型

1. flash 点击劫持
2. 图片覆盖
3. 拖拽劫持
4. 触屏劫持

### Flash 点击劫持

Flash 点击劫持就是在 flash 内容上隐藏了一个看不见的 iframe。

### 图片覆盖

顺着点击劫持本质上是一种视觉欺骗手段这个思路，产生了跨站图片覆盖（Cross Site Image Overlaying）攻击。

XSIO 不同于 XSS ， 它利用的是图片的 style 或能够控制 CSS。
如果应用没有限制 style 的 position 为 absolute，图片就可以覆盖到页面上的任意位置，形成点击劫持。

```html
<html>
  <body>
    <iframe src="https://attact-site-url"></iframe>
    <a href="https://钓鱼网站">
      <!-- 调整图片样式至被攻击站点的相同的位置 -->
      <img src="https://attact-site-domin/image-url" style="position:fixed;" />
    </a>
  </body>
</html>
```

### 拖拽劫持

在浏览器开始支持 Drag & Drop API 后。
浏览器的拖拽对象可以是一个链接，也可以是一段文字，还可以从一个窗口拖拽到另一个窗口。

拖拽劫持的思路是诱导用户从不见的 iframe 中“拖拽”出攻击者希望得到的数据，然后放到攻击者能够控制的另一个页面中，从而窃取数据。

### 触屏劫持

通过讲一个不可见的 iframe 覆盖到当前页面上，可以劫持用户的触屏操作。

在移动互联占据主要市场的这个时代，大多数手机浏览器为了节约空间，隐藏了地址栏，因此手机上的视觉欺骗变得更加容易实施。

## 防御手段

1. frame busting
2. `x-frame-options`

### Frame Busting

通常可以写一段 JavaScript 代码禁止 iframe 嵌套：

```html
<style>
  body {
    display: none;
  }
</style>
<script>
  // 最佳 Frame Busting：
  if (self == top) {
    document.getElementsByTagName('body')[0].style.display = 'block';
  } else {
    top.location = self.location;
  }
</script>
```

常见 iframe busting：

- `if (top != self)`
- `if (top.location != self.location)`
- `if (top.location != location)`
- `if (parent.frames.length > 0)`
- `if (window != top)`
- `if (window.top !== window.self)`
- `if (window.self != window.top)`
- `if (parent && parent != window)`
- `if (parent && parent.frames && parent.frames.length>0)`
- `if ((self.parent&&!(self.parent===self))&&(self.parent.frames.length!=0))`

破坏框架的条件语句：

- `top.location = self.location`
- `top.location.href = document.location.href`
- `top.location.href = self.location.href`
- `top.location.replace(self.location)`
- `top.location.href = window.location.href`
- `top.location.replace(document.location)`
- `top.location.href = window.location.href`
- `top.location.href = "URL"`
- `document.write(’’)`
- `top.location = location`
- `top.location.replace(document.location)`
- `top.location.replace(’URL’)`
- `top.location.href = document.location`
- `top.location.replace(window.location.href)`
- `top.location.href = location.href`
- `self.parent.location = document.location`
- `parent.location.href = self.document.location`
- `top.location.href = self.location`
- `top.location = window.location`
- `top.location.replace(window.location.pathname)`
- `window.top.location = window.self.location`
- `setTimeout(function(){document.body.innerHTML=’’;},1);`
- `window.self.onload = function(evt){document.body.innerHTML`

Frame Busting 可以通过各种手段绕过。
比如 H5 中 iframe 的 sandbox 属性、IE 中 iframe 的 security 属性

### x-frame-options

`x-frame-options` 就是为了解决点击劫持产生的 HTTP 响应头

`X-Frame-Options` 有三个可能的值：

```
X-Frame-Options: deny
X-Frame-Options: sameorigin
X-Frame-Options: allow-from https://example.com/
```

在不支持 x-frame-options 的浏览器中，只能继续使用 Frame Busting。

### Content-Security-Policy

可以通过 [Content-Security-Policy 的 frame-ancestors 指令](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) 控制哪些页面能够 `<frame>`, `<iframe>`, `<object>`, `<embed>`, 或 `<applet>` 嵌入当前页面。

```
Content-Security-Policy: frame-ancestors <source>;
```

## Reference

- 白帽子讲 WEB 安全
- [Busting frame busting: a study of click-jacking vulnerabilities at popular sites](https://seclab.stanford.edu/websec/framebusting/framebust.pdf)
- [Framing Attacks on Smart Phones and Dumb Routers: Tap-jacking and Geo-localization](https://seclab.stanford.edu/websec/framebusting/tapjacking.pdf)
- [HTTP Header Field X-Frame-Options](https://tools.ietf.org/html/rfc7034)
