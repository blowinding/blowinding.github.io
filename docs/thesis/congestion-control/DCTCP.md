## ABSTRACT
数据中心网络中存在不同的应用需求（如带宽敏感性应用以及时延敏感性应用），因此提出`DCTCP`满足以上要求。

## COMMUNICATIONS IN DATA CENTERS
`partition/aggregate`类型的应用对worker有严格的time limit（all-up SLA），否则会影响用户体验；并且对于99.9%分位数的延迟也有要求。而数据中心网络的流量特征呈现出长流短流突发流混杂的情况。
>**Partition/Aggregate（分区 / 聚合）应用**是一类典型的数据中心应用架构模式，尤其常见于大规模分布式系统（例如 Web 搜索、推荐系统、分布式分析系统等）。它的核心思想是：  
**把一个大请求拆分成多个子任务（Partition），让不同的服务器并行处理，然后将结果汇总（Aggregate）。**

在数据中心网络中，会发生incast（多端口打一）、queue buildup（长流积压队列）、buffer pressure（长流减少突发吸纳的缓存量）等问题。
## ALGORITHM
`DCTCP`的核心方法为根据拥塞程度按照比例进行反应。
