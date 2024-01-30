# CSS Border-Image

CSS 的`border-image`属性允许在元素的边框上绘制图像，这使得绘制复杂的外观组件变得更加简单。

> 使用`border-image`时，将会替换掉`border-style`属性所设置的边框样式。

## `border-image` 语法

```css
border-image: <'border-image-source'> || <'border-image-slice'> [ / <'border-image-width'> | / <'border-image-width'>? / <'border-image-outset'> ]? || <'border-image-repeat'>
```

## 示例

[Codepen 在线演示](https://codepen.io/xyy94813/pen/aKyNdL?editors=1100)

```html
<div id="bitmap">文本内容</div>
```

```css
#bitmap { 
  border: 20px solid transparent;
  padding: 0px;
  border-image: url("https://mdn.mozillademos.org/files/4127/border.png") 30;
  /* 
  border-image-source: url("https://mdn.mozillademos.org/files/4127/border.png");
  border-image-slice: 30;
  border-image-width: 1;
  border-image-outset: 0;
  border-image-repeat: stretch; 
  */
}
```

## border-image 的组合属性

`border-image`是以下几个属性组合的简写方式：

* `border-image-source`
* `border-image-slice`
* `border-image-width`
* `border-image-outset`
* `border-image-repeat`

### border-image-source

`border-image-source` 属性被用于声明元素的边框图片的资源。

默认值为：`none`，此时不会使用图片边框

```css
border-image-source: none; /* 默认值 */
border-image-source: url('https://developer.mozilla.org/media/examples/border-diamonds.png');
border-image-source: repeating-linear-gradient(45deg, transparent, #4d9f0c 20px);
```

### border-image-slice

`border-image-slice` 属性被用于切割资源图片的区域，然后将其动态的应用到最终的边框图片。

`border-image-slice`会将资源分割成 9 个区域：四个角，四边以及中心区域

![](https://developer.mozilla.org/files/3814/border-image-slice.png)

* 区域 1-4 为角区域（corner region）。 每一个都用一次来形成最终边界图像的角点。
* 区域 5-8 边区域（edge region）。在最终的边框图像中重复，缩放或修改它们以匹配元素的尺寸。
* 区域 9 为中心区域（ middle region）。它在默认情况下会被丢弃，但如果设置了关键字`fill`，则会将其用作背景图像。

```css
/* 所有的边 */
border-image-slice: 30%; 

/* 垂直方向 | 水平方向 */
border-image-slice: 10% 30%;

/* 顶部 | 垂直方向 | 底部 */
border-image-slice: 30 30% 45;

/* 上 右 下 左 */
border-image-slice: 7 12 14 5; 

/* 使用fill（fill可以放在任意位置） */
border-image-slice: 10% fill 7 12;

/* Global values */
border-image-slice: inherit;
border-image-slice: initial;
border-image-slice: unset;
```

### border-image-width

`border-image-width`定义图像边框宽度。假如border-image-width大于已指定的`border-width`，那么它将向内部（padding/content）扩展。

```css
border-image-width: all                        /* One-value syntax */       
/* E.g. border-image-width: 3; */
border-image-width: vertical horizontal        /* Two-value syntax */       
/* E.g. border-image-width: 2em 3em; */
border-image-width: top horizontal bottom      /* Three-value syntax */     
/* E.g. border-image-width: 5% 15% 10%; */
border-image-width: top right bottom left      /* Four-value syntax */      
/* E.g. border-image-width: 5% 2em 10% auto; */

border-image-width: inherit
```

### border-image-outset

`border-image-outset`属性定义边框图像可超出边框盒的大小。

```css
/* border-image-outset: sides */
border-image-outset: 30%;

/* border-image-outset:垂直 水平 */
border-image-outset: 10% 30%;

/* border-image-outset: 顶 水平 底 */
border-image-outset: 30px 30% 45px;

/* border-image-outset:顶 右 底 左  */
border-image-outset: 7px 12px 14px 5px;

border-image-repeat: inherit;
```

### border-image-repeat

`border-image-repeat`定义图片如何填充边框。或为单个值，设置所有的边框；或为两个值，分别设置水平与垂直的边框。

```css
border-image-repeat: type                      /* One-value syntax */       
/* E.g. border-image-value: stretch; */
border-image-repeat: horizontal vertical       /* Two-value syntax */       
/* E.g. border-image-width: round space; */

border-image-repeat: inherit
```



