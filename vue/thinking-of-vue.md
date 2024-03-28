# 关于 Vue 的思考

官方文档的定位.

> 什么是 Vue？​
>
> Vue (发音为 /vjuː/，类似 view) 是一款用于构建用户界面的 JavaScript 框架。
> 它基于标准 HTML、CSS 和 JavaScript 构建，并提供了一套声明式的、组件化的编程模型，帮助你高效地开发用户界面。
> 无论是简单还是复杂的界面，Vue 都可以胜任。

**以下思考，针对 Vue v3 版本**

## 使用

Vue 整体的使用体验，基本参考自 Angular 的用法。

- 模版
- 指令
- v-model 双向数据绑定

在此之上融合了 React 上的一些概念

- 组件
- VDom
- Hooks/Effect

然后进一步基于社区提出的问题进一步改进 DE。

- [defineModel](https://cn.vuejs.org/guide/components/v-model.html#under-the-hood)
- [watchEffect](https://cn.vuejs.org/guide/essentials/watchers.html#watcheffect)
- 透传 Attributes
- ...等其它功能

<!-- 例如 `defineModel`, `watchEffect` 其使用方式明显脱离 JS 语法范畴，必须借助 Vue 相关的解释器做转译。 -->

例如 `defineModel` 其使用方式明显脱离 JS 语法范畴，必须借助 Vue 相关的解释器做转译。

### 模版语法

与 Angular 类似。
配合内置或自定义指令一同使用。

### 指令

与 Angular 类似。
支持自定义。

### 单文件组件

Vue 自身发展衍生出的概念。
标志性功能之一。

一个文件里包含组件的视图模版、交互逻辑、样式表。

React 以文件目录或 CSSInJS 等达到类似目的。

### Computed

监听依赖的响应式数据计算得到新的值。
非常有用的功能。

### Watchers

监听依赖的响应式数据执行其它 job。
非常有用的功能。

### 透传 Attributes

> “透传 attribute”指的是传递给一个组件，却没有被该组件声明为 props 或 emits 的 attribute 或者 v-on 事件监听器。
> 最常见的例子就是 class、style 和 id。

优点：

- 开发过程，在对某一组件做多次封装后仍暴露同一命名接口非常有优势
- 数据跨多层子节点传递时

```jsx
const Title = ({ level, ...props }) => {
  const Comp = `h${props.level}`;
  return <Comp {...props} />;
};
const Level1Title = (props) => <Title {...props} level="1" />;
const Level2Title = (props) => <Title {...props} level="2" />;
// ....
const PageLevel1Title = (props) => <Level2Title {...props} />;
const PageLevel2Title = (props) => <Level3Title {...props} />;

const ContentLevel1Title = (props) => <PageLevel2Title {...props} />;

ReactDOM.render(
  <ContentLevel1Title onClick={() => {}} />,
  document.getElementById('root'),
);
```

React 中达到类似效果就必须明确往下传递。

缺点：

- 后续维护过程中，难以判断接受 props 的组件收到的数据由谁传递进来的。
  父级组件也难以判断使用的子组件最终会将传递进去的数据最终透传到何处。
- 多个子组件透传属性名相同时会失效，而这是大概率会出现的行为。实际开发估计作用不大。

```vue
// page1-content.vue
<template>
  <content-title title="XXX" />
  <content-paragraph content="XXX" />
  <content-title title="XXX" />
  <content-paragraph content="XXX" />
</template>

//
<template>
  <page1-title />
  <!-- 以下不会工作 -->
  <page1-content @click="do something" />
  <!-- 需要包装一层, 或 page1-content 模版包装一层明确属性，透传的意义其实不大 -->
  <section @click="do something">
    <page1-content />
  </section>
</template>
```

个人观点：为了更好维护应尽可能不利用其特性，甚至明确禁用该功能

### 插槽 Slots

React 是由不同类型 “Node” 组合成 “Render Tree”，组合模式。

可以轻易的实现以下逻辑进行组件之间的组合使用：

```jsx
const Child1 = () => {};
const Child2 = () => {};
const Parent = ({ slot1, slot2, children: slot }) => (
  <div>
    <div>{slot1}</div>
    {slot}
    <div>{slot2}</div>
  </div>
);

ReactDOM.render(
  <Parent slot1={<Child1 />} slot2={<Child1 />}>
    <Child2 />
  </Parent>,
  document.getElementById('root'),
);
```

Vue 是基于模版的，必须指定插入的位置。（待确认）。

> Vue 组件的插槽机制是受原生 Web Component `<slot>` 元素的启发而诞生，同时还做了一些功能拓展

**Vue 考虑了 Web Component 标准，长期发展来看这可能是其一大优势。**
**例如，使用即将发布的语义化标签。**

```vue
// Layout.vue
<template>
  <div>
    <slot name="slot1"></slot>
  </div>
  <slot></slot>
  <div>
    <slot name="slot2" :title="123"></slot>
  </div>
</template>

// use-layout.vue
<template>
  <layout>
    <div />
    <div name="slot1"></div>
    <div name="slot2" v-slot="{ title }">
      {{ title }}
    </div>
  </layout>
</template>
```

Vue 的 slot 机制，可以轻松的传递自身的内部状态至插入的组件。
而 React 得利用 createElement 或 renderProps 机制实现类似效果。

### Provide / Inject

Provide/Inject 分别对应 React 中的 Context.Provider/Context.Customer。

主要解决组件层次过深的数据传递问题

由于 Vue 响应式数据的关系，Inject 的概念更贴切。

### Lazy Component

与 React 类似，`defineAsyncComponent` 对应 `React.lazy`

在其之上提供了 options 选项可以处理 loading/error 状态。

后续也提供了 Suspense 配合使用（Experimental Feature）。

个人看法：
Suspense 的方式更灵活些，可以包裹多个 Lazy Load 的组件。
同一页面异步组件过多时，产品逻辑上不适合每个组件都有 loading 状态

### Transition/TransitionGroup 组件

帮助制作基于状态变化的过渡和动画。对应 CSS Transition 理解。

```vue
<Transition
  name="fade"
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @after-enter="onAfterEnter"
  @enter-cancelled="onEnterCancelled"
  @before-leave="onBeforeLeave"
  @leave="onLeave"
  @after-leave="onAfterLeave"
  @leave-cancelled="onLeaveCancelled"
>
  <p v-if="show">hello</p>
</Transition>

<style scoped>
// 这里的 `.fade-` 前缀，取决于 Transition 属性 name 的值
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

React 的 useTransition hook 与该组件解决的问题完全不是一回事。

### KeepAlive

> 多个组件间动态切换时缓存被移除的组件实例。

避免组件实例重新挂载，是非常有效的功能。

react 参考 [react-activation](https://github.com/CJY0208/react-activation) 自行控制 DOM 更新达成类似效果。
或利用 CSS 属性 `display` 达成类似效果。

### Teleport

> 可以将一个组件内部的一部分模板“传送”到该组件的 DOM 结构外层的位置去。

Model/massage/Notify 等组件实现依赖。
React 类似功能为 [createPortal](https://react.dev/reference/react-dom/createPortal)

## 生命周期

对比 React 存在细微差别，但大差不差。
创建/挂载/更新/卸载。

## 响应式及双向数据绑定

> 在 JavaScript 中有两种劫持 property 访问的方式：getter / setters 和 Proxies。
> Vue 2 使用 getter / setters 完全是出于支持旧版本浏览器的限制。
> 而在 Vue 3 中则使用了 Proxy 来创建响应式对象，仅将 getter / setter 用于 ref
>
> Vue 的响应式系统基本是基于运行时的。
> 追踪和触发都是在浏览器中运行时进行的。

响应式数据绑定意味着当数据发生变化时，视图会自动更新，而双向数据绑定则允许数据在视图和模型之间进行双向同步。

子组件接收到的 props 实际上是 readonly，不能在子组件内部对 props 的值直接进行修改

v-model 使用上与 Angular 类似。
但是实际是基于响应式

## 渲染机制

![Vue Render Pipeline](https://cn.vuejs.org/assets/render-pipeline.sMZx_5WY.png)

根据 VUE 官方的描述，VUE 是 VNode + 编译器优化。
这也是 KeepAlive 组件能够实现的原因。

例如，“静态提升”、“Patch Flags（补丁标记）”、“Tree Flattening”

VUE 编译器的 “静态提升” 功能，则需要 React 开发人员基于经验进行优化。
而人为提升，从代码可读性角度又是不合理的。

React 是 Fiber + ReactElement 做了两层的数据结构。
然后提供了 shouldComponentUpdate/memo 避免创建新的 react element

VUE 协调（reconciliation）相比 React 在 list 的比较上进一步做了优化。

## 生态

以官方名义建立许多延展社区，包括但不限于

- 多语言文档（尤其是中文文档）
- 类似于 Next.js 的入门互动教学教程
- Vue Conf
- [Vue Discard](https://discord.com/invite/HBherRA)
- [Online Paly-Ground](https://play.vuejs.org/)
- [Vue Mastery](https://www.vuemastery.com/courses/)
- [Vue School](https://vueschool.io/)
- 等等

技术生态：

- [Vue Router](https://router.vuejs.org/zh/)（Official），路由
- [pinia](https://pinia.vuejs.org/zh/)，状态管理工具
- [Vue CLI](https://cli.vuejs.org/zh/)，类似于 CRA 的脚手架
- [Vite](https://cn.vitejs.dev/)，基于 rollup 的 nopack 工具。
- @vitejs/plugin-vue
- vue-loader，For Webpack
- @vue/compiler-sfc，vue 语法解释器
- quasar. 企业级跨平台 VueJS 框架. 同时支持 SPA/PWA/SSR
- Nuxt.js， SSR
- uni-app，国内小程序
- Vitest or jest + vite-jest. 单元测试
- eslint-plugin-vue
- Prettier 内置支持 vue
- [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) vscode extends
- UI 组件库
  - Web
    - [Vue Material](https://www.creative-tim.com/vuematerial)
      Material Design 风格的 Vue 实现版本
    - Ant-design-vue。
      Ant Design 风格的 Vue 实现
    - VuetifyVuetify
    - Vant 3.0。
      有赞前端团队开源的移动端组件库
    - Element Plus
      一套为开发者、设计师和产品经理准备的基于 Vue 3.0 的桌面端组件库
    - Naive UI
      一个 Vue 3 组件库，比较完整，主题可调，使用 TypeScript，不算太慢,有点意思
    - Quasar
      构建高性能的 VueJS 用户界面,开箱即用,支持桌面和移动浏览器（包括 iOS Safari！）
    - Arco Design Vue
      字节跳动基于 Arco Design 开源的 Vue UI 组件
    - Element3
      一套 Element 风格，为开发者、设计师和产品经理准备的基于 Vue 3.0 的桌面端组件
    - Varlet
      基于 Vue3 的 Material design 风格移动端组件库
    - Vue-devui
      基于 DevUI Design 的 Vue3 组件库，使用 pnpm + vite + vue3 + tsx 技术搭建
    - Idux
      一个基于 Vue 3.x 和 TypeScript 开发的开源组件库
    - NutUI 3
      京东风格的 Vue 移动端组件库，开发和服务于移动 Web 界面的企业级产品
  - 小程序
    - [FirstUI UNI](https://doc.firstui.cn/)（选项式 API）版组件库，
      是基于 uni-app 开发的一款轻量、全面可靠的跨平台移动端组件库。
    - [uView UI](https://github.com/umicro/uView2.0)，
      是 uni-app 全面兼容 nvue 的 uni-app 生态框架，全面的组件和便捷的工具会让您信手拈来，如鱼得水
    - [ThorUI](https://thorui.cn/doc/)
      轻量、简洁、全面的移动端组件库。能大幅提升开发效率，包含 uniapp 与微信小程序原生版本组件库！
    - Vant Weapp

Vue 不断吸纳各类别框架在不同方面的优势，
建立了非常完善的开源社区生态，
不断改善 DE 是其能够快速发展的重要因素

尤其在国内环境下，从人才储备的角度，Vue 的优势要远胜于 React。
进一步演化下来，相关工具链的中文文档也要远胜 React。
