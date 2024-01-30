# React Diff 算法核心

> **Diff 算法**，即差异比较算法，是 React 实现原理的核心之一，React 通过此算法比较每次更新之后子虚拟 DOM 树种最小的差异点，并进行计算。

首先观察 React 是如何工作的。

假设有一个组件`MyComponent`，长这样

```jsx
class MyComponent extends React.Component {
  render() {
    const { isFirst } = this.props;
    if (isFirst) {
      return (
        <div className="first">
          <span>A Span</span>
        </div>
      );
    }
    return (
      <div className="second">
        <p>A Paragraph</p>
      </div>
    );
  }
}
```

我们希望执行以下的行为：

1. 首先挂载`<MyComponent isFirst={true} />`
2. 然后用`<MyComponent isFirst={flase} />`替换`<MyComponent isFirst={true} />`
3. 卸载这个组件

将这样的一个过程转换成 DOM 指令：

从没有到第一步:

创建一个节点

```html
<div className="first">
  <span>A Span</span>
</div>
;
```

从第一步到第二步:

替换属性`className="first"`为`className="seconde"`.
替换节点`<span>A Span</span>`为`<p>A paragraph</p>`

从第二步到卸载组件:

删除节点

```html
<div className="second">
  <p>A Paragraph</p>
</div>
```

## Diff 算法核心

区分两棵树时，React 首先比较两个根元素。行为因根元素的类型而异。

找到两棵任意的树之间的最小的差异是一个复杂度为 $$O(n^3)$$的问题，React Diff 算法根据 Web 应用的特点 **（Web 应用很少有 component 移动到树的另一个层级，它们大部分只是在相邻的子节点之间移动）** 作出以下两个假设，最终达到了接近 $$O(n)$$ 的复杂度：

1. 不同类型的两个元素将产生不同的树。
2. 开发人员可以使用 props `key` 提示哪些子元素在不同渲染中可以保持稳定。

![How React Compare two tree](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)

### 不同类型的 React Element

每当根元素具有不同类型时，React 都会拆开旧树并从头开始构建新树。从 `<a>` 到 `<img>`，或从 `<Article>` 到 `<Comment>`，或从 `<Button>` 到 `<div>` -任何这些都将导致完全重建。

拆除树时，旧的 DOM 节点将被破坏。
组件实例接收 `componentWillUnmount()`。
建立新树时，会将新的 DOM 节点插入到 DOM 中。
组件实例接收 `componentWillMount()`，然后接收 `componentDidMount()`。
与旧树关联的任何状态都将丢失。
根目录下的所有组件也将被卸载并破坏其状态。

```jsx
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

这将销毁旧的 `Counter`，然后重新安装新的 `Counter`。

### 同类型的 DOM Element

比较两个相同类型的 React DOM 元素时，React 会查看两者的属性，保留相同的基础 DOM 节点，并仅更新更改的属性。

```jsx
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较这两个元素，React 知道只修改底层 DOM 节点上的 className。

在更新样式时，React 还知道仅更新已更改的属性。

```jsx
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

在这两个元素之间进行转换时，React 知道仅修改颜色样式，而不修改 fontWeight。

处理完 DOM 节点后，React 然后在子节点上递归。

### 同类型的 React Element

组件更新时，实例保持不变，因此在渲染之间保持状态。 React 更新基础组件实例的属性以匹配新元素，并在基础实例上调用 `componentWillReceiveProps()`和 `componentWillUpdate()`。

接下来，调用 `render()`方法，并且 diff 算法根据先前的结果和新的结果进行递归。

### 递归子节点

默认情况下，在 DOM 节点的子节点上递归时，React 只会同时遍历两个子节点列表，并在存在差异时生成一个更新操作。

例如，在子元素的末尾添加元素时，在这两个树之间进行转换会很好：

```jsx
<ul>
  <li>first</li>
  <li>second</li>
</ul>
// to
<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React 会匹配两个 `<li>first</li>` 树，两个 `<li>second</li>` 树，然后插入一个 `<li>third</li>` 树。（这看起来很不错）

但是，对于下列例子则会导致糟糕的性能。

```jsx
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
// to
<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

React 会操作每个子节点，而不是使`<li>Duke</li>` 和 `<li>Villanova</li>`保持不变，并在前面插入 `<li>Connecticut</li>`

> React 中渲染多个组件如果不包含 `key` 属性，你将会在控制面板看到这个警告
> "Warning: Each child in an array or iterator should have a unique 'key' prop.

### React Key

React 支持 `key` 属性。当子项具有 `key` 时，React 使用该键将原始树中的子项与后续树中的子项进行匹配。

![Rerender With Key](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/2.png)

[React Render with Key in React v15](https://github.com/facebook/react/blob/d1c08f11d5e1ad03eb92a58b599562a010a68734/src/renderers/shared/reconciler/ReactMultiChild.js#L287-L360)

在对子项进行 diff 时，存在三种类型的操作：

- MOVE_EXISTING -- 存在相同的节点则复用以前的 DOM 节点，做移动操作。
- INSERT_MARKUP -- 新的节点不在旧集合里则插入新的节点。
- REMOVE_NODE -- 新集合里在旧集合中对应的 node 不同，不能直接复用和更新，需要执行删除操作，或者旧集合中的节点不在新集合里的。

1. 遍历 newChildrens，基于 key 判断 newChild 是否在 oldChildrens 存在**相同的节点**.
   1. 如果存在相同节点(prevChild === nextChild)
      1. 判断原先节点的变化顺序（不考虑头部新插入的节点）
         1. 节点的挂载顺序变大（从前往后），移动节点
         2. 节点的挂载顺序变小（从后往前或不变），不做操作
      2. 节点的 `_mountIndex` 变为新集合中的 index
   2. 如果不存在相同节点，
      1. 之前存在相同
      2. 在上一个新集合中的节点后插入新节点
2. 遍历 oldChildrens，移除在新集合中不存在的节点 (???)

```jsx
// befor
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
  <li key="2017">Alex</li>
  <li key="2018">Frank</li>
</ul>

// after
<ul>
  <li key="2014">Connecticut</li>
  <li key="2018">Frank</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

对与上述例子 DOM 节点的变化顺序为

```jsx
// step 1
// 插入新的节点 `<li key="2014">Connecticut</li>`
<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
  <li key="2017">Alex</li>
  <li key="2018">Frank</li>
</ul>

// step 2
// key 2018 向前移动，实际上不做操作
<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
  <li key="2017">Alex</li>
  <li key="2018">Frank</li>
</ul>

// step 3
// `<li key="2015">Duke</li>` 移动至 `<li key="2018">Frank</li>` 后
<ul>
  <li key="2014">Connecticut</li>
  <li key="2016">Villanova</li>
  <li key="2017">Alex</li>
  <li key="2018">Frank</li>
  <li key="2015">Duke</li>
</ul>

// step 4
// `<li key="2016">Villanova</li>` 移动至 `<li key="2015">Duke</li>` 后
<ul>
  <li key="2014">Connecticut</li>
  <li key="2017">Alex</li>
  <li key="2018">Frank</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

// step 5
// 移除新集合中不存在的节点 `<li key="2017">Frank</li>`
<ul>
  <li key="2014">Connecticut</li>
  <li key="2018">Frank</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

所以不得已时，您可以将数组中项目的索引作为键传递。但是使用索引作为 key 式，重新排序会很慢。

- [使用 Key 作为索引可能会导致的问题](https://reactjs.org/redirect-to-codepen/reconciliation/index-used-as-key)
- [不使用索引作为键如何解决这些重新排序，排序和前置问题](https://reactjs.org/redirect-to-codepen/reconciliation/no-index-used-as-key)

## 参考

1. [React’s diff algorithm](https://calendar.perfplanet.com/2013/diff/)
2. [官方文档](https://reactjs.org/docs/reconciliation.html)
3. [React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)
