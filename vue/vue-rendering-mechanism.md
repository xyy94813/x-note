# VUE 渲染机制

## VM

虚拟 DOM (Virtual DOM，简称 VDOM) 是一种编程概念，意为将目标所需的 UI 通过数据结构“虚拟”地表示出来，保存在内存中，然后将真实的 DOM 与之保持同步.

例如，低代码平台/JSON Schema Form/InvokeAI workflow 一样，以数据节点信息表达其需要渲染的内容。

## Render Pipeline

![Vue Render Pipeline](https://cn.vuejs.org/assets/render-pipeline.sMZx_5WY.png)

从高层面的视角看，Vue 组件挂载时会发生如下几件事：

编译：Vue 模板被编译为渲染函数：即用来返回虚拟 DOM 树的函数。这一步骤可以通过构建步骤提前完成，也可以通过使用运行时编译器即时完成。

挂载：运行时渲染器调用渲染函数，遍历返回的虚拟 DOM 树，并基于它创建实际的 DOM 节点。这一步会作为响应式副作用执行，因此它会追踪其中所用到的所有响应式依赖。

更新：当一个依赖发生变化后，副作用会重新运行，这时候会创建一个更新后的虚拟 DOM 树。运行时渲染器遍历这棵新树，将它与旧树进行比较，然后将必要的更新应用到真实 DOM 上去。

## 带编译时信息的虚拟 DOM

Vue 中，框架同时控制着编译器和运行时。
这使得我们可以为紧密耦合的模板渲染器应用许多编译时优化。
编译器可以静态分析模板并在生成的代码中留下标记，使得运行时尽可能地走捷径

### 静态提升

在模板中常常有部分内容是不带任何动态绑定的：

```vue
<div>
  <div>foo</div> <!-- 需提升 -->
  <div>bar</div> <!-- 需提升 -->
  <div>{{ dynamic }}</div>
</div>
```

Vue 编译器自动优化将其提升：

```js
import {
  createElementVNode as _createElementVNode,
  createCommentVNode as _createCommentVNode,
  toDisplayString as _toDisplayString,
  openBlock as _openBlock,
  createElementBlock as _createElementBlock,
} from 'vue';

const _hoisted_1 = /*#__PURE__*/ _createElementVNode(
  'div',
  null,
  'foo',
  -1 /* HOISTED */,
);
const _hoisted_2 = /*#__PURE__*/ _createElementVNode(
  'div',
  null,
  'bar',
  -1 /* HOISTED */,
);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (
    _openBlock(),
    _createElementBlock('div', null, [
      _hoisted_1,
      _createCommentVNode(' hoisted '),
      _hoisted_2,
      _createCommentVNode(' hoisted '),
      _createElementVNode(
        'div',
        null,
        _toDisplayString(_ctx.dynamic),
        1 /* TEXT */,
      ),
    ])
  );
}
```

### Patching Flag / 更新类型标记

VUE 编译器对于单个有动态绑定的元素进行了类型标记。

```vue
<!-- 仅含 class 绑定 -->
<div :class="{ active }"></div>

<!-- 仅含 id 和 value 绑定 -->
<input :id="id" :value="value">

<!-- 仅含文本子节点 -->
<div>{{ dynamic }}</div>
```

在为这些元素生成渲染函数时，Vue 在 vnode 创建调用中直接编码了每个元素所需的更新类型：

```js
createElementVNode(
  'div',
  {
    class: _normalizeClass({ active: _ctx.active }),
  },
  null,
  2 /* CLASS */,
);
```

一个元素可以有多个更新类型标记，会被合并成一个数字。运行时渲染器也将会使用位运算来检查这些标记，确定相应的更新操作.

位运算检查是非常快的。通过这样的更新类型标记，Vue 能够在更新带有动态绑定的元素时做最少的操作。

### Tree Flattening

为了减少虚拟 DOM 协调时需要遍历的节点数量。
VUE 引入一个概念“区块”，内部结构是稳定的一个部分可被称之为一个"区块"。

```vue
<div> <!-- root block -->
  <div>...</div>         <!-- not tracked -->
  <div :id="id"></div>   <!-- tracked -->
  <div>                  <!-- not tracked -->
    <div>{{ bar }}</div> <!-- tracked -->
  </div>
</div>
```

编译的结果会被打平为一个数组，仅包含所有动态的后代节点：

```
div (block root)
- div 带有 :id 绑定
- div 带有 {{ bar }} 绑定
```

当这个组件需要重渲染时，只需要遍历这个打平的树而非整棵树。

## 参考

[Vue 渲染机制](https://cn.vuejs.org/guide/extras/rendering-mechanism.html)
