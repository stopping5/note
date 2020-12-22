

# 远程调用

HttpClient java层面的远程调用

RestTemplate 封装HttpClient

OkHttp

Fegin

# 一、微服务





二、协议

Http：应用层协议

HTTP 的优势在于它的成熟稳定、使用实现简单、被广泛支持、兼容性良好、防火墙友好、消息的可读性高。所以 http 协议在开放 API、跨平台的服务间调用、对性能要求不苛刻的场景中有着广泛的使用

RPC：用来解决服务和服务调用。

RPC 的优势在于高效的网络传输模型（常使用 NIO 来实现），以及针对服务调用场景专门设计协议和高效的序列化技术。

 



# 三、微服务场景

1、服务地址如何维护

- 注册
- 获取

 2、如何对目标服务做负载均衡

- 负载均衡算法 随机算法和轮询算法

3、服务动态感知（某服务挂了如何通知）

- heartbeat



# 四、解决方案 -zookeeper

ZooKeeper is a==centralized service== for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

**分布式协调**组件服务。基于google Chubby思想的实现。

![image-20200601165902303](H:\TyporaImage\咕泡学习\微服务\Zookeeper\image-20200601165902303.png)

视频时间：1:54

zk实现的机制：

1、临时节点、持久化节点

2、watcher机制

3、负载均衡（拿到集群所有的节点合理的分配）

4、命名服务

通过zk管理所有的名字

5、分布式锁

 

## 1、分布式一致性问题

### 什么是分布式一致性问题

多个节点同时发出请求，达成结果一致。简单来说，就是在一个分布式系统中，有多个节点，每个节点都会提出一个请求，但是在所有节点中只能确定一个请求被通过。而这个通过是需要所有节点达成一致的结果，所以所谓的一致性就是在提出的所有请求中能够选出最终一个确定请求。并且这个请求选出来以后，所有的节点都要知道。

### 如何解决

协议（paxos）

## 2、分布式锁服务

从另外一个层面来看，Chubby 提供了一种粗粒度的分布式锁服务，chubby 是通过创建文件的形式来提供锁的功能，server 向 chubby 中创建文件其实就表示加锁操作，创建文件成功表示抢占到了锁。由于 Chubby 没有开源，所以雅虎公司基于 chubby 的思想，开发了一个类似的分布式协调组件 Zookeeper，后来捐赠给了 Apache。所以，大家一定要了解，zookeeper 并不是作为注册中心而设计，他是作为==分布式锁==的一种设计。而注册中心只是他能够实现的一种功能而已。

## 3、zookeeper实现

![image-20200601174905817](H:\TyporaImage\咕泡学习\微服务\Zookeeper\image-20200601174905817.png)

#### 数据结构

#### 解决单点问题

集群

#### 防止单点故障

首先，在分布式架构中，任何的节点都不能以单点的方式存在，因此我们需要解决单点的问题。常见的解决单点问题的方式就是**集群**。

1. 集群中要有主节点和从节点（也就是集群要有角色）主节点（Master）用于增删改其他从节点同步主节点的内容。减少维护工作。zookeeper 是 leader/follower

2. 集群要能做到数据同步，当主节点出现故障时，从节点能够顶替主节点继续工作，但是继续工作的前提是数据必须要主节点保持一直。

3. 主节点挂了以后，从节点如何接替成为主节点？ 是人工干预？还是自动选举。一致性算法。leader选举算法。（paxos->zab协议）

#### 数据同步

接着上面那个结论再来思考，如果要满足这样的一个高性能集群，我们最直观的想法应该是，每个节点都能接收到请求，并且每个节点的数据都必须要保持一致。要实现各个节点的数据一致性，就势必要一个 leader 节点负责协调和数据同步操作。这个我想大家都知道，如果在这样一个集群中没有 leader 节点，每个节点都可以接收所有请求，那么这个集群的数据同步的复杂度是非常大。

所以，当客户端请求过来时，需要满足，事务型数据（增删改）和非事务型数据（查）的分开处理方式，就是leader 节点可以处理事务和非事务型数据。而 follower 节点只能处理非事务型数据。原因是，对于数据变更的操作，应该由**一个节点来维护**，使得集群数据处理的简化。同时数据需要能够通过 leader 进行分发使得数据在集群中各个节点的一致性

![image-20200601202911025](H:\TyporaImage\咕泡学习\微服务\Zookeeper\image-20200601202911025.png)

leader 节点如何和其他节点保证数据一致性，并且要求是强一致的。在分布式系统中，每一个机器节点虽然都能够明确知道自己进行的事务操作过程是成功和失败，但是却无法直接获取其他分布式节点的操作结果。所以当一个事务操作涉及到跨节点的时候，就需要用到分布式事务，分布式事务的数据一致性协议有 **2PC **协议和 **3PC **协议

###### 2pc（Two Phase Commitment Protocol）

当一个事务操作需要跨越多个分布式节点的时候，为了保持事务处理的 ACID 特性，就需要引入一个“协调者”（TM）来统一调度所有分布式节点的执行逻辑，这些被调度的分布式节点被称为 AP。TM 负责调度 AP 的行为，并最终决定这些 AP 是否要把事务真正进行提交；因为整个事务是分为两个阶段提交，所以叫 2pc。

第一阶段：事务查询

第二阶段：事务提交

zookeeper的设计：

在读多写少的情况下，节点越多事务处理越多，如何处理这种情况？

新增Observer角色不参与投票，但是可以同步数据，最终一致性。



# 五、zookeeper的应用

1、官方网站

https://zookeeper.apache.org/releases.html

 2、基本特性

- 创建节点 -create
  - 顺序节点
  - 统一级别的节点不能重复
  - 创建节点时，必须要带上全路径
  - 临时节点的特性
    - 关闭会话自动删除
- 删除节点 -delete
  - 节点只能一层一层的删除
  - deleteall可以递归删除
- 改变节点
  - set /node 123
- 