# Protocol Buffers 基础

协议缓冲区是 Google 的语言中立、平台中立、可扩展的结构化数据序列化机制 —— 对比 XML 和 JSON，更小、更快、更简单。
可以定义一次数据的结构化方式，然后可以使用特殊生成的源代码轻松地使用各种语言在各种数据流中写入和读取结构化数据。

## Message

### 定义 message

```proto
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 page_number = 2;
}
```

### 指定 Field 类型

Field 类型可以为 [Scale](https://developers.google.com/protocol-buffers/docs/overview#scalar)、枚举、复合类型和其它消息类型。

### 分配 Field 编号

每个 Field 都有一个唯一的编号。
这些数字用于在消息二进制格式中标识的 Field，**一旦消息类型被使用，就不应更改**。

请注意，1 到 15 范围内的 Field 编号占用一个字节进行编码，包括 Field 编号和 Field 类型，16 到 2047 范围内的 Field 编号占用两个字节。
因此，频繁出现消息元素保留尽可能 Field 编号 1 到 15，为将来可能添加的频繁出现的元素留出一些空间。

可以指定的最小编号为 1，最大编号为 536,870,911 （2 的 29 次方 - 1）。
不能使用数字 19000 到 19999, 这部分是为 Protocol Buffers 实现保留的。

### Field 规则

Filed 的 规则应是以下之一

- required
- optional，可选字段可设置默认值。`optional int32 result_per_page = 3 [default = 10];`
- repeated

由于历史原因，repeated 标量数值类型的字段编码效率不高。
新代码应该使用特殊选项[packed=true]以获得更有效的编码。

```proto
repeated int32 samples = 4 [packed=true];
```

### 废弃 Field

由于 protocol buffer 的实现原理，我们无法通过注释或删除字段的方式来废弃指定字段。
必须通过 reserved 的方式进行标记，以避免数据解析错误。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

### 更新 message

如果现有的消息类型不再满足所有需求 —— 例如，希望消息格式有一个额外的字段，但仍然希望使用以旧格式创建的代码。
需要遵循[规则](https://developers.google.com/protocol-buffers/docs/overview#updating)

- 不要更改任何现有字段的字段编号。
- 添加的任何新字段都应该是 optional 或 repeated。
- 可以删除非必填字段，只要在更新的消息类型中不再使用字段编号
- 只要类型和编号保持不变，非必填字段可以转换为扩展名，反之亦然。
- int32、uint32、int64、uint64 和 bool 都兼容 —— 这意味着可以将字段从这些类型中的一种更改为另一种，而不会破坏向前或向后的兼容性。
- sint32 并且 sint64 彼此兼容，但与其他整数类型不兼容。

### 扩展 message

允许声明消息中的一系列字段编号可用于第三方扩展。
扩展名是原始 `.proto` 文件未定义类型的字段的占位符。
这允许其他 `.proto` 文件通过定义具有这些字段编号的部分或全部字段的类型来添加到的消息定义中。

```proto
message Foo {
  // ...
  extensions 100 to 199;
}

extend Foo {
  optional int32 bar = 126;
}
```

## Map

如果想创建一个关联映射作为数据定义的一部分，可以使用 map。

```proto
map<key_type, value_type> map_field = N;
```

`key_type` 可以是任何整数或字符串类型。
注意：不包括枚举。

- Map 不支持扩展。
- Map 不能 repeated，optional 或 required。
- Map 值的连线格式排序和 Map 迭代排序未定义，因此不能依赖于特定顺序的 Map 项。
- 为`.proto`生成文本格式时，Map 按 `key` 排序。数字 `key` 按数字排序。
- 从连线解析或合并时，如果有重复的映射键，则使用看到的最后一个键。从文本格式解析 Map 时，如果有重复的键，解析可能会失败。

## package

可以通过定义 package 避免命名冲突。

```proto
package foo.bar;
message Open { ... }

message Foo {
  ...
  required foo.bar.Open open = 1;
  ...
}
```

## Service

如果想在 RPC（远程过程调用）系统中使用的消息类型，可以在.proto 文件中定义一个 RPC 服务接口，协议缓冲区编译器将以选择的语言生成服务接口代码和存根。

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

## 版本

目前，仍常使用的有 proto3 和 proto2.

proto2 可以使用 proto3 中的消息类型，反之亦然。
但是，proto2 枚举不能在 proto3 语法中使用。

## 优点

- 更简单、更快、体积更小。
- RPC 支持：服务器 RPC 接口可以声明为协议文件的一部分。
- 结构验证：具有预定义和更大的结构，与 JSON 相比，一组数据类型，在 Protobuf 上序列化的消息可以由负责交换它们的代码自动验证。

## 缺点

- 非人类可读性： JSON 以文本格式交换，结构简单，易于人类阅读和分析。二进制格式不是这种情况。[不过，现在也有一些方法可以使 protobuf 成为人类可读的。]
- 较少的资源和支持，以及较小的社区：可能是第一个劣势的根本原因。

## 参考

[protocol-buffers guide](https://developers.google.com/protocol-buffers/docs/overview)
