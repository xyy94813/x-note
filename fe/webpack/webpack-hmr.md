# Hot Module Replacement

热模块更换（HMR）允许在运行时更新所有类型的模块，而无需完全刷新。
是 webpack 提供的最有用的功能之一。

主要是通过以下几种方式，来显著加快开发速度：

1. 保留在完全重新加载页面时丢失的应用程序状态。
2. 只更新变更内容，以节省宝贵的开发时间。
3. 调整样式更加快速 - 几乎相当于在浏览器调试器中更改样式。

> In webpack V5, `import.meta.webpackHot` is an alias for `module.hot` which is also available in strict ESM

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

### In Compiler

除了普通资源，编译器(compiler)需要发出 "update" 消息，以允许更新之前的版本到新的版本。"update" 由两部分组成：

1. 更新后的 manifest(JSON)
2. 一个或多个更新后的 chunk (JavaScript)

manifest 包括新的编译 hash 和所有的待更新 chunk 目录。每个更新 chunk 都含有对应于此 chunk 的全部更新模块（或一个 flag 用于表明此模块要被移除）的代码。

编译器确保模块 ID 和 chunk ID 在这些构建之间保持一致。通常将这些 ID 存储在内存中（例如，使用 webpack-dev-server 时），但是也可能将它们存储在一个 JSON 文件中

当 compiler 开始执行、解析和映射应用程序时，它会保留所有模块的详细要点。
这个数据集合称为 "manifest"，当完成打包并发送到浏览器时，runtime 会通过 manifest 来解析和加载模块。
无论选择哪种 模块语法，那些 `import` 或 `require` 语句现在都已经转换为 `webpack_require` 方法，此方法指向模块标识符(module identifier)。
通过使用 manifest 中的数据，runtime 将能够检索这些标识符，找出每个标识符背后对应的模块。

通过 `WebpackManifestPlugin` 插件，可以将 `manifest` 数据提取为一个容易使用的 json 文件。

> CRA 也是基于 manifest 和 Workbox 来支持 PWA

通过使用内容散列(content hash)作为 bundle 文件的名称，这样在文件内容修改时，会计算出新的 hash，浏览器会使用新的名称加载文件，从而使缓存无效。

即使某些内容明显没有修改，某些 hash 还是会改变。这是因为，注入的 runtime 和 manifest 在每次构建后都会发生变化。

### In APP

1. 应用程序代码要求 HMR runtime 检查更新。
2. HMR runtime（异步）下载更新，然后通知应用程序代码。
3. 应用程序代码要求 HMR runtime 应用更新。
4. HMR runtime（同步）应用更新。

可以设置 HMR，以使此进程自动触发更新，或者可以选择要求在用户交互时进行更新。

### In Module

HMR 是可选功能，只会影响包含 HMR 代码的模块。举个例子，通过 style-loader 为 style 样式追加补丁。
为了运行追加补丁，style-loader 实现了 HMR 接口；当它通过 HMR 接收到更新，它会使用新的样式替换旧的样式。

类似的，当在一个模块中实现了 HMR 接口，可以描述出当模块被更新后发生了什么。
然而在多数情况下，不需要强制在每个模块中写入 HMR 代码。
如果一个模块没有 HMR 处理函数，更新就会冒泡(bubble up)。
这意味着一个简单的处理函数能够对整个模块树(complete module tree)进行更新。
如果在这个模块树中，一个单独的模块被更新，那么整组依赖模块都会被重新加载。

### In HMR runtime

对于模块系统的 runtime，附加的代码被发送到 parents 和 children 跟踪模块。
在管理方面，runtime 支持两个方法 check 和 apply。

check 发送 HTTP 请求来更新 manifest。
如果请求失败，说明没有可用更新。
如果请求成功，待更新 chunk 会和当前加载过的 chunk 进行比较。
对每个加载过的 chunk，会下载相对应的待更新 chunk。
当所有待更新 chunk 完成下载，就会准备切换到 ready 状态。

> ./bootstrap.js ?????

apply 方法将所有被更新模块标记为无效。
**对于每个无效模块，都需要在模块中有一个更新处理函数(update handler)，或者在它的父级模块们中有更新处理函数。**
否则，无效标记冒泡，并也使父级无效。
每个冒泡继续，直到到达应用程序入口起点，或者到达带有更新处理函数的模块（以最先到达为准，冒泡停止）。
如果它从入口起点开始冒泡，则此过程失败。

之后，所有无效模块都被（通过 dispose 处理函数）处理和解除加载。
然后更新当前 hash，并且调用所有 `accept` 处理函数。
runtime 切换回闲置状态(idle state)，一切照常继续。
