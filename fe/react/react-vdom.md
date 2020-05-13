# React 虚拟 DOM

## 描述

**虚拟 DOM (Virtual DOM， VDOM) 是一种程序设计概念。**

> “如何实现虚拟 DOM” 这个问题可以转换成 “如何实现虚拟 DOM 技术”。

UI 的“虚拟”表现形式将保存在内存中，并通过诸如 `ReactDOM` 之类的库将“虚拟”表现与“真实” DOM 同步。
这个同步的过程称为 **协调（reconciliation）**。

“虚拟 DOM”更多是一种设计模式，而不是特定的技术，人们有时会说它表示不同的意思。

在 React 中，术语“虚拟 DOM”通常与 **React Element** 相关联。

但是，React 还使用称为 **Fiber** 的内部对象来保存有关组件树的其他信息。
它们也可能被视为 React 中“虚拟 DOM”实现的一部分。

> `Fiber` 是 React 16 中新的协调引擎。其主要目标是支持虚拟 DOM 的增量呈现。

React 中基于 Fiber 的增量变更称之为 Scheduling ？？？？？

## 优势

抽象出属性操纵，事件处理和手动 DOM 更新。只需要告诉 `React` UI 所处的状态，React 就能确保 DOM 匹配该状态。

## 对比 Shadow DOM

虚拟 DOM 与**Shadow DOM**是完全不同的。
**Shadow DOM** 是一种浏览器技术，主要用于确定 Web 组件中的变量和 CSS。虚拟 DOM 是由浏览器 API 之上的 JavaScript 库实现的概念。

## 参考

- [Virtual DOM and Internals](https://reactjs.org/docs/faq-internals.html#what-is-react-fiber)
