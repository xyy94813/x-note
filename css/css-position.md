# CSS Position

通过使用 `position` 属性，可以选择 4 种不同类型的定位，这会影响元素框的生成。

|        |                                                                |
| ------ | -------------------------------------------------------------- |
| 值：   | `static`、`relative`、`absolute`、`fixed`、`sticky`、`inherit` |
| 初始值 | `static`                                                       |
| 应用于 | 所有元素                                                       |
| 继承性 | 无                                                             |
| 计算值 | 根据指定确定                                                   |

## Position 的值含义

### static

元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分，行内元素则会创建一个或多个行框，置于父元素中。

### relative

元素框偏移某个距离。元素仍保持其未定位前的形状，它原本所占的空间仍保留。

### absolute

元素框从文档流中删除，并相对于其包含块定位，包含块可能是文档中的另一个元素或者是初始包含块。元素定位后生一个块级框，而不论原来在正常流中生成何种类型的框。

### fixed

类似于 `absolute`，不同的是其包含块必定是视窗本身。

### sticky

元素被留在正常的流中，直到触发它的粘性的条件出现，这时它被从正常的流中移除，但是它在正常流中的原始空间被保留下来。然后，它的行为就像绝对定位相对于其包含块。一旦不再满足强制粘性的条件，元素将返回到其原始空间中的正常流。

## 被定位的元素的包含块。

对于被定位的元素，其包含块要比浮动元素要来的复杂。

CSS2.1 定义以下行为：

- “根元素”的包含块由用户代理建立。在 HTML 中，根元素就是 `<html />` 元素，小部分浏览器使用 `<body />` 作为根元素。大多数浏览器中，初始包含块是一个视窗大小的矩形。
- 对于非根元素，如果其 `position` 为 `relative` 或 `static`，包含块则由最近的块级框、表单元格或行内块祖先框的内容边界构成。
- 对于非根元素，如果其 `position` 为 `absolute`，包含块设置为最近的 `position` **不是** `static` 的祖先元素。

  - 如果这个祖先是块级元素，包含块则设置为该元素的內边距边界。
  - 如果这个祖先是行内元素，包含块则设置为该祖先元素的内容边界。
  - 如果没有祖先，元素的包含块定义为初始包含块。

- 当涉及到粘贴定位元素时，包含块规则有一个有趣的变体，即一个矩形是与被称为 `sticky-constraint` 的包含块相关的。

> 元素可以定位到其包含块的外部，类似于浮动元素的父边距。“包含块（containing block）” 应该称之为 “定位上下文（Position context）”。不过 CSS 规范中使用的是 “containing block”。 -- 《CSS 权威指南》

## 偏移

`relative`、`absolute`、`fixed`、`sticky` 均支持使用 `top`、`right`、`bottom`、`left` 来描述被定位的元素相对于其包含块的便宜。

|        | top、right、bottom、left                                                                       |
| ------ | ---------------------------------------------------------------------------------------------- |
| 值：   | `<length>`、`<percebtage>`、`auto`、`inherit`                                                  |
| 初始值 | `auto`                                                                                         |
| 应用于 | Positioned Element                                                                             |
| 继承性 | 无                                                                                             |
| 百分数 | 对于 top 和 bottom，相对于包含块的高度；对于 right 和 left 相对于包含块的宽度                  |
| 计算值 | 对于 static 元素为 auto；对于长度值，是相应的绝对长度；对于百分数，则为制定的值；否则为 auto。 |

## 详解绝对定位为

当元素被绝对定位时，它将完全从文档流中删除。
然后，它相对于其包含的块进行定位，并且使用偏移属性(`top`、`left` 等)放置其边缘边缘。
定位元素不围绕其他元素的内容流动，它们的内容也不围绕定位元素流动。
这意味着一个绝对定位的元素可能与其他元素重叠或被它们重叠。

当一个元素处于绝对位置时，它将为其后代元素建立一个包含块。

```css
div {
  position: relative;
  width: 100%;
  height: 10em;
  border: 1px solid;
  background: #eee;
}
div.a {
  position: absolute;
  top: 0;
  right: 0;
  width: 15em;
  height: 100%;
  margin-left: auto;
  background: #ccc;
}
div.b {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 10em;
  height: 50%;
  margin-top: auto;
  background: #aaa;
}
```

```html
<div>
  <div class="a">
    absolutely positioned element A
    <div class="b">
      absolutely positioned element B
    </div>
  </div>
  containing block
</div>
```

只要绝对定位元素不是固定定位元素的后代，如果文档可滚动，定位元素会随着它滚动。
**因为元素最终会相对于正常流的某一部分定位。**

### 定位元素的宽度和高度

通常定位元素也使用 `width` 和 `height` 指定高度。
但是，如果同时使用 `top`、`right`、`bottom`、`left` 来秒速元素的 4 个边的放置位置，那么元素的高宽都由偏移量隐含的确定。

考虑到一个定位元素的宽度和水平放置。可以表示为一个等式：

```
containing_block_width =
    left
    + margin-left
    + border-left-width
    + padding-left
    + width
    + padding-right
    + border-right-width
    + margin-right
    + right;
```

### 绝对定位元素的自动偏移

当绝对定位一个元素，除 `bottom` 外某个便宜属性设置为 `auto` 会存在一种特殊的行为。

```html
<p>
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat
  <span
    style="position: absolute; top: auto;
 left: 0;"
    >[4]</span
  >
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat
</p>
```

对于以上例子，定位元素的顶端会相对于其未定位前本来的顶端位置对齐。

> CSS2.1 指出：
> 元素的 “static position” 的大致含义是 -- 元素在正常流中原本的位置。

### 绝对定位非替换元素

对于非替换绝对定位元素。
如果设置左右外边距为 `auto` 的同时设置 `left` 和 `right`，元素仍会居中显示。

如果只有左外边距为 `auto`，会忽略 `margin-left` 的值。

一般来说，如果只有一个属性设置为 auto，就会修改这个属性来满足上诉等式。

### 绝对定位替换元素

确定替换元素的位置和大小时，所涉及的行为遵循以下规则：

1. 如果 `width` 设置为 `auto`， `width` 的实际使用值由元素内容的固有宽度决定。
2. 在从左向右的读的语言中，如果 `left` 值为 `auto`， 要把 `auto` 替换为静态位置。
   在从右向左的读的语言中，如果 `right` 值为 `auto`， 要把 `auto` 替换为静态位置。
3. 如果 `left` 或 `right` 仍为 `auto`， 则将 `margin-left` 或 `margin-right` 的 `auto` 替换为 `0`。
4. 在此之后，如果只剩下一个 `auto` 值，则将其修改为等于等式的余下部分，以满足等式。

## Z 轴上的放置

`z-index` 属性就是为了解决问题：当两个元素试图放在同一个位置上，其中一个必将覆盖另一个。
利用 `z-index` 可以改变元素相互覆盖的顺序。

|        |                                |
| ------ | ------------------------------ |
| 值：   | `<integer>`、`auto`、`inherit` |
| 初始值 | `auto`                         |
| 应用于 | 定位元素                       |
| 继承性 | 无                             |
| 计算值 | 根据指定确定                   |

一旦一个元素指定了 `z-index` 的值，不是 `auto`，该元素就会建立自己的局部叠放上下文。
这意味着，元素的所有后代相对于该元素都有其自己的叠放顺序。

> Note: z-index 不相对于文档。

```html
<div style="position: fixed; left: 0; top: 0; width: 100%; background: #FFF;">
  back
</div>
<div
  style="position: fixed; left: 0; top: 0; width: 50%; background: #000; z-index: 10;"
>
  <span style="z-index: -1;">
    front
  </span>
</div>
```

CSS2.1 规范对默认值 auto 有以下说明：

> 当前叠放上下文中生成框的栈层次与其父框的层次相同时，这个框不会建立新的局部层叠上下文。
