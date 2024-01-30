# 懒加载资源

如果加载用户可能永远看不到的内容，会占用网络带宽，浪费处理时间，电池和其他系统资源。

懒加载是一种解决该类常见的优化方案。
在页面加载时，推迟**非关键资源**的加载可以带来非常客观的性能提升。

## 懒加载图片

在涉及图像的地方，“非关键”通常等同于“屏幕外”。

### 懒加载 <img>元素中使用的图像

#### `<img/>` 的 loading 属性

大多数现代浏览器上的 `<img/>` 均支持了 `loading` 属性。

loading 属性支持的值：

- `auto`：浏览器的默认延迟加载行为，与不包含该属性的行为相同。**在规范中未提及该值。**
- `lazy`：将资源的加载推迟到到达可视窗口的某个距离为止。
- `eager`：立即加载资源，而不管其在页面上的位置。

距离阈值不是固定的，并且取决于几个因素：

- 正在获取的图像资源的类型
- 是否在 Android 版 Chrome 上启用了精简模式
- [有效的连接类型](https://wicg.github.io/netinfo/#connection-types)

例如，当前 Chrome 版本在快速连接（例如 4G）上的距视口的距离阈值为 1250px；
在较慢的连接（例如 3G）上，阈值为 2500px。

**使用 loading 属性时，图片应该指定 `width` 和 `height`**。

浏览器加载图像时，除非明确指定尺寸，否则它不会立即知道图像的尺寸。
为了使浏览器能够在页面上为图像保留足够的空间，建议所有`<img>`标记都同时包含 width 和 height 属性。

固定图片高宽还能很好的避免[布局偏移](./performance-metrics.md#CLS)

```html
<img src="image.png" loading="lazy" alt="…" width="200" height="200" />
```

使用`<picture>`元素定义的图像也可以延迟加载：

```html
<picture>
  <source media="(min-width: 800px)" srcset="large.jpg 1x, larger.jpg 2x" />
  <img src="photo.jpg" loading="lazy" />
</picture>
```

如果没有指定 `width` 和 `height`，图像初始尺寸是 `0×0` 像素。
因为图像实际上都不会占用空间，也不会将图像推离屏幕，浏览器可能会在一开始确认图像在视图窗口中。
也就是说浏览器会加载所有未指定高宽的图片。

loading 属性的兼容方案：

```html
<script>
  if ('loading' in HTMLImageElement.prototype) {
    const images = document.querySelectorAll('img[loading="lazy"]');
    images.forEach((img) => {
      img.src = img.dataset.src;
    });
  } else {
    // polyfill
    // use IntersectionObserver if support it
    // else use `getBoundingClientRect`
  }
</script>
```

> `<iframe loading=lazy>` 已被标准化，并已在 Chromium 中实施。

#### 基于 IntersectionObserver

[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver) 供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。

```js
document.addEventListener('DOMContentLoaded', () => {
  const lazyImages = [].slice.call(document.querySelectorAll('img.lazy'));

  if ('IntersectionObserver' in window) {
    const options = {
        root: document,
        rootMargin: '0px',
        threshold: 1.0
    };

    const lazyImageObserver = new IntersectionObserver((
      entries,
      observer,
    ) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const lazyImage = entry.target;
          lazyImage.src = lazyImage.dataset.src;
          lazyImage.srcset = lazyImage.dataset.srcset;
          lazyImage.classList.remove('lazy'); // for lazy load css img
          lazyImageObserver.unobserve(lazyImage);
        }
      });
    }, options);

    lazyImages.forEach(lazyImage) => {
      lazyImageObserver.observe(lazyImage);
    });
  } else {
    // Possibly fall back to event handlers here
  }
});
```

#### 基于 `getBoundingClientRect`

[Element.getBoundingClientRect()](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect) 方法返回元素的大小及其相对于视口的位置。

```js
const inView = (element, offset = 0) => {
  const rect = element.getBoundingClientRect();
  return (
    rect.top <= window.innerHeight + offset &&
    rect.bottom >= -offset &&
    rect.right <= window.innerWidth + offset &&
    rect.left >= -offset &&
    getComputedStyle(element).style.display !== 'none'
  );
};

document.addEventListener('DOMContentLoaded', () => {
  const lazyImages = [].slice.call(document.querySelectorAll('img.lazy'));
  let active = false;

  const lazyLoad = () => {
    if (active === false) {
      active = true;

      setTimeout(() => {
        lazyImages.forEach((lazyImage) => {
          const offset = 0;

          if (inView(lazyImage, offset)) {
            lazyImage.src = lazyImage.dataset.src;
            lazyImage.srcset = lazyImage.dataset.srcset;
            lazyImage.classList.remove('lazy'); // for lazy load css img

            lazyImages = lazyImages.filter((image) => image !== lazyImage);

            if (lazyImages.length === 0) {
              document.removeEventListener('scroll', lazyLoad);
              window.removeEventListener('resize', lazyLoad);
              window.removeEventListener('orientationchange', lazyLoad);
            }
          }
        });

        active = false;
      }, 200);
    }
  };

  document.addEventListener('scroll', lazyLoad);
  window.addEventListener('resize', lazyLoad);
  window.addEventListener('orientationchange', lazyLoad);
});
```

### 懒加载 CSS 中的图片

浏览器会在请求外部资源之前检查 CSS 是否应用于 Document。
对于 CSS 中的图片可以通过不同的 class 进行图片懒加载。

```html
<div class="background lazy-background">
  <h1>Here's a hero heading to get your attention!</h1>
  <p>Here's hero copy to convince you to buy a thing!</p>
  <a href="/buy-a-thing">Buy a thing!</a>
</div>
```

```css
.background {
  background-image: url('hero.jpg'); /* The final image */
}

.lazy-background {
  background-image: url('hero-placeholder.jpg') !import; /* Placeholder image */
}
```

```js
if (inView(element)) {
  element.classList.remove('lazy-background');
}
```

## 懒加载视频

与图像一样，也可以延迟加载视频。
如何延迟加载`<video>`取决于用例。

### 不自动播放的视频

不自动播放的视频需要在元素上指定 `preload` 属性为 `none` 来防止浏览器预加载任何视频数据。
当用户启动播放时，使用 `preload="none"` 是在所有平台上推迟视频加载的最简单方法。

```html
<video controls preload="none" poster="one-does-not-simply-placeholder.jpg">
  <source src="one-does-not-simply.webm" type="video/webm" />
  <source src="one-does-not-simply.mp4" type="video/mp4" />
</video>
```

### 自动播放的视频

与 `<img>` 延迟加载示例一样，将视频 URL 存储 `data-src` 在每个`<source>` 元素的属性中。

使用 `<video>` 元素替代 Gif 动画：

```html
<video
  autoplay
  muted
  loop
  playsinline
  width="610"
  height="254"
  poster="one-does-not-simply.jpg"
>
  <source data-src="one-does-not-simply.webm" type="video/webm" />
  <source data-src="one-does-not-simply.mp4" type="video/mp4" />
</video>
```

```js
document.addEventListener('DOMContentLoaded', () => {
  const lazyVideos = [].slice.call(document.querySelectorAll('video.lazy'));

  if ('IntersectionObserver' in window) {
    const options = {
      root: document,
      rootMargin: '0px',
      threshold: 1.0,
    };
    const lazyVideoObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach((video) => {
        if (video.isIntersecting) {
          for (let source in video.target.children) {
            let videoSource = video.target.children[source];
            if (
              typeof videoSource.tagName === 'string' &&
              videoSource.tagName === 'SOURCE'
            ) {
              videoSource.src = videoSource.dataset.src;
            }
          }

          video.target.load();
          video.target.classList.remove('lazy');
          lazyVideoObserver.unobserve(video.target);
        }
      });
    }, options);

    lazyVideos.forEach((lazyVideo) => {
      lazyVideoObserver.observe(lazyVideo);
    });
  }
});
```

## 懒加载 JS 资源

ECMAScript 支持 dynamic import 支持按需加载资源。
此外还有 AMD 以及 SystemJS 等规范。

Webpack，Rollup 等 bundle 工具均对此有很好的支持。

```js
const LazyComponent = React.lazy(() => import('script.js'));
```
