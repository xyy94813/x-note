# gRPC 简介

gRPC 是 Google 实现的一套高性能，开源通用的 RPC 框架。

在 gRPC 中，客户端应用程序可以直接在其他计算机上的服务器应用程序上调用方法，就好像它是本地对象一样，从而使您更轻松地创建分布式应用程序和服务。
与许多 RPC 系统一样，gRPC 围绕定义服务的思想，指定可通过其参数和返回类型远程调用的方法。
在服务器端，服务器实现此接口并运行 gRPC 服务器以处理客户端调用。在客户端，客户端具有一个存根（在某些语言中仅称为 client），提供与服务器相同的方法。

![](https://grpc.io/img/landing-2.svg)

gRPC 客户端和服务器可以在各种环境（从 Google 内部的服务器到您自己的台式机）上运行并相互通信，并且可以使用 gRPC 支持的任何语言编写。
因此，例如，可以使用 Go，Python 或 Ruby 中的客户端轻松地用 Java 创建 gRPC 服务器。
此外，最新的 Google API 的接口将具有 gRPC 版本，可让轻松地在应用程序中构建 Google 功能。

默认情况下，gRPC 使用 [协议缓冲区（Protocol Buffers）](https://developers.google.com/protocol-buffers/docs/overview) 来序列化结构化数据。
它也可以与其他数据格式一起使用，例如 JSON。

## gRPC 使用 Protocol Buffers

[protocol buffers 文档](https://developers.google.com/protocol-buffers/docs/overview)。

使用 Protocol Buffers 的第一步是为要在 _proto file_ 中序列化的数据定义结构：一个带有`.proto` 扩展名的普通文本文件。

Protocol Buffer 的数据结构称之为 `message`。
每条 `message` 都是一条信息的小逻辑记录，其中包含一系列称为 `fields` 的 _name-value_ 对。

```proto
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

然后，一旦指定了数据结构，就可以使用 Protocol Buffers 的 编译器 protoc 从 proto 定义中以首选语言生成数据访问类。这些为每个字段提供了简单的访问器，例如 `name()` 和 `set_name()`，以及将整个结构序列化为原始字节或从原始字节中解析出整个结构的方法。
例如，如果选择的语言是 C++，则在上面的示例中运行编译器将生成一个名为 `Person` 的类。
然后，可以使用此类来填充，序列化和检索 Person 协议缓冲区消息。

可以在普通的 proto 文件中定义 gRPC 服务，并使用 RPC 方法参数和返回类型指定为 Protocol Buffers message：

```proto
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

<!-- gRPC 使用带有特殊 gRPC 插件的 protoc 从 proto 文件生成代码：您将生成生成的 gRPC 客户端和服务器代码，以及用于填充，序列化和检索消息类型的常规协议缓冲区代码。 -->

可以使用 proto2（当前的默认协议缓冲区版本），但最好将 proto3 与 gRPC 一起使用，因为 proto3 可以使用所有 gRPC 支持的语言，并且可以避免 proto2 客户端与 proto3 服务器通信时出现兼容性问题。反之亦然。

Proto3 目前可用于 Java，C ++，Dart，Python，Objective-C，C＃，lite-runtime（Android Java），Ruby 和 JavaScript。
