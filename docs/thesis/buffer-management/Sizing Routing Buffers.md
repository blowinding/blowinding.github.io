## ABSTRACT
n flows仅需要$B=RTT\times{C}/{\sqrt{n}}$的缓存，传统公式$B=RTT\times{C}$有误（C代表Link Capacity）。

## MAIN IDEA
需要保证buffer不会因为sender暂停发送或减缓发送速率而导致缓冲区排空，进而导致链路利用率降低。而传统公式在单TCP流的情况下可以保证缓冲区不排空。
## LONG FLOW
绝大多数情况下，TCP长流一般不会同步，因此经推算可得其链路利用率的下界为：
$$Util\geq{erf(\frac{3\sqrt{3}}{2\sqrt{2}}\frac{B}{\frac{2{\overline{T}_P\cdot{C}+B}}{\sqrt{n}}})}$$
## SHORT FLOW
由于短流一般在TCP拥塞控制算法达到稳定状态前结束，因此不能像长流一样进行稳态分析，而是使用`M/G/1`队列模型分析。
> M/G/1队列指：
> - 突发到达服从泊松过程（M）
> - 每个突发包含的分组数量可以是任意分布（G）
> - 缓冲区只有一个“服务器”，即瓶颈链路（1）
## THINKING
- AQM对3.2节中缓存大小的分布有什么影响，会影响到buffer sizing吗
- 数据中心网络存在的incast问题、长流挤占短流问题，会影响到buffer sizing吗
	- flows同步导致所需buffer增加，不同步可缩减所需buffer，是否和workload有关
	- 长流挤占短流需要有更多的buffer进行突发吸纳（能不能定量分析）
- 论文中研究Internet中的Router，如果拓扑较为复杂，如fat-tree、leaf-spine，该如何考虑每个节点上的buffer sizing