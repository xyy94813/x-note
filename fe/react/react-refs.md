# React Refs

React 提供了 refs 用于访问 DOM 节点或在 render 方法中创建的 React 元素。尽管在 React 单向数据流中， props 是父子组件交互的唯一方式。但是，某些特殊情况下仍需要在典型数据流外强制修改子组件或是 DOM 元素。

> Refs 不能用在无状态组件中

## 使用 Refs 的条件

* 处理焦点、文本选择或媒体控制
* 触发强制动画
* 集成第三方 DOM 库（AMap、Google Map）

> **不要过度使用 Refs。**实际上可以使用 Refs 很“方便”的去处理一些问题，例如通过 Refs 更新子组件的状态，但是提升 state 所在的组件层级或许会是更合适的做法。
>
> 如可以通过声明式实现，则尽量避免使用 Refs

## 创建和访问 Refs 

目前有三种方式能够创建 Refs，通过不同方式创建的 Refs 访问方式也不同。

* `React.createRefs()` API
* 回调 Refs
* 字符串类型的 Refs

> React.createRefs 创建的 Ref 在`componentDidMount()`之前的值均是`object` 只不过该对象的 current 属性此时为 `null`，其余两种则是 `undefined`

### React.createRefs

React v16.3.0 新增了`React.createRefs()` 用于创建 Refs。当一个 ref 属性被传递给一个`render`函数中的元素时，可以使用 ref 中的`current`属性对节点的引用进行访问。

```jsx
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
    }
    componentWillMount() {
        console.log({ ...this.refGenerateByAPI}); // { current: null }
    }
    componentDidMount() {
        this.myRef.current.focus();
    }
    render() {
        return (
            <input ref={this.myRef} type="text" />
        )
    }
}
```

### 回调 Refs

传入回调函数，并绑定到当前实例上。回调函数传入的值就是子元素的实例，可以直接访问。

```jsx
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this._bindRef = this._bindRef.bind(this);
    }
    _bindRef(input) {
        this.myRef = input; 
    }
    componentWillMount() {
        console.log(this.myRef); // undefined
    }
    componentDidMount() {
        this.myRef.focus();
    }

    render() {
        return (
            <input ref={this._bindRef} type="text" />
        )
    }
}
```

> 如果 ref 回调以内联函数的方式定义，在更新期间它会被调用两次，第一次参数是 null ，之后参数是 DOM 元素。这是因为在每次渲染中都会创建一个新的函数实例。因此，React 需要清理旧的 ref 并且设置新的。通过将 ref 的回调函数定义成类的绑定函数的方式可以避免上述问题，但是大多数情况下无关紧要。

### 字符串类型的 Refs

字符串类型的 Refs 是，最早的使用 refs 的方式，由于 String 类型的 refs 存在某些问题，这种使用方式非常有可能在未来的版本中被废弃。

通过这种方式声明的 Refs 最终能通过组件实例的`refs`属性的`{string}`属性进行访问，

```jsx
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
    }
    componentWillMount() {
        console.log(this.refs.myRef); // undefined
    }
    componentDidMount() {
        this.refs.myRef.focus();
    }
    render() {
        return (
            <input ref="myRef" type="text" />
        )
    }
}
```





