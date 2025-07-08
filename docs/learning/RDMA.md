# RDMA

## RDMA基本概念

[RDMA概念](https://zhuanlan.zhihu.com/p/649468433)

## RDMA操作

与一条 RDMA 消息传输相关联的两个远端节点称为 请求端（requester） 和 响应端（responder）。
用户应用与 RDMA 网卡之间的接口由 工作队列元素（Work Queue Elements，简称 WQE，读音为 "wookies"） 提供。

应用程序在每次执行 RDMA 消息传输时都会提交一个 WQE，
该 WQE 包含应用程序为该传输指定的元数据。
WQE 在消息处理过程中被存储在网卡中，并在消息完成后过期。
请求端和响应端网卡上分别提交的 WQE 称为：

Request WQE（请求 WQE）

Receive WQE（接收 WQE）

当消息完成并使得一个 WQE 过期后，网卡会生成一个 完成队列元素（Completion Queue Element，简称 CQE，读音为 "cookie"），
用于通知用户应用该消息已经完成处理。

RDMA 网卡支持四种类型的消息传输操作：

✅ 写（Write）：
请求端将数据写入响应端的内存。

数据长度、源地址和目标地址都在 Request WQE 中指定。

通常不需要接收端有 Receive WQE。

然而，对于 Write-with-immediate（带立即数的写） 操作，用户应用必须在响应端提交一个 Receive WQE，
该 WQE 在完成时过期并生成 CQE，从而通知响应端的应用写操作完成。

✅ 读（Read）：
请求端从响应端的内存中读取数据。

数据长度、源地址和目标地址都在 Request WQE 中指定。

不需要 Receive WQE。

✅ 发送（Send）：
请求端向响应端发送数据。

数据长度和源地址由 Request WQE 指定；

数据目标地址由 Receive WQE 指定。

✅ 原子操作（Atomic）：
请求端从响应端的某个内存位置读取并原子性地更新数据，该位置在 Request WQE 中指定。

不需要 Receive WQE。

原子操作仅限于单个数据包的消息。
