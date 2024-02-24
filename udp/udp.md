# UDP 基本概念

用户数据报协议（UDP，User Datagram Protocol）是一种面向单项数据包的协议。
处于 OSI 模型的传输层

UDP 只提供数据的不可靠传递，它一旦把应用程序发给网络层的数据发送出去，就不保留数据备份

## 特点

- 它是面向事务的，适用于简单的查询响应协议，例如域名系统或网络时间协议。
- 它提供数据报，适合对其他协议（例如 IP 隧道或远程过程调用和网络文件系统）进行建模。
- 它很简单，适合引导或其他没有完整协议栈的用途，例如 DHCP 和简单文件传输协议。
- 它是无状态的，适合大量客户端，例如 IPTV 等流媒体应用程序。
- 由于没有重传延迟，它非常适合实时应用，例如 IP 语音、在线游戏以及许多使用实时流协议的协议。
- 由于它支持组播，因此适用于多种服务发现中的广播信息以及精确时间协议和路由信息协议等共享信息。

## 应用场景

- 域名系统（DNS），查询阶段必须快速，并且只包含单个请求，后跟单个回复数据包；
- 动态主机配置协议（DHCP），用于动态分配 IP 地址；
- 简单网络管理协议（SNMP）；
- 路由信息协议（RIP）；
- 网络时间协议（NTP）
- 流媒体
- 实时多人游戏
- IP 语音 (VoIP)

## 报文结构

UDP 报头包括 4 个字段，每个字段占用 2 个字节（即 16 个二进制位）。

![UDP Header Format](/images/UDP-Header-Format.png)

在 IPv4 中，“来源连接端口”和“校验和”是可选字段（以粉色背景标出）。
在 IPv6 中，只有来源连接端口是可选字段。

**Source port number**:

该字段在使用时标识发送方的端口，并且应假定为需要时要回复的端口。

**Destination port number**:

该字段标识接收器的端口，并且是必需的。

**Length**:

该字段指定 UDP 报头和数据总共占用的长度。可能的最小长度是 8 字节，因为 UDP 报头已经占用了 8 字节。

**Checksum**:

"校验和"字段可用于报头和数据的错误检查。

## 校验和计算

### IPV4

当 UDP 运行在 IPv4 之上时，为了能够计算校验和，需要在 UDP 数据包前添加一个“伪头部”。
**伪标头仅用于校验和计算。**

对于 IPv4，UDP 校验和计算是可选的。如果不使用校验和，则应将其设置为零值。
如果计算出的校验和为零，则将其作为全 1 进行传输（等价于补码算术）。

![IPv4 pseudo header format](/images/IPv4-pseudo-header-format.png)

### IPV6

当 UDP 运行于 IPV6 之上时，校验和是必须的.

> 任何在其校验和计算中包含 IP 标头中的地址的传输协议或其他上层协议都必须进行修改，以便在 IPv6 上使用，以包含 128 位 IPv6 地址而不是 32 位 IPv4 地址

计算校验和时，再次使用模拟真实 IPv6 标头的伪标头：

![IPv6 pseudo header format](/images/IPv6-pseudo-header-format.png)

如果 IPv6 数据包不包含路由标头，则该标头将是 IPv6 标头中的目标地址；
否则，在始发节点，它将是路由标头最后一个元素中的地址，而在接收节点，它将是 IPv6 标头中的目标地址。
Next Header 字段的值是 UDP 的协议值：17。
UDP length 字段是 UDP 头和数据的长度。

## 可靠性和拥塞控制

由于缺乏可靠性，UDP 应用程序可能会遇到一些数据包丢失、重新排序、错误或重复的情况。
如果使用 UDP，最终用户应用程序必须提供任何必要的握手，例如消息已收到的实时确认。
应用程序（例如 TFTP）可以根据需要向应用层添加基本的可靠性机制。
如果应用程序需要高度的可靠性，则可以使用诸如 TCP 之类的协议。

## 参考

- [RFC768，User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768.html)
- [User Datagram Protocol](https://en.wikipedia.org/wiki/User_Datagram_Protocol)
- [IPv6 Jumbograms](https://www.rfc-editor.org/rfc/rfc2675.html)
- [Internet Protocol, Version 6 (IPv6) Specification](https://www.rfc-editor.org/rfc/rfc2460.html)
