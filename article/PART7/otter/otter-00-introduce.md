

# 什么是Otter

**❑ 项目背景**



**❑ 语言**

- 纯java开发



**❑ 定位**

- 基于数据库增量日志解析，准实时同步到本机房或异地机房的MySQL/Oracle数据库



**❑ 模块组成**

 - Zookeeper：分布式一致性协调服务

 - Canal：获取数据库增量日志数据

 - manager：管理中心，用来配置同步信息，接收node模块发来的状态反馈

 - node：node 模块内嵌 Canal，Canal 监听数据库binlog中的变化传送给node的SETL模块

   

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/otter-structure-01.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">otter架构示意图</div>
	<br>
	<br>
</center>
**❑ 架构描述**

- 基于Canal开源产品，获取数据库增量日志数据
- manager（web管理）+ node（工作节点）：典型管理系统架构
  - manager运行时推送同步配置到node节点
  - node节点将同步状态反馈到manager上
- 基于zookeeper，解决分布式状态调度的，允许多node节点之间协同工作





# Otter 提供的解决方案

> **[!NOTE]**
> **❑  异构库同步**
>
> - MySQL -> MySQL/Oracle 
> >（※ 目前开源版本只支持MySQL增量，目标库可以是MySQL/Oracle ，取决于Canal的功能)
>
> **❑  单机房同步 **
>
> >（※ 数据库之间RTT < 1ms)
>
> - 数据库版本升级
> - 数据表迁移
> - 异步二级索引
>
> **❑  异地机房同步**
>
> >（ ※ 比如阿里巴巴国际站就是杭州和美国机房的数据库同步，RTT > 200ms，**亮点**）
>
> - 机房容灾
>
> **❑  双向同步**
>
> - 避免回环算法（通用的解决方案，支持大部分关系型数据库）
> - 数据一致性算法 （保证双A机房模式下，数据保证最终一致性，**亮点**）
>
> **❑  文件同步**
>
> - 站点镜像 （进行数据复制的同时，复制关联的图片，比如复制产品数据，同时复制产品图片）



### 单机房同步复制

![img](https://camo.githubusercontent.com/0467bd29932b2a5c8cc4c3c7b20b507d3ae1f1f709692daa91deae7777f06d79/687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038382f313937352f64656465323263382d353963612d333738612d393064352d3466343562323839616233302e6a7067)

说明：

   a. 数据on-Fly，尽可能不落地，更快的进行数据同步. (开启node loadBalancer算法，如果Node节点S+ETL落在不同的Node上，数据会有个网络传输过程)

   b. node节点可以有failover / loadBalancer.

### 异地机房同步复制

![img](https://camo.githubusercontent.com/900b1d434d634ed907fa694e329b91068882d6b541ccabf0cc468a60b1fba9c1/687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038382f313938312f35333639623533332d356239612d333265362d626263302d3134633430373138386539332e6a7067)

说明：

   a. 数据涉及网络传输，S/E/T/L几个阶段会分散在2个或者更多Node节点上，多个Node之间通过zookeeper进行协同工作 (一般是Select和Extract在一个机房的Node，Transform/Load落在另一个机房的Node)

   b. node节点可以有failover / loadBalancer. (每个机房的Node节点，都可以是集群，一台或者多台机器)



# 相关知识点解释

## Otter 核心model关系图

![img](https://camo.githubusercontent.com/5a6fb96bcea47f40ee166038648522f73954c18dff6ddc228ea828f07e872b03/687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038382f333034382f61353538336463322d613333372d333538332d386239322d3034323938656636666237342e6a7067)



## 名词解释

- Pipeline：从源端到目标端的整个过程描述，主要由一些同步映射过程组成
- Channel：同步通道，单向同步中一个Pipeline组成，在双向同步中有两个Pipeline组成
- DataMediaPair：根据业务表定义映射关系，比如源表和目标表，字段映射，字段组等
- DataMedia : 抽象的数据介质概念，可以理解为数据表/mq队列定义
- DataMediaSource : 抽象的数据介质源信息，补充描述DateMedia
- ColumnPair : 定义字段映射关系
- ColumnGroup : 定义字段映射组
- Node : 处理同步过程的工作节点，对应一个jvm

\----

## Otter 的S/E/T/L stage阶段模型

![img](https://camo.githubusercontent.com/57d239c84c5e6d79f4caa0013a5a9776563cfd07b2e3bf88fe42344c1a6e1e86/687474703a2f2f646c322e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038382f333037332f36643665346630332d323364662d333735392d623764622d3337306530386337623334632e706e67)

说明：为了更好的支持系统的扩展性和灵活性，将整个同步流程抽象为Select/Extract/Transform/Load，这么4个阶段.

Select阶段: 为解决数据来源的差异性，比如接入canal获取增量数据，也可以接入其他系统获取其他数据等。

Extract/Transform/Load 阶段：类似于数据仓库的ETL模型，具体可为数据join，数据转化，数据Load



# Otter FAQ

❑ **Otter与Canal的依赖关系**

- Otter 目前嵌入式依赖 Canal，主要靠Canal解决数据库增量日志解析，两者部署为同一个jvm，目前设计为不产生 *Relay Log* ，数据不落地
- 由于 Otter 受到内嵌的 Canal 的版本限制，对数据库的支持如下：
  - Otter <= v4.2.16 支持 MySQL的5.1 ~ 5.7版本
  - Otter >= v4.2.17（内嵌Canal >= v1.1.2）增加了对 MySQL 8.0 的解析支持
  - 仅支持MySQL作为主库（Master），MySQL/Oracle作为从库（Slave）
- Canal 支持 mixed、row、statement 多种日志协议的解析，**但配合Otter进行数据库同步，目前仅支持row协议的同步**



# 参考

- [Introduction · alibaba/otter Wiki · GitHub](https://github.com/alibaba/otter/wiki/Introduction)

- [Faq · alibaba/otter Wiki · GitHub](https://github.com/alibaba/otter/wiki/Faq)