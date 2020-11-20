Redis 支持 RDB 和 AOF 两种持久化机制，持久化功能有效地避免因进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数据恢复。 

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
appendonly yes  # 开启 aof
appendfilename appendonly.aof # aof 文件
```

### 优点 

### 缺点


### RDB 执行图



![xx](./imgs/aof.png ':size=400')
![xx](./imgs/aof2.png ':size=400')

## 参考文档

- [persistence](https://redis.io/topics/persistence)  
- [persistence（中文）](http://redisdoc.com/topic/persistence.html)