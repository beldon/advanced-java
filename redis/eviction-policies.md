
淘汰策略（Redis Eviction policies）

> [!WARNING]
> 当内存不足时，Redis 会根据配置配置的缓存策略淘汰部分的 keys，以保证写入成功。
> 当无缓存策略时或者没有找到适合淘汰当 key 时，Redis 直接返回 out of memory 错误。


6 种数据淘汰策略

**1.** volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

**2.** volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

**3.** volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

**4.** allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

**5.** allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

**6.** no-enviction（驱逐）：禁止驱逐数据

原文：

The exact behavior Redis follows when the maxmemory limit is reached is configured using the maxmemory-policy configuration directive.

The following policies are available:

- noeviction: return errors when the memory limit was reached and the client is trying to execute commands that could result in more memory to be used (most write commands, but DEL and a few more exceptions).

- allkeys-lru: evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.

- volatile-lru: evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.

- allkeys-random: evict keys randomly in order to make space for the new data added.

- volatile-random: evict keys randomly in order to make space for the new data added, but only evict keys with an expire set.

- volatile-ttl: evict keys with an expire set, and try to evict keys with a shorter time to live (TTL) first, in order to make space for the new data added.

[Using Redis as an LRU cache](https://redis.io/topics/lru-cache)