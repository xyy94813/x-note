# React HMR

## HMR？

HMR（Hot Module Replacement）是 webpack 最常用的共功能之一。

[Webpack HMR](/fe/webpack/webpack-hmr.md) 已经存在很长时间了。

## CRA HMR Client

CRA 的 `react-dev-utils` 实现了 `webpackHotDevClient.js` 来便于 developer 实现 HMR。

See: https://github.com/facebook/create-react-app/blob/master/packages/react-dev-utils/webpackHotDevClient.js

> after react-scripts v3.12.0.
> We can use env variable `FAST_REFRESH` to switch hot update.

`webpackHotDevClient.js` 会下载最新的 `manifest.json`，然后检测 “chunk link” 是否实现了 `module.hot.accept` API

如果没实现，则刷新页面：

```js
// Keep track of if there has been a runtime error.
// Essentially, we cannot guarantee application state was not corrupted by the
// runtime error. To prevent confusing behavior, we forcibly reload the entire
// application. This is handled below when we are notified of a compile (code
// change).
//
// See https://github.com/facebook/create-react-app/issues/3096
```

```log
Update propagation: ./src/views/Home.js -> ./src/common/router.js -> ./src/App.js -> ./src/index.js -> 0
    at hotApply (http://localhost:3000/static/js/bundle.js:525:30)
    at http://localhost:3000/static/js/bundle.js:363:22
```

## React-Hot-Loader

What is React-Hot-Loader do?

React-Hot-Loader 通过 HOC 的方式，注册组件需要 hot update.

当 module 变更后，通过 `forceUpdate` 的方式进行组件的更新。

这意味着我们不再需要自己去实现 `module.hot.accept` API

> React-Hot-Loader 很快将被 React Fast Refresh 取代.
>
> - React Native - supports Fast Refresh since 0.61.
> - parcel 2 - supports Fast Refresh since alpha 3.
> - webpack - no support yet, use React-Hot-Loader
> - other bundler - no support yet, use React-Hot-Loader

### with lazy-load component

因为 react-hot-loader update component instance 的行为是同步的。
但是 lazy-load component 的加载是异步的。
这将会导致 lazy-load component 的更新失败。

react-hot-load use a trick way to support lazy load。
See: https://github.com/gaearon/react-hot-loader/blob/5e226f40aea2f995adbb446199a9cb93656ef465/src/hot.dev.js#L54-L82
可以同过 `setConfig({ trackTailUpdates: false })` 关闭 tailUpdate 行为。

#### 注意

如果 lazy 组件 HMR 失败。
可以尝试使用 hot warp 那些可能被懒加载的组件

```js
// ./AsyncComp.js
import { hot } from 'react-hot-loader';

const AsyncComp = () => {};

// export default process.env.NODE_ENV === 'development' ? hot(AsyncComp) : AsyncComp;
export default hot(AsyncComp);
```

```js
const AsyncComp = React.lazy(() => import('./AsyncComp.js'));

const APP = () => {
  return <AsyncComp />;
};
```

> lazy component HMR 的问题正在修复中
> https://github.com/gaearon/react-hot-loader/pull/1448

另外：
lazy-component hmr 成功后会导致所有组件 re-mount

可以通过 [example](https://github.com/xyy94813/react-hot-loader-lazy-fixes) 体验

该行为貌似不在开发者的期待中
