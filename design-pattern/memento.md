# 备忘录模式

备忘录模式允许在不暴露对象实现细节的情况下保存和恢复对象之前的状态。(快照模式)

## 适用场景

- 需要创建对象状态快照来恢复其之前的状态时
- 当直接访问对象的成员变量、获取器或设置器将导致封装被突破时

## 优/缺点

优点：

- 可以在不破坏对象封装情况的前提下创建对象状态快照
- 可以通过让负责人维护原发器状态历史记录来简化原发器代码

缺点：

- 客户端过于频繁地创建备忘录，程序将消耗大量内存
- 必须完整跟踪原发器的生命周期，这样才能销毁弃用的备忘录
- 绝大部分动态编程语言（例如 PHP、 Python 和 JavaScript）不能确保备忘录中的状态不被修改

## 对比其他模式

- 可以同时使用命令模式和备忘录模式来实现“撤销”。
  在这种情况下， 命令用于对目标对象执行各种不同的操作， 备忘录用来保存一条命令执行前该对象的状态。
- 可以同时使用备忘录和迭代器模式来获取当前迭代器的状态，并且在需要的时候进行回滚。
- 有时候原型模式可以作为备忘录的一个简化版本，条件是需要在历史记录中存储的对象的状态比较简单，不需要链接其他外部资源，或者链接可以方便地重建

## 实现示例

```ts
class Editor {
  private text: sting;
  private curX: number;
  private curY: number;
  private selectionWidth: number;
  setText(text: string) {
    this.text = text;
  }
  setCursor(x: number, y: number) {
    this.curX = x;
    this.curY = y;
  }
  setSelectionWidth(width: number) {
    this.selectionWidth = width;
  }
  createSnapshot(): Snapshot {
    return new Snapshot(
      this,
      this.text,
      this.curX,
      this.curY,
      this.selectionWidth,
    );
  }
}

class Snapshot {
  private editor: Editor;
  private text: sting;
  private curX: number;
  private curY: number;
  private selectionWidth: number;
  constructor(
    editor: Editor,
    text: sting,
    curX: number,
    curY: number,
    selectionWidth: number,
  ) {
    this.editor = editor;
    this.text = text;
    this.curX = curX;
    this.curY = curY;
    this.selectionWidth = selectionWidth;
  }

  restore(): void {
    this.editor.setText(this.text);
    this.editor.setCursor(this.curX, this.curY);
    this.editor.setSelectionWidth(this.selectionWidth);
  }
}
```
