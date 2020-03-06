# gRPC 基础概念

## 概述

### 服务定义 Service definition

与许多 RPC 系统一样，gRPC 围绕定义服 ​​ 务的思想，指定可通过其参数和返回类型远程调用的方法。
默认情况下，gRPC 使用 [protocol buffers](https://developers.google.com/protocol-buffers/) 作为接口定义语言（IDL）来描述服务接口和有效负载消息的结构。

```proto
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC 可以定义四种服务方法：

- Unary RPCs, 客户端向服务器发送单个请求并获得单个响应，就像普通的函数调用一样。

  ```
  rpc SayHello(HelloRequest) returns (HelloResponse) {
  }
  ```

- Server streaming RPCs，客户端向服务器发送请求，并获取流以回读一消息队列。客户端从返回的流中读取，直到没有更多消息为止。 gRPC 保证单个 RPC 调用中的消息顺序。

  ```
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse) {
  }
  ```

- Client streaming RPCs，客户端再次使用提供的流编写一消息队列并将其发送到服务器。客户端写完消息后，它将等待服务器读取消息并返回响应。 gRPC 再次保证了在单个 RPC 调用中的消息顺序。
  ```
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
  }
  ```
- Bidirectional streaming RPCs，双方都使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照自己喜欢的顺序进行读写：例如，服务器可以在写响应之前等待接收所有客户端消息，或者可以先读取消息再写入消息，或其他一些读写组合。每个流中的消息顺序都会保留。

### 使用 API surface

从 `.proto` 文件中的服务定义开始，gRPC 提供了 protocol buffer 编译器插件，这些插件可生成客户端和服务器端代码。
gRPC 用户通常在客户端调用这些 API，并在服务器端实现相应的 API。

- 在服务器端，服务器实现服务声明的方法，并运行 gRPC 服务器来处理客户端请求。 gRPC 基础结构解码传入的请求，执行服务方法，并对服务响应进行编码。

- 在客户端，客户端具有一个称为 `stub` 的本地对象（对于某些语言，可能是 `client`），该对象实现与服务相同的方法。然后，客户端可以只在本地对象上调用这些方法，将调用的参数包装在适当的 protocol buffer 消息类型中-gRPC 在将请求发送到服务器并返回服务器的 protocol buffer 响应之后进行查找。

### 同步与异步

阻塞的同步 RPC 调用直到服务器收到响应为止，是最接近 RPC 所追求的过程调用抽象的近似方法。
另一方面，网络本质上是异步的，因此在许多情况下能够启动 RPC 而不阻塞当前线程很有用。

大多数语言中的 gRPC 编程界面都有同步和异步两种形式。

## RPC 生命周期

### Unary RPC

- 客户端调用 stub/client 对象上的方法后，会通知服务器 RPC 被调用。使用该客户端的 [metadata](https://grpc.io/docs/guides/concepts/#metadata)，方法名称和指定的 [deadline](https://grpc.io/docs/guides/concepts/#deadlines)。

- 服务器既可以立即发送自己的初始 `metadata`（必须在发送任何响应之前发送），也可以等待客户端的请求 message (首先发生的是特定于应用程序。

- 服务器收到客户的请求消息后，它将执行创建和填充其响应所需的所有工作。然后将响应（如果成功）连同状态详细信息（状态代码和可选状态消息）以及可选尾随 `metadata` 一起返回（如果成功）。

- 如果状态为 OK，则客户端将获得响应，从而在客户端完成调用。

### Server streaming RPC

server-streaming RPC 不同之处在于服务器在获得客户端的请求消息后发回响应流。发送回所有响应后，服务器的状态详细信息（状态代码和可选状态消息）和可选的尾 `metadata` 将被发送回服务器端以完成操作。客户端在收到所有服务器的响应后即完成操作。

### Client streaming RPC

Client streaming RPC，不同之处在于客户端将请求流而不是单个请求发送到服务器。服务器通常在收到客户端的所有请求后（但不一定）发送单个响应，以及其状态详细信息和可选的尾 `metadata`。

### Bidirectional streaming RPC

在 Bidirectional streaming RPC 中，调用再次由客户端调用方法，服务器接收客户端 `metadata`，方法名称和 deadline 。同样，服务器可以选择发回其初始元数据，或等待客户端开始发送请求。

接下来发生的情况取决于应用程序，因为客户端和服务器可以按任何顺序进行读写 -- 流完全独立地运行。
所以，例如，服务器可以等到收到所有客户端的消息后再写响应，或者服务器收到请求，然后发回响应，然后客户端发送基于响应的另一个请求，依此类推。

### Deadlines/Timeouts

gRPC 允许客户端指定在终止带有错误 `DEADLINE_EXCEEDED` 的 RPC 之前，他们愿意等待 RPC 完成多长时间。在服务器端，服务器可以查询以查看特定的 RPC 是否超时，或者还剩下多少时间来完成 RPC。

如何指定 deadline/timeout 的方式因语言而异。
例如，并非所有语言都有默认期限，某些语言 API 按照 deadline （固定的时间点）工作，而某些语言 API 根据 timeouts 来工作（持续时间）。

### RPC 终止

在 gRPC 中，客户端和服务器都对呼叫成功进行独立和本地确定，其结论可能不匹配。

例如，在服务器端成功完成 RPC 的 RPC（“我已发送所有响应！”），而在客户端却失败了（“响应在我的 deadline 之后到达！”）。

服务器也有可能在客户端发送所有请求之前决定完成。

### 取消 RPC

客户端或服务器都可以随时取消 RPC。取消操作会立即终止 RPC，因此无需进行进一步的工作。
这不是“撤消”：取消之前所做的更改不会回滚

### Metadata

`Metadata` 是以键值对列表的形式提供的有关特定 RPC 调用的信息（例如身份验证详细信息），其中键是字符串，值通常是字符串（但可以是二进制数据）。
`Metadata` 对于 gRPC 本身是不透明的，它允许客户端向服务器提供与调用相关的信息，反之亦然。

对元数据的访问取决于语言。

关于身份验证可参考：[gRPC Authentication](https://grpc.io/docs/guides/auth)

### Channels

gRPC 通道提供到指定主机和端口上的 gRPC 服务器的连接，并在创建客户端 stub/client 时使用。
客户可以指定通道参数来修改 gRPC 的默认行为，例如打开和关闭消息压缩。通道具有状态，包括 `connected` 和 `idle`。
