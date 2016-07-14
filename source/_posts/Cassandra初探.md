title: Cassandra初探
tags: []
categories:
  - Distributed System
date: 2016-07-13 11:38:00
---
title: Cassandra初探
tags: 
---

Cassandra是一个去中心化的、高可靠性高可扩展性的、高吞吐率的分布式存储系统。它是由Facebook开发，论文发表在[paper address](http://dl.acm.org/citation.cfm?id=1773922)上。这里给出这篇论文中描述的关键概念。

# Data Model
表是一组索引到一组多维数据的映射，就像传统数据库一样。数据列被组织为*column family*，一个column family可以包括多个列，此时存取单列记为*column_family : column*。除此之外还有*super column family*，一个super column family可以包括多个column family，此时存取单列记为*column_family : super_column : column*。

# API
三个简单的方法：
```java
insert(table,key,rowMutation)
get(table,key,columnName)
delete(table,key,columnName)
```
这里的*columnName*可以是column family中的column，可以是column family，可以是super column family，可以是super column中的column。

# Partitioning
Partitioning就是将数据分布到不同节点的特性。通过将负载高节点的数据转移到低负载节点，可以提高系统的可扩展性。并且Cassandra支持增量式扩展(scale incrementally)。使用的核心技术是*consistent hashing*，节点随机分配到标识符环上，key都被hash到一个标识符（和节点的标识符属于同一个标识符空间）。key找到第一个满足标识符大于key的标识符的节点，这个节点是这个key的coordinator。对这个key的请求会被route到这个节点。节点的加入和离开只会影响邻居节点管制的key，容易增量式扩展。基本的*consistent hashing*带来两个问题，一个是数据负载可能不均衡(?)，第二个是节点的性能可能存在差异。Cassandra的做法是分析负载信息，移动低负载节点的位置从而降低高负载节点的压力。

# Replication
Cassandra使用复制提高可用性和持续性(availability and durability)。可以指定一个复制因子$N$，系统会将数据复制到$N$个不同的节点。Coordinator负责将数据复制到其他$N-1$个节点中。有几种复制策略选择：“Rack Unaware”会复制到coordinator后面的$N-1$个节点；“Rack Aware”和“Datacenter Aware”会利用Zookeeper选择出一个leader，leader指定数据的其他$N-1$replica位置，节点加入时leader告知它负责哪些range，并且leader维护一个不变式使得每个节点不作为超过$N-1$个range的replica。Cassandra借用Dynamo的说法，将负责某个range的那些节点称为range的“preference list”。

通过将每行数据复制到不同的data center（推测应该是Datacenter Aware策略的结果），Cassandra提供了持续性（durability）保障。这样即使整个data center failure都可以处理。

# Membership
Cluster membership基于Scuttlebutt（我并不了解），Scuttlebutt是一个高效的anti-entropy Gossip based mechanism（并没有详细介绍）。Cassandra中gossip并不仅仅用于membership还被用来分发系统控制状态。

Cassandra使用修改版的$\Phi$ Accural Failure Detector来探测节点failure。这个failure detection module为每个节点释放一个$\Phi$值，这个值在一个量表中表征某个含义（含义略）。

# Bootstrapping
节点在加入时随机选择一个标识符，这个映射会被存储到硬盘盒Zookeeper上。标识符信息会被gossip给cluster的所有节点，cluster的节点就可以知道标识符环的节点位置。节点bootstrap的加入需要初始contact points，Cassandra称之为seeds，通常由Zookeeper配置。

（Facebook相关）Facebook环境中节点会暂时断开但并不意味着永久离开，因此不需要partitioning的重新平衡和unreachable replica的repair。因此节点的加入和离开由人来控制，然而人可能犯错，因此一个Cassandra实例的消息都带有这个实例的名字。当人工操作失误时，操作将被制止。

# Local Persistence
涉及到两个存储结构：*commit log*以及*in-memory data structure*。写操作会触发对commit log和in-memory data structrue的写入。对后者的写入发生在对前者的写入完成后，这样即使发生错误也能从commit log恢复。commit log dump到在每个节点配备的较为昂贵的磁盘中，in-memory data structure dump到在廉价磁盘中。当in-memory data structure超过一定大小会被dump。对commit log和in-memory data structure的写入会产生一个索引便于后续查找。一个后台过程会负责把dump到磁盘的文件合并为一个文件。

读操作首先查询in-memory data structure，如果没找到会从最新到最旧查看文件。可能会从多个文件查找某个key，于是Cassandra为了避免对不包括key的文件进行查找，采用了*bloom filter*方法。bloom filter存储在文件和内存中，查找文件前会首先查询bloom filter。由于所要求的column可能包含多个column，并且column数据可能在文件中与key距离很远，所以Cassandra维护column索引快速存取column数据。一个key的columns是串行dump到文件中，Facebook每256K创建一个索引。

# Implementation Details
包含这几个模块：partitioning模块、cluster membership and failure detection模块、storage engine模块。这些模块由事件驱动，下层是消息处理流水线和任务流水线。网络层使用的是非阻塞型io。系统控制消息使用的是UDP，和应用相关的消息（复制、request routing）使用的是TCP。

request routing模块是一个状态机：
1. 找到具有这个key的节点（们）
1. 把请求route到节点（们）并等待回应
1. 如果在给定timeout内没有回应，向client返回失败
1. 根据时间戳确定最新的回应
1. schedule repair数据，即repair没有最新的数据的replica

写入操作可以是同步型或者异步型，同步型需要一定数量的replica的回应，异步型则（或许）只需要一个replica的回应。