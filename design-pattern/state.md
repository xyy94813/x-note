# 状态模式

状态模式能在一个对象的内部状态变化时改变其行为，使其看上去就像改变了自身所属的类一样

## 适用场景

- 对象需要根据自身当前状态进行不同行为，同时状态的数量非常多且与状态相关的代码会频繁变更的话，可使用状态模式
- 某个类需要根据成员变量的当前值改变自身行为，从而需要使用大量的条件语句时，可使用该模式
- 相似状态和基于条件的状态机转换中存在许多重复代码时，可使用状态模式

## 优/缺点

优点：

- 单一职责原则。将与特定状态相关的代码放在单独的类中。
- 开闭原则。无需修改已有状态类和上下文就能引入新状态。
- 通过消除臃肿的状态机条件语句简化上下文代码。

缺点：

- 如果状态机只有很少的几个状态，或者很少发生改变，那么应用该模式可能会显得小题大作。

## 对比其他模式

- 桥接模式、状态模式和策略模式（在某种程度上包括适配器模式）模式的接口非常相似。
  都是基于组合模式 -- 将工作委派给其他对象
- 状态可被视为策略的扩展。两者都基于组合机制。
  策略使得这些对象相互之间完全独立，它们不知道其他对象的存在。
  状态模式没有限制具体状态之间的依赖，且允许它们自行改变在不同情景下的状态。

## 实现示例

```ts
class AudioPlayer {
  private state: AudioPlayerState;
  private ui: any;
  private volume: any;
  private playlist: any;
  private curSong: any;

  constructor() {
    this.state = new AudioPlayerState(this);
    this.ui = new UserInterface();
    this.ui.lockButton.onClick(this.clickLock);
    this.ui.playButton.onClick(this.clickPlay);
    this.ui.nextButton.onClick(this.clickNext);
    this.ui.prevButton.onClick(this.clickPrevious);
  }

  changeState(state: AudioPlayerState) {
    this.state = state;
  }

  clickLock() {
    this.state.clickLock();
  }
  clickPlay() {
    this.state.clickPlay();
  }
  clickNext() {
    this.state.clickNext();
  }
  clickPrevious() {
    this.state.clickPrevious();
  }

  //   startPlayback() {}
  //   stopPlayback() {}
  //   nextSong() {}
  //   previousSong() {}
  //   fastForward(time) {}
  //   rewind(time) {}
}

abstract class AudioPlayerState {
  constructor(player) {
    this.player = player;
  }

  clickLock(): void;
  clickPlay(): void;
  clickNext(): void;
  clickPrevious(): void;
}

class LockedState extends State {
  clickLock() {
    if (player.playing) {
      player.changeState(new PlayingState(player));
    } else {
      player.changeState(new ReadyState(player));
    }
  }
  clickPlay(): void {
    /*undo anything*/
  }
  clickNext(): void {
    /*undo anything*/
  }
  clickPrevious(): void {
    /*undo anything*/
  }
}

class ReadyState extends State {}
class PlayingState extends State {}
```
