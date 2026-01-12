## PLAN(WEEKLY UPDATE)
### OVERALL PLAN
- BACKGROUND & MOTIVATION 1 week DELAY
- DESIGN 2 week TODO
- IMPLEMENTATION 6 weeks TODO
- EXPERIMENTS 4 weeks TODO
### GOALS FROM PREVIOUS WEEK
- 完成BACKGROUND和MOTIVATION
### UPDATES FROM PREVIOUS WEEK
- 阅读SP-PIFO、AIFO、PACKS、vPIFO、BC-PQP等论文，总结BACKGROUND
- 思考MOTIVATION和DESIGN GOALS
### NEXT WEEK PLAN
- 完善BACKGROUND和MOTIVATION
	- BACKGROUND
		- ISP流量分类、调度具体讲清楚，加上论据
		- 路由器结构、调度策略详细解释（画图）
	- MOTIVATION
		- related work
			- PACKS
			- vPIFO
			- BMW tree
		- 添加论据
		- 把问题具体定义（精确描述）
## DETAILS
### BACKGROUND
- HQoS
	- QoS定义、QoS功能、为什么QoS重要
	- 目前业界实现QoS的方式（引出HQoS）
		- HQoS定义
		- HQoS功能及重要性（可画图描述）
- 网络设备（路由器）
	- QoS（HQoS）在网络设备（路由器）中实现
	- 路由器的结构
		- 路由器的QoS调度
			- 最低层级为物理队列
		- 路由器如何维护队列
			- Queue Info Table，每个表项维护头尾指针以及元数据

### MOTIVATION
- 目前路由器遇到的瓶颈
	- 目前，支持更丰富的调度策略，则需要更多的队列
		- 如何定义更丰富的调度策略
			- 更精细的流量标记和分类
				- 依照用户组分类->依照用户分类，分类更加精细
				- 不同用户的流量可以进行不同的调度
		- 目前的路由器架构，为什么更丰富的调度策略需要更多的队列
			- 当前路由器架构，带有某种标记的流被引导至相应的物理队列（TAR）
		- 需要维护更多的队列会带来什么代价
			- queue info table（或其他维护开销）增大
		- 为什么这个代价需要优化
			- 维护队列的缓存存放在片内缓存
			- 片外缓存或许可以存放报文，但是片内缓存无法维护更多的队列（存在极限）
			- 需要说明这个极限对于目前研制片上缓存技术无法突破
- 更丰富的调度策略与更多的物理队列并不等价
	- QoS服务的本质是差异化处理
	- 调度物理队列存在的本质意义是改变包进入发送队列的分布
	- 目前存在其他的方式改变包进入发送队列的分布(SP-PIFO、AIFO、PACKS）
- PIFO系列
	- FIFO原语
		- SP-PIFO
			- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
		- AIFO
			- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
		- PACKS
			- 不支持Hierarchy Schedule以及non work-conserving scheduling algorithms.
	- PIFO原语
		- vPIFO
			- 每一层节点都需要消耗SRAM，需要额外的面积容纳RPU
		- BMW tree
			- 每一层节点都需要消耗SRAM，需要额外的面积容纳RPU
- BC_PQP
	- 无优先级调度
- 使用虚拟队列替换物理队列
	- Challenge
		- 如何使用虚拟队列近似实现HQoS
			- 调度算法的近似程度
				- $\Delta(t) = \frac{\lvert P(t) \setminus A(t) \rvert + \lvert A(t) \setminus P(t) \rvert}{\lvert P(t) \rvert + \lvert A(t) \rvert}$ 
					- 单位时间窗内带宽占用
				- 
		- 低优先级的包在缓存中等待的代价与直接丢包的代价的trade off
			- 可以减轻缓存负担以及队列维护负担
			- 需要考虑丢包的代价
### DESIGN
- 问题
	- 如何在NP中正确实现多层调度（SP、DWRR等）
	- 如何使用较少的队列（减少Queue Info Table占用）执行正确调度策略
- 设计目标
	- 使用虚队列代替物理队列，减少带宽收敛比
		- 虚队列的阈值
		- 丢包带来的影响
		- 缓存开销能否减少
	- 将调度策略移动到NP中，可以自定义调度策略
		- 如何保证调度策略正确或与传统HQoS方案相近
		- 在NP中实现的调度策略的速度如何与TM相近
