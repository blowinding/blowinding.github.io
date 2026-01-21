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
### NEXT WEEK PLAN
- 初步设计design
- 补全background&motivation
## DETAILS
### BACKGROUND
- HQoS
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
- 网络设备（路由器）
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
### MOTIVATION
- 目前路由器遇到的瓶颈
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
			- ***维护队列的缓存存放在片内缓存（TODO）***
			- 片外缓存或许可以存放报文，但是片内缓存无法维护更多的队列（存在极限）
			- 片上 SRAM 缩放已经远落后于逻辑电路缩放，造成功耗更高、面积效率更低，因此SRAM不能大容量扩展[13]、[14]、[15]
			- 目前单位带宽的SRAM容量下降了4倍[16]
- 更丰富的调度策略与更多的物理队列并不等价
	- QoS服务的本质是差异化处理
	- 调度本质意义是改变包进入发送队列的分布（时间、顺序）[17]
	- 目前存在其他的方式改变包进入发送队列的分布(SP-PIFO[18]、AIFO[19]、PACKS[20]）
- ***为什么其他工作不行（TODO）***
	- PIFO系列
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
- 使用虚拟队列替换物理队列
	- Challenge
		- 低优先级的包在缓存中等待的代价与直接丢包的代价的trade off
			- 可以减轻缓存负担以及队列维护负担
			- 需要考虑丢包的代价
### DESIGN
- 目标
	- 使用虚拟队列近似实现HQoS
		- 调度算法的近似程度
			- 时间：单个流中包的出队时间间隔
				- ***TODO***
			- 顺序：单位时间窗内出队包顺序差异
				- ***TODO***
- thoughts
	- 虚拟队列的阈值含义
		- token bucket size
		- 调度导致的虚拟队列的变化，会在长时间尺度上影响实体队列的排布
	- 流映射
	- 虚拟队列入队和出队
- 基础调度器模型
	- 分为SP调度、WFQ调度
	- 根节点处有限速，限制输出总带宽
	- 节点
		- 向上游队列发送入队指令
		- 向下游队列发送出队指令
	- 队列
		- 使用计数器维护
			- 维护报文长度
		- 执行入队、出队操作
			- 入队时，计数器加上报文长度
		- 属性
			- 阈值
			- 限速
- SP
	- 目标
		- 为某些流提供绝对的优先
	- 出队策略
		- 当最高优先级计数器为正数时，首先对最高优先级队列出队
	- 准入分析
		- 优先级从高到低排列，除了第一个计数器为正数的队列，其他低优先级队列均满载，即入速率大于出速率，最终执行准入策略后得到的结果为理想结果
			- 丢掉的低优先级包会损害低优先级流的性能，故最佳策略是丢掉的低优先级包即使进入缓存系统也会被丢掉
			- 当低优先级队列未满载时，实际出队的包不符合优先级调度策略，可通过分类策略纠正
- WFQ
	- 目标
		- 公平分配每个流的带宽，做到per-flow isolation
	- 出队策略
		- 可按照GPS，叶子队列出队时按照权重同时出队
	- 准入分析
		- 每个队列均满载时，即入速率大于出速率，最终执行准入策略后得到的结果为理想结果
			- 
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