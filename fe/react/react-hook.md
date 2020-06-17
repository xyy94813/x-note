# React Hook

React Hook 是 React 16.8 中的新增功能。它们使得无需编写类即可使用 state 和其他 React 功能。

Hook 的引入没有带来任何的 Break Change。

- 是完全可选择的。
- 100% 向后兼容
- 即刻使用

**Hook 的引入并不能完全替代 Class 组件。**

**Hook 不会改变对 React 概念的了解。**
相反，Hooks 基于现有的 React 概念提供了更直接的 API：道具，状态，上下文，引用和生命周期。

## 为什么需要 Hook

### 很难在组件之间重用状态逻辑

React 没有提供一种将可重用行为“附加”到组件的方法（例如，将其连接到 Store）。
此类问题可以采用 HOC 的模式进行解决，但是这些模式要求在使用组件时对其进行重组，这可能很麻烦，并使代码难以遵循。
如果查看 React DevTools 中的典型 React 应用程序，就会发现组件的 **“wrapper hell”**，这些组件被提供者，使用者，高阶组件，渲染道具和其他抽象层包围。
尽管可以在 DevTools 中过滤掉它们，但这指出了一个更深层的潜在问题：React 需要更好的原始操作来共享状态逻辑。

**使用 Hooks，可以从组件中提取状态逻辑，以便可以对其进行独立测试和重用。**
Hook 允许重用状态逻辑，而无需更改组件层次结构。
这使得在许多组件之间或与社区共享 Hook 变得容易。

例如 `react-async-hook`。

### 复杂的组件变得难以理解

常常不得不维护一些组件，这些组件起初很简单，但是发展成为状态逻辑和副作用难以控制的混乱状态。
每个生命周期方法通常包含不相关逻辑的混合。

例如，组件可能在 componentDidMount 和 componentDidUpdate 中执行某些数据获取。
但是，相同的 componentDidMount 方法可能还包含一些不相关的逻辑，这些逻辑用于设置事件侦听器，并在 componentWillUnmount 中执行清理。
在一起变化的相互关联的代码被分开，但是完全不相关的代码最终被合并为一个方法。

在许多情况下，由于有状态逻辑无处不在，因此无法将这些组件分解为较小的组件。测试将会变得困难。
这是许多人倾向于将 React 与单独的状态管理库（Redux）结合的原因之一。
**（Redux 的最大优势并不是全局的状态共享，而是将计算逻辑抽离出来单独测试。）**

但是，这通常会引入过多的抽象，需要在不同的文件之间跳转，并使重用组件更加困难。

为了解决这个问题，**Hooks 允许根据相关的部分（例如设置订阅或获取数据）将一个组件拆分为较小的功能**，而不是基于生命周期方法进行拆分。
还可以选择使用 reducer 管理组件的本地状态，以使其更加可预测。

### Class 会使人和机器混淆

除了使代码重用和代码组织变得更加困难之外，class 可能是学习 React 的一大障碍。
人们可以很好地理解 prop，state 和自上而下的数据流，但仍会遇到类问题。

JavaScript 中的 class 的实现是基于原型链的，这与大多数语言中的工作方式截然不同。

还需要理解 JavaScript 中的上下文的概念，这也是最近几年前端面试该问题激增的原因
如果没有不稳定的语法建议，代码将非常冗长。**（bind hell）**

```jsx
class MyComponent extends Component {
  consturctor(props) {
    super(props);
    this.state = {};
    this.fn = this.fn.bind(this); // bind hell!!!!!!!!
  }

  fn() {}
}
```

[组件的提前编译](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)具有很大的未来潜力。特别是不限于模板。

如果使用 Prepack 进行 [component folding](https://github.com/facebook/react/issues/7323)，可以带来一定程度的性能优化。
但是 class 组件鼓励无意识的模式，这种模式会使这些优化无效。

Class 组件还有其它类似的问题。
比如，Class 组件无法很好的 minify，使得热加载变得 容易碎片化（flaky）且不可靠。

为了解决这些问题，Hooks 让无需类即可使用 React 的更多功能。
从概念上讲，React 组件更接近 Function 组件。
Hook 拥抱 Function 组件，但不影响 React 的实践精神。
Hook 提供对命令式 “逃生舱” 的访问，并且不需要学习复杂的功能或反应性编程技术。

## 内置 Hook

Basic：

- useState
- useEffect
- useContext

Additional：

- useReducer
- useCallback
- useMemo
- useRef
- useImperativeHandle
- useLayoutEffect
- useDebugValue

### useState

`const [state, setState] = useState(initialedValue)`

example：

```jsx
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

React 保证了 setState 标识是稳定的并且不会在重新渲染的过程中更新。
所以可以在 useEffect 或 useCallback 依赖项中忽略。

如果更新函数返回的值与当前状态完全相同，则后续的重新渲染将被完全跳过。

> 与 class 组件 setState 不同的是，useState 不会自动合并更新对象。
> 可以通过 `setState(prevState => ({ ...prevState, ...updatedValues }))` 达到同样的目的，或者使用 useReducer 代替。

#### Lazy 初始值

initialState 参数是在初始渲染期间使用的状态。
在后续渲染中，将忽略它。
如果初始状态是昂贵的计算结果，则可以提供一个函数，该函数仅在初始渲染器上执行

```jsx
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

#### 摆脱状态更新

如果将 State Hook 更新为与当前状态相同的值（使用 `Object.is` 比较算法），React 将不渲染子级以及触发副作用。

请注意，React 可能仍需要在渲染之前再次渲染该特定组件。
如果渲染时要进行昂贵的计算，则可以使用 `useMemo` 进行优化 。

### useEffect

语法：

```js
useEffect(() => {
  // do effect

  // return a optional callback fn, clear effect in fn
  return function cleanup() {
    // clear effect
  };
}, [...dependencies]);
```

> `useEffect` 可以看作是 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 功能的结合。
> 但是 `useEffect` 在 React 的生命周期与上述三者有着极大的不同。
> `useEffect` 在每次渲染(after render)后运行!!!!!! 而不是 “mounting” 和 “updating”

React 组件有两种常见的副作用：不需要清除的副作用和需要清除的副作用。

#### 无需清理的副作用

在 React 更新 DOM 之后进行网络请求，手动变更 DOM 和日志记录是不需要清除副作用的常见示例。

example base on `useState` example:

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

> 与 `componentDidMount` 或不同 `componentDidUpdate`，预定的副作用 `useEffect` 不会阻止浏览器更新屏幕。
> 这使应用程序反应更快。因为大多数副作用不需要同步发生。
> 在不常见的情况下（例如测量布局），会有一个单独的 `useLayoutEffectHook`，其 API 与相同 `useEffect。`

#### 需清理的副作用

对某些外部数据源的订阅。

example：

```jsx
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

当组件卸载就会调用 cleanup

#### Effects 在每一次更新都会执行

可以从[上一个例子](#需清理的副作用)中发现，如果仅在组件 render 和卸载后执行副作用。
假设中途 `friend.id` 变化，则会导致最后一个变化之前的订阅均未取消，这将会导致内存泄漏。

以下是此组件随时间推移可能产生的一系列订阅和取消订阅调用：

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange); // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange); // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange); // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

#### 跳过副作用执行

可以通过 dependencies 跳过副作用执行

```jsx
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

> 注意：目前 dependencies 只会进行浅比较，对于复杂对象可以考虑 stringify 后进行比较. [optimize by use JSON.stringify ](https://github.com/facebook/react/issues/14476#issuecomment-471199055)

> 注意：如果存在 dependencies ，且 dependencies 出现变化，也会执行 “cleanup” 函数

将来，依赖项可能会通过构建时转换自动添加。

> 如果想运行一个 effect 并且仅将其清理一次（在挂载和卸载时），则可以传递一个空数组（[]）作为第二个参数。
> 这告诉 React 当前的 effect 不依赖于 props 或 state 的任何值，因此它不需要重新运行。

React 会推迟 useEffect 直到浏览器绘制完成后再运行，因此进行额外的工作不会有太大问题。

### useContext

语法：`const value = useContext(MyContext);`

组件使用 useContext 时，如果 Context Value 变化，组件调用将始终 re-render。
如果重新渲染组件很昂贵，则可以使用缓存来优化它。

```jsx
const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

### useReducer

`const [state, dispatch] = useReducer(reducer, initialArg, init);`

与 Redux 类似。
有复杂的状态逻辑，其中涉及多个子值，或者下一个状态取决于前一个值时，最好采用 useReducer 代替 useState

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

与 setState 一样，React 保证了 dispatch 标识是稳定的并且不会在重新渲染的过程中更新。
所以可以在 useEffect 或 useCallback 依赖项中忽略。

#### 初始化 useReducer 状态

有两种不同的方法来初始化 useReducer 状态。

`const [state, dispatch] = useReducer(reducer, {count: initialCount});`

> React 不使用 Redux 流行的 `state = initialState` 参数约定。
> 初始值有时需要依赖于 props，因此可以从 Hook 调用中指定。
> 可以调用 `useReducer（reducer，undefined，reducer` 来模拟 Redux 行为，但是不建议这样做。

以将 `init` 函数作为第三个参数传递。初始状态将设置为 `init（initialArg）`。

```jsx
const init = (initialCount) => ({ count: initialCount });

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({ type: 'reset', payload: initialCount })}
      >
        Reset
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

如果返回了与当前一样的状态（使用 `Object.is` 比较算法），将不会触发子组件更新或触发副作用。

### useCallback

`const memoizedCallback = useCallback(() => { do(a, b); }, [a, b])`;

返回一个 memoized 的函数，只有当 dependencies 变化才会返回新的 callback。

等同于 `useMemo(() => fn, deps)`

> 依赖性数组不会作为参数传递给回调。不过，从概念上讲，这就是它们所代表的含义：回调中引用的每个值也应出现在依赖项中。

### useMemo

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

返回一个 memoized 的 value。仅在其中一个依赖项已更改时才重新计算存储的值。

需要昂贵计算的值都可以采用 `useMemo` 进行优化。

### useRef

`const refContainer = useRef(initialValue);`

useRef 返回一个可变的 ref 对象， 该对象的 `.current` 属性已初始化为传递的参数（initialValue）。
返回的对象将在组件的整个生存期内持续存在。

```jsx
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

本质上，useRef 就像一个“盒子”，可以在其 `.current` 属性中保存可变的值。

通常使用其访问 DOM 节点。

但是，`useRef()` 不仅对 ref 属性有用。
与在类中使用实例字段的方式类似，保留任何可变值都很方便。

之所以有效，是因为 `useRef()` 创建了一个普通的 JavaScript 对象。
`useRef()` 和自己创建一个 `{current：...}` 对象之间的唯一区别是 `useRef` 将在每次渲染时提供相同的 `ref` 对象;

> 更改.current 属性不会导致重新渲染

### useImperativeHandle

`useImperativeHandle(ref, createHandle, [deps])`

useImperativeHandle 自定义使用 ref 时公开给父组件的实例值。
与往常一样，在大多数情况下应避免使用 ref 的命令性代码。
useImperativeHandle 应该与 forwardRef 一起使用：

```jsx
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
  }));
  return <input ref={inputRef} />;
}
FancyInput = forwardRef(FancyInput);
```

### useLayoutEffect

与 useEffect 类似，但在所有 DOM 挂载后都会同步触发。
一般使用它从 DOM 读取布局并同步重新渲染。
useLayoutEffect 有机会在浏览器绘制之前，计划在内部进行的更新将被同步刷新。

尽可能使用标准 `useEffect` 以避免阻塞视觉更新。

> `useLayoutEffect` 和 `componentDidMount` 以及 `componentDidUpdate` 在同一阶段触发。
> 即便如此，在从类组件进行迁移时，最好优先采用 `useEffect`.

> 如果使用 SSR，在下载 JavaScript 之前，它们 useLayoutEffect 和 useEffect 都不能运行。
> 这就是为什么当 SSR 的组件包含时 useLayoutEffect，React 会发出警告的原因
> 要解决此问题，请将该逻辑移至 useEffect（如果不需要第一次渲染），或者将显示该组件的时间延迟到客户端渲染之后（如果 HTML 在 useLayoutEffect 运行之前看起来很破）。

### useDebugValue

`useDebugValue(value)`

`useDebugValue` 可用于在 React DevTools 中显示自定义钩子的标签。

```jsx
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // Show a label in DevTools next to this Hook
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

> 不建议将调试值添加到每个自定义的挂钩。对于共享库中的自定义钩子而言，这最有价值。

## Hook 规则

Hook 是 JavaScript 函数，但是它们施加了两个附加规则：

- 仅在顶层调用 Hook。不要在循环，条件或嵌套函数中调用 Hook。
- 仅从 React 函数组件调用 Hooks。不要通过常规 JavaScript 函数调用 Hook。 （除了自定义 Hooks 内部）

可以在项目中使用 `eslint-plugin-react-hooks` 进行检查避免犯错。

**React 依赖于 Hook 的调用顺序来保证 useState 对应的 state 相同。**

这就是为什么必须在组件的顶层调用 Hook 且保持 Hook 执行顺序相同的原因。

## 自定义 Hook

自定义挂钩是自然遵循挂钩设计的约定，而不是 React 功能。

必须以 “use” 开头命名自定义挂钩。否则将无法自动检查是否违反了 Hooks 规则。

两个组件使用相同的 Hook 不共享状态吗

```js
function useFriendStatus(friendId) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendId, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendId, handleStatusChange);
    };
  }, [friendId]); // Only re-subscribe if friendId changes

  return isOnline;
}

function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }

  return isOnline ? 'Online' : 'Offline';
}
```

#### 推迟格式化 Debug 值

`useDebugValue(date, date => date.toDateString());`

## Hook 深入

// TODO

React 基于 Hook 的调用顺序来保证 hook 的返回值是正确的。

每个组件都有一个内部的“内存单元”列表。
它们只是 JavaScript 对象，React 在其中放置一些数据。
当调用 Hook 时，ie： `useState()`，它将读取当前单元格（或在第一个渲染期间对其进行初始化），然后将指针移至下一个单元格。
这就是多个 `useState()` 调用各自获得独立的本地状态的方式。

Hook 的值与当前需要进行 render 的 version 有关联。

## 对于 Hook 的思考

不需将所有的类组件迁移至函数组件。这会带来昂贵的成本，应该考虑在新的代码中试用。

长期来看，React 团队更期望贴近于 Function 组件，所以应该尽可能的使用 Function 组件。

Hook 并未覆盖所有的生命周期， 如 `getSnapshotBeforeUpdate` `getDerivedStateFromError` `componentDidCatch`。
如果需要使用这些生命周期，还是需要使用 class 组件。

应该 尽可能 使用 Hook 替代 HOC 模式，这能够减少 React DOM Tree 的嵌套。

Hook 极大程度的利用的 JavaScript 的闭包特性。
在现代浏览器中，闭包的原始性能与类相比，除极端情况外没有显着差异。
但是闭包可能会导致内存泄漏，这可能是 Hook 的缺陷之一。

虽然 React 官方指出，class 带来了额外的学习成本，但是很好的使用 Hook 所需的学习成本并不比之前要低。
初期的开发人员使用 Hook 可能更容易的产生 Bug。

比如，轻易的写出一个死循环:

```jsx
function AComp() {
  const [data, setData] = useState(null);
  const now = new Date();
  useEffect(() => {
    const getLast6HoursData = async (now) => {
      const data = await getData();
      setData(data);
    };
    getLast6HoursData();
    return () => {};
  }, [now]);
}
```

## 参考

[hooks-intro](https://reactjs.org/docs/hooks-intro.html)
[Hooks at a Glance](https://reactjs.org/docs/hooks-overview.html)
[Using the State Hook](https://reactjs.org/docs/hooks-state.html)
[Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html)
[Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
[Building Your Own Hooks](https://reactjs.org/docs/hooks-custom.html)
[Hooks API Reference](https://reactjs.org/docs/hooks-reference.html)
[Hooks FAQ](https://reactjs.org/docs/hooks-faq.html)
