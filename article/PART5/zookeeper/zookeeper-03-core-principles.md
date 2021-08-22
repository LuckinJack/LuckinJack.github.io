## 集群数量



> **[!NOTE]**
>
> **❑** ZooKeeper集群节点数必须是奇数
>
> - ZooKeeper集群中需要一个主节点，也称为Leader节点，并且Leader节点是集群通过选举规则从所有节点中选出来的，简称为选主。选主规则中很重要的一条是：**可用节点数量 > 总节点数量 / 2**，这种规则也叫 **节点数过半机制**。
> - **节点数过半机制** 的作用是为了防止 **集群脑裂（Split-Brain）**，即网络断了，一个集群被分成了两个集群。（......................）。ZooKeeper 集群、ElasticSearch 集群都使用 **节点数过半机制** 来确保集群被分裂后还能正常工作。
>
> 





## leader选举机制





## Paxos算法

https://blog.csdn.net/u013679744/article/details/79222103

![img](C:\Users\Administrator\Desktop\epub_38894783_76)

## Zookeeper一致性协议——Zab协议

[Zookeeper——一致性协议:Zab协议 - 简书 (jianshu.com)](https://www.jianshu.com/p/2bceacd60b8a)

[Zookeeper系列（5）--ZAB协议，消息广播，崩溃恢复，数据同步](https://blog.csdn.net/u013679744/article/details/79240249)

在上面介绍了经典的分布式数据一致性算法Paxos算法，但事实上zookeeper采用的是简化版的Paxos算法，并没有完全使用，而是采用了一种称为 **Zookeeper Atomic Broadcast (ZAB，zookeeper原子消息广播协议)**。 ZAB 协议是为分布式协调服务 ZooKeeper 专门设计的一种支持崩溃恢复的原子广播协议。在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，ZooKeeper 实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性，同时其崩溃恢复过程也确保看zk集群的高可用性（HA）
