## ABSTRACT
在RCN（RDMA-offloaded container networks）中，发现大多数性能问题都和RNIC有关。因此论文进行**组合因果测试**，以高效地推理 RNIC 的架构模型，有效地近似其性能模型，从而主动优化网络功能（NF）卸载调度。
## MOTIVATION
在RDMA硬件卸载的实践过程中，出现了一些可扩展性的性能瓶颈，这些症状出现在以下三方面：
- Virtual Switch
- RNIC Driver
- RNIC Hardware
然而归根结底，以上问题都由网卡本身的设计缺陷造成的，而RNIC网卡本身为黑盒系统，因此定位问题比较困难。
### DESIGN
分为三个步骤
- 组合因果测试
- 性能解释和预测
- 按需性能优化