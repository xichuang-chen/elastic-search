# Q&A

## 为什么要使用elasticsearch
- 模糊查询，会造成查询引擎放弃索引（例如mysql）， 进行全表扫描; 而es倒排索引
- es有很完善的生态， 可以与很多框架结合使用

## elasticsearch master节点

### 选举流程
- es选 master 由 ZenDiscovery 模块负责，主要通过Ping（节点之间通过RPC发现彼此）和 Unicast（单播模块包含一个主机列表以控制
哪些节点需要 ping 通）这两部分
- 对所有可以成为master的节点（node.master:true）根据nodeId字典排序，每次选举，每个节点都把自己知道的节点排一次序，然后选第一个节点
暂定它为master节点
- 如果某个节点的投票数达到一定的值（可以成为master节点数 n/2+1）并且该节点自己选举自己，那这个节点就是master

### master职责
- 集群、节点和索引的管理。 索引的创建、删除等； 决定分片分到哪个node等
- 尽量不要负责data node的事情，因为很耗cpu，会增大延迟，容易造成其他节点以为 master 挂掉

## elasticsearch 集群脑裂问题
什么是脑裂？
集群中出现了多个master节点

### 原因
- 网络问题  
    集群间网络延迟导致一些节点访问不到master，认为master挂掉，而选举出新的master
- 节点负载  
    master 节点即为 master 又为 data，访问量较大造成ES停止响应造成大面积延迟，其他节点以为挂掉，重新选取主节点

### 解决方案
- 减少误判  
    `discovery.zen.ping_timeout`节点状态响应时间，默认3s, 适当调大，减少误判
- 角色分离  
    master 节点 master 和 data 的分离

## 并发情况下，es如何保证读写一致
- 通过版本号乐观锁并发控制，确保新版本不会被旧版本覆盖，由应用层处理冲突
- 设置只有当副分片拷贝完之后，才认为插入成功，从而保证 主片和副片数据一致