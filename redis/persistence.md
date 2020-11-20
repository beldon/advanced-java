Redis 支持 RDB 和 AOF 两种持久化机制，持久化功能有效地避免因进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数据恢复。 

> [!NOTE]
> 如果主要充当缓存功能，或者可以承受数分钟数据的丢失, 通常生产环境一般只需启用RDB可，此也是默认值
> 如果数据需要持久保存，一点不能丢失，可以选择同时开启RDB和AOF，一般不建议只开启AOF


## RDB

> [!NOTE]
> RDB 持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是 fork 一个子进程，
> 先将数据集**写入临时文件**，写入成功后，再**替换**之前的文件，用**二进制**压缩存储。

> [!TIP]
> - save 命令，塞当前Redis服务器，直到RDB过程完成为止。
> - bgsave 命令，Redis 进程执行 fork 操作创建子进程，RDB 持久化过程由**子进程**负责，完成后自动结束。阻塞只发生在 fork 阶段，一般时间很短。

> [!TIP]
> - 如果从节点执行全量复制操作，主节点自动执行 bgsave 生成 RDB 文件并发送给从节点。
> - 执行debug reload命令重新加载Redis时，也会自动触发save操作。
> - 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则 自动执行bgsave。
 
### 配置  

```shell
save 10 1
# 如“save m n”。表示m秒内数据集存在n次修改 时  
dbfilename dump.rdb  # rdb 文件名
dir ./  # rdb 文件路径

 ## 动态配置 config set dir{newDir} 和 config set dbfilename{newFileName}
```
### 优点 

- RDB 是一个紧凑压缩的二进制文件，代表 Redis 在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每6小时执行 bgsave 备份，并把 RDB 文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复。

- Redis 加载 RDB 恢复数据远远快于 AOF 的方式。


### 缺点

- RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运 行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。  

- RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式 的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题。  

- 针对RDB不适合实时持久化的问题，Redis提供了AOF持久化方式来解决。

### RDB 执行图

![xx](./imgs/rdb.png ':size=400')
![xx](./imgs/rdb2.png ':size=400')

## AOF

> [!NOTE]
> AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，
> 以文本的方式记录，可以打开文件看到详细的操作记录。

### 配置

```shell
#是否开启AOF日志记录，默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了，但是redis如果中途宕机，会导致可能有几分钟的数据丢失(取决于dump数据的间隔时间)，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性，Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。默认不启用此功能
appendonly yes  

appendfilename "appendonly-${port}.aof"          #文本文件AOF的文件名，存放在dir指令指定的目录中

#aof持久化策略的配置
#no表示由操作系统保证数据同步到磁盘,Linux的默认fsync策略是30秒，最多会丢失30s的数据
#always表示每次写入都执行fsync，以保证数据同步到磁盘,安全性高,性能较差
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据,此为默认值,也生产建议值
appendfsync everysec    

dir /path               #快照文件保存路径，示例：dir "/apps/redis/data"

#默认为no,表示"不暂缓",新的aof记录仍然会被立即同步到磁盘，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题
#为yes,相当于将appendfsync设置为no，这说明并没有执行磁盘操作，只是写入了缓冲区，因此这样并不会造成阻塞（因为没有竞争磁盘），但是如果这个时候redis挂掉，就会丢失数据。丢失多少数据呢？Linux的默认fsync策略是30秒，最多会丢失30s的数据,但由于yes性能较好而且会避免出现阻塞因此比较推荐
#rewrite 即对aof文件进行整理,将空闲空间回收,从而可以减少恢复数据时间
no-appendfsync-on-rewrite yes     #在aof rewrite期间,是否对aof新记录的append暂缓使用文件同步策略,主要考虑磁盘IO开支和请求阻塞时间。

auto-aof-rewrite-percentage 100             #当Aof log增长超过指定百分比例时，重写AOF文件，设置为0表示不自动重写Aof日志，重写是为了使aof体积保持最小，但是还可以确保保存最完整的数据
auto-aof-rewrite-min-size 64mb              #触发aof rewrite的最小文件大小

#是否加载由于某些原因导致的末尾异常的AOF文件(主进程被kill/断电等)，建议yes
aof-load-truncated yes                      

#redis4.0新增RDB-AOF混合持久化格式，在开启了这个功能之后，AOF重写产生的文件将同时包含RDB格式的内容和AOF格式的内容，其中RDB格式的内容用于记录已有的数据，而AOF格式的内容则用于记录最近发生了变化的数据，这样Redis就可以同时兼有RDB持久化和AOF持久化的优点（既能够快速地生成重写文件，也能够在出现问题时，快速地载入数据）,默认为no,即不启用此功能
aof-use-rdb-preamble no 
```

### AOF 重写

### 优点 

- 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而每修改同步，我们可以将其视为同步持久化，即每次发生的数据变化都会被立即记录到磁盘中。可以预见，这种方式在效率上是最低的。至于无同步，无需多言，我想大家都能正确的理解它。

- 由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。然而如果我们本次操作只是写入了一半数据就出现了系统崩溃问题，不用担心，在Redis下一次启动之前，我们可以通过redis-check-aof工具来帮助我们解决数据一致性的问题。

- 如果日志过大，Redis 可以自动启用 rewrite 机制。即 Redis 以 append 模式不断的将修改数据写入到老的磁盘文件中，同时 Redis 还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

- AOF 包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

### 缺点

- 即使有些操作是重复的也会全部记录，AOF 的文件大小要大于 RDB 格式的文件
- AOF 在恢复大数据集时的速度比 RDB 的恢复速度要慢
- 根据 fsync 策略不同，AOF 速度可能会慢于 RDB


### RDB 执行图



![xx](./imgs/aof.png ':size=400')
![xx](./imgs/aof2.png ':size=400')

## 参考文档

- [persistence](https://redis.io/topics/persistence)  
- [persistence（中文）](http://redisdoc.com/topic/persistence.html)
- [10分钟彻底理解Redis的持久化机制：RDB和AOF](https://juejin.cn/post/6844903874927525902)