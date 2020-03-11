# CSS Float

CSS `float` 属性使得元素浮动。

|        |                            |
| ------ | -------------------------- |
| 值     | left、right、none、inherit |
| 初始值 | none                       |
| 应用于 | 所有元素                   |
| 继承性 | 无                         |
| 计算值 | 根据指定确定               |

```css
.ad {
  float: right;
}
```

```html
<img src="xxx.png" class="ad" />
```

如果对非替换元素浮动，必须为其声明 `width`。

```css
div.ad,
span.ad {
  width: 5em;
}
```

```html
<div class="ad">float</div>
<span class="ad">float</span>
```

## 浮动元素

当给任意元素增加 `float` 属性后，会使其成为**浮动元素**。
浮动元素会以某种方式从正常的文档流中移除。

### 环绕浮动元素

一个元素浮动时，其它的内容会“环绕”该元素。

```css
p.aside {
  float: right;
  margin: 10em;
}
```

```html
<p class="aside">
  float float float float float float float float float float float float float
  float float float float float float float float float float float
</p>
<p>
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
</p>
```

此外，浮动元素周围的外边距不会合并。

可以对之前的 Demo 的 CSS 调整进行验证。

```css
p {
  margin: 5em;
}
p.aside {
  float: right;
  margin: 10em;
}
```

### 浮动元素的包含块

浮动元素会生成一个块级框，无论该元素本身是什么。
也就是说，对浮动元素声明 `display: block;` 是无意义的.

浮动元素的 **包含块（containning block）** 是其最近的块级祖先元素。

```html
<div class="ancestor-0">
  <div class="ancestor-1">
    <span class="ancestor-2">
      <img style="float: left;" />
    </span>
  </div>
</div>
```

对于这个例子，浮动元素 `<img />` 的包含块是 `class="ancestor-1"` 的 `div`。

### 浮动元素的规则

1. 浮动元素的左外边界不能超出其包含块的左內边界，右外边界不能超出其包含块的右內边界。
2. 浮动元素的左外边界必须是包含块中之前出现的左浮动元素的右外边界，右外边界必须是包含块中之前出现的右浮动元素的左外边界。除非，后出现的浮动元素在先出现的浮动元素的底端。
3. 左浮动元素的右外边界不会在其右边浮动的左外边界的右边。一个右浮动元素左外边界不会在其左边任何左浮动元素的右外边界的左边。
4. 浮动元素的顶端不能比其父元素的内顶端更高。
5. 浮动元素的顶端不能比之前所有浮动元素或块级元素的顶端更高。
6. 如果源文档中一个浮动元素之前出现另一个元素，浮动元素的顶端不能比包含该元素所生成框的仍和行框的顶端要高。
7. 左浮动元素的左边有另一个浮动元素，前者的右边界不能在其包含块的右边界的右边。右浮动元素的右边有另一个浮动元素，前者的左边界不能在其包含块的左边界的左边边。
8. 浮动元素必须尽可能高地放置。
9. 左浮动元素必须向左尽可能远，右浮动元素必须向右尽可能的远。

> 简单的总结就是浮动元素不能超出其包含块，且尽可能的贴着包含块的内边界“漂浮”。

### 浮动元素与正常流重叠

如果一个浮动元素在内容流过的边上有负外边距，最终导致浮动元素与正常流重叠。
对此，CSS 2.1 指出以下规则：

- 行内框与一个浮动元素重叠时，其边框、背景和内容都在该浮动元素“之上”显示。
- 块框与一个浮动元素重叠时，其边框和背景在该浮动元素“之下”显示，而内容在浮动元素“之上”显示。

## 清除浮动

利用 `clear` 属性可以确保某个元素不与浮动元素在同一行。

|        |                                  |
| ------ | -------------------------------- |
| 值     | left、right、both、none、inherit |
| 初始值 | none                             |
| 应用于 | 块级元素                         |
| 继承性 | 无                               |
| 计算值 | 根据指定确定                     |

```css
h3 {
  clear: both;
  background: rgba(0, 0, 0, 0.6);
}
```

```html
<p>
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat
</p>
<img src="XXX.png" alt="img1" style="float: left;" />
<p>
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat
</p>
<img src="XXX.png" alt="img2" style="float: right;" />
<h3>
  h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3 h3
  h3 h3 h3 h3 h3 h3 h3 h3 h3 h3
</h3>
<p>
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat repeat
  repeat repeat repeat repeat repeat repeat repeat
</p>
```
