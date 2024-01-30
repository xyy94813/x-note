# 页面布局之单列布局

单列布局的 CSS 实现方式

## 水平居中

使用`inline-block`和`text-align`实现

> 兼容性好，但是需要同时设置子元素和父元素

```css
.parent {
    text-align: center;
}
.child {
    display: inline-block;
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

使用`margin`实现

> 兼容性好，但是只适用于子元素指定来宽度的情况

```css
.child {
    width: 200px;
    margin: 0 auto;
}
```

```html
<div>
    <div class="child"></div>
</div>
```

使用`table`+`margin`实现

> 需要对 IE6、IE7 做兼容

```css
.child {
    display: table;
    margin: 0 auto;
}
```

```html
<div>
    <div class="child"></div>
</div>
```

使用绝对定位+`transform`实现

> 需要对`transform`做兼容，IE9 以上才可用

```css
.parent { 
    position: relative; 
}
.child {
    position: absolute;
    left: 50%;
    transform: translate(-50%);
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

使用`flex`实现

> 需要对`flex`进行兼容，大面积布局可能会影响效率

```css
.parent { 
    position: flex;
    justify-content: center;
}
```

```html
<div class="parent">
    <div></div>
</div>
```

## 垂直居中

使用`vertical-align`实现

> 只有一个元素属于`inline`或是`inline-block`（`table-cell`也可以理解为`inline-block`），  
> 其身上的`vertical-align`属性才会起作用。

```css
.parent {
    display: table-cell;
    vertical-align: middle;
    height: 20px;
}
```

或是

```css
.parent { 
    display: inline-block;
    vertical-align: middle;
    line-height: 20px;
}
```

```html
<div class="parent">
    <div></div>
</div>
```

使用绝对定位实现

```css
.parent {
    position: relative;
    height: 200px;
}
.child {
    position: absolute;
    top: 50%;
    transform: translate(0,-50%);
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

使用`flex`实现

```css
.parent {
    display: flex;
    align-items: center;
    height: 200px;
}
```

```html
<div class="parent">
    <div></div>
</div>
```

## 水平垂直全部居中

使用`vertical-align`，`text-align`，`inline-block`实现

```css
.parent {
    display: table-cell;
    vertical-align: middle;
    text-align: center; 
    height: 200px;
}
.child {
    display: inline-block;
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

使用绝对定位实现

```css
.parent {
    position: relative;
    height: 200px;
}
.child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

```html
<div class="parent">
    <div class="child"></div>
</div>
```

使用`flex`实现

```css
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 200px;
}
```

```html
<div class="parent">
    <div></div>
</div>
```



