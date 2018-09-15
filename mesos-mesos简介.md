# mesos

## 一：mesos简介

### 1：认识mesos

&emsp;&emsp; Mesos 简介

Apache Mesos是由美国伯克利大学(UCB)的AMPLab研发并贡献到 Apache 基金会的一款开源群集管理系统，支持 Hadoop、ElasticSearch、Spark、Storm 和 Kafka 等应用架构。Mesos特性如下：

- 弹性扩展支持10,000个计算节点
- 使用ZooKeeper 实现 Master 和 Slave 的多容错副本技术
- 支持 Docker 容器技术
- 支持原生的 Linux 容器技术隔离任务
- 基于多资源调度，包括内存，CPU、磁盘、端口
- 提供 Java，Python，C++等多种语言 API接口开发新的分布式应用
- 提供 Web UI 界面查看集群状态

Apache Mesos架构图如下：

![architecture3](C:\Users\Administrator\Desktop\architecture3.jpg)

通过以上架构图，我们可以了解到：

- Mesos本身包含两个组件:Master Daemon和Agent Daemon。
  - Master Daemon
    - 管理所有的 Slave Daemon。
    - 用 Resource Offers 实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。
    - 限制和提供资源给应用框架使用。
    - 使用可拔插的模块化的架构，方便增加新的策略控制机制。
  - Agent Daemon
    - 负责接收和管理 Master 发来的需求任务(Task)
    - 支持使用各种环境运行各种任务(Task)，如Docker、VM、进程调度(纯硬件)。
- Mesos上的任务(Task)由2个组件管理:调度器(Scheduler)和执行进程(Executor Process)
  - 调度器(Scheduler)
    - 调度器通过注册 Mesos Master获得集群资源调度权限
    - 调度器( Scheduler)可通过 MesosSchedule Driver 接口和 Mesos Master 交互
  - 执行进程(Executor Process)
    - 用于启动框架内部的任务(Task)
    - 不同的调度器使用不同的执行进程(Executor Process)
- Mesos 集群为了避免单点故障，所以使用 Zookeeper 提供高容错的副本机制。

### 2 mesos的运行方式

下图描述了一个 Framework 如何通过调度来运行一个 Task 

![architecture-example](C:\Users\Administrator\Desktop\architecture-example.jpg)

事件流程:

1. Agent1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Agent1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Agent1，一个 Task 需要<2个cpu，1 gb内存="">，另外一个Task需要<1个cpu，2 gb内存="">
4. 最后，Master 发送这些 Tasks 给 Agent1。然后，Agent1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

> 注意：当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。

**资源供给**
如同其他的许多集群管理器一样 ， Mesos 集群由一组称为 master 和 slave 的机器组成。集群中的每一个 Mesos slave 以资源供给的形式通告它有效的 CPU、内存和存储资源，如图 1.2 所示，这些资源供给定期地从 slave 发送给 Mesos master，通过调度算法的处理后，再提供给运行在 Mesos 集群上的企amework 调度器 。
**两层调度**
在 Mesos 集群中，资源调度的职责是由 Mesos master 的分配模块和 framework的调度器共同承担的，这就是所谓的两层调度概念 。 如前所述， 资源供给从 Mesosslave 发送到 master 的分配模块，随后分配模块负 责将资源提供给不同的 framework调度器，台amework 调度器依据工作负载情况来决定接受还是拒绝。分配模块是 Mesos master 中的一个可插拔组件，它所实现的算法决定在什么时候将哪一个资源分配给哪一个 framework。组件的模块化性质能让工程师为他们组织定制自己的资源共享策略。
### 3 mesos组件
#### masters
Mesos master 的职责是管理集群中在每台机器上运行的 Mesos slave 守护进程 。通过 ZooKeeper 和 master 之间协调哪个节点是主 master，哪些节点作为备用存在，它们将在主 master 离线时接管服务 。主 master节点使用可插拔的分配模块或调度算法来分发资源供给至各种调度器，从而决定将什么资源提供给某一特定的 framework 。 调度器依据其上是否有任务需要执行来决定接收或拒绝资源供给 。
Mesos 集群至少要求有一个 master 节点 。 在生产环境为了保证高可用性，推荐采用三个甚至更多的 master 节点 。 你可以将 ZooKeeper 在与 master 相同的机器上运行，或者使用独立 ZooKeeper 集群 。
#### slaves
在集群中负责执行famework 任务的服务器被称为 Mesos slave 节点，它们访问ZooKeeper 来确定主 master 节点，将 CPU、内存、存储资源以资源供给的形式宣告给主 master。当调度器从主 master 接收资源供给后，在 slave 节点上启动一个或多个执行器，执行器负责运行 framework 的任务。
Mesos slave 也能够基于属性与资源进行配置，从而允许它们定制特定环境。属性配直是键值对形式，可以包含类似于节点所在机房位置信息。资源配直可以替代Mesos 自动探测发现 slave 节点的有效资源，并由用户指定具体的 CPU、内存、磁
盘资源信息。属性配置与资源配置的示例信息如下：
```
-- attributes='datacenter:pdxl;rack :l-l; os :rhel7 ’
--resources=’ cpu :24 ;mem: 24576 ;disk :409600 ’
```
#### framework

如你前面所了解到的， framework 是表示 Mesos 应用的术语，它负责在集群上调度与执行任务 。 仕amework 由两个组件组成：调度器与执行器。
**提示**：本书写作时存在的 framework 清单请见附录 B 。
##### 调度器
调度器是典型的长运行态服务，负责与 Mesos Master连接，接收或拒绝资源供给 。Mesos 将调度的职责委派给framework，而不是试着由自己调度所有的任务执行。调度器基于当下是否有任务需要运行来决定是否接受或拒绝资源供给 。调度器通过与 ZooKeeper 通信来探测主 master 的存在，之后将其自己注册到 master 中 。
##### 执行器
执行器是在 Mesos slave 上启动的一个进程，负责运行famework 的任务 。 在本书写作之时， Mesos 内建的执行器允许台amework 执行 shell 脚本、 Docker 容器等。Mesos 支持多种编程语 言执行器，新的执行器可以与 framework 绑定在一起，当任务需要它时由 Mesos slave 从famework 获取。如你所看到的 ， Mesos 提供了 一个分布式、高可用的架构， master 负责整个集群的调度工作， slave 将有效资源通知调度器，并在集群中执行任务 。