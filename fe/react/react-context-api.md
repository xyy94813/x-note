# React Context API

context 就是用来设计分享全局数据的。但是，官方并不推荐使（滥）用 `context` ，因为这是一个实验性的 API，在未来仍会变动，除非你打算实现一个类似于 React-Router、Redux 之类的全局组件。

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
            <span>{this.context.color}</span>
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
    static childContextTypes = {
        color: PropTypes.string
    }
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

## 旧版本 Context API Key 冲突

旧版本的 Context API 中，如果子组件的 父组件 A 和父组件 B 的 Context key 相同，会产生冲突

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class CComponent extends Component {

    static contextTypes = {
        color: PropTypes.string
    }

    render() {
        return (
            <span>{this.context.color}</span>
        );
    }
}

class MComponent extends Component {
    static childContextTypes = {
        color: PropTypes.string
    }

    getChildContext() {
        return {
            color: 'blue',
        }
    }

    render() {
        return (
            <CComponent onClick={this.props.onClick} />
        );
    }
}

class Root extends Component {
    static childContextTypes = {
        color: PropTypes.string
    }

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

## 新版本 Context API 的使用

React v16.3.0 推出了新的 Context API，其中包括：

* React.createContext\(defaultValue\) // 创建一个 `Context` 对象
* Provider // 容器组建，类似于 Redux 的 `Provider`
* Consumer // 消费组件，消费 Provider 提供的数据

```jsx
import React, { Component, createContext } from 'react';

export const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
};

export const ThemeContext = createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});

function ThemeTogglerButton(props) {
  // The Theme Toggler Button receives not only the theme
  // but also a toggleTheme function from the context
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button
          onClick={toggleTheme}
          style={{backgroundColor: theme.background}}>
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

// An intermediate component that uses the ThemedButton
class Toolbar extends Component {
  shouldComponentUpdate() {
    return false
  }
  render() {
    return (
      <ThemeTogglerButton onClick={this.props.changeTheme}>
        Change Theme
      </ThemeTogglerButton>
    );
  }
}

class App extends Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    }
  }

  render() {
    // The entire state is passed to the provider
    return (
      <ThemeContext.Provider value={this.state}>
        <Toolbar changeTheme={this.toggleTheme} />
      </ThemeContext.Provider>
    );
  }
}
```

新版本的 Context API 解决了之前提到的旧版本 Context API 存在的问题。

> _如果组件的层级只有几层的话，不建议采用 Context_

## 使用新版本 Context API 取代 Redux？

就目前来说，新版的 Context API 仍旧无法解决 Redux 能够解决的下问题：

* **逻辑/数据/视图分离的代码结构**
* **在不同项目之间通用的存储和事件机制**，从而允许`redux-devtools`这种通用的开发工具、以及类似`redux-observable`这种强大中间件的存在

## 新版 Context API 的其它问题

新版 Context API 仍存在其它值得思考的方向，由于不限制 Context 的数量，因此引发了以下几个问题。

* 多个 Context 如何管理
* 子组件消费多个 Context 导致的多层 Comsumer 嵌套问题
* 子组件更新 Context 值，需要向 `Provider` 传递 handler，这并不符合单一职责原则 
* 子组件触发多个 Context 状态更新，导致的多次更新问题



