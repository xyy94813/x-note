# CSS 盒模型

所有文档元素都会生成一个名为 element box 的矩形框，它描述了一个元素在文档布局中所占的空间量。

![CSS Box Model](https://gdut_yy.gitee.io/doc-csstdg4/figures/ch8/fg8-1.png)

## CSS 标准盒模型计算公式

```
CSS box width = margin-left + border-left + padding-left + width(content width) + padding-right + border-right + margin-right;

CSS box height = margin-top + border-top + padding-top + width(content width) + padding-bottom + border-bottom + margin-bottom;
```

这也是为什么负外边距使得元素盒模型宽度增加的原因。

DEMO：

```html
<style>
  .box {
    margin: 10px 20px 30px 40px;
    padding: 10px 20px 30px 40px;
    width: 200px;
    height: 300px;
    border: 8px solid #000;
    background: green;
  }
</style>
<div class="box" id="box">box</div>
<script>
  const $box = document.querySelector("#box");
  const $boxRect = $box.getBoundingClientRect();
  console.log($boxRect.width === 200 + 20 + 40 + 8 * 2); // true
  console.log($boxRect.height === 300 + 10 + 30 + 8 * 2); // true
</script>
```

> `$Element.getBoundingClientRect()` API 可以获得 CSS 盒模型的在页面上的具体信息。

## IE 盒模型

由于历史原因，早期 IE 实现的 CSS Box Model 与上图不同。

为了方便区分，通常分别称为**标准盒模型**和**IE 盒模型**。

以下是常见的两个模型的对比图。

![标准盒模型](https://images2015.cnblogs.com/blog/793040/201511/793040-20151130140140858-2462296.jpg)

![IE 盒模型](https://images2015.cnblogs.com/blog/793040/201511/793040-20151130140151233-1527652250.jpg)

与标准盒模型不同的是：

1. IE 盒模型中，样式的 `width` 和 `height` 直接决定了盒模型的最终宽度和高度。
2. `border` 属于内容区的一部分。

> 实际内容宽度 = `width` - `border-width` - `padding`;

```html
<style>
  .box {
    margin: 10px 20px 30px 40px;
    padding: 10px 20px 30px 40px;
    width: 200px;
    height: 300px;
    border: 8px solid rgba(0, 0, 0, 0.3);
    background: green;
  }
</style>
<div class="box" id="box">box</div>
<script>
  const $box = document.querySelector("#box");
  const $boxRect = $box.getBoundingClientRect();
  console.log($boxRect.width === 200); // true
  console.log($boxRect.height === 300); // true
</script>
```

## box-sizing

可以通过 CSS 属性 `box-sizing` 切换盒模型的计算公式。

- `content-box`。 标准盒模型
- `border-box`。IE 盒模型
