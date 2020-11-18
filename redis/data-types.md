

## 概述

- String
- Lists
- Sets
- Hashes
- Sorted sets
- Bitmaps
- HyperLogLogs

![xx](./imgs/data-types.png ':size=500x500')![xx](./imgs/data-types2.png ':size=500x500')

## 类型



### Stream

Redis Stream 是 Redis 5.0 版本新增加的数据结构，提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。



参考链接：

- [Redis Stream](https://www.runoob.com/redis/redis-stream.html)

- [Introduction to Redis streams](https://redis.io/topics/streams-intro)

### GEO

GEO 主要用于存储地理位置信息，并对存储的信息进行操作，该功能在 Redis 3.2 版本新增。

命令：

#### geoadd：添加地理位置的坐标

```shell
GEOADD key longitude latitude member [longitude latitude member ...]
```

#### geopos：获取地理位置的坐标

```shell
GEOPOS key member [member ...]
```

#### geodist：计算两个位置之间的距离

```shell
GEODIST key member1 member2 [m|km|ft|mi]
```

- 最后一个距离单位参数说明
	- m ：米，默认单位。
	- km ：千米。
	- mi ：英里。
	- ft ：英尺。

#### georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合

```shell
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```

#### georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合

```shell
GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```

- 参数说明：

  - m ：米，默认单位。
  - km ：千米。
  - mi ：英里。
  - ft ：英尺。
  - WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
  - WITHCOORD: 将位置元素的经度和维度也一并返回。
  - WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。
  - COUNT 限定返回的记录数。
  - ASC: 查找结果根据距离从近到远排序。
  - DESC: 查找结果根据从远到近排序。

#### geohash：返回一个或多个位置对象的 geohash 值

```shell
GEOHASH key member [member ...]
```

### HyperLogLogs

A HyperLogLog is a probabilistic data structure used in order to count unique things (technically this is referred to estimating the cardinality of a set).

| 命令                                      | 说明                                      |
| ----------------------------------------- | ----------------------------------------- |
| PFADD key element [element ...]           | 添加指定元素到 HyperLogLog 中。           |
| PFCOUNT key [key ...]                     | 返回给定 HyperLogLog 的基数估算值。       |
| PFMERGE destkey sourcekey [sourcekey ...] | 将多个 HyperLogLog 合并为一个 HyperLogLog |

### Bitmap

命令

- setbit key offset value
- getbit key offset
- bitcount key [start] [end]
- bitpos
- bitop operation destkey key [key ...]
	- operation` 可以是 `AND` 、 `OR` 、 `NOT` 、 `XOR` 这四种操作中的任意一种：
	- `　　BITOP AND destkey key [key ...]` ，对一个或多个 `key` 求逻辑并，并将结果保存到 `destkey` 。 
	- `　　BITOP OR destkey key [key ...]` ，对一个或多个 `key` 求逻辑或，并将结果保存到 `destkey` 。
	- `　　BITOP XOR destkey key [key ...]` ，对一个或多个 `key` 求逻辑异或，并将结果保存到 `destkey` 。
	- `　　BITOP NOT destkey key` ，对给定 `key` 求逻辑非，并将结果保存到 `destkey` 。

除了 `NOT` 操作之外，其他操作都可以接受一个或多个 `key` 作为输入。　　　　

返回值：保存到 `destkey` 的字符串的长度，和输入 `key` 中最长的字符串长度相等

- bitfield

应用场景

- 位图计数统计
- 日活跃用户数

## 参考链接

- [Data types short summary](https://redis.io/topics/data-types)
- [Introduction to Redis data types](https://redis.io/topics/data-types-intro)

