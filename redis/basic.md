
## Redis 常用命令

全局命令

| 命令                | 描述           |
| ------------------- | :------------- |
| Keys *              | 查看所有命令   |
| dbsize              | 查看键总数     |
| exists key          | 查看键是否存在 |
| del key             | 删除键         |
| Expire key seconds  | 键过期         |
| type key            | 键类型         |
| object encoding key | 查看内部编码   |

> [!TIP]
> type 命令实际返回的是当前键的数据结构类型， object encoding 是查看内部编码。


```
字符串：
        setnx(key,value)  只在键 key 不存在的情况下， 将键 key 的值设置为 value 。key存在,不做任何操作。
        setex(key,seconds,value)   将key设置及生存时间seconds秒,原值存在覆盖。
        psetex(key,milliseconds,value) 与setex同样,只是单位是毫秒。
        getset(key,value)   设置新值并返回旧值,不存在返回nil
        setrange(key,offset,value) 从偏移量开始offset开始
        mset 同时给多个key复制
哈希表(map)：
        hset(hash field value)  将哈希表 hash 中域 field 的值设置为 value
        hmset key field value [field value …] 同时将多个 field-value (域-值)对设置到哈希表 key 中。
        hget hash field  返回哈希表中给定域的值。
        hgetall key      返回哈希表 key 中，所有的域和值。
队列(queue):
        lpush key value [value …] 将一个或多个值 value 插入到列表 key 的表头
        lpop key   移除并返回列表 key 的头元素,不存在返回nil
        lset key index value  将列表 key 下标为 index 的元素的值设置为 value 。
        brpop key [key …]  timeout 在超时时间内移除列表尾元素，阻塞的。
集合：
        sadd key member [member …]  将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略
        sismember key member  如果 member 元素是集合的成员，返回 1 。 如果 member 元素不是集合的成员，或 key 不存在，返回 0 。
        spop key  移除集合key的随机元素
        smembers key  返回集合 key 中的所有成员。
        sdiff key [key …]  返回给定多个集合之间的差集。
有序集合：
        zadd key score member [[score member] [score member] …]   将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
        zscore key member 返回有序集 key 中，成员 member 的 score 值。
        zcount key min max   score 值在 min 和 max 之间的成员的数量。
        zrange key start stop [withscores]  返回有序集 key 中，指定区间内的成员(从小到大)。
        zrank key member  返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列。
        zrem key member [member …]   移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。
时效性：
        expire(key,seconds) 为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。
        expireat( key,timestamp) 设置过期时间戳,expireatcache1355292000# 这个 key 将在 2012.12.12 过期
        ttl(key) 返回剩余时间
        persist key 移除key有效期，转换成永久的
数据指令：
        keys pattern   符合给定模式的 key 列表。阻塞的
        scan  异步的  有重复
```

> [!WARNING]
> - persist 命令可以删除任意类型键的过期时间，但是 set 命令也会删除**字符串类型键**的过期时间。
> - set 命令只会删除**字符串类型键**的过期时间。

[Redis data types](https://redis.io/topics/data-types-intro)

## 为什么单线程还能这么快

> [!NOTE]
> - 纯内存访问
> - 非阻塞I/O（select/poll/epoll模型）
> - 单线程避免了线程切换和竞态产生的消耗

> [!WARNING]
> 由于是单线程，所有 redis 对每个命令的执行时间是有要求的。如果某个命令执行过长，会造成其他命令的阻塞。

![xx](./imgs/poll.png ':size=500')

## 最大缓存设置

在 Redis 中，允许用户设置的最大使用内存是 512G。

配置是: **server.maxmemory**

在内存限定的情况下是很有用的。譬如，在一台 8G 机子上部署了 4 个 redis 服务点，每一个服务点分配 1.5G 的内存大小，减少内存紧张的情况，由此获取更为稳健的服务。


## Redis 回收策略

过期键删除策略种类
- 立即删除

- 事件删除

- 惰性删除

放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，就返回该键

- 定时任务删除

每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定

**间隔小则占用CPU,间隔大则浪费内存**


redis采用后两种结合的方式

- 读写一个key时，触发惰性删除策略
- 惰性删除策略不能及时处理冷数据，因此redis会定期主动淘汰一批已过期的key
- 内存超过maxmemory时，触发主动清理

## 缓存穿透、缓存击穿、缓存雪崩、缓存预热

### 缓存穿透

指查询一个一定不存在的数据，由于缓存是不命中时查询数据库。若流量大时，数据库可能会宕机。

- 解决方案

**1.** 布隆过滤器：将所有可能存在的key放到一个足够大bitmap中。一定不存在的数据直接被bitmap拦截掉。  
**2.** 短时缓存：若数据库查询的数据为空，扔把key缓存起来，设置一个短时间的过期时间。数据插入后清理。

### 缓存击穿

某个热点key失效后，大并发访问,导致数据库宕机。

- 解决方案

**1.** 加锁排队：互斥锁对某个key只允许一个线程查数据写缓存，其他线程等待。  
**2.** 永久缓存：不设置超时时间  
**3.** 永久缓存，定时刷新：定时任务刷新或者删除缓存

### 缓存雪崩

缓存时集中在某一时段同时失效，请求全部转发到数据库，数据库瞬时压力过重导致雪崩效应。

- 解决方案

缓存时间增加随机值：每个缓存时间不一样，避免集体失效。

### 缓存预热

系统上线前把热点数据加入redis缓存中。     
