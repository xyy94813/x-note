# 命令模式

命令模式可将请求转换为一个包含与请求相关的所有信息的独立对象。
该转换让能根据不同的请求将方法参数化、延迟请求执行或将其放入队列中，且能实现可撤销操作。

## 适用场景

- 如果需要通过操作来参数化对象，可使用命令模式
- 如果想要将操作放入队列中、操作的执行或者远程执行操作，可使用命令模式
- 如果想要实现操作回滚功能，可使用命令模式

## 优/缺点

优点：

- 单一职责原则。可以解耦触发和执行操作的类
- 开闭原则。可以在不修改已有客户端代码的情况下在程序中创建新的命令
- 可以实现撤销和恢复功能
- 可以实现操作的延迟执行
- 可以将一组简单命令组合成一个复杂命令

缺点：

- 代码可能会变得更加复杂，因为在发送者和接收者之间增加了一个全新的层次。

## 对比其他模式

- 责任链模式、命令模式、中介者模式和观察者模式用于处理请求发送者和接收者之间的不同连接方式：
  - 责任链按照顺序将请求动态传递给一系列的潜在接收者，直至其中一名接收者对请求进行处理。
  - 命令在发送者和请求者之间建立单向连接。
  - 中介者清除了发送者和请求者之间的直接连接，强制它们通过一个中介对象进行间接沟通。
  - 观察者允许接收者动态地订阅或取消接收请求。
- 责任链的管理者可使用命令模式实现。在这种情况下，可以对由请求代表的同一个上下文对象执行许多不同的操作。
- 可以同时使用命令和备忘录模式来实现“撤销”。在这种情况下，命令用于对目标对象执行各种不同的操作，备忘录用来保存一条命令执行前该对象的状态。
- 命令和策略模式看上去很像，因为两者都能通过某些行为来参数化对象。但是，它们的意图有非常大的不同。
- 原型模式可用于保存命令的历史记录。
- 可以将访问者模式视为命令模式的加强版本， 其对象可对不同类的多种对象执行操作

## 实现示例

```ts
abstract class Command {
  private backup: any;
  private editor: Editor;

  constructor(editor: Editor) {
    this.editor = editor;
  }
  saveBackup(): void {
    this.backup = this.editor.current;
  }
  undo(): void {
    this.editor.current = this.backup;
  }
  execute(): void;
}

class CopyCommand extends Command {
  execute(): void {}
}

class CutCommand extends Command {
  execute(): void {}
}

class UndoCommand extends Command {
  private app: Application;
  constructor(app: Application) {
    this.app = app;
  }
  execute(): void {
    app.undo();
  }
}

class Editor {
  current: any;
}

class Application {
  commandHistory: Command[];
  executeCommand(command: Command) {
    command.execute();
    this.commandHistory.push(command);
  }
  undo() {
    this.commandHistory.pop()?.undo();
  }
}
```
