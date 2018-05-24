# React Context API

React v16.3.0 正式推出了新的 Context API，其中包括：

* React.createContext
* Provider
* Consumer

context 就是用来设计分享全局数据的。但是，官方并不推荐使（滥）用 context ，因为这是一个实验性的 API，在未来仍会变动，甚至在将来（v17.0.0）会被删除，除非你打算实现一个类似于 React-Router、Redux 之类的全局组件。

## 旧版本的 context API 的 shouldComponentUpdate 问题

如果使用了上下文组件的父组件中`shouldComponentUpdate()`返回了`false`，则该子组件并不会更新。

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class CComponent extends Component {

    static contextTypes = {
        color: PropTypes.string
    }

    render() {
        return (
            <span>this.context.color</span>
        );
    }
}

class MComponent extends Component {
    shouldComponentUpdate() {
        return false;
    }
    render() {
        return (
            <CComponent />
        );
    }
}

class Root extends Component {
    getChildContext() {
        return {
            color: 'red',
        }
    }

    render() {
        return (
            <MComponent />
        );
    }
}
```

## 





