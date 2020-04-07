# CSS 表布局

表布局的行为十分复杂，导致了在页面布局时需要尽可能的避免使用表布局。

## 表格式化

**表格式化（Table Formatting）**，是 CSS 中最复杂的一部分。
CSS 定义了表格式化中涉及的许多对象，如下图所示：

![Table-Formatting-Object-Tree](https://s1.ax1x.com/2020/03/25/8jD4C4.png)

- **表标题（table-and-caption）** 是一个块级格式化对象，它返回包括表及其标题的块区域。 `table-and-caption` 包含一个可选的 `table-caption` 格式对象和一个必需的 `table` 格式对象。
- "table-caption" 是一个容器格式化对象，它保存一个块堆栈作为表标题的内容。
- **表（table）** 是一个块级格式化对象，它返回一个包含表的块区域，该表包含一个或多个可选的`table-column`格式化对象，一个可选的`table-header`格式化对象，一个可选的`table-footer`格式化对象，以及一个必需的`table-body`格式对象。
- **表列（table-column）** 是一个辅助格式化对象，不返回任何区域。它用于配置每列的宽度和其他属性。
- **表头（table-header）**是一种容器格式化对象，其中包含一个或多个表头行。
- **表页脚（table-footer）**是一种容器格式化对象，其中包含一个或多个表页脚行。
- **表主体（table-body）**是一种容器格式化对象，其中包含一个或多个表主体行。
- **表行（table-row）**是一种容器格式化对象，其中包含一个或多个`table-cell`格式化对象。
- **表单元格（table-cell）**是一个容器格式化对象，它保存块堆栈作为表主体，页眉或页脚行的单个单元格的内容。

### 表的视觉编排

CSS 对于表和内部表元素有很分明的界限。
在 CSS 中，内部表元素生成矩形框，这些框有内容，内边距和边框，但是没有外边距。

### 排列规则

表格的编排有 6 个规则。

- 每个行框包含一行表单元格。
- 一个行组包含多个行框，该行组就包含多少个表格单元格。
- 列框包含一列或多列表格单元。所有列框都按其出现的顺序一次相邻放置。
- 列组中包含多少个列框，该列组框就包含多少个表格单元。
- 虽然单元格可能跨越几行或几列，但 CSS 没有定义这是如何发生的。而是让文档语言来定义生成。
- 单元格的框“不能”扩展到表或行组的最后一行框之外。如果表结构会导致这种情况，则必须缩短单元格，直到它适合包围它的表或行组。

这些的规则的基础是 “表格单元”，这是绘制表的表格线之间的区域。

![Grid cells form the basis of table layout](https://gdut_yy.gitee.io/doc-csstdg4/figures/ch14/fg14-1.png)

### 表的 dispaly 值

通过修改元素的 display 值，可以使得元素的展示与表格相关的元素相似？？？

| Value                | 对应的 HTML 元素     | 描述                                   |
| :------------------- | :------------------- | :------------------------------------- |
| `table`              | `<table />`          | 指定元素定义块级表。                   |
| `inline-table`       | none                 | 此值指定元素定义内联级表。             |
| `table-row`          | `<tr />`             | 此值指定元素是表单元格的行。           |
| `table-row-group`    | `<tbody />`          | 此值指定元素对一个或多个表行进行分组。 |
| `table-header-group` | `<thead />`          |                                        |
| `table-footer-group` | `<tfoot />`          |                                        |
| `table-column`       | `<col />`            | 此值声明元素描述表单元格的列。         |
| `table-column-group` | `<colgroup />`       | 此值声明元素对一个或多个列进行分组。   |
| `table-cell`         | `<th />` 或 `<td />` | 此值指定元素表示表中的单个单元格。     |
| `table-caption`      | `<caption />`        | 此值定义表的标题。                     |

> CSS 没有定义如果多个元素有值“标题”会发生什么，但它明确警告：作者不应该把一个以上的元素与 “显示:标题” 在一个表或内联表元素。

CSS 将表模型定义为“以行为主（row primacy）”。
模型假设开发人员会显示声明行，而列是从单元格行的布局推导出来。

尽管 CSS 表模型是面向行的，单元格在文档流中是行元素的后代，但它们可能同时属于两个上下文（行和列）。

CSS 中的列只能接受 4 种样式，同时还需满足特殊规则。

| Value        | Description                                                                       |
| :----------- | :-------------------------------------------------------------------------------- |
| `border`     | 只有 `boder-collapse` 属性为 `collapse` 是才能生效                                |
| `background` | 只有 单元格及其航有透明背景时                                                     |
| `width`      | 列或列组的最小宽度                                                                |
| `visibility` | 如果一个列或列组的 `visibility` 为 `collapse`，从合并列跨到其它列的单元格会被裁剪 |

### 匿名表对象

标记语言可能未包含足够的元素以至于无法按 CSS 的定义重返表示表。例如：

```html
<table>
  <td>Name</td>
  <td>Alex</td>
</table>
```

CSS 对此做了兼容性处理：

```html
<table>
  [anonymous table-row object start]
  <td>Name</td>
  <td>Alex</td>
  [anonymous table-row object end]
</table>
```

插入规则：

1. 如果一个 `table-cell` 父元素不是 `table-row`，则会在该 `tabel-cell` 及其父元素之间插入匿名 `table-row` 对象。
2. 如果一个 `table-row` 父元素不是 `table`、`inline-table` 或 `table-row-group`，则会在该 `tabel-row` 及其父元素之间插入匿名 `table` 对象。
3. 如果一个 `table-column` 父元素不是 `table`、`inline-table` 或 `table-column-group`，则会在该 `tabel-column` 及其父元素之间插入匿名 `table` 对象。
4. 如果一个 `table-row-group`、`table-header-group`、`table-footer-group` 或 `table-caption` 对象的父元素不是 `table`，则会在该元素及其父元素之间插入匿名 `table` 对象。
5. 如果一个 `table`、`inline-table`对象的子元素不是 `table-row-group`、`table-header-group`、`table-footer-group`、 `table-caption` 或 `table-row`，则在该 `table` 元素与其子元素之间插入一个匿名 `table-row` 对象。该匿名对象将包含所有 `table-row-group`、`table-header-group`、`table-footer-group`、 `table-caption` 或 `table-row` 的兄弟节点。
6. 如果一个 `table-row-group`、`table-header-group`、`table-footer-group` 对象的子元素不是 `table-row` 元素，则在该元素及其子元素之间插入一个匿名 `table-row` 对象。这个匿名对象包含该子元素的多有飞 `table-row` 对象的连续兄弟。
7. 如果一个 `table-row` 对象的子元素不是 `table-cell` 元素，则在该元素及其子元素之间插入一个匿名 `table-cell` 对象。这个匿名对象包含该子元素的所有非 `table-cell` 对象的连续兄弟。

用一个示例解释规则 5、6、7：

```html
<table>
  [anonymous table-row object start] [anonymous table-cell object start]
  <span>Last</span>
  <span>Name</span>
  [anonymous table-cell object end]
  <td>Alex</td>
  [anonymous table-row object end]
  <tr>
    <td>address</td>
    <td>AAA st.</td>
  </tr>
  [anonymous table-row object start]
  <td>Sex:</td>
  <td>male</td>
  [anonymous table-row object end]
</table>
```

### 表布局的层次

CSS 为了完成表的显示，定义了 6 个不同的层，分别放置表的不同结构：

![Table Layout layers](https://gdut_yy.gitee.io/doc-csstdg4/figures/ch14/fg14-3.png)

## 表单元格

CSS 存在两种不同的边框模型。一种是分隔边框模型，一种是合并边框模型。

Developer 可以通过 CSS 属性 `border-collapse` 控制。
`border-collapse` 仅支持应用于 `display` 值为 `table` 或 `inline-table`。

如果 `border-collapse` 值为 `collapse`，则使用合并边框模型，如果值为 `separate` 使用分割边框模型。
默认值为 `separate`。

分隔单元格模型可使用 `border-spacing` 控制边框的边距。

如果一个单元格內包含内容，则不能认为是空的。
如果一行中的所有单元格都为空，而且 `empty-cells` 都是 `hide`，则整行将处理为这个行元素设置为 `display:none` 的效果。
`empty-cells` 仅在 `border-collapse: separate` 时生效。

合并单元格模型的布局上相比分隔单元格模型要复杂。

- display 值为 `table` 或 `inline-table` 的元素不能有任何内边距，但可以有外边距。因此表的外围边框与最外单元格的边界不会有间隔。
- 边框可以应用到单元格、行、行组、列和列组。
- 单元格边框之间绝对不会有任何间隔。
- 一旦合并，单元格之间的边框会在单元格间的家乡表格线上居中。

![使用合并边框模型的表行布局](http://gdut_yy.gitee.io/doc-csstdg4/figures/ch14/fg14-8.png)

在该模型中，外边框超出了表的宽度，行的宽度只包含表边框的一半。
假设，表行中存在 n 个单元格，那么可以得到行宽为：

```
row_width = (0.5 * border-width0) +
            padding-left1 + width1 + padding-right1 + border-width1 +
            padding-left2 + ... + padding-rightn + (0.5 * border-widthn);

// border-width(i) 为单元格 i 与下一个单元格之间的边框，border-width3 为单元格 3 和 4 之间的边框。
```

> 如果边框宽度是一个奇数，User-Agent 必须觉得如何将边框在表格线上居中。
> 可能会移动该边框，使之稍稍偏离中序，或者宽度向上或向下调整为一个偶数。

此外，在开始建立合并表框布局时， User-Agent 会为表本身计算一个初始的左右边框。
检查表中第一行第一个单元格的左边框宽度的一半作为表的初始左边框，最后一个单元格的右边框的宽度的一半作为表的初始右边框。

### 相邻边框的优先级

如果两个或多个边框相邻，会相互“合并”。

- 如果某合并边框的 `border-style` 为 `hidden`。该边框优先级最高，这个位置所有边框都隐藏
- 如果某合并边框的 `border-style` 为 `none`。该边框优先级最低。
- 如果至少有一个合并边框的 `border-style` 不为 `none`，且所有边框 的 `border-style` 不为 `hidden`，则更宽的边框优先级更高。
- 如果合并边框的样式和宽度一致，但是颜色不同，则优先级按以下顺序：`cell`、`row`、`row-group`、`column`、`column-group`、`table`。

![Manipulating border widths, styles, and colors leads to some unusual results](http://gdut_yy.gitee.io/doc-csstdg4/figures/ch14/fg14-9.png)

## 表格大小

确定表的宽度有两种不同的方法，固定宽度布局和自动宽度布局。
可以通过 `table-layout` 进行选择。

### 固定宽度布局

**固定宽度表布局相对于自动宽度表布局速度更快。**

固定宽度布局模型步骤简单：

1. `width` 属性不是 `auto` 的所有元素会根据 `width` 值设置该列的宽度。
2. 如果一个列的宽度为 `auto` 则根据该单元格宽度设置此列的宽度。如果单元格跨多列，则宽度在这些列上平均分配。
3. 如果列的宽度仍为 `auto`，会自动确定其大小，使其尽可能相等。

此时，表的宽度设置为 `Math.max(表的 width 值, 列宽度之和)`。
如果 `表宽度 > 列宽总和`，将 `(表宽度 - 列宽总和) / 列数` 宽度增加到每一列上。

一位所有列宽都有表的第一行定义，所以速度很快。

### 自动宽度布局

在大多数 User-Agent 中 只要 table 的 width 为 auto，就会触发自动布局，即使设置了 table-layout。

自动宽度布局步骤繁琐：

1. 对于一列中的各个单元格，计算最小和最大单元格宽度。
   1. 确定显示内容所需的最小宽度。若单元格的 width 值大于最小可能宽度，则把最小单元格宽度设置为该 width。如果单元格的 width 为 auto，最小单元格宽度则设置为最小内容宽度。
   2. 最大宽度为完全显示内容而且不包括换行符所需的宽度。
2. 对于各一列，计算最小和最大列宽。
   1. 列的最小宽度由该列中最小单元格宽度的最大值确定。如果为该列指定的 width 大于列中所有最小单元格宽度，则最小列宽度设置为该 width 值。
   2. 取该列中所有单元格的最大单元格宽度的最大值。如果为该列指定的 width 大于列中所有最大单元格宽度，则最大列宽度设置为该 width 值。
3. 如果一个单元格跨多列，最小列宽之和必须等于这个跨列单元格的最小单元格宽度。最大列宽之和等于跨列单元格的最大宽度。如果存在差距，User-Agent 会把这个差距在所跨列上平均分配。
4. 如果表的计算宽度值不是 auto，将这个计算表宽度值与所有列宽、边框和单元格间隔之和相比较。取二者较大一方作为最终宽度。如果计算宽度值大于列宽、边框和单元格间隔之和，所有列的宽度会增加一个相等的量，使得刚好将表完全填充。
5. 如果表的计算宽度值为 auto，列宽、边框和单元格间隔之和为表的最终宽度。

### 表格内容对齐

可以通过 text-align 控制单元格中的内容水平对齐方式。

可以通过 vertical-align 控制单元格中的内容垂直对齐方式。