## PLAN(WEEKLY UPDATE)
### OVERALL PLAN
- BACKGROUND & MOTIVATION 2 week DONE
	- 继续阅读相关论文
- DESIGN 2 week ONGOING
	- 初步design ONGOING
- IMPLEMENTATION 6 weeks TODO
- EXPERIMENTS 4 weeks TODO
### GOALS FROM PREVIOUS WEEK
- 完成BACKGROUND和MOTIVATION
### UPDATES FROM PREVIOUS WEEK
- 完成MOTIVATION和DESIGN GOALS
---
### NEXT WEEK PLAN
- 初步设计design
- 补全background&motivation
---
## DETAILS
### BACKGROUND
#### QoS
- QoS
	- QoS描述了网络在资源受限条件下对不同流量类型进行差异化处理的能力，以满足各种业务的性能需求（例如带宽、延迟、抖动和丢包率）的保证。[1]
	- SLA是服务提供商与客户之间的一种正式承诺性合同，用于定义具体的服务质量指标、责任和保障措施。SLA 的核心作用是在业务层面将用户的 QoS 需求转化为可衡量、可执行的服务等级目标。[2]
	- 传统 IP 网络的 Best-Effort 机制无法保证性能，因而 QoS 技术（如 DiffServ）引入了按一些机制来实现可控的服务质量。[3]
	- 为了保证QoS，目前有work conserving（schedule）和non work conserving（traffic shaping）等机制
		- schedule有DWRR[4]、SP[5]、WFQ[6]等算法
		- traffic shaping
		- traffic policing一般使用token bucket[7]
	- HQoS
		- HQoS定义
			- 随着用户规模增加和业务种类多样化，传统单层 QoS 已难满足对不同用户和不同业务的细粒度服务要求。因此在接入网尤其是运营商边缘网关中引入了层次化 QoS（Hierarchical QoS, HQoS）机制。[8]
		- HQoS功能
			- HQoS 在传统 QoS 的基础上增加了多级调度结构，通过树形队列结构实现对服务、用户和用户组等多个层次的差异化调度，从而在接入网环境中更有效地保证 SLA 级别的 QoS 目标。
			- ***画图举个例子（TODO）***
---
#### 网络设备（路由器）
- QoS（HQoS）大多在网络设备（路由器）中实现
	- 博通[9]、[10]
	- 华为[11]
	- 思科[12]
- 路由器的结构（BCM88480）
	- 路由器进行QoS完整的过程
		- 由于Ingress缓存较大，主要的QoS大多发生在Ingress内
		- 首先在NP进行流分类和流标记
		- Ingress入队
			- 标记映射到VOQ
			- The TAR returns to the CGM Packet-Enqueue-Request that is evaluated for admission by the CGM based on available resources, queue-size, and queue parameters, resulting in an admit/drop decision. 
			- If the packet is admitted, the queue number and start-SRAM buffer, along with additional packet information, are passed to the SRAM-Queue-Manager (SQM). Otherwise, the packet copy is discarded. If all packet copies have been discarded, then the packet’s Start-Buffers are sent to the SPB to be freed. The SQM allocates a Packet-Descriptor (PD) and links it to the queue.
		- 路由器如何维护队列
			- Queue Info Table、PD Table、SRAM buffer
			- Queue Info Table每个表项维护头尾指针
			- 在broadcom中，有CGM、ingress credit scheduler、SQM维护相关信息
		- VOQ一旦有报文积压，ingress credit scheduler向egress credit scheduler发送flow status 
		- egress credit scheduler进行调度
			- The function of the egress credit scheduler is to allocate bandwidth to the VOQs. The scheduler allocates the bandwidth by sending credits to the competing VOQs. When a VOQ receives a credit, it is granted the right to send data to the target port. The worth, in bytes, of one credit is configurable (for example, 2 KB). Traffic shaping at all levels (that is, device, interface, OTM port, aggregate, and queue) is accomplished by shaping the credit allocation by the scheduler.
		- Ingress出队
			- When credit balance is positive, a Dequeue-Command with Dequeue-Command-Bytes is issued to the SRAM queue manager. The SRAM queue manager traverses the queue, dequeuing several packets. Until the queue becomes empty OR until the Dequeue-Command-Bytes are exhausted
			- The SRAM buffer manager reads the packet, updates the SRAM buffer user-count, and frees the buffer. The packet data is read and sent to the Ingress Transmit Packet Processor (ITPP) and then to the Transmit queues that have multiple contexts according to the source (SRAM/DRAM).
---
### MOTIVATION
#### 目前路由器遇到的瓶颈
- 目前，支持更丰富的调度策略，则需要更多的队列
	- 如何定义更丰富的调度策略
		- 更精细的流量标记和分类
			- 依照用户组分类->依照用户分类，分类更加精细
			- 不同用户的流量可以进行不同的调度
			- ***画图举个例子（TODO）***
		- 目前的路由器架构，为什么更丰富的调度策略需要更多的队列
			- 当前路由器架构，带有标记的包会被引导至对应的物理队列
		- 需要维护更多的队列会带来什么代价
			- queue info table开销增大
				- 对于BCM88480，一共有32K个VOQ，按照每个QIT只存储head ptr、tail ptr，字长为64bit估计，需要0.48MB缓存存储QIT，占约1/16
		- 为什么这个代价需要优化
			- 维护队列的缓存存放在片内缓存[23]
			- 片外缓存或许可以存放报文，但是片内缓存无法维护更多的队列（存在极限）
			- 片上 SRAM 缩放已经远落后于逻辑电路缩放，造成功耗更高、面积效率更低，因此SRAM不能大容量扩展[13]、[14]、[15]
			- 目前单位带宽的SRAM容量下降了4倍[16]
--- 
#### 更丰富的调度策略与更多的物理队列并不等价
- QoS服务的本质是差异化处理
- 调度本质意义是改变包进入发送队列的分布（时间、顺序）[17]
- 目前存在其他的方式改变包进入发送队列的分布(SP-PIFO[18]、AIFO[19]、PACKS[20]）
---
#### ***为什么其他工作不行（TODO）***
#####  PIFO系列
- FIFO原语
	- SP-PIFO
		- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
	- AIFO
		- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
	- PACKS
		- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
- PIFO原语
	- vPIFO[21]
		- 不支持non work-conserving scheduling algorithms
	- BMW tree[22]
		- 不支持non work-conserving scheduling algorithms
	- BBQ
		- 不支持non work-conserving scheduling algorithms
	- Loom
		- 不支持过多的流，只能小规模使用
- BC_PQP[23]
	- 无优先级调度
---
### BASIC IDEA
#### 虚拟队列
- 基础结构
	- 根节点处有限速，限制输出总带宽
	- 节点
		- 虚拟队列（计数器）
		- 调度策略
		- 入队自叶节点开始，向根节点逐级增加计数器
		- 出队自根节点开始，按调度策略选择叶节点扣减计数器
---
#### Challenge
##### drop can be harmful
- 文献[24]指出类似policer的策略会对TCP丢包敏感流损害严重，需要有足够大的阈值
- 若阈值过大，会导致更多的突发流进入TM，导致缓存系统丢包严重，影响结果的确定性
---
##### burst make result uncertain
- 以SP调度举例
	- 当高优先级backlog，仅调度高优先级的数据包
	- 假设根节点调度出速率为10Mbps，q1到达速率10Mbps，q2到达速率10Mbps
	- 当q1、q2计数器未满时，两个虚拟队列所对应的数据包均可正常发送
	- 根节点一直使q1计数器扣减，导致q2计数器以10Mbps的速率上涨
	- 当q2计数器到达阈值，到达q2的包丢包，此时只有到达q1的数据包可发送
	- 在q2计数器上涨的过程中，q2的包一直可发送，导致调度结果与预期不一致
- 如果阈值低，可以迅速达到上述效果，但是会造成很大程度的丢包，对流性能损害严重
---
### DESIGN
#### GOALS
- 使用虚拟队列近似实现HQoS
	- 模拟SP调度、WFQ调度、shaping
	- 调度算法的近似程度
		- 时间：单个流中单位时间出包的个数
			- ***TODO***
		- 顺序：单位时间窗内出队包顺序差异
			- ***TODO***
---
#### OVERVIEW
- 准入
	- 虚拟队列控制准入
		- 计数器维护通过节点的数据包长度
			- 一条流中数据包长度不一
    - 为什么需要有准入
        - non-work-conserving schedule算法需要准入近似控制，准入模块通过丢包可以暂缓数据包发送
        - buffer-management有对队列进行准入控制的模块[9] 、[10]，如果将物理队列从TM中剔除，原有对队列的准入控制应当也一并在虚拟队列中实现
        - 准入可改变数据包的分布，从而近似模拟调度的结果[19]
---
- 映射
	- 为什么需要有映射
		- 通过映射可以使用粗粒度优先级队列模拟细粒度优先级队列[18]
		- 准入策略不足以完全实现调度，当虚拟队列未满载，并且入速率大于出速率，准入无法对进入的数据包进行控制，因此需要优先级调度
---
#### SP
- 目标
	- 为某些流提供绝对的优先
- 出队策略
	- 优先级从高到低排列，当高优先级计数器为正数时，低优先级出队速率为0
- 映射策略
	- SP-PIFO
		- ***为什么使用SP-PIFO（TODO）***
---
- 准入分析
	- 优先级从高到低排列，除了第一个计数器为正数的队列，其他低优先级队列均满载，即入速率大于出速率，最终执行准入策略后得到的结果为理想结果
		- 按照出队策略，优先级从高到低排列，当高优先级计数器为正数时，低优先级出队速率为0，因此低优先级队列会按照入队速率累积
- 映射分析
	- ***（TODO）***
---
#### WFQ
- 目标
	- 公平分配每个流的带宽，做到per-flow isolation
- 出队策略
	- 可按照GPS，叶子队列出队时按照权重同时出队
---
- 准入分析
	- 每个队列均满载时，即入速率大于出速率，最终执行准入策略后得到的结果为理想结果
---
#### Shaper
- 目标
	- 对流进行限速
- 出队策略
	- 按照限定速率进行出队
---
## IMPLEMENTATION
- ***传统HQoS的实现（TODO）***

## REFERENCE
[1] https://support.huawei.com/enterprise/zh/doc/EDOC1100335696/f1b1b569
[2] https://en.wikipedia.org/wiki/Service-level_agreement
[3] https://en.wikipedia.org/wiki/Quality_of_service
[4] Efficient Fair Queueing using Deficit Round Robin
[5] 
[6] Analysis and Simulation of a Fair Queueing Algorithm
[7] https://info.support.huawei.com/info-finder/encyclopedia/en/HQoS.html
[8] An Internet-Wide Analysis of Traffic Policing
[9] https://docs.broadcom.com/doc/88480-DG1-PUB
[10] https://docs.broadcom.com/doc/88800-DG1-PUB
[11] https://support.huawei.com/enterprise/zh/doc/EDOC1100464252/da76c2c0?idPath=24030814|9856750|22715517|9858994|15969
[12] https://www.ciscolive.com/c/dam/r/ciscolive/global-event/docs/2024/pdf/BRKARC-2096.pdf
[13] Enabling static random-access memory cell scaling with monolithic 3D integration of 2D field-effect transistors
[14] SRAM Cell Design Challenges in Modern Deep Sub-Micron Technologies: An Overview
[15] https://semiengineering.com/sram-scaling-issues-and-what-comes-next/
[16] Occamy: A Preemptive Buffer Management for On-chip Shared-memory Switches
[17] BBQ: A Fast and Scalable Integer Priority Queue for Hardware Packet Scheduling
[18] SP-PIFO: Approximating Push-In First-Out Behaviors using Strict-Priority Queues
[19] Programmable packet scheduling with a single queue
[20] Everything Matters in Programmable Packet Scheduling
[21] vPIFO: Virtualized Packet Scheduler for Programmable Hierarchical Scheduling in High-Speed Networks
[22] BMW Tree: Large-scale, High-throughput and Modular PIFO Implementation using Balanced Multi-Way Sorting Tree
[23]  [System Monitoring Configuration Guide for Cisco NCS 5500 Series Routers, IOS XR Release 7.8.x - Graceful Handling of Out of Resource Situations \[Cisco Network Convergence System 5500 Series\] - Cisco](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/system-monitoring/78x/b-system-monitoring-cg-ncs5500-78x/oor-handling.html)
[24] Improving TCP performance with bufferless token bucket policing: A TCP friendly policer