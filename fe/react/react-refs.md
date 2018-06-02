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

## 转发 Refs（Forwarding Refs）

假设我们有一组件 A 经过`context.Consumer`之类的高阶组件或是 `withRouter`之类的函数封装成了 B，此时又希望能够使用 B 组件的时候，能够操作 A 的实例或对应的 DOM。

比如，现有一组件实现如下，并且希望对 ThemeInput 的实例进行 focus 的操作

```jsx
function ThemeInput(props) {
  return (
    <input className="theme-input" />
  );
}
```

这种情况下，可以使用 React v16.3.0 推出的新 API —— React.forwardRefs\(fn\) 重新实现 ThemeInput

```jsx
const ThemeInput = React.forwardRef((props, ref) => (
  <input ref={ref} className="theme-input" />
));

class XXXForm extends React.Component {
  consturctor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  componentDidMount() {
    this.myRef.current.focus();
  }
  render() {
    return (
      <ThemeInput ref={this.myRef} />
    );
  }
}
```

上述示例具体行为：

1. 我们通过调用 React.createRef 创建一个 React ref 并将其分配给一个 ref 变量。
2. 通过将它指定为 JSX 属性，我们将我们的 ref 传递给 `<ThemeInput ref = {ref}>`。
3. React 将 ref 传递给 forwardRef 中的`(props，ref) => {}`函数作为第二个参数。
4. 我们通过指定它作为一个JSX属性，将这个ref参数转发给 `<input ref = {ref}>`。
5. 当ref被附加时，ref.current 将指向`<input>`DOM节点。

> 第二个参数只有在您使用 React.forwardRef 调用定义组件时才存在。 常规的功能或类组件不会收到参数参数，并且参考也不可用于道具中。
>
> 引用转发不限于DOM组件。 您也可以将引用转发给类组件实例。

forwardRef 在 HOC 中的应用的 React 官方示例，通过额外的 `forwardedRef`属性结合 `React.forwardedRef()` 的方式，更加轻易的实现传递 refs 的功能

```jsx
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      return <Component ref={forwardedRef} {...rest} />;
    }
  }
  
  return React.forwardRef((props, ref) => (
    <LogProps {...props} forwardedRef={ref} />
  ));
}
```



