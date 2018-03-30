# 页面布局之多列布局

常见的多列布局：

* 左列定宽，右列自适应
* 右列定宽，左列自适应
* 一列不定宽，一列自适应
* 两侧定宽，中栏自适应
* 多列等分布局

### 左列定宽，右列自适应

使用`float`和`margin`实现

```css
.left {
  float: left;
  width: 100px;
}
.right {
  margin-left: 100px;
}
```

```html
<div class="float-and-margin">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`float`和`margin(fix)`实现。可以优先渲染右边部分。

```css
.left {
    width: 100px; 
    float: left; 
}
.right-fix {
    width: 100%;
    margin-left: -100px;
    float: right;
}
.right {
    margin-left: 100px;
}
```

```html
<div class="float-and-margin-fix">
    <div class="right-fix">
        <div class="right"></div>
    </div>
    <div class="left"></div>
</div>
```

使用`float`和`overflow`实现。

> 左侧 left 设置`float`脱离文档流，右侧利用`overflow: hidden`触发 **BFC模式**。  
> 浮动无法影响，隔离其它元素（IE6 不支持）

```css
.left {
  width: 100px;
  float: left;
}
.right {
  overflow: hidden;
}
```

```html
<div class="float-and-overflow">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`table`和`table-cell`实现

```css
.parent {
  display: table;
  table-layout: fixed;
  width: 100%;
}
.left {
  width: 100px;
  display: table-cell;
}
.right {
  display: table-cell;
}
```

```html
<div class="table-and-table-cell parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`flex`布局实现

> 利用右侧容器的`flex:1`，均分了剩余的宽度。而`align-items`默认值为`stretch`，保证左右高度一致。

```css
.parent {
  display: flex;
}
.left {
  width: 100px;
}
.right {
  flex: 1;
}
```

```html
<div class="flex-layout parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`grid`布局实现

```css
.parent {
  display: grid;
  grid-template-columns: 100px 1fr;
}
```

```html
<div class="grid-layout parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

### 右列定宽，左列自适应

使用`float`和`margin`实现

```css
.left {
  margin-right: -100px; 
  width: 100%;
  float: left;
}
.right {
  float: right; 
  width: 100px;
}
```

```html
<div class="float-and-margin">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`table`和`table-cell`实现

```css
.parent {
  display: table;
  table-layout: fixed;
  width: 100%;
}
.left {
  display: table-cell;
}
.right {
  width: 100px;
  display: table-cell;
}
```

```html
<div class="table-and-table-cell parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`flex`布局实现

```css
.parent {
  display: flex;
}
.left {
  flex: 1;
}
.right {
  width: 100px;
}
```

```html
<div class="flex-layout parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`grid`布局实现

```css
.parent {
  display: grid;
  grid-template-columns: 1fr 100px ;
}
```

```html
<div class="grid-layout parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

### 左列不定宽，右列自适应

使用`float`和`overflow`实现

```css
.left {
  float: left
}
.right {
  overflow: hidden;
}
```

```html
<div class="float-and-overflow">
    <div class="left">inline</div>
    <div class="right"></div>
</div>
```

使用`table`布局实现

```css
.parent {
  display: table;
  table-layout: fixed;
  width: 100%;
}
.left {
  width: 10%;
  display: table-cell;
}
.right {
  display: table-cell;
}
```

```html
<div class="tabel-layout parent">
    <div class="left"></div>
    <div class="right"></div>
</div>
```

使用`flex`布局实现

```css
.parent {
  display: flex;
}
.right {
  flex: 1;
}
```

### 两侧定宽，中栏自适应

使用`float` 和`margin`实现

```css
.parent {
  position: relative;
}
.left {
  width: 100px;
  float: left;
}
.right {
  width: 100px;
  float: right;
}
.center {
  margin-left: 100px;
  margin-right: 100px;
}
```

```html
<div class="float-and-margin parent">
    <div class="left"></div>
    <div class="right"></div>
    <div class="center"></div>
</div>
```

使用`table`布局实现

```css
.parent {
  width: 100%; 
  display: table; 
  table-layout: fixed;
}
.left {
  display: table-cell;
  width: 100px;
}
.right {
  display: table-cell;
  width: 100px;
}
.center {
  display: table-cell;
}
```

```html
<div class="table-layout parent">
    <div class="left"></div>
    <div class="center"></div>
    <div class="right"></div>
</div>
```

使用`flex`布局实现

```css
.parent {
  display: flex;
}
.left {
  width: 100px;
}
.right {
  width: 100px;
}
.center {
  flex: 1;
}
```

```html
<div class="flex-layout parent">
    <div class="left"></div>
    <div class="center"></div>
    <div class="right"></div>
</div>
```

使用`grid`布局实现

```css
.parent {
  display: grid;
  grid-template-columns: 100px 1fr 100px;
}
```

```html
<div class="grid-layout parent">
    <div class="left"></div>
    <div class="center"></div>
    <div class="right"></div>
</div>
```

圣杯布局

```css
.parent {
  overflow: hidden;
  padding: 0 100px;
}

.left {
  float: left;
  position: relative;
  left: -100px;
  width: 100px;
  margin-left: -100%;
}
.right {
  width: 100px;
  margin-left: -100px;
  float: left;
  right: -100px;
  position: relative;
}
.center {
  float: left;
  width: 100%;
}
```

```html
<div class="holy-grail-layout parent">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>
```

双飞翼布局

```css

.parent {
  overflow: hidden;
}
.left {
  float: left;
  margin-left: -100%;
  width: 100px;
}
.right {
  float: left;
  margin-left: -100px;
  width: 100px;
}
.center {
  float: left;
  width: 100%;
}
```

```html
<div class="double-wing-layout parent">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>
```

## 多列等宽布局

使用`flex`布局实现

```css
.parent {
  display: flex;
}
.column {
  flex: 1;
}
```

```html
<div class="flex-layout parent">
    <div class="left column"></div>
    <div class="center column"></div>
    <div class="right column"></div>
</div>
```

使用`grid`布局实现

```css
.parent {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
}
```

```html
<div class="grid-layout parent">
    <div class="left"></div>
    <div class="center"></div>
    <div class="right"></div>
</div>
```

使用`column`布局实现

```css
.parent {
  column-count: 3;
  column-gap: 0;
}
```

```html
<div class="column-layout parent">
    <div class="left"></div>
    <div class="center"></div>
    <div class="right"></div>
</div>
```



