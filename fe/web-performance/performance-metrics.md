# WEB 性能指标

性能是相对的.

- 站点可能对于 “高端用户” -- 较好的网络以及高性能设备的，很快，但是对于“低端用户” -- 较差的网络和低性能设备，很慢。
- 两个站点可能在完全相同的时间完成加载，但是其中一个站点加载内容更快。
- 站点加载速度很快，但是对用户交互的响应缓慢

在过去的几年中，Chrome 团队的成员与 [W3C Web Performance Working Group](https://www.w3.org/webperf/)，一直在努力标准化一组新的 API 和指标，以更准确地衡量用户体验网页性能的方式。

Google Chrome 团队在 2020 年 5 月的 Chrome DEV 上如何对站点对性能进行量化，进一步提出了相应的解决方案.

> https://www.youtube.com/watch?v=t8YBZLjL-KU

## 历史站点性能衡量

在之前，常常是通过 [`Load`](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event) 事件和 [`DOMContentLoad`](https://developer.mozilla.org/en-US/docs/Web/API/Window/DOMContentLoaded_event) 事件来衡量一个站点的性能。

但是，即使 `Load` 是页面生命周期中定义明确的时刻，该时刻也不一定与用户关心的任何事情相对应。

例如，服务器返回一个能够很快加载的小页面，然后在 `load` 事件后延迟请求数据并展示。
尽管技术角度该页面加载很快，但是从用户角度该时间与用户时间并不对应。

## 性能指标

为了帮助确保指标与用户相关，所有指标应围绕以下关键问题：

|                   |                                   |
| ----------------- | --------------------------------- |
| Is it happening?  | 导航正常启动? 服务器是否响应?     |
| Is it useful?     | 是否存在足够内容让用户参与?       |
| Is it usable?     | 用户是否能够与页面进行互动?       |
| Is it delightful? | 交互是否顺畅自然，没有滞后和颠簸? |

性能指标通常以以下两种方式之一进行度量：

- In the lab：使用 lighthouse 之类都工具在一致的受控环境中模拟页面加载
- In the field: 真实的数据

### 指标类型

- 感知的加载速度（Perceived load speed）：页面能够以多快的速度加载所有可视元素并将其呈现到屏幕。
- 加载响应度（Load responsiveness）：页面加载和执行任何必需的 JavaScript 代码以使组件快速响应用户交互的速度。
- 运行时响应（Runtime responsiveness）：页面加载后，页面对用户交互的响应速度有多快。
- 视觉稳定性（Visual stability）：页面上的元素是否以用户不期望的方式移动，并可能干扰它们的交互？
- 平滑度（Smoothness）：转场和动画是否以一致的帧速率渲染并从一种状态流畅地流动到另一种状态？

### 指标测量工具

In the lab:

- [Lighthouse](https://developers.google.com/web/tools/lighthouse/)
- [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/)
- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)

In the field:

- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
- [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report)
- [Search Console (Speed Report)](https://webmasters.googleblog.com/2019/11/search-console-speed-report.html)
- [Firebase Performance Monitoring (beta)](https://firebase.google.com/docs/perf-mon/get-started-web)

JS SDK:

- [web-vitals](https://github.com/GoogleChrome/web-vitals)

## FP

FP（First Paint）包含一个 DOMHighResTimeStamp，报告用户代理在导航后首次呈现的时间。
**不包括默认背景画**，但包括**非默认背景画和 iframe 的封闭框**。

## FCP

First Content Paint（FCP）指标，测量从页面开始加载到屏幕上呈现页面内容的任何部分的时间。

> 这里的 Content 包括文本、图片、SVG 元素以及非全白的 Canvas 对象。

### FCP 指标

**FCP 应该尽可能正在 1s 以内。**

### 通过 JavaScript 测量 FCP

通过 Google 提供的 [web-vitals](https://github.com/GoogleChrome/web-vitals) 库测量 FCP。

```js
import { getFCP } from 'web-vitals';

// Measure and log the current FCP value,
// any time it's ready to be reported.
getFCP(console.log);
```

通过 [Paint Timing API](https://w3c.github.io/paint-timing/) 测量该指标

```js
let firstHiddenTime = document.visibilityState === 'hidden' ? 0 : Infinity;
document.addEventListener(
  'visibilitychange',
  (event) => {
    firstHiddenTime = Math.min(firstHiddenTime, event.timeStamp);
  },
  { once: true },
);

try {
  const onPaintEntry = (entry) => {
    if (entry.name === 'first-contentful-paint') {
      // Only report if the page wasn't hidden prior to the first paint.
      if (entry.startTime < firstHiddenTime) {
        po.disconnect();
        // sendToAnalytics({ fcp: entry.startTime });
      }
    }
  };

  const po = new PerformanceObserver((entryList, po) =>
    entryList.getEntries().map((entry) => onPaintEntry(entry, po)),
  );
  po.observe({
    type: 'paint',
    buffered: true,
  });
} catch {
  // not to do anything
}
```

通过 [Performance Timeline API](https://w3c.github.io/performance-timeline/#dom-performance-getentriesbyname)

> unstable

```js
window.performance.getEntriesByName('first-contentful-paint');
```

### 提升 FCP

- [Eliminate render-blocking resources](https://web.dev/render-blocking-resources)
- [Minify CSS](https://web.dev/unminified-css)
- [Remove unused CSS](https://web.dev/unused-css-rules)
- [Preconnect to required origins](https://web.dev/uses-rel-preconnect)
- [Reduce server response times (TTFB)](https://web.dev/time-to-first-byte)
- [Avoid multiple page redirects](https://web.dev/redirects)
- [Preload key requests](https://web.dev/uses-rel-preload)
- [Avoid enormous network payloads](https://web.dev/total-byte-weight)
- [Serve static assets with an efficient cache policy](https://web.dev/uses-long-cache-ttl)
- [Avoid an excessive DOM size](https://web.dev/dom-size)
- [Minimize critical request depth](https://web.dev/critical-request-chains)
- [Ensure text remains visible during webfont load](https://web.dev/font-display)
- [Keep request counts low and transfer sizes small](https://web.dev/resource-summary)

## LCP

Largest Contentful Paint (LCP）衡量可视窗口内可见的最大图像或文本块的渲染时间。

### LCP 指标

**LCP 应在 2.5s 内，且 `LCP / Load Time < 0.75`**

- Good：0-2.5s
- Need Improvement：2.5-4.0s
- Poor：大于 4s

### LCP 考虑的元素类型

- `<img />` tag
- `<image />` 内的 `<svg />` 元素
- `<video />` 元素
- 具有通过 `url()` 功能加载的背景图片的元素
- 包含文本节点或其它内联级文本元素子级的**块级元素**。

> 将元素限制在此有限范围内，以便一开始就保持简单。将来可能会添加其它元素（例如`<svg>`，`<video>`）。

### LCP 如何计算元素的大小

LCP 观测的元素的大小通常是**用户在视图窗口中的可见大小**，如果元素被裁剪或溢出不可见，这些部分不会被记入。

对于 Image 元素，则观测的元素大小取可视大小（visible size）和**固有尺寸（intrinsic size）**中较小的值。

```js
const imgLCPSize = Math.min(imgVisibleSize, imgIntrinsicSize);
```

对于文本元素，仅考虑其文本节点的大小（包含所有文本节点的最小矩形）。

对于所有元素，不考虑通过 CSS 应用的任何边距，填充或边框。

> 确定哪个文本节点属于哪个元素有时可能很棘手，尤其是对于其子元素包括嵌入式元素和纯文本节点以及块级元素的元素而言。
> 关键是每个文本节点都属于（并且仅属于）其最接近的块级祖先元素。
> 在规范中：[每个文本节点都属于生成其包含块的元素。](https://wicg.github.io/element-timing/#set-of-owned-text-nodes)

网页通常是分阶段加载的，因此，页面上最大的元素可能会发生变化。

为了处理这种潜在的更改，浏览器会在绘制第一帧后立即调度一个类型为 `maximum-contentful-paint` 的 `PerformanceEntry`，以标识最大的内容元素。
但是，在渲染后续帧之后，只要有最大内容的元素发生变化，它将分配另一个 `PerformanceEntry`。

除了延迟加载图像和字体外，页面可能会在新内容可用时向 DOM 添加新元素。
如果这些新元素中的任何一个大于先前最大的内容元素，那么还将报告新的 `PerformanceEntry`。

如果页面从 DOM 中删除了一个元素，则不再考虑该元素。
同样，如果元素的关联图像资源发生更改（例如，通过 JavaScript 更改 img.src），则该元素将停止考虑，直到加载新图像为止。

> 将来，从 DOM 中删除的元素仍可以视为 LCP 候选对象。
> 目前[正在进行研究以评估此更改的影响](https://github.com/WICG/largest-contentful-paint/issues/41#issuecomment-583589387)。
> 可以遵循 [CHANGELOG](http://bit.ly/chrome-speed-metrics-changelog) 指标以保持最新。

**一旦用户与页面进行交互，浏览器将停止报告新条目，因为用户交互通常会改变用户可见的内容。**

> 由于用户可以在背景标签中打开页面，因此，**只有当用户将标签聚焦后，最大的内容绘画才会出现**，这可能比第一次加载时要晚得多。
> FCP 也是如此。

出于安全考虑，对于缺少 `Timing-Allow-Origin` 标头的跨域图像，不会公开图像的渲染时间戳，而是只公开其加载时间。

### LCP 如何处理元素布局和大小更改

为了使计算和分配新性能条目的性能开销保持较低，对元素大小或位置的更改不会生成新的 LCP 候选对象。
仅考虑元素的初始大小和在视口中的位置。

这意味着可能不会报告最初在屏幕外渲染然后在屏幕上过渡的图像。
这也意味着最初在视口中渲染的元素随着被“下推”到视线外，仍然会报告其初始的视口大小。

但是，如果某个元素已从 DOM 中删除或与其关联的图像资源发生了更改，则该元素将从考虑中删除。

[Example](https://web.dev/lcp/#examples)

### 通过 JavaScript 测量 LCP

实验和线上环境均能测量该指标

通过 Google 提供的 [web-vitals](https://github.com/GoogleChrome/web-vitals) 库测量 FCP。

```js
import { getLCP } from 'web-vitals';

getLCP(console.log);
```

基于 [Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/)

```js
// Keep track of whether (and when) the page was first hidden, see:
// https://github.com/w3c/page-visibility/issues/29
// NOTE: ideally this check would be performed in the document <head>
// to avoid cases where the visibility state changes before this code runs.
let firstHiddenTime = document.visibilityState === 'hidden' ? 0 : Infinity;
document.addEventListener(
  'visibilitychange',
  (event) => {
    firstHiddenTime = Math.min(firstHiddenTime, event.timeStamp);
  },
  { once: true },
);

// Use a try/catch instead of feature detecting `largest-contentful-paint`
// support, since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create a variable to hold the latest LCP value (since it can change).
  let lcp;

  function updateLCP(entry) {
    // Only include an LCP entry if the page wasn't hidden prior to
    // the entry being dispatched. This typically happens when a page is
    // loaded in a background tab.
    if (entry.startTime < firstHiddenTime) {
      // NOTE: the `startTime` value is a getter that returns the entry's
      // `renderTime` value, if available, or its `loadTime` value otherwise.
      // The `renderTime` value may not be available if the element is an image
      // that's loaded cross-origin without the `Timing-Allow-Origin` header.
      lcp = entry.startTime;
    }
  }

  // Create a PerformanceObserver that calls `updateLCP` for each entry.
  const po = new PerformanceObserver((entryList, po) => {
    entryList.getEntries().forEach((entry) => updateLCP(entry, po));
  });

  // Observe entries of type `largest-contentful-paint`, including buffered entries,
  // i.e. entries that occurred before calling `observe()` below.
  po.observe({
    type: 'largest-contentful-paint',
    buffered: true,
  });

  // Log the final LCP score once the
  // page's lifecycle state changes to hidden.
  addEventListener(
    'visibilitychange',
    function fn(event) {
      if (document.visibilityState === 'hidden') {
        removeEventListener('visibilitychange', fn, true);

        // Force any pending records to be dispatched and disconnect the observer.
        po.takeRecords().forEach((entry) => updateLCP(entry, po));
        po.disconnect();

        // If LCP is set, report it to an analytics endpoint.
        if (lcp) {
          // sendToAnalytics({ lcp });
        }
      }
    },
    true,
  );
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```

> 如果页面是在后台选项卡中加载的，则不应报告 LCP。
> 上面的代码部分解决了这个问题，但是由于页面可能已经隐藏然后在运行此代码之前显示出来，所以它并不是完美的。
> [Page Visibility API 规范](https://github.com/w3c/page-visibility/issues/29)中正在讨论此问题的解决方案

#### 提升 LCP

LCP 主要受以下四个因素影响

- 服务器响应时间
- JavaScript 和 CSS 导致的渲染阻塞
- 资源加载时间
- 客户端渲染（Client-side rendering）

提升手段

- [Apply instant loading with the PRPL pattern](https://web.dev/apply-instant-loading-with-prpl)
- [Optimizing the Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [Optimize your CSS](https://web.dev/fast#optimize-your-css)
- [Optimize your Images](https://web.dev/fast#optimize-your-images)
- [Optimize web Fonts](https://web.dev/fast#optimize-web-fonts)
- [Optimize your JavaScript](https://web.dev/fast#optimize-your-javascript)

## TTI

可交互时间（Time to Interactive, TTI）衡量的是从页面开始加载到页面主要子资源加载之间的时间，它能够反应用户最快的可交互时间。

要基于网页的 [performance trace](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference) 来计算 TTI。

1. 从 FCP 开始。
2. 及时向前搜索至少五秒钟的**静默窗口（quite window）**，_静默窗口_ 定义为：无长任务且处理中的 GET 请求不超过两个。
3. 向后搜索静默窗口之前的最后一个长任务，如果找不到长任务，则在 FCP 处停止。
4. TTI 是静默窗口之前的最后一个长任务的结束时间（如果找不到长任务，则为与 FCP 相同的值）。

![如何通过页面加载时间表显示了计算 TTI ](https://webdev.imgix.net/tti/tti-timeline.svg)

> 图片来自于 [Time to Interactive (TTI)](https://web.dev/tti/)

SSR 之类的技术可能会导致页面看上去是交互式的（即，链接和按钮在屏幕上可见）的场景 。
但实际上并不是交互式的，因为主线程被阻塞或因为控制这些线程的 JavaScript 代码元素尚未加载。

当用户尝试与看似互动但实际上不互动的页面进行交互时，他们可能会以以下两种方式之一进行响应：

- 在最佳情况下，他们会因为页面响应缓慢而感到烦恼。
- 在最坏的情况下，他们会假设页面已损坏并且可能会离开。他们甚至可能对您的品牌价值失去信心或信任。

### TTI 指标

**良好的 TTI 应该小于 5S**

lighthouse 评分：

- fast: 0-0.38s
- moderate: 3.9–7.3s
- slow: 大于 7.3s

### 提升 TTI

- [缩小 JavaScript](https://web.dev/unminified-javascript/)
- [预连接到所需的起点](https://web.dev/uses-rel-preconnect/)
- [预加载关键请求](https://web.dev/uses-rel-preload/)
- [减少第三方代码的影响](https://web.dev/third-party-summary/)
- [最小化关键请求深度](https://web.dev/critical-request-chains/)
- [减少 JavaScript 执行时间](https://web.dev/bootup-time/)
- [减少主线程工作](https://web.dev/mainthread-work-breakdown/)
- [保持低请求数量和小传输大小](https://web.dev/resource-summary/)

## FID

**首次输入延迟（First Input Delay，FID）**度量标准有助于衡量用户对网站的交互性和响应性的第一印象。

使用 FCP 可以衡量用户对您的网站加载速度有第一印象。
但是，您的网站可以在屏幕上绘制像素的速度只是故事的一部分。
当用户尝试与这些像素进行交互时，您的网站的响应速度同样重要！

FID 衡量的是从用户第一次与页面进行交互到浏览器实际上能够开始处理交互事件的时间。

通常，FCP 与交互时间（TTI）之间会出现较长的首次 FID，因为页面已呈现了其某些内容，但尚未可靠地进行交互。
FID 常常是因为浏览器的主线程在处理其它操作导致（通常是加载大型 JS 脚本）。

![使用FCP，TTI 和 FID 的示例页面加载跟踪](https://web.dev/fid/fid-full.svg)

> 图片来自于 [First Input Delay (FID)](https://web.dev/fid/)

在上图中，用户恰好在主线程最繁忙时段的开始时与页面进行交互。
如果用户只是在片刻之前（在空闲期间）与页面进行了交互，则浏览器可能会立即做出响应。
FID 的这种差异强调了在报告度量标准时查看 FID 值分布的重要性。

FID 测量接收输入事件与下一次主线程空闲之间的增量。这意味着即使在未注册事件侦听器的情况下，也将测量 FID 。

> FID 仅在事件处理中测量“延迟”。
> **它不测量事件执行时间本身，也不测量事件处理程序运行后浏览器更新 UI 所花费的时间。**

将测量事件执行时间和浏览器更新 UI 的时间其作为 FID 的一部分包括在内可能会激励开发人员添加实际上会使体验更糟的变通办法，但更可能导致糟糕的情况，尽管这段时间确实会影响用户体验。

例如：

他们可以将事件处理程序逻辑包装在异步中回调（通过 `setTimeout()` 或 `requestAnimationFrame()`），以便将其从与事件关联的任务中分离出来。
结果将是度量得分的提高，但用户感知到的响应变慢。

但是，尽管 FID 仅测量事件延迟的“延迟”部分，但是想要跟踪更多事件生命周期可以使用 [Event Timing API](https://wicg.github.io/event-timing/) 进行此操作。

### FID 指标

优秀的站点 FID 的时间应该小于 100ms，或 `FID < Load Time * 75%`

- Good：0-100ms
- Need Improvement：100-300ms
- Poor：大于 300ms

### 通过 JavaScript 测量 FID

计算 FID 时要注意以下事项

- 忽略页面在后台标签时触发的 `first-input` 条目
- 如果页面在第一次输入发生之前已被后台处理，PerformanceObserver API 还会触发 `first-input` 条目，但是在计算 FID 时也应忽略这些页面（仅当页面始终处于前台时才考虑输入）。
- 从后退/前进缓存还原页面时，PerformanceObserver API 不会报告 `first-input` 条目，但是在这种情况下，应该对 FID 进行测量，因为用户将它们视为不同的页面访问。
- PerformanceObserver API 不会报告 `iframe` 中发生的输入，但是要正确衡量 FID，您应该考虑这些输入。子 `iframe` 可以使用 API​​ 将其 `first-input` 条目报告给父 `iframe` 进行聚合。
- 无法测量在 cross-orgin iframe 的 FID

基于 `web-vitals`：

```js
import { getFID } from 'web-vitals';

// Measure and log FID as soon as it's available.
getFID(console.log);
```

基于 `PerformanceObserver`：

```js
// 参考 https://github.com/GoogleChrome/web-vitals/blob/master/src/getFID.ts
const onHidden = (cb, once) => {
  const onVisibilityChange = (event: Event) => {
    if (document.visibilityState === 'hidden') {
      cb(event);
      if (once) {
        removeEventListener('visibilitychange', onVisibilityChange, true);
      }
    }
  };
  addEventListener('visibilitychange', onVisibilityChange, true);
};

let firstHiddenTime = -1;

const initHiddenTime = () => {
  return document.visibilityState === 'hidden' ? 0 : Infinity;
};

const trackChanges = () => {
  // Update the time if/when the document becomes hidden.
  onHidden(({ timeStamp }) => {
    firstHiddenTime = timeStamp;
  }, true);
};

// 浏览器回退需要重新统计
const onBFCacheRestore = (cb) => {
  addEventListener(
    'pageshow',
    (event) => {
      if (event.persisted) {
        cb(event);
      }
    },
    true,
  );
};

const getFirstHidden = () => {
  if (firstHiddenTime < 0) {
    if (self.__WEB_VITALS_POLYFILL__) {
      firstHiddenTime = self.webVitals.firstHiddenTime;
      if (firstHiddenTime === Infinity) {
        trackChanges();
      }
    } else {
      firstHiddenTime = initHiddenTime();
      trackChanges();
    }

    // Reset the time on bfcache restores.
    onBFCacheRestore(() => {
      setTimeout(() => {
        firstHiddenTime = initHiddenTime();
        trackChanges();
      }, 0);
    });
  }
  return {
    get timeStamp() {
      return firstHiddenTime;
    },
  };
};

const finalMetrics = new WeakSet();

const bindReporter = (callback, metric, reportAllChanges) => {
  let prevValue;
  return () => {
    if (metric.value >= 0) {
      if (
        reportAllChanges ||
        finalMetrics.has(metric) ||
        document.visibilityState === 'hidden'
      ) {
        metric.delta = metric.value - (prevValue || 0);

        if (metric.delta || prevValue === undefined) {
          prevValue = metric.value;
          callback(metric);
        }
      }
    }
  };
};

const initMetric = (name, value) => {
  return {
    name,
    value: typeof value === 'undefined' ? -1 : 0,
    delta: 0,
    entries: [],
    // id: generateUniqueID()
  };
};

const getFID = (onReport, reportAllChanges) => {
  const firstHidden = getFirstHidden();
  let metric = initMetric('FID');
  let report;

  const entryHandler = (entry) => {
    // Only report if the page wasn't hidden prior to the first input.
    if (entry.startTime < firstHidden.timeStamp) {
      metric.value = entry.processingStart - entry.startTime;
      metric.entries.push(entry);
      finalMetrics.add(metric);
      report();
    }
  };

  const po = new PerformanceObserver((l) => l.getEntries().map(entryHandler));
  po.observe({ type: 'first-input', buffered: true });

  report = bindReporter(onReport, metric, reportAllChanges);

  if (po) {
    onHidden(() => {
      po.takeRecords().map(entryHandler);
      po.disconnect();
    }, true);
  }

  if (self.__WEB_VITALS_POLYFILL__) {
    // Prefer the native implementation if available,
    if (!po) {
      window.webVitals.firstInputPolyfill(entryHandler);
    }
    onBFCacheRestore(() => {
      metric = initMetric('FID');
      report = bindReporter(onReport, metric, reportAllChanges);
      window.webVitals.resetFirstInputPolyfill();
      window.webVitals.firstInputPolyfill(entryHandler);
    });
  } else {
    // Only monitor bfcache restores if the browser supports FID natively.
    if (po) {
      onBFCacheRestore(() => {
        metric = initMetric('FID');
        report = bindReporter(onReport, metric, reportAllChanges);
        resetFirstInputPolyfill();
        firstInputPolyfill(entryHandler);
      });
    }
  }
};
```

### 提升 FID

改进 FID 与改进实验室指标 **总阻塞时间（Total Blocking Time，TBT）** 相同。

- [减少第三方代码的影响](https://web.dev/third-party-summary/)
- [减少 JavaScript 执行时间](https://web.dev/bootup-time/)
- [减少主线程工作](https://web.dev/mainthread-work-breakdown/)
- [保持低请求数量和小传输大小](https://web.dev/resource-summary/)

## TBT

**最大阻塞时长（Total Blocking Time， TBT）** 是衡量加载响应时长的重要指标。

TBT 度量标准度量了 FCP 和 TTI 之间的总时间，在该时间中，主线程长时间阻塞无法进行输入响应。
因此 TBT 有助于量化页面在变为可靠交互之前的非交互性的严重程度。

只要出现在主线程上耗时超过 **50ms** 的长任务，则视为主线程阻塞。
而 TBT 为 FCP 和 TTI 之间所有长任务**阻塞时间（大于 50ms 的部分）**之和。

![主线程上的任务时间表](https://webdev.imgix.net/tbt/tbt-all-tasks.svg)

> 图片来自于 [Total Blocking Time (TBT)](https://web.dev/tbt/)

上图中 `TBT = (250 - 50) + (90 - 50) + (155 - 50) = 345ms`

如果主线程至少有五秒钟没有执行长任务，则 TTI 认为页面是 "reliably interactive"。
这意味着，在 10s 均匀内分布的三个 51ms 任务可以将 TTI 推迟到 10s????

### TBT 指标

在一般的移动硬件上进行测试时，站点应努力使总阻止时间小于 300ms。

- fast: 0-300ms
- moderate: 300-600ms
- slow: >600ms

### 提升 TBT

- [减少第三方代码的影响](https://web.dev/third-party-summary/)
- [减少 JavaScript 执行时间](https://web.dev/bootup-time/)
- [减少主线程工作](https://web.dev/mainthread-work-breakdown/)
- [保持低请求数量和小传输大小](https://web.dev/resource-summary/)

## CLS

**累计布局偏移（Cumulative Layout Shift， CLS）**是衡量用户视觉稳定性的重要指标。

页面内容的意外移动通常是由于异步加载资源或将 DOM 元素动态添加到现有内容上方的页面而发生的。
通常是由于尺寸未知的图像或视频、呈现比其后备更大或更小的字体或者是动态调整自身大小的第三方广告或小部件。

CLS 会度量页面整个生命周期中发生的每个**意外的布局偏移（unexpected layout shift）**的**布局移位分数（layout shift scores）**的总和。

每当可见元素从一个渲染帧到下一帧改变其位置时，都会发生布局偏移。

### CLS 指标

- Good: < 0.1
- Need Improvement: 0.1-0.25
- Poor: > 0.25

### 如何计算 CLS

```
布局移位分数(layout shift score) = 影响分数(impact fraction) * 距离分数(distance fraction)
```

**影响分数：**

所有不稳定元素前一帧和当前帧的**可见区域**的并集的比例。

**距离分数：**

不稳定元素相对于视口移动的距离，多个不稳定元素取其最大移动距离。

例如，第一帧 `<img width="100vw" height="50vh"/>`，第二帧 `<img width="100vw" height="50vh" marginTop="25vh" />`。

该元素的

```
影响分数 = 0.75
距离分数 = 0.25
布局移位分数 = 0.75 * 0.25 = 0.1875
```

CLS 仅计算视图可视！

例如，第一帧 `<img width="100vw" height="50vh" marginTop="50vh"/>`，第二帧 `<img width="100vw" height="50vh" marginTop="75vh" />`。

该元素的

```
影响分数 = 0.5
距离分数 = 0.25
布局移位分数 = 0.5 * 0.25 = 0.125
```

影响分数为所有不稳定影响元素影响区域占视图比例！
距离分数为所有不稳定影响元素最大移动距离！

例如：

第一帧 `<img width="100vw" height="10vh" />`, `<img width="100vw" height="10vh" marginTop="30vh" />`
第一帧 `<img width="100vw" height="10vh" marginTop="10vh" />`, `<img width="100vw" height="10vh" marginTop="50vh" />`

```
影响分数 = 0.2 + 0.3 = 0.5
距离分数 = Math.max(0.1 - 0, 0.5 - 0.3) = 0.2
布局移位分数 = 0.5 * 0.2 = 0.1
```

_如果距离分数为 0，布局移位分数也为 0 ？_

例如，第一帧 `<img width="100vw" height="0vh" />`，第二帧 `<img width="100vw" height="50vh" />`。

PS：
最初，仅基于影响分数来计算 CLS 的偏移分数。
加入距离分数，是为了避免过于惩罚其中大元件由少量偏移的情况下。

### 预期与意外布局的变化

并非所有的布局转换都是不好的。
实际上，许多动态 Web 应用程序经常更改页面上元素的开始位置。

在用户输入的 500ms 内发生的布局转换将设置 `hadRecentInput` 标志，因此可以将其从计算中排除。

> 注意： 该 `hadRecentInput` 标志仅对离散输入事件（如敲击，单击或按键）有效。
> 滚动，拖动或捏和缩放手势之类的连续交互不被视为“最近输入”。

**用户启初始化布局偏移 (User-initiated layout shifts)**

仅当用户不期望布局转换时，它才是不好的。
另一方面，响应于用户交互（单击链接，按下按钮，在搜索框中键入内容等）而发生的布局偏移通常很好。
只要该偏移发生在与交互关系足够近的位置即可，该位置对于用户是清晰大。

**动画和过渡**

动画和过渡效果是一种在不引起用户惊讶的情况下更新页面内容的好方法。
在页面上突然发生意外变化的内容几乎总是会产生不良的用户体验。
但是，内容逐渐自然地从一个位置移动到另一个位置通常可以帮助用户更好地了解发生了什么，并在状态更改之间进行指导。

`CSStransform` 属性使您可以为元素设置动画，而不会触发布局转换：

- 使用 `transform: scale()` 替代 `height` 和 `width` 的变化。
- 使用 `transform: translate()` 来代替左右移动的元件，避免改变 `top`，`right`，`bottom`，或`left` 属性。

### 通过 JavaScript 测量 CLS

基于 `web-vitals` 测量 CLS

```js
import { getCLS } from 'web-vitals';

getCLS(console.log);
```

TODO 参考 https://web.dev/cls/#measure-cls-in-javascript

### 提升 CLS

- 请务必在图片和视频元素上包含 `size` 属性，否则请使用 CSS 宽高比框保留所需的空间。
  这种方法可确保浏览器在加载图像时可以在文档中分配正确的空间量。
  请注意，您也可以使用 unsize-media 功能部件策略 在支持功能部件策略的浏览器中强制执行此行为。
- 除非响应用户交互，否则切勿在现有内容上方插入内容。这样可以确保可以预期发生任何版式移位。
- 首选将转换动画替换为触发布局更改的属性动画。对过渡进行动画处理，以提供状态与状态之间的上下文和连续性。

## INP

INP(Interaction to Next Paint) 是一种评估响应性能的测试指标。`web-vitals` 已对其支持。
INP 记录整个页面生命周期中所有交互的延迟。
这些交互的最高值为该页面的 INP。

`Math.max(INP0, INP1, ..., INPn)`

> 对于少于 50 次交互的页面，INP 是延迟最差的交互。
> 对于过多交互的页面。INP 通常是 98% ???
> 对于某些网页，用户会存在更多交互（例如文本编辑器）
> 此时对最差页面进行抽样会产生误导，通过高百分比，可以评估大多数交互是否得到及时响应。

交互（interaction）是在同一逻辑下，用户触发的一组输入事件。

单个交互的延迟作为交互一部分的任何事件的单个最长持续时间组成。
交互的持续时间（duration）是从用户于页面交互的点开始测量，直到在所有关联的时间处理程序都执行后呈现下一帧位置。

```
交互持续时间(duration)= 输入延迟(input delay) + 执行时间(processing time) + 呈现延迟(presentation delay)
```

![交互的完整过程](https://web-dev.imgix.net/image/jL3OLOhcWUQDnR4XjewLBx4e3PC3/Ng0j5yaGYZX9Bm3VQ70c.svg)

### INP 指标

- Good: < 300ms
- Need Improvement: 200ms - 500ms
- Poor: > 500ms

> INP 仍处于实验性的指标，阈值可能会时间改变。
> 需要关注 [CHANGELOG](http://bit.ly/chrome-speed-metrics-changelog)

### INP 与 FID 的差异

FID 是加载阶段的响应指标。

INP 是用户从加载到离开页面可能发生的整个交互范围。
INP 更类似于 CLS

> 如果用户未进行交互，不会产生 INP 值

### 如何计算 INP

// TODO

### 如何提高 INP

#### 加载阶段

> 根据 HTTP 存档，总阻塞时间 (TBT)与 INP 的相关性是 FID 的两倍。

- 删除未使用的代码。
- 代码拆分
- 懒加载的非必要的慢速第三方 JavaScript 。
- 使用性能分析器查找可以优化的长任务。
- 确保您在 JavaScript 完成后不会对浏览器渲染提出太多要求——即大型组件树重新渲染、大型图像解码、太多繁重的 css 效果等等。

#### 加载后

- 使用 `postTaskAPI` 适当地确定任务的优先级。
- 当浏览器空闲时安排非必要的工作 `requestIdleCallback`。
- 使用性能分析器来评估离散交互（例如，切换移动导航菜单）并找到需要优化的长任务。
- 审核页面中的第三方 JavaScript 查看其是否影响页面响应能力。

## Reference

- [Web Vitals](https://web.dev/vitals)
- [User-centric performance metrics](https://web.dev/user-centric-performance-metrics/)
- [Defining the Core Web Vitals metrics thresholds](https://web.dev/defining-core-web-vitals-thresholds/)
- [Time to Interactive (TTI)](https://web.dev/tti/)
- [First Input Delay (FID)](https://web.dev/fid/)
- [Total Blocking Time (TBT)](https://web.dev/tbt/)
- [Interaction to Next Paint (INP)](https://web.dev/inp/#why-not-the-worst-interaction-latency)
