# React-Hot-Loader

## What is Hot Module Loader

webpack has supported HRM for a long time

## CRA HMR Client

See: https://github.com/facebook/create-react-app/blob/master/packages/react-dev-utils/webpackHotDevClient.js

```js
// Keep track of if there has been a runtime error.
// Essentially, we cannot guarantee application state was not corrupted by the
// runtime error. To prevent confusing behavior, we forcibly reload the entire
// application. This is handled below when we are notified of a compile (code
// change).
// 
// See https://github.com/facebook/create-react-app/issues/3096
```

> after react-scripts v3.12.0.
> We can use env variable `FAST_REFRESH` to switch hot update.

CRA HMR Client will download newest manifest, and check the "chunk link" if implements `module.hot.recept`;

if not implement it got an error and reload page:

```
Update propagation: ./src/views/Home.js -> ./src/common/router.js -> ./src/App.js -> ./src/index.js -> 0
    at hotApply (http://localhost:3000/static/js/bundle.js:525:30)
    at http://localhost:3000/static/js/bundle.js:363:22
```

## React-Hot-Loader

What is React-Hot-Loader do?

React-Hot-Loader HOC to regesitry cur component need to hot update.

It means we let webpack dev clent known which on need hot reload.

### with lazy-load component

Because react-hot-loader sync update component instance,
but lazy-load component is lazy to load. 

So, react-hot-load use a trick way to support lazy load

See: https://github.com/gaearon/react-hot-loader/blob/5e226f40aea2f995adbb446199a9cb93656ef465/src/hot.dev.js#L54-L82

You can close it by `setConfig({ trackTailUpdates: false })`

If you lazy component hot reload fail.
You can try to wrap the component that would be lazy load.

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
    return <AsyncComp />
}
```

### with redux

Redux provider should not be re-render after updated.
It will create a new store, it will cause some problem

