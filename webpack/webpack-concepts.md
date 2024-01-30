# Webpack 基本概念

webpack 是用于现代 JavaScript 应用程序的**静态模块捆绑器（static module bundler）**。
当 webpack 处理应用程序时，它会在内部构建一个依赖关系图（Dependencies Graph），该图映射您项目所需的每个模块并生成一个或多个捆绑包（bundles）。

## 基本概念

- Entry
- Output
- Loaders
- Plugins

### Entry

一个 Entry Point 表明 webpack 应该使用哪个模块来开始构建它的依赖关系图。

webpack 递归地建立一个依赖关系图，其中包括你的应用程序所需要的每一个模块，然后将所有这些模块捆绑成一个小数量的捆绑包。

任何时候，当一个文件依赖于另一个文件时，webpack 都会将其视为依赖关系。

这使得 webpack 可以采用非代码资产，如图片或网页字体，并将其作为你的应用程序的依赖关系。

### Output

output 属性告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件。

### Loaders

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。
loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖关系图中。

### Plugins

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。
包括：打包优化，资源管理，注入环境变量。

### Mode

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。

Webpack 会根据 mode 自动进行一些默认的优化。
