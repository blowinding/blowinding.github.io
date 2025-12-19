## Organization
![[25-12-11-6.png]]
## Simulator
### Events and Simulator
ns-3是一个离散事件网络模拟器，模拟器需要以下组成：
- 一个能访问 **事件队列（event queue）** 并负责执行事件的对象（即 `Simulator`）。
- 一个负责事件插入、删除和调度的调度器（Scheduler）。
- 一种表示模拟时间的机制。
- 事件本身。
#### Events
##### 事件定义
**事件** 是改变模拟状态的东西。在两个事件之间，模拟状态不发生改变。
一个更直接的理解是：
👉 “事件” 就像 **⏱ 延迟调用一个函数**。它不是立即执行的，而是在 _将来某个时间点_ 才会被触发。
注意：
- 模拟时间 ≠ 实际真实时间；
- 模拟时间可以比真实时间快或慢；
- 但模拟时间总是 _向前推进_ 的。
一个事件由以下几部分组成：
- 事件发生的时间；  
- 指向“处理该事件”的函数的指针； 
- 事件处理函数所需的参数（如果有）；
- 以及其他内部使用的数据结构。
##### 事件调度和管理
事件通过调用 `Simulator::Schedule` 进行调度。一旦事件被调度，就可以被**取消（cancel）** 或**移除（remove）**：
- **移除（remove）**：  
    表示将事件从调度器的数据结构中彻底删除。
- **取消（cancel）**：  
    事件仍然保留在调度器的数据结构中，但会设置一个布尔标志，使得在计划的时间点**不调用**该事件绑定的处理函数。
当事件通过 `Simulator` 调度时，会返回一个 **EventId**。用户可以使用该事件 ID 在之后取消或移除事件。
##### **事件的执行顺序**
事件被模拟器存储在调度器的数据结构中，并按照**模拟时间递增的顺序**进行处理。
如果两个事件具有**相同的调度时间**，则：
- **唯一 ID 较小的事件会先执行**；
- 这个唯一 ID 是一个单调递增的计数器。
换句话说，**同一时刻的事件按照 FIFO（先到先服务）顺序执行**。
##### 事件执行和时间推进
一般来说，**取消事件在计算开销上比移除事件更低**，但被取消的事件仍然会占用调度器的数据结构内存，这可能会对调度器性能产生影响。
在事件执行期间，**模拟时间不会前进**，也就是说，**每个事件都被认为是在零时间内完成的**。
这是离散事件模拟中的一个常见假设：当事件中执行的计算复杂度可以忽略不计时，该假设是合理的。当这一假设不成立时（例如事件中包含计算量很大的操作），就必须通过**调度第二个事件**来模拟该计算任务的结束。
#### Simulator
`Simulator` 类是访问事件调度机制的**公共入口**。  
当用户已经调度了一些用于启动模拟的事件之后，就可以通过进入模拟器的主循环（调用 `Simulator::Run`）来开始执行模拟。
一旦主循环开始运行，模拟器就会**按时间顺序**（从最早到最新）**依次执行所有已调度的事件**，直到出现以下两种情况之一：
- 事件队列中不再有任何事件；
- 调用了 `Simulator::Stop`。
##### 事件调度接口
为了让模拟器主循环执行事件，`Simulator` 类提供了一组  
**`Simulator::Schedule*` 系列函数** 来调度事件。如：
```
void handler(int arg0, int arg1)
{
  std::cout << "handler called with argument arg0=" << arg0
            << " and arg1=" << arg1 << std::endl;
}

Simulator::Schedule(Seconds(10), &handler, 10, 5);
```

> **注意事项**
> ns-3 的 `Schedule` 方法 **自动识别的函数或方法参数个数最多为 4 个**（即少于 5 个参数）。

`Simulator::Schedule` 方法本质上就是一种**自动构造完全绑定函子（fully-bound）的方式**，完全绑定函子表示一个被调用时不需要再传任何参数的可调用对象。
```
// std::bind
auto f = std::bind(foo, 10, 5);
f();   // OK

// lambda
int x = 10, y = 5;
auto f = [x, y]() {
    foo(x, y);
};
f();   // OK

```
##### **常见的事件调度操作**
`Simulator` API 的设计目标之一就是：**让大多数事件调度操作尽可能简单**。  
它提供了三类常用调度方式（按使用频率从高到低排序）：
1. **`Schedule` 方法**
	- 用于调度一个未来事件；
	- 通过指定相对于当前模拟时间的延迟来确定事件执行时间。
2. **`ScheduleNow` 方法**
    - 用于在**当前模拟时间**调度事件；
    - 该事件会在**当前事件执行完成之后**、**下一个事件的模拟时间推进之前**执行。
3. **`ScheduleDestroy` 方法**
    - 用于在模拟器关闭阶段挂钩清理逻辑；
    - 当用户调用 `Simulator::Destroy` 时，所有 destroy 事件都会被执行。
##### **维护模拟上下文（Simulation Context）**
事件可以通过两种方式调度：
`Simulator::Schedule(Time const &time, MEM mem_ptr, OBJ obj);`
或：`Simulator::ScheduleWithContext(uint32_t context, Time const &time, MEM mem_ptr, OBJ obj);`
在 ns-3 中，**上下文（context）通常用于表示当前事件所属的网络节点 ID**。
ns-3 日志系统的一个关键特性是：
> **自动显示“当前正在执行事件”的网络节点 ID**

这个节点 ID 正是由 `Simulator` 类维护的上下文信息。
- 当前事件的上下文可通过 `Simulator::GetContext()` 获取；
- 上下文是一个 32 位整数；
- 如果某个事件不属于任何特定节点，其上下文会被设置为 `0xffffffff`。
- `Schedule` 和 `ScheduleNow` 方法会 **自动继承当前事件的上下文**
- 也就是说，新调度的事件默认与当前执行事件属于同一个节点
在某些情况下（最典型的是**数据包从一个节点发送到另一个节点**），这种自动继承上下文的行为是**不正确的**。
例如：
- 发送事件的上下文是 **发送节点**
- 接收事件的上下文应该是 **接收节点**
为了解决这个问题，`Simulator` 提供了：`Simulator::ScheduleWithContext(...)`.该接口允许显式指定事件所属的节点 ID。
在极少数情况下，开发者可能需要理解或修改**第一个事件的上下文是如何被设置的**。  
这一过程由 `NodeList` 类完成：
1. 每当创建一个新节点时，`NodeList` 使用 `ScheduleWithContext`  
    为该节点调度一个 **initialize 事件**
2. initialize 事件在节点上下文中执行
3. 该事件调用 `Node::Initialize`
4. `Node::Initialize` 调用每个对象的 `DoInitialize`
5. 某些对象（尤其是 `Application`）会在 `DoInitialize` 中调度：
    - `StartApplication`
    - 流量生成事件
    - 网络层事件
##### 可用的Simulator引擎
- DefaultSimulatorImpl
- DistributedSimulatorImpl
- NullMessageSimulatorImpl
除了基础引擎外，ns-3 还提供了一种通用机制来构建**模拟器适配器**，用于在不修改核心引擎的情况下引入小的行为变化。
- 适配器基类：`SimulatorAdapter`
- 继承自 `SimulatorImpl`
- 使用 **PIMPL（指向实现的指针）** 设计模式
- 默认将所有调用转发给底层核心引擎
- 只需重载特定接口即可实现定制行为
- 适配器可以链式叠加，每个适配器在基础引擎上提供一层行为修改
**当前已有的适配器**
1. **RealtimeSimulatorImpl**
    - 尝试与真实时间同步执行
    - 采用“尽力而为”的时间对齐
    - 通常与 `DefaultSimulatorImpl` 搭配使用
    - 也可用于分布式仿真的实时同步
2. **VisualSimulatorImpl**
    - 提供模拟运行时的实时可视化
    - 显示网络拓扑和数据包传输过程
3. **LocalTimeSimulatorImpl**
    - 为节点引入带噪声的本地时钟
    - 事件基于本地时钟而非全局模拟时间调度
##### **事件级定制钩子**
除了适配器机制外，还有一个特殊的事件级钩子：`SimulatorImpl::PreEventHook(const EventId & id)`。它允许在事件真正执行之前进行一些额外处理（例如状态维护）。
#### Time
ns-3 在内部使用 **64 位有符号整数** 来表示仿真时间点和时间间隔（符号位用于表示负时间间隔）。  
这些时间值并不直接携带单位，而是相对于一个全局配置的 **“时间分辨率（resolution）”单位** 来解释的，该单位采用常见的 SI 单位体系：
> fs、ps、ns、us、ms、s、min、h、d、y

这个单位决定了 **Time 能表示的最小时间步长**。  
时间分辨率 **可以在第一次调用 `Simulator::Run()` 之前设置一次**，之后不能再更改。  
注意：**分辨率并不会存储在 64 位时间值本身中**。
##### Time的构造和操作
Time 对象可以通过以下方式构造：
- 使用所有标准数值类型（采用当前配置的默认时间单位）
- 使用显式单位（例如 `Time MicroSeconds(uint64_t value)`）
Time 支持以下操作：
- 时间比较
- 判断正负或是否为零
- 按指定单位进行取整
- 转换为指定单位的标准数值类型
- 基本算术运算：
    - 加法
    - 减法
    - 与标量（数值类型）的乘法或除法
- 从/向 IO 流读写（输出时可以指定不同于分辨率的单位）
> 当使用浮点数构造时间（如 `Seconds(1.5)`）时，**如果数值小于当前时间分辨率，将被舍入为 0**。
#### Scheduler
Scheduler 类的主要职责是 **维护未来事件的优先队列**。
调度器可以通过全局变量来设置，方式与选择 `SimulatorImpl` 类似：
`GlobalValue::Bind("SchedulerType",                   StringValue("ns3::DistributedSimulatorImpl"));`
也可以在运行时通过以下接口更换调度器：
`Simulator::SetScheduler();`
默认使用的调度器是 **MapScheduler**，它使用 `std::map<>` 按时间顺序存储事件。
##### **为什么有多种调度器？**
由于不同模型中的事件分布差异很大，不存在一种“万能”的事件优先队列实现。因此 ns-3 提供了多种调度器，具有不同的 **时间复杂度** 和 **空间开销**。
工具 `utils/bench-scheduler.c` 可用于测试在特定事件分布下不同调度器的性能。
实践经验：
- 对于运行时间较短（例如少于 1 小时）的仿真，调度器选择通常 **不是主要瓶颈**
- 使用 **优化构建（optimized build）** 对性能的影响通常更大
##### **可用调度器及其复杂度**

下表列出了各调度器在 **Insert()** 和 **RemoveNext()** 操作下的时间和空间复杂度（仅列出主要操作）：

|调度器类型|底层实现|Insert()|RemoveNext()|额外开销|每事件内存|
|---|---|---|---|---|---|
|CalendarScheduler|`<std::list>`|常数|常数|24 字节|16 字节|
|HeapScheduler|`std::vector` 上的堆|对数|对数|24 字节|0|
|ListScheduler|`std::list`|线性|常数|24 字节|16 字节|
|MapScheduler|`std::map`|对数|常数|40 字节|32 字节|
|PriorityQueueScheduler|`std::priority_queue<std::vector>`|对数|对数|24 字节|0|
## Callbacks
