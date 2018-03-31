# React Diff算法核心

> **Diff 算法**，即差异比较算法，是 React 实现原理的核心之一，React 通过此算法比较每次更新之后子虚拟 DOM 树种最小的差异点，并进行计算。

首先观察 React 是如何工作的。

假设有一个组件`MyComponent`，长这样

```jsx
class MyComponent extends React.Component {
    render () {
        const { isFirst } = this.props;
        if (isFirst ) {
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
<div className="first">
    <span>A Span</span>
</div>;
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

#### 按层级划分

找到两棵任意的树之间的最小的差异是一个复杂度为`O(n3)`的问题。对此，React 根据 Web 应用的特点**（Web 应用很少有 component 移动到树的另一个层级，它们大部分只是在相邻的子节点之间移动）**，尝试将树按照层级进行分解，最终达到了接近`O(n)`的复杂度。

![](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)

#### 列表

假设有一个 component，一个循环渲染了多个 component，随后又在列表中间插入一个新的 component，根据现有的信息很难知道如何在这两个组件列表之间做映射。

默认情况下，react 会将前一个列表第一个 component 和后一个列表的第一个 component 关联起来，如此类推。因此你可以给 list 中每一个组件设置一个`key`属性帮助 react 来处理它们之间的对应关系。

> React 中渲染多个组件如果不包含`key`属性，你将会在控制面板看到这个警告
> "Warning: Each child in an array or iterator should have a unique 'key' prop.

![](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/2.png)


#### 组件比对

React 应用通常由用户定义的 component 组合而成，这些 component 通常是一个由很多`div`组成的树。对此，React 只会匹配相同`class`的 component。例如`<AComponent/>`被`<BComponet/>`替换掉，React  会直接移除`AComponent`，然后创建`Bcomponent`，不会再对这两个组件的内部细节进行比对。

## 参考

1. [React’s diff algorithm](https://calendar.perfplanet.com/2013/diff/)
2. [官方文档](https://reactjs.org/docs/reconciliation.html)
