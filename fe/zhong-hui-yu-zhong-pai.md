# 重绘与重排

当浏览器下载完页面所有的资源后，浏览器的 HTML 解析器会开始构建 DOM Tree，同时 CSS 解析器会构建 Style Tree。合并这两个 Tree 上的信息并最终形成 Render Tree。

PS：RenderLayerTree???

![](https://user-gold-cdn.xitu.io/2018/11/13/1670b0b818763b3a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

一旦 Render Tree 构建完成，浏览器便开始绘制（paint）页面元素。

## 重排

当元素的几何属性发生变化，浏览器重新计算元素的几何属性的过程就称之为**重排**。
eg：改变了元素宽度变化，内容增加导致行数增加（高度增加）

> 涉及到盒模型的 CSS 属性基本会造成重排。

## 重绘

当元素发生变化浏览器将其重新绘制到屏幕上的过程称之为**重绘**。

> 重排一定会引起重绘，重绘不一定伴随重排。


## 关于重绘与重排优化的切入点

重绘和重排操作都是代价昂贵的操作，会消耗 WEB 应用的性能。

### 减少盒模型操作

```js

var $ele = document.querySelector('.ele');

// before
$ele.style.borderLeft = '1px';
$ele.style.borderRight = '2px';
$ele.style.padding = '5px';
// 大多数现代浏览器对上面代码进行了优化，但是老版浏览器可能导致严重的性能问题


// after
$ele.style.cssText='border-left: 1px; border-right: 2px; padding: 5px;' // 一次性修改

// or
$ele.className = 'newClass';
```

### 批量修改元素

基本思路：

* 让该元素脱离文档流
* 对其进行多重改变
* 将元素带回文档中

#### 隐藏元素，进行修改后，然后再显示该元素

```js

function appendNode($node, data) {
  var a, li;
  
  for(let i = 0, max = data.length; i < max; i++) {
    a = document.createElement('a');
    li = document.createElement('li');
    a.href = data[i].url;
    
    a.appendChild(document.createTextNode(data[i].name));
    li.appendChild(a);
    $node.appendChild(li);
  }
}

// before
let ul = document.querySelector('#mylist');
appendNode(ul, data);

// after
let ul = document.querySelector('#mylist');
ul.style.display = 'none';
appendNode(ul, data);
ul.style.display = 'block';

```

#### 使用文档片段创建一个子树，然后再拷贝到文档中

```js
let fragment = document.createDocumentFragment();
appendNode(fragment, data);
ul.appendChild(fragment);
```

#### 将原始元素拷贝到一个独立的节点中，操作这个节点，然后覆盖原始元素

```js
let old = document.querySelector('#mylist');
let clone = old.cloneNode(true);
appendNode(clone, data);
old.parentNode.replaceChild(clone, old);
```

#### others

* 移动通过 translate 来实现
* 浏览器原理中(root 节点，css position:relative，absolute，transform，有节点溢出 overflow，alpha mask，reflection，css filter，2dCanvas，WebGl，video) 都会为这些元素创造一个新的 Layer 以便浏览器加速渲染，这些元素的修改只会涉及自己重排重绘。

## Tools

[Site Report](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk)
