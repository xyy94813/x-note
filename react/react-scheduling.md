# React Scheduling

**[React Scheduling](https://github.com/acdlite/react-fiber-architecture#scheduling)** 是 React v16 后 Reconciler (Fiber Reconciler) 的一部分，
指的是 React 何时确定何时应执行 **[work](https://github.com/acdlite/react-fiber-architecture#scheduling)** 的过程。

## 问题背景

这个问题看 Dan 在 JSConf 2018 的 [Part 1](https://www.youtube.com/watch?v=nLF0n9SACd4) 就能够理解.

由于浏览器中 JS 的单线程环境，需要面对以下问题：

1. **长时间运行的任务会导致帧丢失。**
   所以需要确保所有任务都较小，并且可以在几毫秒内完成，以便可以在一帧内运行它们。
2. **不同的任务具有不同的优先级。**
   优先考虑用户输入总体上会带来更好的体验。所以需要一种定义顺序并相应安排任务的方法。

React 采用了两种模式来解决上诉问题：

- **Concurrent React（时间切片）**。
  在渲染期间暂停和增量更新。类似与 Git Commit。
- **Scheduler。**
  在浏览器中注册具有不同优先级的回调。

> [~~Concurrent React~~](https://reactjs.org/docs/concurrent-mode-intro.html) ~~还在 Experimental 阶段~~
>
> [Concurrent Features](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react) 在 React 18 已经正式发布

## Scheduler

[Scheduler](https://github.com/facebook/react/blob/4c6470cb3b821f3664955290cd4c4c7ac0de733a/packages/scheduler)。
通用协作主线程 Scheduler 是由 React Core 团队开发的，可以在浏览器中注册具有不同优先级的回调。

Scheduler 目前支持的优先级：

https://github.com/facebook/react/blob/4c6470cb3b821f3664955290cd4c4c7ac0de733a/packages/scheduler/src/SchedulerPriorities.js

React 中的采用的 Scheduler 的优先级：

https://github.com/facebook/react/blob/4c6470cb3b821f3664955290cd4c4c7ac0de733a/packages/react-reconciler/src/SchedulerWithReactIntegration.new.js#L57-L63

- `Immediate` 用于需要同步运行的任务。
- `UserBlocking` （250 毫秒超时）用于应因用户交互（例如，单击按钮）而运行的任务。
- `Normal` （5s 超时），让您不必感到瞬间更新。
- `Low` （10 秒超时）用于可以推迟但最终仍必须完成的任务（例如，分析通知）。
- `Idle` （无超时）用于根本不需要运行的任务（例如，隐藏的屏幕外内容）。

> 制定超时时间以避免饥饿（starvation）问题。
> 即使有很多高优先级的工作要做，低优先级的工作也可以连续运行。

**Scheduler 会将所有已注册的回调存储在按到期时间（注册回调的时间加上优先级超时）的顺序排列的列表中。**
然后，Scheduler 将自己注册一个回调，该回调在浏览器绘制下一帧之后运行。
在此回调中，Scheduler 将执行尽可能多的已注册回调，直到需要渲染下一帧为止。

[当前实现中](https://github.com/facebook/react/blob/3e94bce765d355d74f6a60feb4addb6d196e3482/packages/scheduler/src/forks/SchedulerHostConfig.default.js#L115-L118)，Scheduler 通过在 `requestAnimationFrame()` 回调中使用 `postMessage()`实现的。

<!-- 处理完第一个按键事件后，浏览器会在其队列中看到未决事件，并决定在渲染框架之前运行事件侦听器。 -->

### Scheduler 的使用场景

仅仅在需要优化的时候才需要 Scheduler。
如果当前 APP 运行流畅，完全没有必要。

### 使用 Scheduler

```js
import {
  unstable_next,
  unstable_LowPriority,
  unstable_scheduleCallback,
} from "scheduler";

// 高消耗的 API，对其降级处理
function sendDeferredAnalyticsNotification(value) {
  unstable_scheduleCallback(unstable_LowPriority, function () {
    sendAnalyticsNotification(value);
  });
}

function SearchBox(props) {
  const [inputValue, setInputValue] = React.useState();

  function handleChange(event) {
    const value = event.target.value;

    setInputValue(value);
    unstable_next(function () {
      props.onChange(value);
      sendAnalyticsNotification(value);
    });
  }

  return <input type="text" value={inputValue} onChange={handleChange} />;
}
```

## Scheduler 的局限

使用 Scheduler，可以控制执行回调的顺序，并且可以在 Concurrent 模式下直接使用。

目前仍有两个限制：

- **资源斗争。**
  Scheduler 尝试使用所有可用资源。
  如果 Scheduler 的多个实例在同一线程上运行并争夺资源，则会导致问题。我们需要确保应用程序的所有部分都将使用相同的 Scheduler 实例。
- **使用户定义的任务与浏览器工作保持平衡。**
  由于 Scheduler 在浏览器中运行，因此它只能访问浏览器公开的 API。
  无法影响比如 Html 的渲染、回收周期。

[Scheduling API in the Browser](https://github.com/WICG/main-thread-scheduling) 就是为了解决该问题

## 参考

- [Beyond React 16 -- JSConf 2018](https://www.youtube.com/watch?v=nLF0n9SACd4)
- [scheduling-in-react](https://philippspiess.com/scheduling-in-react/)
- [React Codebase Overview](https://reactjs.org/docs/codebase-overview.html#fiber-reconciler)
- [精读《Scheduling in React》](https://www.jianshu.com/p/3b9545f338c7)
- [react-v18 change log](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react)
