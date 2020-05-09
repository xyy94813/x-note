# Hot Module Replacement

热模块更换（HMR）允许在运行时更新所有类型的模块，而无需完全刷新。
是 webpack 提供的最有用的功能之一。

## 启用 HMR

在 Webpack 集成了 webpack-dev-server 后，启用 HMR 变得更加简单。
只需要设置 `devServer.hot=true` 即可。

```js
module.exports = {
  entry: {
    app: './src/index.js',
  },
  devtool: 'inline-source-map',
  devServer: {
    contentBase: './dist',
    hot: true, // Only one config to open HMR
  },
  plugins: [
    // new CleanWebpackPlugin(['dist/*']) for < v2 versions of CleanWebpackPlugin
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Hot Module Replacement',
    }),
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

然后在模块內实现 `module.hot.accept` API

```js
// index.js
import _ from 'lodash';
import printMe from './print.js';

function component() {
  const element = document.createElement('div');
  const btn = document.createElement('button');

  element.innerHTML = _.join(['Hello', 'webpack'], ' ');

  btn.innerHTML = 'Click me and check the console!';
  btn.onclick = printMe;

  element.appendChild(btn);

  return element;
}

document.body.appendChild(component());

// 约定重于设计
if (module.hot) {
  module.hot.accept('./print.js', function () {
    console.log('Accepting the updated printMe module!');
    printMe();
  });
}
```

这样，当 `./print.js` 变更时就会触发 callback. 在 callback 內执行 `./print.js` 变更后需要的操作就可以实现 HMR

### 注意事项

热模块更换的处理函数实际逻辑要复杂的多。
上述示例中。HRM 后单击示例页面上的按钮，控制台仍在打印旧的 `printMe` 函数。
发生这种情况是因为按钮的 `onclick` 事件处理程序仍绑定到原始的 `printMe` 函数。

```js
// index.js
document.body.appendChild(component());
let element = component(); // Store the element to re-render on print.js changes
document.body.appendChild(element);

if (module.hot) {
  module.hot.accept('./print.js', function () {
    console.log('Accepting the updated printMe module!');
    //  printMe();
    document.body.removeChild(element);
    element = component(); // Re-render the "component" to update the click handler
    document.body.appendChild(element);
  });
}
```

### HMR with Stylesheets

`style-loader` 支持了 HMR。
当 CSS 依赖项更新时，此加载器在后台使用 `module.hot.accept` 修补 `<style>`标签。

## 原理

当 compiler 开始执行、解析和映射应用程序时，它会保留所有模块的详细要点。
这个数据集合称为 "manifest"，当完成打包并发送到浏览器时，runtime 会通过 manifest 来解析和加载模块。
无论你选择哪种 模块语法，那些 `import` 或 `require` 语句现在都已经转换为 `webpack_require` 方法，此方法指向模块标识符(module identifier)。
通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

通过 `WebpackManifestPlugin` 插件，可以将 `manifest` 数据提取为一个容易使用的 json 文件。

> CRA 也是基于 manifest 和 Workbox 来支持 PWA

通过使用内容散列(content hash)作为 bundle 文件的名称，这样在文件内容修改时，会计算出新的 hash，浏览器会使用新的名称加载文件，从而使缓存无效。

即使某些内容明显没有修改，某些 hash 还是会改变。这是因为，注入的 runtime 和 manifest 在每次构建后都会发生变化。
