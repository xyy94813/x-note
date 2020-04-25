# React Reconciliation

React 提供了声明式的 API 让你不需要担心每次更新都改变了什么，这使得编写 APP 变得更加容易。

使用 React 时，您可以在单个时间点将 `render()` 函数视为创建 React Element Tree。
在下一个 `state` 或 `props` 更新时，该 `render()` 函数将返回不同的 React Element Tree。

然后，React 需要弄清楚如何有效地更新 UI 以匹配最新的树。

以最小数量的操作以将一棵树转换为另一棵树的算法问题，现有的解决方案复杂度约为 $$ O(n^2) $$。
如果比较 1000 个元素，则需要 **10 亿** 次的比较，这太昂贵了。

React 基于两个假设实现了一种启发式 $$ O(n) $$ 算法：

1. 不同类型的两个元素将产生不同的树。
2. 开发人员可以使用 props `key` 提示哪些子元素在不同渲染中可以保持稳定。

![](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)

## [React Diff 算法](react-diff-algorithm.md)

## 权衡 React Reconciliation

React 可以在每次操作时重新渲染整个应用程序；最终结果将是相同的。需要明确的是，在这种情况下，重新渲染意味着为所有组件调用渲染，但这并不意味着 React 会卸载并重新安装它们。它将仅遵循前几节中所述的规则应用差异。

在当前的实现中，子树在其兄弟姐妹之间移动 React 无法直到子树已经在其他地方移动了。该算法将重新渲染该完整的子树。

由于 React 依赖于启发式算法，因此如果不满足其背后的假设，性能将会受到影响。

1. 该算法不会尝试匹配不同组件类型的子树。如果您发现自己在两种具有非常相似输出的组件类型之间交替，则可能需要使其成为相同的类型。
2. 密钥应稳定，可预测且唯一。不稳定的键（如 `Math.random()` 产生的键）将导致不必要地重新创建许多组件实例和 DOM 节点，这可能导致性能下降和子组件中的状态丢失。

## 参考

- [React Reconciliation](https://reactjs.org/docs/reconciliation.html)
