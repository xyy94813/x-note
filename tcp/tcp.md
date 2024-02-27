# 传输控制协议（TCP，Transmission Control Protocol）

**TCP 为应用程序提供可靠、有序的字节流服务**，是互联网协议栈中重要的传输层协议。

## 特点

- 可靠的，有序的
- 面向连接。尽管本身并不包含活动检测功能。
- 使用端口号来识别应用程序服务并在主机之间多路复用不同的流
- 持双向数据流，但应用程序可以自由地仅单向发送数据

## 应用场景

## 报文结构

```
   #####################
   # TCP Header Format #
   #####################

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |       |C|E|U|A|P|R|S|F|                               |
   | Offset| Rsrvd |W|C|R|C|S|S|Y|I|            Window             |
   |       |       |R|E|G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                           [Options]                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               :
   :                             Data                              :
   :                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Source Port:**

源端口，16 bits。

**Destination Port:**

目标端口，16 bits。

**Sequence Number:**

序列号，32 bits。

该段中第一个数据八位位组的序列号（设置 SYN 标志时除外）。
如果设置了 SYN，则序列号是初始序列号 (ISN)，第一个数据八位字节是 ISN+1。

**Acknowledgment Number:**

确认编号，32 bits。

如果设置了 ACK 控制位，则该字段包含该段的发送方期望接收的下一个序列号的值。
一旦建立连接，就会始终发送。

**Data Offset (DOffset):**

数据偏移量，4 bits。

TCP 标头中 32 位字的数量。指示数据开始的位置。

TCP 标头（甚至包括选项）的长度是 32 位的整数倍。

**Reserved (Rsrvd):**

保留供将来使用的一组控制位。
如果发送或接收主机未实现相应的未来功能，则生成的段中必须为零，并且接收的段中必须被忽略。

**Control bits:**

控制位也称为“标志”。
其分配由 IANA 从“TCP 标头标志”注册表进行管理。

当前分配的控制位：

- CWR: 1 bit，拥塞窗口减少
- ECE: 1 bit，显式拥塞通知，用于指示 TCP 对等方支持 ECN
- URG: 1 bit，紧急，用于指示在正常的 TCP 通信中需要中断进程以接受异步事件的控制数据
- ACK: 1 bit，确认
- PSH: 1 bit，推送功能 (see the Send Call description in Section 3.9.1).
- RST: 1 bit，重制链接
- SYN: 1 bit，同步序列号
- FIN: 1 bit，无更多数据发送

**Window:**

16 bits。
用于指示发送方可以发送多少数据而不需要等待确认。
窗口字段的值是接收方通知发送方的，以便控制数据流量并实现流量控制。

窗口字段的大小是动态变化的，它会根据网络条件和接收方的处理能力进行调整。

**Checksum:**

校验和字段是标头和文本中所有 16 位字的补码和的 16 位补码。
校验和计算需要确保求和数据的 16 位对齐。

校验和还涵盖概念上作为 TCP 标头前缀的伪标头。
在校验和中包含伪标头可以为 TCP 连接提供保护，防止误路由段。
此信息携带在 IP 标头中，并在 IP 层上的 TCP 实现的调用参数或结果中通过 TCP/网络接口进行传输。

IPv4 的伪标头为 96 位，IPv6 的伪标头为 320 位。

```
    ######################
    # IPv4 Pseudo-header #
    ######################

    +--------+--------+--------+--------+
    |           Source Address          |
    +--------+--------+--------+--------+
    |         Destination Address       |
    +--------+--------+--------+--------+
    |  zero  |  PTCL  |    TCP Length   |
    +--------+--------+--------+--------+

    ######################
    # IPv6 Pseudo-header #
    ######################
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +                                                               +
    |                                                               |
    +                         Source Address                        +
    |                                                               |
    +                                                               +
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    +                                                               +
    |                                                               |
    +                      Destination Address                      +
    |                                                               |
    +                                                               +
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                   Upper-Layer Packet Length                   |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                      zero                     |  Next Header  |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Urgent Pointer:**

紧急指针，16bits

该字段将紧急指针的当前值传达为距该段中的序列号的正偏移量。
紧急指针指向紧急数据之后的八位字节的序号。该字段仅在设置了 URG 控制位的段中进行解释。

**Options:**

```
# present only when only DOffset > 5
size(Options) == (DOffset - 5) * 32;
```

选项可以从任何八位字节边界开始，且都包含在校验和中.

选项的格式有两种情况：

- 选项类型的单个八位位组。
- 选项种类 (Kind) 的八位字节、选项长度的八位字节以及实际的选项数据八位字节。

当前定义的所有选项列表由 IANA [ 62 ]管理：

| Kind | Length | Meaning          |
| ---- | ------ | ---------------- |
| 0    | -      | 选项列表选项结束 |
| 1    | -      | 无操作           |
| 2    | 4      | 最大片段大小     |

## TCP 状态机

```
    #################################
    # TCP Connection State Diagram  #
    #################################

                                +---------+ ---------\      active OPEN
                                |  CLOSED |            \    -----------
                                +---------+<---------\   \   create TCB
                                  |     ^              \   \  snd SYN
                     passive OPEN |     |   CLOSE        \   \
                     ------------ |     | ----------       \   \
                      create TCB  |     | delete TCB         \   \
                                  V     |                      \   \
              rcv RST (note 1)  +---------+            CLOSE    |    \
           -------------------->|  LISTEN |          ---------- |     |
          /                     +---------+          delete TCB |     |
         /           rcv SYN      |     |     SEND              |     |
        /           -----------   |     |    -------            |     V
    +--------+      snd SYN,ACK  /       \   snd SYN          +--------+
    |        |<-----------------           ------------------>|        |
    |  SYN   |                    rcv SYN                     |  SYN   |
    |  RCVD  |<-----------------------------------------------|  SENT  |
    |        |                  snd SYN,ACK                   |        |
    |        |------------------           -------------------|        |
    +--------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +--------+
       |         --------------   |     |   -----------
       |                x         |     |     snd ACK
       |                          V     V
       |  CLOSE                 +---------+
       | -------                |  ESTAB  |
       | snd FIN                +---------+
       |                 CLOSE    |     |    rcv FIN
       V                -------   |     |    -------
    +---------+         snd FIN  /       \   snd ACK         +---------+
    |  FIN    |<----------------          ------------------>|  CLOSE  |
    | WAIT-1  |------------------                            |   WAIT  |
    +---------+          rcv FIN  \                          +---------+
      | rcv ACK of FIN   -------   |                          CLOSE  |
      | --------------   snd ACK   |                         ------- |
      V        x                   V                         snd FIN V
    +---------+               +---------+                    +---------+
    |FINWAIT-2|               | CLOSING |                    | LAST-ACK|
    +---------+               +---------+                    +---------+
      |              rcv ACK of FIN |                 rcv ACK of FIN |
      |  rcv FIN     -------------- |    Timeout=2MSL -------------- |
      |  -------            x       V    ------------        x       V
       \ snd ACK              +---------+delete TCB          +---------+
         -------------------->|TIME-WAIT|------------------->| CLOSED  |
                              +---------+                    +---------+
```

## 序列号

TCP 连接发送的每个八位字节数据都有一个序列号。
由于每个八位位组都是有序的，因此每个八位位组都可以被确认。
该机制允许在存在重传的情况下直接进行重复检测。

序列号通常是按照递增的顺序生成和使用的，但 TCP 协议本身并没有要求严格的顺序。
其作用更多地是用于唯一标识和管理数据流

TCP 序列号的大小是固定的，从 0 到(2^32-1) = 4GB，这意味着不能发送超过 4GB 的数据，每个数据字节都有一个唯一的序列号。

TCP 序列号的重复使用是通过循环利用序列号来维护数据传输的连续性，这被称为 Wrap Around 概念。

TCP 序列号比较的典型种类：

1. 确定确认是指已发送但尚未确认的某个序列号。
2. 确定一个段占用的所有序列号都已被确认（例如，从重传队列中删除该段）。
3. 确定传入段包含预期的序列号（即该段“重叠”接收窗口）

为了响应发送数据，TCP 端点将收到确认。处理确认需要进行以下比较：

- SND.UNA = 最旧的未确认序列号
- SND.NXT = 下一个要发送的序列号
- SEG.ACK = 来自接收 TCP 对等体的确认（接收 TCP 对等体期望的下一个序列号）
- SEG.SEQ = 段的第一个序列号
- SEG.LEN = 段中数据占用的八位字节数（包括 SYN 和 FIN）
- SEG.SEQ+SEG.LEN-1 = 段的最后一个序列号

新的确认（称为“可接受的确认”）是满足以下不等式的确认： `SND.UNA < SEG.ACK =< SND.NXT`

TCP 初始序列号是根据单调递增直至回绕的数字序列生成的，通俗地称为“时钟”。

```
ISN = M + F(本地IP、本地端口、远程IP、远程端口、秘密密钥)
```

## 建立连接

```
    TCP Peer A                                           TCP Peer B

1.  CLOSED                                               LISTEN
2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED
3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED
4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED
5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
```

建立链接经历了 3 次通信，因此称为 3 次握手 (3WHS)。

**三向握手的主要原因是为了防止旧的重复连接启动造成混乱。**

如果接收 TCP 对等方处于非同步状态（即 SYN-SENT、SYN-RECEIVED），则在接收到可接受的重置后返回到 LISTEN。
如果 TCP 对等方处于同步状态之一（ESTABLISHED、FIN-WAIT-1、FIN-WAIT-2、CLOSE-WAIT、CLOSING、LAST-ACK、TIME-WAIT），它将中止连接并通知其用户。

```
##############
# 同时连接同步 #
##############

    TCP Peer A                                       TCP Peer B

1.  CLOSED                                           CLOSED
2.  SYN-SENT     --> <SEQ=100><CTL=SYN>              ...
3.  SYN-RECEIVED <-- <SEQ=300><CTL=SYN>              <-- SYN-SENT
4.               ... <SEQ=100><CTL=SYN>              --> SYN-RECEIVED
5.  SYN-RECEIVED --> <SEQ=100><ACK=301><CTL=SYN,ACK> ...
6.  ESTABLISHED  <-- <SEQ=300><ACK=101><CTL=SYN,ACK> <-- SYN-RECEIVED
7.               ... <SEQ=100><ACK=301><CTL=SYN,ACK> --> ESTABLISHED
```

## 关闭连接

CLOSE 是一个操作，意思是“我没有更多的数据要发送”。
CLOSE 的用户可以继续 RECEIVE，直到 TCP 接收方被告知远程对等点也已 CLOSE。

```
###########
# 一般关闭 #
###########

    TCP Peer A                                           TCP Peer B

1.  ESTABLISHED                                          ESTABLISHED
2.  (Close)
    FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  --> CLOSE-WAIT
3.  FIN-WAIT-2  <-- <SEQ=300><ACK=101><CTL=ACK>      <-- CLOSE-WAIT
4.                                                       (Close)
    TIME-WAIT   <-- <SEQ=300><ACK=101><CTL=FIN,ACK>  <-- LAST-ACK

5.  TIME-WAIT   --> <SEQ=101><ACK=301><CTL=ACK>      --> CLOSED
6.  (2 MSL)
    CLOSED
```

## 分割数据

术语“分段”是指 TCP 在从发送应用程序获取字节流并将该字节流打包为 TCP 分段时执行的活动。

应用程序可以在上层协议中以消息的粒度进行写入，但 TCP 保证发送和接收的 TCP 段的边界与用户应用程序数据的读或写缓冲区的边界之间没有关联。

发送更大数据段的目标包括：

- 减少网络中传输的数据包数量。
- 限制 TCP 标头的开销。
- 通过启用更少数量的中断和层间交互来提高处理效率和潜在性能。

发送较大数据段的性能优势可能会随着大小的增加而降低，并且可能存在优势相反的边界。
例如，在某些实现架构上，一个段内的 1025 字节可能会导致比 1024 字节更差的性能，这纯粹是由于复制操作上的数据对齐所致。

发送更小数据段的目标包括：

- 避免发送会导致 IP 数据报大于 IP 网络路径上最小 MTU（最大最小传输单元） 的 TCP 段，因为这会导致数据包丢失或数据包碎片。
- 防止应用程序数据流延迟，特别是当 TCP 正在等待应用程序生成更多数据时，或者当应用程序正在等待来自其对等方的事件或输入以生成更多数据时。
- 启用 TCP 段和低层数据单元之间的“命运共享”（例如，低于 IP，对于单元或帧大小小于 IP MTU 的链路）。

当 TCP 接收的 MSS 不同于 IPv4 的默认 536 或 IPv6 的 1220 (SHLD-5) 时，
TCP 实现应该在每个 SYN 段中发送 MSS 选项，
并且可以始终发送它 (MAY-3)。

```
Eff.snd.MSS = min(SendMSS+20, MMS_S) - TCPhdrsize - IPoptionsize
```

## 超时重传

由于报文段可能由于错误（校验和测试失败）或网络拥塞而丢失，因此 TCP 使用重传来确保每个报文段的传送。
由于网络或 TCP 重传，会收到重复的的数据段可能会到达。

由于构成互联网络系统的网络的可变性以及 TCP 连接的广泛使用，必须动态地确定重传超时（RTO，Retransmission Timeout）。

计算 RTO 的算法：

- RFC 6298 (newest)
- RFC 2988
- RFC 1122
- RFC 793 (oldest)

计算当前 RTO，TCP 发送方维护两个状态变量，平滑往返时间（SRTT，Smoothed Round-Trip Time）和 往返时间变化（RTTVAR，round-trip time variation）。

假设时钟粒度为 G 秒。SRTT、RTTVAR 和 RTO 的计算规则如下

1. 直到完成往返时间 (RTT) 测量。
   在发送者和接收者之间发送的段，发送者应该设置 `RTO <- 1` 秒
2. 当进行第一个 RTT 测量时，主机必须设置

   ```
    SRTT <- R
    RTTVAR <- R/2
    RTO <- SRTT + max(G, K*RTTVAR)

   where K = 4.
   ```

3. 当进行后续 RTT 测量 R' 时，

   ```
    RTTVAR <- (1 - beta) * RTTVAR + beta * |SRTT - R'|
    SRTT <- (1 - alpha) * SRTT + alpha * R'
    RTO <- SRTT + max (G, K*RTTVAR)
   ```

   更新 RTTVAR 时使用的 SRTT 的值就，在使用第二个分配更新 SRTT 本身之前的值。
   建议使用 alpha=1/8 和 beta=1/4 计算

4. 每当计算 RTO 时，如果小于 1 秒，则 RTO 应四舍五入为 1 秒。
5. RTO 可以设置最大值，前提是它至少为 60 秒。

## 阻塞控制

RFC 1122 要求实施 Van Jacobson 的拥塞控制算法：慢启动和拥塞避免，以及同一网段的连续 RTO 值的指数退避

慢启动：

在建立 TCP 连接时逐渐增加发送速率，以便适应网络的容量。
慢启动算法通过逐渐增加拥塞窗口的大小来实现。
拥塞窗口是指在不发生数据包丢失的情况下，可以发送到网络上的数据包数量。
当网络拥塞时，TCP 慢启动算法会减少发送速率，以避免网络拥塞的发生。

拥塞避免：

通过动态调整拥塞窗口的大小，以便在网络容量的范围内发送数据。
拥塞避免算法通过增加拥塞窗口的大小来逐渐增加发送速率，同时也会对网络拥塞做出响应性的调整。

## 参考

- [Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293.html)
- [Internet Protocol, Version 6 (IPv6) Specification](https://www.rfc-editor.org/rfc/rfc8200.html)
- [Computing TCP's Retransmission Timer](https://www.rfc-editor.org/rfc/rfc6298.html)
- [TCP Congestion Control](https://www.rfc-editor.org/rfc/rfc2581.html)
