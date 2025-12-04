# RDMA over Ethernet for Distributed AI Training at Meta Scale

## abstract

本文介绍了 Meta 公司为分布式 AI 训练设计、实现与运维的 基于以太网的远程直接内存访问（RoCE）网络。

- 网络拓扑（Network Topology） —— 为了支持不断演进的多代 AI 硬件平台，我们将基于 GPU 的训练流量划分到独立的“后端”网络中。

- 路由（Routing） —— 训练类工作负载天生具有负载不均衡和突发性，因此我们部署了多轮路由方案迭代，以实现接近最优的流量分布。

- 传输层（Transport） —— 我们最初尝试使用 DCQCN 进行拥塞管理，但后来放弃了 DCQCN，转而利用 collective 通信库 自身来管理拥塞。

- 运维（Operations） —— 我们分享了大规模 AI 网络的运维经验，包括开发的工具以及典型的故障排查案例。