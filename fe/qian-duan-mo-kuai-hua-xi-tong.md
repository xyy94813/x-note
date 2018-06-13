# 前端模块化系统

模块系统主要解决模块的定义、以来和导出。


## 现有的模块化规范

### `<script/>` 标签

最原始的 JavaScript 文件加载方式，如果把每一个文件看作是一个模块，那么他们的接口通常是暴露在全局作用域下，（定义在 window 对象中），不同模块的接口调用都在一个作用域中，一些复杂的框架，会使用命名空间的概念来组织这些模块的接口，例如 YUI 库。

缺陷：
* 全局作用域下容易造成变量冲突
* 文件只能按照`<script/>`的顺序加载
* 开发人员必须主管解决模块和代码库的以来关系
* 在大型项目中各种资源难以管理，长期累计的问题导致代码库混乱不堪



### CommonJS

服务器段的Node.js遵循CommonJS规范，该规范的核心思想是运行模块通过require方法来同步加载所要以来的其他模块，然后通过exports或module.exports来到处需要暴露的接口。2013年5月，Node.js的内核开发者已经废弃了该规范。

CommonJS是同步加载模块，其原理是将所有模块定义好，并通过id进行索引，这样就可以方便的在浏览器环境中解析。

优势：
* 服务器端模块便于重用
* [NPM](https://www.npmjs.com/)中已经有将近 20 万个可以使用模块包
* 简单并容易使用

缺点：
* 同步的模块加载方式不适合在浏览器环境中，同步意味着阻塞加载，浏览器资源是异步加载的
* 不能非阻塞的并行加载多个模块

实现：
* 服务器端的  [Node.js](http://www.nodejs.org/)
* [Browserify](http://browserify.org/)，浏览器端的 CommonJS 实现，可以使用 NPM 的模块，但是编译打包后的文件体积可能很大
* [modules-webmake](https://github.com/medikoo/modules-webmake)，类似 Browserify，还不如 Browserify 灵活
* [wreq](https://github.com/substack/wreq)，Browserify 的前身



### AMD

[Asynchronous Module Definition](https://github.com/amdjs/amdjs-api)规范其实只有一个主要接口`define(id?, dependencies?, factory)`，它要在声明模块的时候指定所有的依赖 dependencies，并且还要当做形参传到 factory 中，对于依赖的模块提前执行，依赖前置。

```js
define("module", ["dep1", "dep2"], function(d1, d2) {  
 return someExportedValue;  
});  
require(["module", "../file"], function(module, file) { 
 /* ... */ 
});
```

优点：
* 适合在浏览器环境中异步加载模块
* 可以并行加载多个模块

缺点：
* 提高了开发成本，代码的阅读和书写比较困难，模块定义方式的语义不顺畅
* 不符合通用的模块化思维方式，是一种妥协的实现

实现：
* [RequireJS](http://requirejs.org/)
* [curl](https://github.com/cujojs/curl)

### CMD

[Common Module Definition](https://github.com/cmdjs/specification/blob/master/draft/module.md)规范和 AMD 很相似，尽量保持简单，并与 CommonJS 和 Node.js 的 Modules 规范保持了很大的兼容性。

```js
define(function(require, exports, module) {
 var $ = require('jquery');
 var Spinning = require('./spinning');

 exports.doSomething = ...
 module.exports = ...

})
```

优点：
* 依赖就近，延迟执行
* 可以很容易在 Node.js 中运行

缺点：
* 依赖 SPM 打包，模块的加载逻辑偏重

实现：
* [Sea.js](http://seajs.org/)
* [coolie](https://github.com/cloudcome/coolie)

### UMD

[Universal Module Definition](https://github.com/umdjs/umd)规范类似于兼容 CommonJS 和 AMD 的语法糖，是模块定义的跨平台解决方案。

### ES6 模块

EcmaScript6 标准增加了 JavaScript 语言层面的模块体系定义。[ES6 模块](http://es6.ruanyifeng.com/#docs/module)的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。

```js
import "jquery";  
export function doStuff\(\) {}  
module "localModule" {}
```

优点：
* 容易进行静态分析
* 面向未来的 EcmaScript 标准

缺点：
* 原生浏览器端还没有实现该标准
* 全新的命令字，新版的 Node.js 才支持 (V9.0.0 以上版本)

实现：
* [Babel](https://babeljs.io/)

（本机浏览器chrome 52.x 版本不支持 ES6 的模块系统，据说 firefox 已经支持未验证）



## 期望的模块系统

可以兼容多种模块风格，尽量可以利用已有的代码，不仅仅只是 JavaScript 模块化，还有 CSS、图片、字体等资源也需要模块化。

### 前端模块加载

前端模块要在客户端中执行，所以他们需要增量加载到浏览器中。

模块的加载和传输，我们首先能想到两种极端的方式，一种是每个模块文件都单独请求，另一种是把所有模块打包成一个文件然后只请求一次。显而易见，每个模块都发起单独的请求造成了请求次数过多，导致应用启动速度慢；一次请求加载所有模块导致流量浪费、初始化过程慢。这两种方式都不是好的解决方案，它们过于简单粗暴。

分块传输，按需进行**懒加载**，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。

要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。

### 所有资源都是模块

在上面的分析过程中，我们提到的模块仅仅是指 JavaScript 模块文件。然而，在前端开发过程中还涉及到样式、图片、字体、HTML 模板等等众多的资源。这些资源还会以各种方言的形式存在，比如 coffeescript、 less、 sass、众多的模板库、多语言系统（i18n）等等。

如果他们都可以视作模块，并且都可以通过 require 的方式来加载，将带来优雅的开发体验，比如：

```js
require("./style.css");  
require("./style.less");  
require("./template.jade");  
require("./image.png");
```

### 静态分析

在编译的时候，要对整个代码进行静态分析，分析出各个模块的类型和它们依赖关系，然后将不同类型的模块提交给适配的加载器来处理。比如一个用 LESS 写的样式模块，可以先用 LESS 加载器将它转成一个 CSS 模块，在通过 CSS 模块把他插入到页面的`<style />`标签中执行。[Webpack](https://webpack.js.org/) 就是在这样的需求中应运而生。

同时，为了能利用已经存在的各种框架、库和已经写好的文件，我们还需要一个模块加载的兼容策略，来避免重写所有的模块。







