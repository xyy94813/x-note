# Web 拖拽实现方案

## 基于 HTML 自带事件

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

## 基于 鼠标事件和 getBoundingclientRect 等 API 进行计算

TODO

### 进一步基于 IntersectionObserver 判断是否相交

TODO
