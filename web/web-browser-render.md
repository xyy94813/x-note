# 浏览器的渲染过程

## 浏览器中的进程

目前浏览器多为多进程架构，页面的加载、渲染和交互是由多个进程配合完成。

- 浏览器主进程。
  - 负责界面显示
  - 用户交互
  - 子进程管理
  - 同时提供存储等功能
- 渲染进程
  - 将 HTML、CSS 和 JavaScript 转换为用户可以与之交互的网页
  - 渲染引擎 Blink 和 JavaScript 引擎 V8 都是运行在该进程中
  - 出于安全考虑，渲染进程都是运行在沙箱模式下，无法访问系统资源
  - 出于性能 Chrome 会为每个 Tab 标签创建一个渲染进程，其它内核的浏览器只有一个渲染进程
- 网络进程
  - 主要负责页面的网络资源加载。
  - 之前是作为一个模块运行在浏览器主进程里面的，直至最近才独立出来，成为一个单独的进程。
- GPU 进程
  - GPU 的使用初衷是为了实现 3D CSS 的效果
  - 随后网页、Chrome 的 UI 界面都选择采用 GPU 来绘制，这使得 GPU 成为浏览器普遍的需求
- 插件进程
  - 负责插件的运行
  - 出于鲁棒性考虑，插件进程隔离插件执行保证插件异常时页面不受影响
- 其它进程

## 浏览器引擎种类

浏览器引擎（Browser Engine），或者称为浏览器排版引擎（Browser Layout Engine），目前主流有以下几种。

> 浏览器排版引擎和浏览器渲染引擎不是一回事，两者紧密相关互相耦合

- [Webkit (Safari)](https://webkit.org/)
- [Blink (Chromium)](https://www.chromium.org/blink/)。
- [Gecko (Firefox)](https://developer.mozilla.org/en-US/docs/Glossary/Gecko)
- [Presto (Opera)](https://developer.mozilla.org/en-US/docs/Glossary/Presto)
- [Trident (Internet Explorer)](https://developer.mozilla.org/en-US/docs/Glossary/Trident)

IE 已死，Edge、Opera 都已改为采用 Blink.
而 Blink 是 Webkit 的变种，其对自身的定义为“渲染引擎（rendering engine）”。

大体上浏览器引擎仅剩 Webkit 和 Gecko 两大分支

## 渲染进程详细

### 渲染进程的线程类别

导航过程完成之后，浏览器进程把数据交给了渲染进程，渲染进程负责 Tab 选项卡内的所有事情，核心目的就是将 HTML/CSS/JavaScript 代码，转化为用户可进行交互的 Web 页面。

在该过程中渲染进程会开启多个线程协作完成，主要的线程以及作用如下：

- GUI 渲染线程
- JS 引擎线程
- 事件触发线程
- 定时器触发线程
- 异步 http 请求线程

> GUI 渲染线程与 JS 引擎线程是相互排斥的。
> 因为 JS 引擎线程在执行的过程中可能会发生回流和重绘，所以 GUI 渲染线程执行时候，JS 引擎线程会被挂起，等待 GUI 渲染线程执行完毕之后。
> 同理，当 JS 引擎执行时 GUI 线程会被挂起，GUI 更新会被保存起来等到 JS 引擎空闲时立即被执行。
> 所以如果 JS 执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染阻塞。

### JS 引擎类别

- SpiderMonkey
  - Firefox
- V8
  - Chromium
  - Node
  - Opera
- JavaScriptCore
  - Safari

### 渲染进程工作流程

渲染引擎的基本流程:

1. 解析文档
   - 构建文档对象模型树
   - 构建样式对象模型树
   - 脚本异步加载
2. 构建渲染对象树
3. 布局/排版（Layout/flow）。
4. 绘制（Paint）
   1. 光栅化（Rasterization）
   2. 合并（Composite）

> 这个过程是逐步完成的，
> 为了更好的用户体验，渲染引擎将会尽可能早地将内容呈现到屏幕上，
> 并不会等到所有 HTML 都解析完成之后再去构建和布局渲染树，
> 它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。

另外，不同渲染引擎也有细微差异....

Gecko 将视觉格式化元素树称为“Frame Tree”，每个元素都是一个 Frame。 WebKit 使用术语 “Render Tree”，它由 “Render Objects” 组成

Webkit 工作流程：

![Webkit Workflow](/images/webkit-workflow.png)

Gecko 工作流程：

![Gecko Workflow](/images/webkit-workflow.png)

### 构建文档对象模型树

DOM 是文档对象模型 (Document Object Model) 的简称。
它是 HTML 文档的对象呈现方式，也是 HTML 元素与 JavaScript 等外部环境之间的接口。

DOM 与 HTML 标记之间几乎是一对一的关系。

创建解析器时，也会创建 Document 对象。
在树构建阶段，以 Document 为根部的 DOM 树将被修改，并且向其中添加元素。
标记生成器发出的每个节点都将由树构造函数进行处理。
对于每个词法单元，规范都会定义与其相关的 DOM 元素，并将为这个词元创建这些元素。
该元素随即会添加到 DOM 树以及开放元素的堆栈中。
此堆栈用于更正嵌套错误和未关闭的标记。
该算法也可描述为状态机。
这些状态称为“插入模式”。

### 构建样式对象模型树

渲染引擎无法直接理解 CSS 文件的内容，所以需要将其解析成渲染引擎能够理解的结构，即 styleSheets。

WebKit 使用 Flex 和 Bison 解析器生成器，根据 CSS 语法文件自动创建解析器。
Bison 会创建一个自下而上的移位归约解析器。

Gecko 使用手动编写的自上而下解析器。

CSSOM（CSS Object Model）其作用为：

- 提供给 JavaScript 操作样式表的能力
- 图层树的合成提供基础的样式信息。

### 构建渲染对象树

在 DOM 树构建期间，浏览器会构建另一个树，即渲染树。
该树状图中的视觉元素按照显示顺序排列。
它是文档的可视化表示。
这种树旨在确保按正确的顺序绘制内容。

渲染程序与 DOM 元素相对应，但并非一对一关系。
非可视化 DOM 元素不会插入渲染树中。例如“head”元素。
此外，`display:none` 的元素也不会显示在树中

构建渲染树需要计算每个渲染对象的视觉属性。
这是通过计算每个元素的样式属性来完成的。
样式包括不同来源的样式表、内嵌样式元素和 HTML 中的视觉属性

WebKit 节点会引用样式对象 (RenderStyle)。
在某些情况下，这些对象可以由节点共享。

为了简化样式计算，Gecko 还采用了另外两种树：规则树（rule tree）和样式上下文树（style context tree）。
WebKit 也有样式对象，但它们并不存储在样式上下文树这样的树中，只有 DOM 节点指向其相关样式。

示例：

```html
<!-- HTML -->
<html>
  <body>
    <div class="err" id="div1">
      <p>
        this is a <span class="big"> big error </span>
        this is also a
        <span class="big"> very big error</span> error
      </p>
    </div>
    <div class="err" id="div2">another error</div>
  </body>
</html>
```

```css
/* CSS */
div {
  margin: 5px;
  color: black;
}
.err {
  color: red;
}
.big {
  margin-top: 3px;
}
div span {
  margin-bottom: 4px;
}
#div1 {
  color: blue;
}
#div2 {
  color: green;
}
```

![Rule Tree](/images/stylesheet-rule-tree.png)

WebKit 使用一个标记来标记是否所有顶级样式表（包括 @imports）均已加载完毕。如果在添加样式时样式未完全加载，则会使用占位符并在文档中对其进行标记，并在样式表加载完毕后重新计算占位符。

### 布局（Paint）

渲染引擎在创建完成并添加到呈现树时，并不具有位置和大小。
计算这些值的过程称为布局或自动重排。
最终会创建图层树（Layer Tree）它将确定每个节点在屏幕上的确切坐标。

布局阶段会从图层树的根节点开始遍历，然后确定每个元素盒子在页面中的确切大小与坐标位置的绝对像素值。布局流程：

1. 父 renderer 会自行确定宽度。
2. 父级会处理子级，并执行以下操作
   1. 放置子 renderer。（设置其 x 和 y）
   2. 根据需要调用子布局
3. 父级使用子级的累计高度以及外边距和内边距的高度来设置自己的高度，
   此值将由父级渲染器的父级使用
4. 将其 dirty 设置为 false

通常情况下，并不是图层树的每个节点都包含一个图层，如果一个节点没有对应的层，那么这个节点就从属于父节点的图层。

[何时创建层叠上下文（Stacking Context），参考 MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context)。

HTML 使用基于流的布局模型。
“流中”后期的元素通常不会影响更早“流中”元素的几何形状，因此布局可以按从左到右、从上到下的顺序遍历文档。
除了 `<Table />` 外，只需一次遍历即可计算出几何图形。

如果渲染程序发生更改或添加，则会将其自身及其子项标记为“dirty”需要布局。

在整个渲染树上触发布局，这就是“全局”布局。例如：

- 屏幕大小调整。
- 影响所有渲染程序的全局样式更改，例如字体大小更改

全局布局通常会被同步触发。

当渲染程序处于 dirty 时，会（异步）触发增量布局。
例如，当额外内容来自网络并添加到 DOM 树之后，新的渲染程序才附加到渲染树。

### 绘制（Paint）

构建完图层树（Layer Tree）之后，渲染引擎会遍历渲染树，并调用渲染程序的“paint()”方法，并绘制每个节点

与布局一样，绘制也可以是全局（绘制整个树）或增量绘制。

绘制顺序：

1. 背景颜色
2. 背景图片
3. 边框
4. 子节点内容
5. Outline

绘制列表只是用来记录绘制顺序和绘制指令的列表，而实际上绘制操作是由渲染引擎中的合成线程来完成的。
当图层的绘制列表准备好之后，主线程会把该绘制列表提交（commit）给合成线程

#### 显示列表

Gecko 遍历渲染树，为绘制的矩形构建一个显示列表。
重新绘制时，只需遍历一次树。
Gecko 还对处理过程进行了优化，如完全位于其他不透明元素下方的元素

#### 栅格化（Rasterization）

WebKit 会将旧的矩形保存为位图，然后只绘制新旧矩形之间的增量。

所谓栅格化，是指将图块（tile）转换为位图（这些图块的大小通常是 256x256 或者 512x512）。
而图块是栅格化执行的最小单位。

合成线程会按照视口附近的图块来优先生成位图，实际生成位图的操作是由栅格化来执行的。

#### 合并（Composite）

在光栅化完成之后，每一个图块都对应一张图片，合成线程会将这些图片合成为“一张”图片，将其绘制到内存中，最后再将内存显示在屏幕上。

## 参考资料

- [Browser Engine and Types of Browser Engines](https://codinglap.com/browser-engine-and-types-of-browser-engines/)
- [How browsers work](https://web.dev/articles/howbrowserswork)
- [浏览器渲染流程和性能优化](https://zhuanlan.zhihu.com/p/516574343)
- [渲染进程的内部机制](https://tsejx.github.io/javascript-guidebook/browser-object-model/browser-working-principle/workflow/)
