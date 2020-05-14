# webp 兼容性方案

- `<picture>` 标签
- JS 替换图片的 URL
- webp polyfill
- webp-loader
- 服务器处理

## `<picture>` 标签

h5 新增的`<picture>`

HTML `<picture>` 元素通过包含零或多个 `<source>` 元素和一个 `<img>` 元素来为不同的显示/设备场景提供图像版本。
浏览器会选择最匹配的子 `<source>` 元素，如果没有匹配的，就选择 `<img>` 元素的 src 属性中的 URL。
然后，所选图像呈现在`<img>`元素占据的空间中。

`<picture>` 的常见使用场景：

- Art direction —— 针对不同 media 条件裁剪或修改图像
- 遇到所有浏览器都不支持的特定格式时，提供不同的图像格式
- 通过加载最适合查看者显示的图像来节省带宽并加快页面加载时间。

> `<audio />` 和 `<video />` 也支持指定多个源。

```html
<picture>
  <source type="image/webp" srcset="./img.webp" />
  <img src="./img.jpg" alt="it is a jpg" />
</picture>
```

对于不支持 H5 的浏览器使用 [html5shiv](https://github.com/aFarkas/html5shiv) 做兼容

> css 中 webp 不兼容

## JS 替换图片的 URL

使用最小化的 webp 图片去判断浏览器是否支持 webp.
然后替换 src。

```js
// check_webp_feature 出自 Google 官方文档
function check_webp_feature(feature, callback) {
  var kTestImages = {
    lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
    lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
    alpha:
      "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
    animation:
      "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA",
  };
  var img = new Image();
  img.onload = function () {
    var result = img.width > 0 && img.height > 0;
    callback(feature, result);
  };
  img.onerror = function () {
    callback(feature, false);
  };
  img.src = "data:image/webp;base64," + kTestImages[feature];
}

check_webp_feature("lossless", function (feature, support) {
  if (!support) {
    const $imgs = document.querySelectorAll('img[src*="webp"]');
    const replaceImgSrc = (img) => img.src.replace(".webp", ".jpg");
    $imgs.forEach(img, replaceImgSrc);
  }
});
```

> 性能极低

## webp polyfill

引入 [webp-hero](https://github.com/chase-moskal/webp-hero)

引入 polyfill 本身就是一个巨大的成本

## 服务器处理

不支持 webp 的浏览器请求 webp 文件时，accept 中不包含 `image/webp`.
服务器判断不支持时返回其它格式的 image

完美兼容 CSS 与 image，需要提供额外的 webp 副本。

[Rewriting to WebP was easy with caddy](https://vcsjones.dev/2017/12/24/caddy/):

```
header /images {
    Vary Accept
}

rewrite /images {
    ext .png .jpeg .jpg
    if {>Accept} has image/webp
    to {path}.webp {path}
}
```

CDN 是否支持该功能？

## webpack-loader

使用 [multi-loader](https://yarnpkg.com/package/multi-loader) 复制文件副本
使用 [webp-loader](https://yarnpkg.com/package/webp-loader) 将图片转成 webp
