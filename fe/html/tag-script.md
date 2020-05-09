# script

Script 标签用于插入 JavaScript 脚本或引入 JavaScript 脚本

## async 和 defer

Script 脚本会阻塞 HTML Parser。
async 和 defer 用于标记脚本加载与 HTML Parser 并发运行

### async

指示浏览器是否在允许的情况下异步执行该脚本。
加载脚本完成后立即执行，执行脚本时会阻塞 HTML Parser。

```html
<script async src="https://example.com/xxx.js" />
```

### defer

defer 表明 JS 为异步加载，并且脚本将在文档完成解析后，触发 `DOMContentLoaded` 事件前执行
对动态嵌入的脚本使用 `async=false` 来达到类似的效果。

```html
<script defer src="https://example.com/xxx.js" />
```

### 比较 async and defer

![normal script compare async and defer](https://images2015.cnblogs.com/blog/1089472/201706/1089472-20170620090132491-1002446728.png)

A Demo:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <div>123</div>
    <script
      defer
      src="./defer.js"
      onload="console.log('defer onload')"
    ></script>
    <script
      async="false"
      src="./async.js"
      onload="console.log('async onload')"
    ></script>
    <script
      src="https://cdn.bootcdn.net/ajax/libs/antd/4.2.0/antd.js"
      onload="console.log('antd')"
    ></script>
    <script>
      console.log("last script", document.querySelector("#bottom"));
      document.addEventListener(
        "DOMContentLoaded",
        function () {
          console.log("DOMContentLoaded");
        },
        false
      );
    </script>
    <div id="bottom">bottom</div>
  </body>
</html>
```

```js
// ./async.js
console.log("aync1", document.querySelector("#bottom"));
```

```js
// ./defer.js
console.log("defer", document.querySelector("#bottom"));
```

```log
aync1 null
async onload
antd
last script null
defer `<div id="bottom">bottom</div>`
defer onload
DOMContentLoaded
```

利用 antd lib 增加网络消耗，更清晰的展示 defer 和 async 的区别。

如果去掉 antd 的 script. 由于 JavaScript 文件太小，HTML Parser 过快能够产生以下结果

```log
last script null
defer `<div id="bottom">bottom</div>`
defer onload
DOMContentLoaded
aync1 `<div id="bottom">bottom</div>`
async onload
```

## crossorigin

那些没有通过标准 CORS 检查的正常 script 元素传递最少的信息到 window.onerror。
可以使用本属性来使那些将静态资源放在另外一个域名的站点打印错误信息。

| 关键字          | 描述                                                                            |
| --------------- | ------------------------------------------------------------------------------- |
| anonymous       | 对此元素的 CORS 请求将不设置凭据标志。                                          |
| use-credentials | 对此元素的 CORS 请求将设置凭证标志；这意味着请求将提供凭据。                    |
| ""              | 设置一个空的值，如 crossorigin 或 crossorigin=""，和设置 anonymous 的效果一样。 |

<script src="https://example.js" crossorigin="anonymous"></script>
