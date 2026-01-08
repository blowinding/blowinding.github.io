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
	- 路由器的结构（可画图描述）

### MOTIVATION
- 目前路由器遇到的瓶颈
	- 在需要支持大规模用户的情况下，特定流量模式下的缓存收敛比较高（需要实验证明）
- PIFO系列
	- SP-PIFO
		- 可扩展性不足，需要占用大量队列
		- 不天然支持HQoS，若支持HQoS则队列数量不足
	- AIFO
		- 较长队列积累则调度失真（实验配置可说明）
		- 实际上没有任何优先级调度，强依赖Rank分布（变式AQM）
		- 不适合用于ISP（丢包损害更严重）
		- 和phantom queue机理相似（都是准入控制）
	- PACKS
		- 将准入控制和优先级映射结合
		- TODO
	- vPIFO
		- 基本实现HQoS，但是除了调度没有其他策略（shaping、policy）
		- 没有对出口队列的拥塞控制策略
	- BMW tree
		- 没有对出口队列的拥塞控制策略
- 是否可以使用虚拟队列替换物理队列（challenge、TODO）
- 路由器的调度策略在TM实现，是否可以在NP实现以增加灵活性
- 本质上是如何设计一个最小化队列占用，以实现（TODO）多级调度的的调度算法
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
