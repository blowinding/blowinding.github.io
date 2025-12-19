## RDMA-testbed
### RDMA-testbed所需材料
- 交换机-交换机（中性-中性\*16）
	- 8\*DAC电缆（100G,2m,中性-中性，万兆通/1sfpcom）
	- 16\*光模块（100G，QSFP28，中性，飞速）
	- 8\*光纤（MTP，OM4，1m，飞速）
- 交换机-Mellanox网卡（中性-Mellanox\*16）
	- 8\*光模块（100G,QSFP28,Mellanox,飞速）
	- 8\*光模块（100G，QSFP28，中性，飞速）
	- 8\*光纤（MPO，OM4，2m，万兆通/1sfpcom）
	- 5\*AOC光缆（100G，3m，中性-Mellanox，万兆通/1sfpcom）
	- 2\*光模块（100G，QSFP28，中性，飞速）、2\*光模块（100G，QSFP28，Mellanox，飞速）、2\*光纤（MTP，OM4，3m，飞速）
	- 1\*光模块（100G，QSFP28，中性，万兆通/1sfpcom）、1\*光模块（100G，QSFP28，Mellanox，万兆通/1sfpcom）、1\*光纤（MPO，OM4，3m，万兆通/1sfpcom）
- 总计
	- 光模块（100G,QSFP28,Mellanox,飞速）\*10 √
	- 光模块（100G，QSFP28，中性，飞速）\*26 √
	- 光模块（100G，QSFP28，中性，万兆通/1sfpcom）\*1 √
	- 光模块（100G，QSFP28，Mellanox，万兆通/1sfpcom）\*1 √
	- 光纤（MPO，OM4，3m，万兆通/1sfpcom）\*1 √
	- 光纤（MPO，OM4，2m，万兆通/1sfpcom）\*8 √
	- 光纤（MTP，OM4，3m，飞速）\*2 √
	- 光纤（MTP，OM4，1m，飞速）\*8 √
	- DAC电缆（100G,2m,中性-中性，万兆通/1sfpcom）\*8 √
	- AOC光缆（100G，3m，中性-Mellanox，万兆通/1sfpcom）\*5 √
### RDMA-testbed拓扑

| ip             | port | PCIe          |
| -------------- | ---- | ------------- |
| 10.150.240.201 | 7/0  | enp153s0f0np0 |
| 10.150.240.202 | 8/0  | enp153s0f1np1 |
| 10.150.240.203 | 10/0 | enp174s0f0np0 |
| 10.150.240.204 | 9/0  | enp174s0f1np1 |
| 10.150.240.211 | 5/0  | enp10s0f0np0  |
| 10.150.240.212 | 6/0  | enp10s0f1np1  |
| 10.150.240.213 | 12/0 | enp173s0f0np0 |
| 10.150.240.214 | 11/0 | enp173s0f1np1 |
| 10.150.240.221 | 3/0  | enp10s0f0np0  |
| 10.150.240.222 | 4/0  | enp10s0f1np1  |
| 10.150.240.223 | 14/0 | enp173s0f0np0 |
| 10.150.240.224 | 13/0 | enp173s0f1np1 |
| 10.150.240.231 | 1/0  | enp10s0f0np0  |
| 10.150.240.232 | 2/0  | enp10s0f1np1  |
| 10.150.240.233 | 16/0 | enp173s0f0np0 |
| 10.150.240.234 | 15/0 | enp173s0f1np1 |
## extra experiment
### 随机丢包
实验初步结果

|            | 1000      | 10000    | 100000   | 1000000  | no loss  |
| ---------- | --------- | -------- | -------- | -------- | -------- |
| mean_small | 1728.622  | 404.418  | 256.625  | 243.433  | 257.176  |
| mean_large | 2094.184  | 1159.794 | 928.069  | 909.798  | 919.574  |
| p99_small  | 17714.336 | 6743.304 | 4898.529 | 4677.402 | 4881.993 |
| p99_large  | 18728.538 | 9335.806 | 7524.924 | 7221.196 | 7355.450 |

### 拥塞丢包
#### random packet spray
实验配置
- DCQCN参数
	- DCQCN_K_MIN = 580
	- DCQCN_K_MAX = 2520
	- DCQCN_P_MAX = 0.1
- buffer size
	- 50000cells->4000cells
### PFC副作用
### slow-restart+incast
### AI workload
## motivation
- rate limit
	- 收到CNP和loss的速率变化
		- 只开启DCQCN
		- 只开启slow start，但设置不同的DCQCN参数
		- 同时开启DCQCN和slow start
