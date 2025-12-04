## ABSTRACT
本文提出了**Flor**——一个在异构 RNIC 之上提供统一硬件数据平面，并在主机 CPU、RNIC 的 NPU 或 DPU 上运行灵活软件控制平面的开放框架。
## MOTIVATION
- 不同厂商或不同版本的RNIC的性能有显著差异（和拥塞控制算法、流量控制算法有关系）
- PFC带来了很多运维上的问题，因此希望在flow control中消除PFC
## DESIGN
核心目标：维持RDMA的数据平面在硬件中，将控制平面解耦到软件中。
### Architecture
- Data Path
	- 只要使用所有 RNIC 都支持的标准 RDMA 操作，就能够在异构 RNIC 间实现可比的性能。Flor 采用 **RDMA WRITE** 和 **SEND** 作为数据传输与接收的基础 WQE，因为它们同时支持 RC 和 UC，并且能保持 RDMA 的高性能。
- Control Path
	- 控制平面有五个模块构成
		- 连接管理（Connection Management）
		- 分片（Chunking）
			- Flor 将 chunk 作为选择重传算法和拥塞控制算法的基本单位，而不是传统传输层中的 packet。每个 chunk 会通过 WRITE 或 SEND WQE 发送到网络。
		- RTT 测量（RTT Measurement）
		- 可靠性（Reliability）
		- 拥塞控制（Congestion Control）
### Optimization
- 通过软硬件协同设计维持RDMA性能
	- 采用动态分片机制，在执行软件层拥塞控制与丢包恢复时调节消息大小，从而减缓较慢控制面的影响。
- 基于选择性重传增强UC
	- UC具有这样一种特性：RNIC在传输数据时不必等待之前的消息完成即可将后续消息交付给主机。Flor利用这一特性，在无需修改硬件的前提下，实现了更高效的选择性重传机制，以加速应用处理性能
- 在确保正确性的前提下增强RC
	- RC 是 Flor 支持的数据面传输协议之一。然而 RC 内建的 Go-back-N 重传机制众所周知效率低下。Flor 在此基础上加入了额外的软件重传机制以提升效率。