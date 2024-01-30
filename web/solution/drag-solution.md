# Web 拖拽实现方案

## 基于 Drag and Drop API

HTML5 支持

| - 事件名称 - | - 型事件处理程序 - | - 触发时刻 -                                                            |
| ------------ | ------------------ | ----------------------------------------------------------------------- |
| drag         | ondrag             | 当拖拽元素或选中的文本时触发。                                          |
| dragend      | ondragend          | 当拖拽操作结束时触发 (比如松开鼠标按键或敲“Esc”键).                     |
| dragenter    | ondragenter        | 当拖拽元素或选中的文本到一个可释放目标时触发（见 指定释放目标）。       |
| dragleave    | ondragleave        | 当拖拽元素或选中的文本离开一个可释放目标时触发。                        |
| dragover     | ondragover         | 当元素或选中的文本被拖到一个可释放目标上时触发（每 100 毫秒触发一次）。 |
| dragstart    | ondragstart        | 当用户开始拖拽一个元素或选中的文本时触发（见开始拖拽操作）。            |
| drop         | ondrop             | 当元素或选中的文本在可释放目标上被释放时触发（见执行释放）。            |

Drag 和 Drop 之间可以通过 `DragEvent.dataTransfer.setData` 和 `DragEvent.dataTransfer.getData` 进行数据传输。
但是数据仅支持基本类型。

[兼容性参考](https://caniuse.com/dragndrop)

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <style>
      .droptarget {
        float: left;
        width: 100px;
        height: 35px;
        margin: 15px;
        padding: 10px;
        border: 1px solid #aaaaaa;
      }
    </style>
  </head>
  <body>
    <div class="droptarget" ondrop="drop(event)" ondragover="allowDrop(event)">
      <p ondragstart="dragStart(event)" draggable="true" id="dragtarget">
        拖动我!
      </p>
    </div>
    <div
      class="droptarget"
      ondrop="drop(event)"
      ondragover="allowDrop(event)"
    ></div>
    <p id="demo"></p>
    <script>
      function dragStart(event) {
        event.dataTransfer.setData("Text", event.target.id);
        document.getElementById("demo").innerHTML = "开始拖动 p 元素";
      }
      function allowDrop(event) {
        event.preventDefault();
      }
      function drop(event) {
        event.preventDefault();
        var data = event.dataTransfer.getData("Text");
        event.target.appendChild(document.getElementById(data));
        document.getElementById("demo").innerHTML = " p 元素已被拖动";
      }
    </script>
  </body>
</html>
```

[React 基于该 API 的简单示例](https://codesandbox.io/s/drag-list-demo-9nm4zl)

## 基于 鼠标事件和 getBoundingclientRect 等 API 进行计算

TODO

### 进一步基于 IntersectionObserver 判断是否相交

TODO
