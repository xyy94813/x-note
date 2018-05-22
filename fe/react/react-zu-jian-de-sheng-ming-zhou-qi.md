# React 组件的生命周期

每一个 React 组件的实例都拥有着一套完善的生命周期，主要分为挂载、更新和卸载三个大阶段，每一个阶段都拥有着更为细化的生命周期。

## 三个阶段与相关 API

![](https://pic4.zhimg.com/80/v2-610ad32e1ed334b3b12026a845e83399_hd.jpg)

### 挂载

挂载阶段相关的生命周期 API：

1. `static getDefaultProps`
2. `initialize state`
3. `constructor(props)`
4. `static getDerivedStateFromProps(nextProps, prevState)`_\(V 16.3.0\)_
5. `componentWillMount()` / `UNSAFE_componentWillMount()`
6. `render()`
7. `componentDidMount()`

### 更新

更新阶段相关的生命周期 API：

1. `componentWillReceiveProps(nextProps)` / `UNSAFE_componentWillreceiveProps(nextProps)`
2. `static getDerivedStateFromProps(nextProps, prevState)`_\(V 16.3.0\)_
3. `shouldComponentUpdate(nextProps, nextState)`
4. `componentWillUpdate(nextProps, nextState)` / `UNSAFE_componentWillUpdate(nextProps, nextState)`
5. `render()`
6. `getSnapshotBeforeUpdate(prevProps, prevState)`_\(V 16.3.0\)_
7. `componentDidUpdate(prevProps, prevState, snapshot)`

### 卸载

卸载阶段相关的生命周期 API：

1. `componentWillUnmount()`



## API 详情

#### static getDefaultProps

定义默认属性

```js
class MyComponent extends React.Component {
    static getDefaultProps = {}
    // ...
}
```

#### initialize state

初始化 state。

> 当前版本的官方文档\(V 16.3.0\)未说明该阶段。在 React V 16.0.0 之前，React.createClass 仍未废弃的情况下，还包含着这个阶段相关的 api。当前的官方文档中的示例，初始化`state`都是在`constructor`中完成。但是文档中说到_\(The constructor is the right place to initialize state. \_To do so, just assign an object to _`this.state`\)，\_猜测实际上仍存在该阶段

```js
class MyComponent extends React.Component {
    // ES7 语法实现初始化 state
    state = {}
    // ...
}
```

#### constructor\(props\)

构造器函数，在此阶段进行初始化 state 和事件处理器\(event handler\)的上下文绑定

> * `React.Component`的子类的构造器函数里，一定得调用`super(props)`，否则`this.props`将会是`undefined`
> * 不要在这个阶段调用this.setState\(\)，倘若要改变 state，只需要 state 对象进行 assign 操作即可
> * 如果，不需要初始化 state 也不打算给 class 实例绑定，事件处理器，就没有必要实现 `conostructor`
> * 在此阶段改变 state 时，不处理受到 props 影响的值，而是在`componentWillMount`或`getDerivedStateFrom`内处理

```js
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = {};
        this.btnClickHandler = this.btnClickHandler.bind(this);
    }
    btnClickHandler (e) {}
    // ...
}
```

#### static getDerivedStateFromProps\(nextProps, prevState\)

React v16.3.0新增 API。`getDerivedStateFromProps()`在组件初始化和接收新的 props 时调用。将会返回一个 object 去更新 state，或是返回一个`null`表示新的 props 不会导致 state 更新。

> * 无论 props 是否改变，该函数都会被调用
> * 通常情况下，`this.setState()`不会触发该函数
> * 一旦使用了这个新的生命周期函数，不安全的生命周期函数将不会被调用

```js
class MyComponent extends React.Component {
    static getDerivedStateFromProps(nextProps, prevState) {
        if (nextProp.value !== prevState.selectedVal) {
            return {
                selectedVal: nextProps.value,
            }
        }
        return null;
    }
    // ...
}
```

#### UNSAFE\_componentWillMount\(\)

`componentWillMount()` 的别名。在即将挂载的时候调用。由于该阶段在`render`之前，因此，在该阶段异步的调用`setState()`不会触发额外的渲染。在挂载之前，最后一次允许改变 state 的阶段。

> * `componentWillMount` 这个名称计划在 React V17.0.0 之后废弃掉。react 提供了 [rename-unsafe-lifecycles codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles) 这个工具去自动更新
> * 该生命周期函数会受到`static getDerivedStateFromProps()`和`getSnapshotBeforeUpdate()`的影响而无法使用

```js
class MyComponent extends React.Component {
    UNSAFE_componentWillMount() {
        if (this.props.mod === 'mod1') {
            this.setState({
                mod1State
            });
        }
    }
    // ...
}
```

#### componentDidMount\(\)

`componentDidMount()`在挂载完成后调用。在该阶段可进行加载远程数据，订阅事件，设置定时器等操作。

> 该阶段调用 setState\(\) 会触发额外的渲染，处理不当可能会导致死循环

```js
class MyComponent extends React.Component {
    _loadData() { // ... }
    componentDidMount() {
        this._loadData();
        this.autoLoadDataIntevalKey = setInterval(this._loadData, 30 * 1000);
        this.subscriptKey = subscriptNoticeMsg((msg) => {
            alert(msg);
        });
    }
    // ...
}
```

#### UNSAFE\_componentWillReceiveProps\(nextProps\)

`componentWillReceiveProps()`的新名称，在一个已经挂载的组件接收到新 props 时被调用。如果需要在此阶段根据 props 更新 state，最后先比较 `this.props`和`nextProps`后再调用`setState()`

> * 如果是父组件导致重新渲染，即使 props 没变更也会被调用
> * `componentWillReceiveProps`计划在 React V17.0.0 之后废弃掉
> * 该生命周期函数会受到`static getDerivedStateFromProps()`和`getSnapshotBeforeUpdate()`的影响而无法使用

```js
class MyComponent extends React.Component {
    componentWillReceiveProps(nextProps) {
        if (nextProps.value === this.props.value) {
            this.setState({
                selectedVal: nextProps.value,
            });
        }
    }
    // ...
}
```

#### shouldComponentUpdate\(nextProps, nextState\)

`shouldComponentUpdate()`在一个已经挂载的组件接收到新的 props 或 state 时被调用，其返回一个布尔值告诉 React 是否应该进行更新。默认情况下，每一次的状态变更都会进行重新渲染。

当`shouldComponentUpdate()`的返回`false`时，`UNSAFE_componentWillUpdate()`，`render()`和`componentDidUpdate()`都不会被调用。

对于`React.PureComponent`的子类，`shouldComponentUpdate()`对 props 和 state 进行了一次浅比较。

> 如果，`React.PureComponent`的子类重写了`shouldComponentUpdate()` React 在开发环境下会有以下警告：
>
> Warning: XXXX has a method called shouldComponentUpdate\(\). shouldComponentUpdate should not be used when extending React.PureComponent. Please extend React.Component if shouldComponentUpdate is used.

```js
class MyComponent extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
        for (let key in nextProps) {
            if (nextProps[key] === this.props[key]) {
                return true;
            }
        }
        for (let key in nextState) {
            if (nextState[key] === this.state[key]) {
                return true;
            }
        }
    }
    // ...
}
```

#### UNSAFE\_componentWillUpdate\(nextProps, nextState\)

`componentWillUpdate()`的新名称，在一个已经挂载的组件接收到新的 props 或 state，并且`shouldComponentUpdate()`返回`true`时被调用。这是在更新之前最后一次准备的机会。

> * 不要在这个阶段调用`this.setState()`，dispatch Redux 的 action 也一样，否则，可能会导致 React 组件在`UNSAFE_componentWillUpdate()`返回前就进行了一次更新。
> * `componentWillUpdate`计划在 React V17.0.0 之后废弃掉
> * 该生命周期函数会受到`static getDerivedStateFromProps()`和`getSnapshotBeforeUpdate()`的影响而无法使用

#### getSnapshotBeforeUpdate\(prevProps, prevState\)

React V16.3.0 新增的生命周期。在当前已挂载的组件调用 render\(\) 之后调用。该函数的返回值将作为`componentDidMount()`的第三个参数传递下去。

> 一旦使用了这个新的生命周期函数，不安全的生命周期函数将不会被调用

```jsx
class ScrollingList extends React.Component {
  listRef = React.createRef();

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the current height of the list so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      return this.listRef.current.scrollHeight;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    if (snapshot !== null) {
      this.listRef.current.scrollTop +=
        this.listRef.current.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

#### componentDidUpdate\(prevProps, prevState, snapshot\)

`componentDidUpdate()`在更新结束后立刻被调用，可以在这个阶段进行 DOM 的操作、网络请求等。

```js
class MyComponent extends React.Component {
  _loadData() {}
  componentDidUpdate(prevProps, prevState, snapshot) {
    if (prevProps.province !== this.props.province) {
      this._loadData();
      this.amap.setProvince(this.props.province)
    }
  }
  // ...
}
```

#### componentWillUnmount\(\)

当一个组件即将被卸载和销毁时会立刻调用`componentWillUnmount()`函数，在这个阶段可以进行取消网络、清空订阅等操作。

```js
class MyComponent extends React.Component {
  componentWillUnmount() {
    clearInterval(this.intervalKey);
    unsubscript(this.subscriptKey);
  }
  // ...
}
```



