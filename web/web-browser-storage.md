# WEB 客户端存储

web 客户端存储有以下几种形式

- Cookie
- Web Storage
  - session storage
  - local storage
- IE User Data
- 应用程序缓存（APP Cache）
- 浏览器端数据库
  - Web SQL Database
  - IndexedDB

## Cookie

> Cookie 是一种被服务端脚本使用的客户端存储机制，能够存储少量的信息。
> 每一次 HTTP 请求，无论是否需要，都会把这些数据传输到服务器端。目前绝大多数的浏览器都支持。

_cookie_ 一词在计算机历史上，很早就被使用。
_cookie_ 和 _magic cookie_ 均表示少量的数据，特别是指类似密码这种用于识别身份和访问许可的保密数据。
不过，**在 javascript 中，不会对 cookie 采取任何对加密措施**，因此 **cookie 是不安全的**。

cookie 的默认有效期很短，只能持续在 Web 浏览器的会话期间，一旦用户关闭浏览器（并不是局限于单个窗口或标签页），cookie 保存的数据将会丢失。
可以通过设置`max-age`属性，明确表明 cookie 的有效期（单位为秒）。
一旦设置了有效期，浏览器会将 cookie 数据存储在一个文件中，直到过了有效期才会删除该文件。

**cookie 的作用域通过文档源和文档路径来确定**。
默认情况下，某个 web 页面创建的 cookie 能够允许该页面同目录或是子目录的其它 Web 页面可见。例如：

- [http://localhost:3000/parent/index.html](http://localhost:3000/parent/index.html) =&gt; 该页面创建了 cookie A
- [http://localhost:3000/parent/index2.html](http://localhost:3000/parent/index2.html) =&gt; 能够访问 cookie A
- [http://localhost:3000/parent/child/index.html](http://localhost:3000/parent/child/index.html) =&gt; 能够访问 cookie A
- [http://localhost:3000/uncle/index.html](http://localhost:3000/uncle/index.html) =&gt; 无法访问
- [http://localhost:3000/index.html](http://localhost:3000/index.html) =&gt; 无法访问

在创建 cookie 时，可以通过设置 cookie 路径的方式，来限制 cookie 的作用域。比如，设置路径为`/`，则该域名下所有的页面都能共享该 cookie。

只能在 `Set-Cookie` 头中明确指定域的情况下，子域名共享父域名的 Cookie。

```
Set-Cookie: name=value; domain=example.com
```

此时，cookie 能够作用域子域名 `img.example.com`。

> 大多数现代浏览器支持 [RFC 6265](http://tools.ietf.org/html/rfc6265) 不需要前导点。
> 否则需要设置为 `Set-Cookie: name=value; domain=.example.com` 方可作用于子域名。

设置 `secure` 使得 cookie 仅作用于 https 链接中。
设置 `httpOnly` 使得 cookie 不能被 JavaScript 调用。
设置 `sameSite` 使得 cookie 跨站请求时无效。

`sameSite` 采用三个可能的值：`Strict`，`Lax` 和 `None`。
使用 `Strict`，Cookie 仅发送到与它起源的站点相同的站点；
`Lax` 与之类似，但用户从外部站点导航至 URL 时（例如通过链接）除外。
没有一个对跨站点请求没有限制。

> 浏览器正在将 cookie 的默认值从 `none` 迁移至 `Lax`.
> 如果 cookie 使用跨域使用，需要明确指定 `sameSite` 属性为 `none`.

### 读写 Cookie

给当前文档设置 cookie：

```js
// 使用 '; ' 分隔
document.cookie = `key=${val}; max-age=${second}; path=${path}; domain=${domain}; secure; httpOnly; sameSite`;
```

读取 cookie 中的数据：

```js
let _cookies = (function () {
  const obj = {};
  String.prototype.split.call(document.cookie, "; ").forEach((item) => {
    const [key, val] = String.prototype.split.call(item, "=").filter(Boolean);
    obj[key] = val;
  });
  return obj;
})();
```

## WebStorage

> web storage 最初作为 HTML5 的一部分被定义成 API 的形式，包含了 localStorage 对象和 sessionStorage 对象，这两个对象实际上是持久化关联数组，是键值对的映射表，“key” 和 “value” 都是字符串。  
> 目前，Web Storage 已经被剥离出来作为独立的一份标准。

### localStorage

**localStorag 存储的数据是永久性的**，除非 Web 应用或用户刻意删除存储的数据。

**localStorage 的作用域限定在文档源（document origin）级别**。

> 文档源通过协议、主机名和端口三部分确定

同源的文档间共享同一份 localStorage 数据，即可以互相读写对方存储在 localStorage 中的数据。

**API:**

- localStorage.setItem\(key, val\)
- localStorage.getItem\(key\)
- localStorage.removeItem\(key\)
- localStorage.clear\(\)

### sessionStorage

**sessionStorage 存储的数据在窗口或标签页关闭后将会丢失。**

> 不过，现在许多现代浏览器具备了重新打开最近关闭的标签页，并恢复上一次浏览的绘画功能。

**sessionStorage 的作用域不但限定在文档源中，还限制在窗口中。**不同浏览器窗口或标签页之间的文档的 sessionStorage 是各自独立的。

**API:**

- sessionStorage.setItem\(key, val\)
- sessionStorage.getItem\(key\)
- sessionStorage.removeItem\(key\)
- sessionStorage.clear\(\)

### onstorage 事件

当 localStorage 或 sessionStorage 的数据发生改变时，浏览器会在**其它对该数据可见的窗口对象**上触发 onstorage 事件。

回调函数参数对象`obj`：

| 属性            | 描述                             |
| :-------------- | :------------------------------- |
| obj.key         |                                  |
| obj.newValue    |                                  |
| obj.oldValue    |                                  |
| obj.storageArea | local or session                 |
| obj.url         | 触发该存储变化脚本所在文档的 url |

## IE userData 持久化数据

> IE5 以及 IE5 以上版本的浏览器是通过在 document 元素后面附加一个专属的“DHTML 行为”来实现客户端存储。

```js
// IE 浏览器的又一特殊设计
var memory = document.createElement("div");
memory.id = "_memory";
memory.style.display = "none";
memory.style.behavior = 'url("#default#userData")';
document.body.appendChild(memory);
```

一旦给原属赋予了"userData"行为， 该元素就拥有了`load`和`save`方法。并且可以通过`getAttribute`和`setAttribute`方法查询和设置数据，然后调用`save`方法可以存储新的数据。

## ~~应用程序缓存~~

> HTML5 中新增了“应用程序缓存”，开发者可以使用`Application Cache`接口设定浏览器应该缓存的资源。允许 Web 应用在处于离线状态时，也能正常加载与工作。
>
> _**不过该特性已经从 Web 标准中删除，未来将会使用 Service Workers 的方式来实现应用程序缓存**_

优势：

- 离线浏览
- 更快的速度
- 减轻服务器的负载

### 启用 Application Cache

在 页面中的`<html>`元素上增加`manifest`特性：

```html
<html manifest="example.appcache">
  ...
</html>
```

manifest 特性与**缓存清单（cache manifest）**文件关联，这个文件包含了需要缓存的资源列表。

### 加载过程

1. 当浏览器访问一个包含`manifest`特性的文档时，如果缓存不存在，浏览器会加载文档，然后获取所有在清单文件中的资源，生产缓存的第一个版本；

2. 对该文档的后续访问会使浏览器直接从应用缓存中加载文档和缓存清单文件中列出的资源，此外还会向`window.applicationCache`对象发送一个`checking`事件；

3. 如果当前缓存的清单副本是最新的，浏览器将向`applicationCache`对象发送一个`noupdate`事件，更新过程结束；

4. 如果清单文件已经改变，文件中列出的所有文件（包括调用`applicationCache.add`方法添加到缓存中的那些文件）会被获取并放到一个临时缓存中。对于每个加入到缓存中的文件，浏览器会向`applicationCache`对象发送一个`progress`事件。如果出现任何错误，浏览器会发送一个`error`事件，并暂停更新；

5. 一旦所有文件都获取成功，它们会自动移送到真正的离线缓存中，并向`applicationCache`对象发送一个`cached`事件。

### 缓存清单文件

缓存清单文件可以使用任意扩展名，但传输它的 **MIME 类型**必须为`text/cache-manifest`

缓存清单的三个段落：

1. **CACHE（缓存资源）**
2. **NETWORK（非缓存资源）**
3. **FALLBACK（后备页面，当资源无法访问时，使用该页面）**

## 浏览器端数据库

使用 sessionStorage 和 localStorage 应对小规模的数据绰绰有余，但是面对需要存储大规模结构化数据的场景时，还是显得不够灵活和强大。HTML5 引入了，**Web SQL Database** 的概念，基于浏览器嵌入的 **Sqlite** 实现。该规范曾经在 W3C 的推荐规范上，但是目前陷入了僵局之中，已经停止。W3C 目前的推荐的客户端大规模数据解构存储的方案是 **IndexedDB。**

### Web SQL Database

> 该规范的僵局：目前的所有实现都基于同一个 SQL 后端 —— SQLite，但是需要更多的独立实现来规范化

Web SQL Database 基于浏览器内嵌的 sqlite 实现，提供了一组 API 供 Web App 创建、存储、查询数据库。虽然该规范已经停止，但是目前大多数主流浏览器都已经支持 Web SQL Database。

### IndexedDB

**IndexedDB** 是一个用于在浏览器中储存较大数据结构的 Web API，并提供索引功能以实现高性能查找。 IndexedDB 是一个事务型的数据库系统，类似于关系型数据库系统（RDBMS），不过, 它是使用 JavaScript 对象而非列数固定的表格来储存数据的。

IndexedDB 可以存储所有 [structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm) 支持的数据类型，并且是永久有效的，online 和 offline 模式下均能使用。

IndexedDB 访问权限严格基于同源策略。
