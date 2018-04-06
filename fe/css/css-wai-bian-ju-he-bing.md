# CSS 外边距合并

> 外边距合并指的是，当两个垂直外边距相遇时，它们将只形成一个外边距，合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。

只有普通文档流中块级元素的垂直外边距才会发生外边距合并，行内元素、 浮动框以及绝对定位之间的外边距不会合并。

当一个元素出现在另一个元素上面时，第一个元素的下边距会和第二个元素的上边距发生合并

![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_1.gif)



当一个元素包含在另一个元素中，倘若没有内边距或边框将两个元素的外边距分隔开，那么，这两个元素的上边距或下边距也会发生合并

![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_2.gif)

假设有一个空元素，它有外边距，但是没有边框以及填充。这种情况下，上外边距，与下外边距碰到了一起，此时，该元素的上外边距和下外边距也会触发外边距合并。

![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_3.gif)

 倘若此时遇到另外一个元素的上边距或下边距，还会继续发生合并

![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_4.gif)

