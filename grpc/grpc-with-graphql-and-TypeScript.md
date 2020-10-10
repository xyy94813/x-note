# GRPC with GraphQL and TypeScript

gRPC 和 GraphQL 都是 2015 年推出的接口设计方案，各有各的优势。

GraphQL 在提供了 [GraphQL schema stitching](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html) 以支持微服务架构。
但是，由于 GraphQL 请求被解析了两次，所以可能导致性能瓶颈。

所以，APP 与独立的 GraphQL Server 集群进行通信，而 GraphQL Server 通过 gRPC 与微服务应用进行通信是一个不错的方案。

![Sample Architecture diagram with GraphQL & gRPC](https://miro.medium.com/max/700/1*o9_bjlKXlMjii3G7BCNb-Q.png)

- GraphQL Server 作为唯一的 **BFF（Backend for Frontend）** 入口。
- gRPC Server，以执行所有功能操作。

> 这两类微服务托管在 K8S（Kubernetes） 上，因此还需要在 gRPC 中实现运行状况检查机制。

## 技术选型参考

For GraphQL Server:

- as GraphQL Server
  - [express-graphql(Offical)](https://github.com/graphql/express-graphql)
  - [graphql-relay-js(Offical)](https://github.com/graphql/graphql-relay-js)
  - [Apollo-Server](https://github.com/apollographql/apollo-server)
- as gRPC Client
  - [gRPC-node(Offical)](https://github.com/grpc/grpc-node)
    - [@grpc/grpc-js](https://www.npmjs.com/package/@grpc/grpc-js)
    - [grpc](https://www.npmjs.com/package/grpc)

> Tthe difference between `grpc` and `@grpc/grpc-js` see [there](https://github.com/grpc/grpc-node/blob/master/PACKAGE-COMPARISON.md)

如果你的应用还作为 gRPC Server:

- [gRPC-node(Offical)](https://github.com/grpc/grpc-node)
  - [@grpc/grpc-js](https://www.npmjs.com/package/@grpc/grpc-js)
  - [grpc](https://www.npmjs.com/package/grpc)
- [mali](https://github.com/malijs/mali)
- ...

### gRPC 直接转 GraphQL

存在一些现成的轮子做了 gRPC 直接转 GraphQL 的工作。

- [rejoiner](https://github.com/google/rejoiner)，Java 版本，Google 维护。
- [grpc-graphql-gateway](https://github.com/ysugimoto/grpc-graphql-gateway)，golang 版本，个人项目

## 生成 TypeScript 声明文件

gRPC、GraphQL 以及 TypeScript 都做了类型声明的工作，需要以某种方式做映射，尽可能保证类型统一易于维护。

### 基于 proto files 生成 Typescript 类型

可以使用以下工具基于 proto files 直接生成 TypeScript 声明文件

- [protobuf.js](https://github.com/protobufjs/protobuf.js#pbts-for-typescript)
- [grpc_tools_node_protoc_ts](https://github.com/agreatfool/grpc_tools_node_protoc_ts)
- [ts-proto](https://github.com/stephenh/ts-proto)，与以上两者不同的是仅支持 `*.proto` to `*.ts`

### 基于 GraphQL Schema 生成 Typescript 类型

可以使用以下工具基于 GraphQL Schema 直接生成 TypeScript 声明文件

- [gql2ts](https://github.com/avantcredit/gql2ts)
- [GraphQL Code Generator](https://github.com/dotansimha/graphql-code-generator)

如果 GraphQL Schema 是基于 proto files 生成的，那么推荐的流程应该是：

1. proto files to GraphQL Schema
2. proto files to client and TS types.
3. GraphQL Schema to TS types.

## Reference

- [When GraphQL meets gRPC…](https://medium.com/@svengau/when-graphql-meets-grpc-3e9729d32e05)
- [Creating a TypeScript API that consumes gRPC and GraphQL via generated types](https://medium.com/attest-engineering/fully-typed-typescript-api-consuming-grpc-and-graphql-5d5ae6b33bf1)
