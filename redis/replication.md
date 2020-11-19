

## 集群

### 哨兵模式（sentinel）

核心是主从复制（一主多从方式），能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。竞选机制的实现，是依赖于在系统中启动一个 sentinel 进程。
哨兵 sentinel 特点：监控、消息通知、自动故障转移、提供主服务器地址。

哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，很难支持在线扩容。

![xx](./imgs/sentinel.png ':size=500x500')


### 集群（cluster）

redis-cluster 采用无中心结构，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。集群部署至少要 3 台以上的 master 节点，最好使用 3 主 3 从六个节点的模式：

![xx](./imgs/cluster.png ':size=500x500')

其结构特点

- 所有的 redis 节点彼此互联（PING-PONG机制），内部使用二进制协议优化传输速度和带宽。
- 节点的 fail 是通过集群中超过半数的节点检测失效时才生效。
- 客户端与 redis 节点直连，不需要中间 proxy 层.客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。
- redis-cluster 把所有的物理节点映射到 [0-16383] slot 上（不一定是平均分配），cluster 负责维护 node<->slot<->value。
- Redis集群预分好 **16384** 个桶，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384 的值，决定将一个 key 放到哪个桶中。
- redis-cluster 选举: 选举过程中所有 master 参与，超过半数以上 master 间节点通信，认为当前 master 节点挂掉。如果集群任意 master 挂掉且没有 slave 节点，集群不可用。超过半数以上 master 挂掉，也不可用。

相对于单机redis，功能上有一些限制：
- key 批量操作支持有限；
- 只支持多 key 在同一节点上的事务操作，当多个 key 分布在不同节点时，不支持事务操作；
- key 作为数据分区的最小粒度，不能将一个大的键值对象如 hash, list 等映射到不同的节点；


## 参考资料

[Replication](https://redis.io/topics/replication)