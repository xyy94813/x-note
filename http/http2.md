## HTTP-2.0

HTTP-2 为 HTTP 语义提供了优化的传输，引进报头字段压缩。
同时支持多路复用来更有效利用网络资源、减少感知延迟。
该协议的重点是性能。特别是最终用户感知的延迟，网络和服务器资源的使用情况。

HTTP-2 向后兼容 HTTP-1.1 的所有核心功能，但旨在通过多种方式提高效率。

HTTP-2 的出现改变了目前业内表达 HTTP 的方式（HTTP-2 or HTTP1）。

> 对于 HTTP-2 与 HTTP-1 的理解可以类比 ES6，不是一次彻底的技术革新（not rewrited），而是一次巨大的变更。

## HTTP-1 的缺点

HTTP-1.1 是针对 90 年代的情况而不是现代 web 应用的性能而设计的，导致它的一些特点已经对现代应用程序的性能产生负面影响。

HTTP-1.0 只允许在一个连接上建立一个当前未完成的请求。

HTTP-1.1 管道只部分处理了请求并发和报头阻塞的问题。
因此客户端需要发起多次请求通过数次连接服务器来减少延迟。

HTTP-1.1 的报头字段经常重复和冗长。在产生更多或更大的网络数据包时，可能导致小的初始 TCP 堵塞窗口被快速填充。这可能在多个请求建立在一个新的 TCP 连接时导致过度的延迟。

## 约定和术语

所有数值均以网络字节顺序排列。除非另有说明，否则值是无符号的。适当时以十进制或十六进制提供文字值。十六进制文字以 0x 为前缀，以区别于十进制文字。

**keywords**：

- **客户（Client）**：
  启动 HTTP / 2 连接的端点。客户端发送 HTTP 请求并接收 HTTP 响应。
- **连接（Connection）**：
  两个端点之间的传输层连接。
- **连接错误（Connection Error）**：
  影响整个 HTTP / 2 连接的错误。
- **端点（Endpoint）**：
  连接的客户端或服务器。
- **帧（Frame）**：
  HTTP / 2 连接中的最小通信单元，由头和根据帧类型构造的八位字节的可变长度序列组成。
- **对等端（peer）：**
  一个端点。在讨论特定端点时，“对等”是指远离（Remote）讨论主要主题的端点？？？？？
- **接收端（Receiver）：**
  正在接收帧的端点。
- **发送端（Sender）：**
  正在传输帧的端点。
- **服务器（Server）：**
  接受 HTTP / 2 连接的端点。服务器接收 HTTP 请求并发送 HTTP 响应。
- **流（Stream）：**
  HTTP / 2 连接内的双向帧流。
- **流错误（Stream Error）：**
  单个 HTTP / 2 流上的错误。

## HTTP-2 帧

所有帧均以固定的 9 字节首部开头，后跟可变长度的有效载荷。

![Frame Layout Format](https://s1.ax1x.com/2020/05/02/JvMAne.jpg)

## HTTP-2 的版本标识

- `h2` 使用 TLS 安全协议
- `h2c` 明文传输，不使用安全协议

## HTTP-2 流和多路复用

> **多路复用(Multiplexing)**。数据通信系统或计算机网络系统中，传输媒体的带宽或容量往往会大于传输单一信号的需求。
> 为了有效地利用通信线路，希望一个信道同时传输多路信号，这就是所谓的多路复用技术。

### 流的特性

- 单个 HTTP-2 连接可以包含多个并发打开的流，其中任一端点都可以交错多个流中的帧。
- 流可以单方面建立和使用，也可以由客户端或服务器共享。
- 可以通过任一端点关闭流。
- 在流上发送帧的顺序很重要。收件人按接收顺序处理帧。特别地，`HEADERS` 和 `DATA` 帧的顺序在语义上很重要。
- 流由整数标识。通过端点启动流，将流标识符分配给流。

### 流的状态与生命周期

![HTTP2-Stream-Lifecycle](https://s1.ax1x.com/2020/05/03/YpQaGj.jpg)

流（Stream）具有以下状态：

- idle
- reserverd
  - reserved (local)
  - reserved (remote)
- open
- half-closed
  - half-closed (local)
  - half-closed (remote)
- closed

#### idle

所有流以 `idle(空闲)` 状态开始

此状态下，以下转换有效：

- 发送或接受一个 `HEADERS` 帧，使得流变成 `open`. 相同的 `HEADERS` 帧可以使得流立刻变成 `half-close`
- 发送一个 `PUSH_PROMISE` 帧保留一个空闲流，标识为以后使用的。Reserverd 流状态变成 "reserved (local)"
- 接受一个 `PUSH_PROMISE` 帧保留一个空闲流，标识为以后使用的。Reserverd 流状态变成 "reserved (remote)"

> 注意，PUSH_PROMISE 帧不是在空闲流上发送的，而是在 Promised Stream ID 字段中引用新保留的流。

#### Reserved （Local）

处于 “Reserved （Local）” 的流是一个已经通过发送 `PUSH_PROMISE` 帧 promised 的状态流。
`PUSH_PROMISE` 帧通过将流与由远程对等端发起的开放流关联保留空闲流？

此状态下，以下转换有效：

- 端可以发送 `HEADERS` 帧。这将导致流以“half-closed（Remote）”状态打开。
- 任一端均可发送 `RST_STREAM` 帧以使流变为“关闭”。将释保留的放流。

在这种状态下，端点不得发送除 `HEADERS`，`RST_STREAM` 或 `PRIORITY` 以外的任何类型的帧。

在这种状态下可以接收一个 `PRIORITY` 或 `WINDOW_UPDATE` 帧。
在流上接收除 `RST_STREAM`，`PRIORITY` 或 `WINDOW_UPDATE` 以外的任何类型的帧，都必须视为 `PROTOCOL_ERROR` 类型的连接错误

#### Reserved （Remote）

处于 “保留（远程）” 状态的流被远程对等端 保留

此状态下，以下转换有效：

- 接收 `HEADERS` 帧会使流转换为“Reserved （Local）”。
- 任一端点均可发送 `RST_STREAM` 帧以使流变为“关闭”。这将释放预存流。

端点可以在这种状态下发送一个 `PRIORITY` 帧来重新排序保留的流。在这种状态下，端点不得发送除 `RST_STREAM，WINDOW_UPDATE` 或 `PRIORITY` 以外的任何类型的帧。

在这种状态下在流上接收除 `HEADERS`，`RST_STREAM` 或 `PRIORITY` 以外的任何类型的帧，都必须视为 `PROTOCOL_ERROR` 类型的连接错误

#### Open

处于 “Open” 状态的流，两个对等端都可以使用来发送任何类型的帧。
在此状态下，发送对等端遵守的 **流级别（stream-level）** **流量控制(flow-control)**限制

从此状态，任一端点都可以发送设置了 `END_STREAM` 标志的帧，这将导致流转换为 “half-closed” 状态。
发送 `END_STREAM` 的端点使流状态变为 “half-closed(local)”；
接收 `END_STREAM` 的端点使流状态变为 “half-closed(remote)”。

任一端点都可以从该状态发送 `RST_STREAM` 帧，从而使其立即转换为“closed”。

#### half-closed(local)

处于“半封闭（本地）”状态的流，不能用于发送 `WINDOW_UPDATE`，`PRIORITY` 和 `RST_STREAM` 以外的帧。

当接收到包含 `END_STREAM` 标志的帧或任一对等端发送 `RST_STREAM` 帧时，流将从此状态转换为“closed” 。

端点可以在此状态下接收任何类型的帧。
要继续接收流控制的帧，必须使用 `WINDOW_UPDATE` 帧提供流控制验证。
在这种状态下，接收器可以忽略 `WINDOW_UPDATE` 帧，这些帧可能在发送带有 `END_STREAM` 标志的帧后的短时间内到达。

在此状态下接收的 `PRIORITY` 帧用于重新确定已标识流的流的优先级。

#### half-closed(remote)

处于 “half-closed(remote)” 的流，将不再用于对等端发送帧。
在这种状态下，端点不再必须维护接收方流量控制（flow-control）窗口。

处于此状态的流， 如果端点收到 `WINDOW_UPDATE`，`PRIORITY` 或 `RST_STREAM` 以外的其他帧，则它必须以流错误响应

端点可以使用“半封闭（远程）”流发送任何类型的帧。
在这种状态下，端点继续遵守通告的流级别（stream-level）流量控制（flow-control）限制。

#### closed

“closed”状态是最终的状态。

端点不得在 “closed” 流上发送除优先级以外的帧。

接收到 `RST_STREAM` 后接收到除 `PRIORITY` 以外的任何帧的端点必须将其视为 `STREAM_CLOSED` 类型的流错。
同样，在接收到 `END_STREAM` 帧之后接收任何帧的端点务必将其视为 `STREAM_CLOSED` 类型的连接错误。

发送包含 `END_STREAM` 标志的 `DATA` 或 `HEADERS` 帧后，可以在此状态下短时间内接收 `WINDOW_UPDATE` 或 `RST_STREAM` 帧。
在远程对等端接收并处理 `RST_STREAM` 或带有 `END_STREAM` 标志的帧之前，它可能会发送 `WINDOW_UPDATE` 或 `RST_STREAM` 帧。
端点必须忽略在这种状态下接收到的 `WINDOW_UPDATE` 或 `RST_STREAM` 帧.
尽管端点可以选择将发送 `END_STREAM` 之后很长时间到达的帧视为 `PROTOCOL_ERROR` 类型的连接错误。

可以在封闭流上发送 `PRIORITY` 帧，以区分依赖于封闭流的流的优先级。
端点应该处理 `PRIORITY` 帧。但是如果从依赖关系树中删除了流，则可以忽略它们。

如果由于发送 `RST_STREAM` 帧而达到此状态，则接收 `RST_STREAM` 的对等端可能已经在无法撤消的流上发送了（或已排队发送）帧。
端点发送 `RST_STREAM` 帧后，必须忽略其在关闭流上接收的帧。
端点可以选择限制其忽略帧的时间段，并将在此时间之后到达的帧视为错误。

发送 `RST_STREAM` 之后收到的流控制的帧（例如 DATA）被计入连接流控制窗口。
即使这些帧可能会被忽略，因为它们是在发送方接收 `RST_STREAM` 之前发送的，因此发送方将考虑将这些帧计入流控制窗口。

端点在发送 `RST_STREAM` 之后可能会收到 `PUSH_PROMISE` 帧。
即使相关联的流已被重置，`PUSH_PROMISE` 也会使流变为“保留”。
因此，需要 `RST_STREAM` 来关闭不需要的承诺流。

### 流（Stream）标识符

流用无符号的 31 位整数标识。
客户端发起的流必须使用奇数流标识符。
由服务器启动的那些务必使用偶数流标识符
流标识符零（0x0）用于连接控制消息；流标识符零不能用于建立新流。

升级到 HTTP-2 的 HTTP-1.1 请求将以流标识符 1（0x1）进行响应。
升级完成后，流 0x1 被 “half-closed(local)” 到客户端。
因此，从 HTTP-1.1 升级的客户端无法将流 0x1 选择为新的流标识符。

新建立的流的标识符必须在数值上大于发起端点已打开或保留的所有流。
这控制使用 `HEADERS` 帧打开的流和使用 `PUSH_PROMISE` 保留的流。
接收到意外流标识符的端点必须以 `PROTOCOL_ERROR` 类型的连接错误做出响应。

首次使用新的流标识符会隐式关闭处于 “idle” 状态的所有流，该流可能已由该对等端使用值较低的流标识符启动。

> 例如，如果客户端在流 “7” 上发送 `HEADERS` 帧而没有在流 “5” 上发送帧，则当发送或接收流 “7” 的第一个帧时，流 5 转换为 “closed” 状态。

流标识符不能重复使用，长期连接会导致端点耗尽可用范围的流标识符。
无法建立新的流标识符的客户端可以为新的流建立新的连接。
无法建立新流标识符的服务器可以发送 `GOAWAY` 帧，以便客户端被迫为新流打开新连接。

### 流并发

对等端可以使用 `SETTINGS` 帧内的 `SETTINGS_MAX_CONCURRENT_STREAMS` 参数来限制并发活动流的数量.
最大并发流设置特定于每个端点，并且仅适用于接收该设置的对等端。

> 即，客户端指定服务器可以启动的并发流的最大数量；服务器指定客户端可以启动的并发流的最大数量。

处于 “open” 状态或处于 “half-closed” 状态中的任一流都计入允许端点打开的最大流数.
这三种状态中的任何一种流都将计入 `SETTINGS_MAX_CONCURRENT_STREAMS` 设置中公布的限制。

端点不得超过其对等端设置的限制。
接收到导致其通告的并发流限制被超过的 `HEADERS` 帧的端点必须将此视为 `PROTOCOL_ERROR` 或 `REFUSED_STREAM` 类型的流错误.

### 流量控制

使用流进行多路复用会导致对 TCP 连接的使用产生争用，从而导致流阻塞。
流量控制方案可确保同一连接上的流不会造成相消干扰。
流控制既用于单个流，也用于整个连接。

HTTP-2 通过使用 `WINDOW_UPDATE` 框架来提供流控制。

#### 流量控制原理

流量控制算法必须满足以下特性：

1. **流量控制特定于链接**。流控制的类型在单个跃点（hop）端之间，而不是 e2e 路径上。？？？
2. **流量控制基于 `WINDOW_UPDATE` 帧**。接收方通告它们准备在流上以及整个连接中接收多少个字节（octets ）。这是基于信用的方案。
3. **流量控制是方向性的，由接收器提供总体控制**。接收者可以选择设置每个流和整个连接所需的任何窗口（window）大小。发送方必须遵守接收方施加的流量控制限制。客户端，服务器和（intermediaries ）都独立地将其流控制窗口发布为接收方，并在发送时遵守其对等方设置的流控制限制。
4. **对于新流和整个连接，流控制窗口的初始值为 65,535 字节（octets）**
5. **帧类型确定流控制是否适用于帧**。仅 `DATA` 帧受流控制，仅在 `DATA` 帧中受流控制。所有其他帧类型都不会占用通知（advertised）流控制窗口中的空间。这确保了重要的控制框架不会被流控制阻塞。
6. **流量控制无法禁用**。
7. HTTP-2 仅定义 `WINDOW_UPDATE` 帧的格式和语义。

流量控制算法的实现还负责管理如何更具优先级发送请求和响应，选择如何避免对请求的行头阻塞以及管理新流的创建。

#### 适当使用 flow-control

流控制解决了接收器无法处理一个流上的数据，但希望继续处理同一连接中的其他流的问题。
例如，代理需要在许多连接之间共享内存，并且可能具有较慢的上游连接和较快的下游连接。

但是，如果在不了解带宽延迟乘积的情况下启用了流量控制，则会导致可用网络资源的最佳使用。

**使用流控制时，接收者务必及时从 TCP 接收缓冲区中读取。如果未读取并执行关键帧（例如 `WINDOW_UPDATE`），则可能导致死锁。**

### 流的优先级（Stream Priority）

客户端可以通过在打开该流的 `HEADERS` 帧中包含优先级信息来为新流分配优先级.
优先级排序的目的是允许端点表达在管理并发流时希望其对等方分配资源的方式。
当发送能力有限时，可以使用优先级来选择要发送帧的流。

可以通过将流标记为依赖于其他流的完成来区分流的优先级。
每个依赖项被分配一个相对权重，该数字用于确定分配给依赖于相同流的流的可用资源的相对比例。

#### 流依赖

每个流可以被赋予对另一个流的显式依赖。包括依赖性表示优先选择将资源分配给所标识的流而不是依赖性流。

**不依赖于任何其他流的流的流依赖性为 0x0**。换句话说，不存在的流 0 形成树的根。

依赖于另一个流的流是从属流。流所依赖的流是父流。对当前不在树中的流（例如处于“空闲”状态的流）的依赖性导致该流被赋予默认优先级

在分配对另一个流的依赖关系时，该流将作为父流的新依赖关系添加。
共享同一父对象的从属流彼此之间没有顺序。
例如，如果流 B 和 C 依赖于流 A，并且如果创建的流 D 依赖于流 A，则这将导致 A 的依赖顺序，然后是 B，C 和 D 的任何顺序。

![Default Dependency Creation](https://s1.ax1x.com/2020/05/03/Yp9076.jpg)

排他标志允许插入新级别的依赖关系。
如果创建的流 D 具有对流 A 的排他性依赖关系，则这将导致 D 成为 B 和 C 的依赖关系父级。

![Exclusive Dependency Creation](https://s1.ax1x.com/2020/05/03/Yp92jA.jpg)

在依赖关系树内部，只有在依赖流所有依赖流（父流链最高为 0x0）关闭或无法对其进行处理时，才应为其分配资源。

#### 依赖权重

为所有相关流分配了 1 到 256（$$ 2^{8} $$） 之间的整数权重。

具有相同父项的流应根据其权重按比例分配资源。
因此，如果流 B 依赖于权重为 4 的流 A，流 C 依赖于权重为 12 的流 A，并且流 A 无法进行任何处理，则流 B 理想地接收分配给流 C 的资源的三分之一

```
B(4) ---|
        |----> A ---> X
C(12)---|
```

#### 重新排序

可以使用 `PRIORITY` 帧更改流优先级。设置依赖关系会导致流变得依赖于已标识的父流。

如果重新确定父级的优先级，则从属流将与其父级流一起移动。
使用重排标志为排定优先级的流设置从属关系会导致新父流的所有从属关系都变为已排定优先级的流。
如果使流依赖于其自身的依赖关系之一，则首先将先前依赖的流移动为依赖于重新排序优先级的流的先前父对象。

例如，考虑一个原始的依赖树，其中 B 和 C 依赖于 A，D 和 E 依赖于 C，F 依赖于 D。
如果使 A 依赖于 D，则 D 代替 A。通常所有其他依赖关系保持不变相同。如果 A 排他，则 F 依赖于 A。

![Dependency Reordering](https://s1.ax1x.com/2020/05/03/Ypi5Ax.jpg)

#### 优先状态管理

从依赖关系树中删除流时，可以将其依赖关系移动为依赖于 closed 流的父级。
通过基于封闭流依赖关系的权重按比例分配封闭流的依赖关系权重，可以重新计算新依赖关系的权重。

**从依赖关系树中删除的流会导致某些优先级信息丢失**。
资源在具有相同父流的流之间共享，这意味着，如果该集中的流关闭或被阻塞，则分配给该流的任何备用容量都将分配给该流的直接邻居。但是，如果从树中删除了公共依存关系，则这些流与下一个最高级别的流共享资源。

> 假设流 A 和 B 共享父级，而流 C 和 D 都依赖于流 A。
> 在删除流 A 之前，如果流 A 和 D 无法继续，则流 C 接收专用于流 A 的所有资源。
> 如果从树中删除了流 A，则将流 A 的权重分配到流 C 和 D 之间。
> 如果流 D 仍然无法进行，则导致流 C 接收到的资源比例减少。
> 对于相等的起始权重，C 接收可用资源的三分之一而不是一半。

**在传输依赖于该流的优先级信息时，流可能会关闭。**
如果在依赖项中标识的流没有关联的优先级信息，则将为依赖项流分配默认优先级。
这可能会产生次优的优先级，因为可以为流赋予与预期的优先级不同的优先级。

所以，**端点应该在流关闭后的一段时间内保留流优先级状态。状态保留的时间越长，为流分配不正确或默认优先级值的机会就越小。**
同理，处于“idle”状态的流可以被分配优先级或成为其他流的父级。

对于未计入 `SETTINGS_MAX_CONCURRENT_STREAMS` 设置的限制的流，保留优先级信息可能会给端点造成很大的状态负担。
因此，可以限制保留的优先级状态的数量。

端点为优先级维护的其他状态的数量可能取决于负载。
在高负载下，可以丢弃优先级状态以限制资源承诺。
在极端情况下，端点甚至可能丢弃活动或保留流的优先级状态。
如果应用了限制，则端点应至少保持其 `SETTINGS_MAX_CONCURRENT_STREAMS` 设置所允许的流数量。
实现也应该尝试为优先级树中正在使用的流保留状态。

#### 默认优先级

最初，为所有流分配对流 `0x0` 的非排他性依赖关系。
推送流最初取决于其关联的流。
在这两种情况下，流均被分配默认权重 16。

### 错误处理

HTTP-2 存在两大类错误：

- 使整个连接无法使用的错误条件是连接错误。
- 单个流中的错误是流错误

连接错误是指阻止帧层进一步处理或破坏任何连接状态的任何错误。

流错误是与特定流相关的错误，不影响其他流的处理。

如果在流保持 “open” 或 “half-closed” 状态时关闭或重置 TCP 连接，则无法自动重试受影响的流。

## 帧的定义

[More Details](https://httpwg.org/specs/rfc7540.html#FrameTypes)

### DATA

`DATA` 帧（type=`0x0`）传达与流相关的任意长度可变的八位字节序列。

### HEADERS

`HEADERS` 帧（`type=0x1`）用于打开流，并另外带有标题块片段。
`HEADERS` 帧可以在 “idle”，“reversed (local)”，“open” 或 “half-closed（remote）”状态下在流上发送

`HEADERS` 帧必须与流关联。

#### PRIORITY

`PRIORITY` 帧（`type=0x2`）指定发送方建议的流优先级
它可以以任何流状态发送，包括空闲或关闭的流。

#### RST_STREAM

`RST_STREAM` 帧（`类型=0x3`）允许立即终止流。发送 `RST_STREAM` 以请求取消流或指示发生了错误情况。

#### SETTINGS

`SETTINGS` 帧（`类型=0x4`）传达影响端点通信方式的配置参数.
例如，对等方行为的偏好和约束。

`SETTINGS` 框架还用于确认已接收这些参数。单独地，`SETTINGS`

##### SETTINGS 可设置的参数

- `SETTINGS_HEADER_TABLE_SIZE（0x1）`：允许发送方以字节为单位通知远程端点用于解码标头块的标头压缩表的最大大小。
- `SETTINGS_ENABLE_PUSH（0x2）`：此设置可用于禁用服务器推送
- `SETTINGS_MAX_CONCURRENT_STREAMS（0x3）`：指示发送方允许的最大并发流数。
- `SETTINGS_INITIAL_WINDOW_SIZE（0x4）`：指示用于流级别流控制的发送方的初始窗口大小（以八位字节为单位）。
- `SETTINGS_MAX_FRAME_SIZE（0x5）`：指示发送方愿意接收的最大帧有效负载的大小
- `SETTINGS_MAX_HEADER_LIST_SIZE（0x6）`：此建议设置以八位字节的形式通知对等方发件人准备接受的报头列表的最大大小。

#### 设置同步

SETTINGS 帧中的值必须按它们出现的顺序进行处理，而值之间不得进行其他帧处理。
不支持的参数必须被忽略。
一旦处理完所有值，接收者必须立即发出设置了 ACK 标志的 SETTINGS 帧。
在接收到设置了 ACK 标志的 SETTINGS 帧后，更改后的参数的发送方可以依赖于已应用的设置。

### PUSH_PROMISE

`PUSH_PROMISE` 帧（`类型=0x5`）用于在发送方打算初始化的流之前通知对等端终结点。

`PUSH_PROMISE` 帧只能在处于 “open” 或 “half-closed（remote）” 状态的对等端发起的流上发送。
`PUSH_PROMISE` 帧的流标识符指示与之关联的流。如果流标识符字段指定值为 `0x0`，则接收者必须以 `PROTOCOL_ERROR` 类型的连接错误进行响应。

承诺的流不需要按承诺的顺序使用。`PUSH_PROMISE` 仅保留流标识符供以后使用。

### PING

`PING` 帧（`类型=0x6`）是一种机制，用于测量来自发送方的最小往返时间，以及确定空闲连接是否仍然有效。
可以从任何端点发送 `PING` 帧。

HEALTH CHECK?

### GOAWAY

`GOAWAY` 帧（`类型=0x7`）用于启动连接关闭或发出严重错误情况信号。GOAWAY 允许端点优雅地停止接受新流，同时仍然完成对先前建立的流的处理。这将启用管理操作，例如服务器维护。

### WINDOW_UPDATE

`WINDOW_UPDATE` 帧（`类型=0x8`）用于实现流控制

流控制在两个级别上运行：

- 在每个单独的流上
- 在整个连接上。

### CONTINUATION

`CONTINUATION` 帧（`类型=0x9`）用于继续一系列头块片段

## HTTP 消息交换

HTTP-2 旨在与 HTTP 的当前使用尽可能兼容。
这意味着，从应用程序角度来看，该协议的功能在很大程度上没有变化。
为此，尽管传达这些语义的语法已更改，但保留了所有请求和响应语义。

### Request 和 Response

客户端使用先前未使用的流标识符在新流上发送 HTTP 请求。
服务器在与请求相同的流上发送 HTTP 响应。

HTTP 消息（请求或响应）包括：

1. 对与唯一响应，零个或多个 `HEADERS` 帧（每个帧后跟零个或更多 `CONTINUATION`） 含有信息（1xx 上的消息头帧）的 HTTP 响应。
2. 一个 `HEADERS` 帧包含消息头（接着是零个或更多 `CONTINUATION` 帧）
3. 零个或多个 `DATA` 包含有效载荷体
4. （可选）一个 HEADERS 帧，后跟零个或多个包含尾部（如果存在）的 `CONTINUATION` 帧

序列中的最后一个帧带有 `END_STREAM` 标志.

> 带有 `END_STREAM` 标志的 `HEADERS` 帧之后可以跟随 `CONTINUATION` 帧，这些帧携带了头块的任何剩余部分

来自任何流的其他帧不得在 `HEADERS` 帧和随后的任何 `CONTINUATION` 帧之间出现。

`HEADERS` 帧（和相关 `CONTINUATION` 帧）只能出现在一个流的开始或结束。

HTTP-2 使用`DATA`帧来承载消息有效负载。

一个流中帧的顺序:

```
HEADERS -> (CONTINUATION) -> HEADERS -> (CONTINUATION) -> (CONTINUATION)
-> ...
-> DATA -> DATA -> DATA -> DATA
-> ...
-> HEADERS with END_STREAM FLAG
-> (CONTINUATION)
```

#### Upgrading from HTTP-2

HTTP-2 不在支持状态交换协议状态代码 101。
101（交换协议）的语义不适用于多路复用协议。
替代协议能够使用与 HTTP-2 协商使用相同的机制。

#### HTTP 头部字段

HTTP 标头字段将信息作为一系列键值对携带。

就像在 HTTP-1.x 中一样，标头字段名称是 ASCII 字符的字符串，以不区分大小写的方式进行比较。
**但是，报头字段名称必须先转换为小写**，然后再使用 HTTP-2 进行编码。

##### 伪头字段

HTTP-1.x 使用消息起始行传达 URI,请求的方法和响应的状态码。
而 HTTP-2 使用特殊的伪头字段，以 `:` (ASCII 0x3a) 字符开头

> 伪头字段不是 HTTP 头字段。

**伪标题字段仅在定义它们的上下文中有效。**
为请求定义的伪头字段不得出现在响应中；为响应定义的伪标题字段不得出现在请求中。

- `:method`
- `:schema`
- `:authority`
- `:path`
- `:status`

##### 特定于连接的标题字段

http-2 不使用 `Connection` 标头字段来指示特定于连接的标头字段
唯一的例外是 `TE` 标头字段，该字段可以出现在 HTTP-2 请求中。如果是，则除 “trailers” 外，不得包含其他任何值

也就是说，将 HTTP-1 消息转换成 HTTP-2 的代理需要删除由 Connection 指定的所有字段。
还包括 keep-alive，proxy-connection， transfer-encoding 和 upgrade。

#### 对比 HTTP-1 和 HTTP-2 报文

GET 请求报文：

```
// http-1
GET /resource HTTP/1.1
Host: example.org
Accept: image/jpeg
```

```
// http-2
HEADERS
  + END_STREAM
  + END_HEADERS
    :method = GET
    :scheme = https
    :path = /resource
    host = example.org
    accept = image/jpeg
```

GET 响应报文：

```
// http-1
HTTP/1.1 304 Not Modified
Etag: "hash"
Expires: Thu, 23 Jan ...
```

```
// http-2
HEADERS
  + END_STREAM
  + END_HEADERS
    :status = 304
    etag = "hash"
    expires = Thu, 23 Jan ...
```

POST 请求报文：

```
// http-1
POST /resource HTTP/1.1
Host: example.org
Content-Type: image/jpeg
Content-Length: 12

{binary data}
```

```
// http-2  包含 CONTINUATION 帧
HEADERS
  - END_STREAM
  - END_HEADERS
    :method = POST
    :scheme = https
    :path = /resource
CONTINUATION
  + END_HEADERS
    content-type = image/jpeg
    host = example.org
    content-length = 123

DATA
  + END_STREAM
{binarydata}
```

POST 响应：

```
// http-1
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: 123

{binary data}
```

```
// http-2
HEADERS
  - END_STREAM
  + END_HEADERS
    :status = 200
    content-type = image/jpeg
    content-length = 123

DATA
  + END_STREAM
{binarydata}
```

#### HTTP2 请求可靠性

HTTP-1 中，client 因为无法确定错误的性质，所以无法重试非幂等请求。
能在错误之前发生了一些服务器处理，如果重新尝试了该请求，可能导致不良后果。

HTTP-2 提供了两种机制，可向客户端保证未处理请求：

- `GOAWAY` 帧表明可能已经被处理的最高流编号。因此，可以保证对具有最高编号的流的请求可以安全的重试。
- `REFUSED_STEAM` 错误代码可以被包括在一个 `RST_STREAM` 帧，以指示该流杯之前已经发生的任何处理关闭。可以安全的重试在重置流觞发送的任何请求。

### 服务器推送

HTTP-2 允许服务器与先前的客户端发起的请求相关联地抢先向客户端发送（或“推送”）响应（以及相应的“承诺”请求）

客户端可以请求禁用服务器推送，尽管这是针对每个跃点?单独协商的。

该 `SETTINGS_ENABLE_PUSH` 设置可以设置为 0，表明服务器推送被禁用。

**承诺的请求必须是可缓存的，必须是安全的，并且不得包含请求正文。**

如果客户端实现了 HTTP 缓存，则可以存储可缓存的推送响应。
**不可缓存的推送响应绝不能由任何 HTTP 缓存存储。它们可以单独提供给应用程序。**

服务器必须在 `:authority` 伪头字段中包含一个值，该值是服务器具有权威性的值

代理可以从服务器接收推送，并选择不将其转发到客户端。
同样，中介可能会选择对客户端进行其他推送，而服务器不会采取任何措施。

## 连接管理

**HTTP-2 连接是持久的。**为了获得最佳性能，期望客户端在确定不需要与服务器进行进一步通信之前（例如，当用户离开特定网页时）或直到服务器关闭连接之前，不会关闭连接。

## HTTP-2 头部压缩

随着网页增长到需要数十到数百个请求，这些请求中的冗余标头字段不必要地消耗了带宽，从而显着增加了延迟。

HTTP=2 使用标题字段表将标题字段映射到索引值，从而通知编码。这些标头字段表可以在编码或解码新标头字段时进行增量更新。

## 小结

HTTP2 向后兼容 HTTP-1 绝大多数功能。

通过帧和流控制（Flow-control）实现多路复用，有效的利用 TCP 链接，减少了 TCP 链接的数量。

使用伪头。

使用类似 SPDY 的方式压缩头部字段。

支持了服务器推送功能。

只有持久连接。直到服务器关闭连接之前，不会关闭连接。

## 参考

- ~~[RFC7540: HTTP/2](https://httpwg.org/specs/rfc7540.html)~~
- [RFC7541: HPACK Header Compression for HTTP/2](https://httpwg.org/specs/rfc7541.html)
- [RFC9113: HTTP/2](https://httpwg.org/specs/rfc9113.html)
