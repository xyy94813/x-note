# @babel/preset-env

`@babel/preset-env` 可以动态的引入目标目标环境所需的语法转换（以及可选的浏览器 polyfill）。
从而使得 JavaScript 包更小！

> `@babel/preset-env` 不支持属于 `stage-x` 插件

## 工作原理

`@babel/preset-env` 基于 [browserslist](https://github.com/browserslist/browserslist), [compat-table](https://github.com/kangax/compat-table), 以及 [electron-to-chromium](https://github.com/Kilian/electron-to-chromium) 等优秀等开源数据源。

Babel 团队利用这些数据源来维护哪个版本的受支持目标环境获得了 JavaScript 语法或浏览器功能的支持，
以及这些语法和功能到 Babel Transformation 插件和 [core-js polyfills](https://github.com/zloirock/core-js) 的映射。

`@babel/preset-env` 接受指定的任何目标环境，并根据其映射检查它们，以编译插件列表，并将其传递给 Babel。

## 与 Browserslist 集成

对于基于浏览器或 Electron 的项目，建议使用 `.browserslistrc` 文件指定目标。
因为该文件已被生态系统中的许多工具所使用，例如 `autoprefixer`，`stylelint`，`eslint-plugin-compat` 等。

默认情况下，除非设置了 `target` 或 `ignoreBrowserslistConfig` 选项，否则 `@babel/preset-env` 将使用 browserslist 配置源。

### 未指定目标环境

由于 `preset-env` 的最初目标之一是帮助用户轻松地从使用 `preset-latest` 过渡，因此当未指定目标时，
其行为类似：`preset-env` 会将所有 `ES2015-ES2020` 代码转换为与 `ES5` 兼容

> 不建议以这种方式使用预设环境，因为它没有利用针对特定环境/版本的功能。

```json
{
  "presets": ["@babel/preset-env"]
}
```

因此，`preset-env` 的行为与 `browserslist` 不同：当在 Babel 或 `browserslist` 配置中未找到目标时，它不使用 `defaults` 查询。
如果要使用 `defaults` 查询，则需要显式将其作为目标传递：

```json
{
  "presets": [["@babel/preset-env", { "targets": "defaults" }]]
}
```

## 可选配置项

[所有的配置项](https://babeljs.io/docs/en/babel-preset-env#options)

### useBuiltIns

`"entry"`

此选项启用了一个新插件，该插件可以根据环境将单独的 `core-js` 替换为 `import "core-js/stable";` 和 `import "regenerator-runtime/runtime"` 语句（
或 `require("core-js")`and `require("regenerator-runtime/runtime")`）。

```js
// input
import 'core-js';
```

```js
// output (based on environment)
import 'core-js/modules/es.string.pad-start';
import 'core-js/modules/es.string.pad-end';
```

`"usage"`

利用捆绑器将只加载一次相同的 polyfill。
在每个文件中使用 polyfill 时，添加特定的导入。

```js
// a.js
var a = new Promise();
```

```js
// b.js
var b = new Map();
```

```js
// a.ouput.js
import 'core-js/modules/es.promise';
var a = new Promise();
```

```js
// b.output.js

// if env support Map, next line will not exist.
// import "core-js/modules/es.map";
var b = new Map();
```

`false`

不要为每个文件自动添加 polyfill，也不转换 `import "core-js";` 或 `import "@babel/polyfill"`

## 示例

```js
// entry.js
class NameVector {
  constructor(...names) {
    this.names = names;
  }

  toString() {
    return this.names.reduceRight((str, name) => `${str},${name}`, '');
  }
}

const namsList = new NameVector('name1', 'name2', 'name3', 'name4');

const nameListProxy = new Proxy(nameList, {
  get(target, property) {
    if (property !== 'toString') {
      return target[property];
    }

    return () => {
      return 'Proxy' + target[property]();
    };
  },
});

console.log(namsList.toString());

const wkMap = new WeakMap();
wkMap.set({}, {});

const sdURL = new URL('https://www.shundaochuxing.com');
sdURL.searchParams.set('channel', 'qrcode');
```

```json
//.babelrc
{
  "presets": [
    [
      "@babel/env",
      {
        // use targets from `browserslistrc` if not set property `targets`
        // "targets": {
        //   "browsers": "> 0.25%, not dead",
        //   "edge": "12",
        //   "firefox": "40",
        //   "chrome": "37", // for andriod 5
        //   "safari": "9",
        //   "node": "current"
        // },
        "useBuiltIns": "usage", // 仅使可用部分
        "corejs": "3.6.4"
        // all options see: https://babeljs.io/docs/en/babel-preset-env#options
      }
    ]
  ]
}
```

```sh
#.browserslistrc
> 0.25%, not dead
current node
Edge 12
Firefox 40
Chrome 37
Safari 9
```

```js
// ouput.js
'use strict';

require('core-js/modules/es.array.concat');

require('core-js/modules/es.array.iterator');

require('core-js/modules/es.array.reduce-right');

require('core-js/modules/es.date.to-string');

require('core-js/modules/es.object.define-property');

require('core-js/modules/es.object.to-string');

require('core-js/modules/es.regexp.to-string');

require('core-js/modules/es.string.iterator');

require('core-js/modules/es.weak-map');

require('core-js/modules/web.dom-collections.iterator');

require('core-js/modules/web.url');

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError('Cannot call a class as a function');
  }
}

function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ('value' in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  return Constructor;
}

var NameVector = /*#__PURE__*/ (function () {
  function NameVector() {
    _classCallCheck(this, NameVector);

    for (
      var _len = arguments.length, names = new Array(_len), _key = 0;
      _key < _len;
      _key++
    ) {
      names[_key] = arguments[_key];
    }

    this.names = names;
  }

  _createClass(NameVector, [
    {
      key: 'toString',
      value: function toString() {
        return this.names.reduceRight(function (str, name) {
          return ''.concat(str, ',').concat(name);
        }, '');
      },
    },
  ]);

  return NameVector;
})();

var namsList = new NameVector('name1', 'name2', 'name3', 'name4');
var nameListProxy = new Proxy(nameList, {
  get: function get(target, property) {
    if (property !== 'toString') {
      return target[property];
    }

    return function () {
      return 'Proxy' + target[property]();
    };
  },
});
console.log(namsList.toString());
var wkMap = new WeakMap();
wkMap.set({}, {});
var sdURL = new URL('https://www.shundaochuxing.com');
sdURL.searchParams.set('channel', 'qrcode');
```

> `Proxy` 不存在 core-js 中，如果使用 `Proxy`，需要单独引入 [proxy-polyfill](https://github.com/GoogleChrome/proxy-polyfill)

## 参考

- [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env#how-does-it-work)
- [@babel/plugin-transform-runtime with corejs 3 polyfill issue](https://github.com/babel/babel/issues/10133#issuecomment-506249922)
