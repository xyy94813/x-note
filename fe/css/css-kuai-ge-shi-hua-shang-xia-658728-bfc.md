# CSS 块格式化上下文\(BFC\)

> **块格式化上下文（Block Formatting Context，BFC）**是 Web 页面的可视化 CSS 渲染的一部分，是布局过程中生成块级盒子的区域，也是浮动元素与其他元素的交互限定区域。

浮动元素和绝对定位元素，非块级盒子的块级容器（例如 `inline-blocks`，`table-cells`和`table-captions`），以及`overflow`值不为`visiable`的块级盒子，都会为他们的内容创建新的BFC（块级格式上下文）。创建了块格式上下文的元素中的所有内容都会被包含到该 BFC 中。

浮动定位和清除浮动时只会应用于同一个 BFC 内的元素。

浮动不会影响其它 BFC 中元素的布局，而清除浮动只能清除同一 BFC 中在它前面的元素的浮动。

外边距折叠（Margin collapsing）也只会发生在属于同一BFC的块级元素之间。



## 参考

* [块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)



