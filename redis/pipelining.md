

> [!NOTE]
> RTT（Round Trip Time）,往返时间    
> redis执行一条命令有四个过程：发送命令、命令排队、命令执行、返回结果；整个过程是一个往返时间（RTT）

Pipeline(流水线) 为了解决批量处理而消耗的大量网络传输时间（因为redis执行是微妙级，大量操作更多消耗的是网络时间）



| **命令**             | **时间**          | **数据量** | **特性** |
| -------------------- | ----------------- | ---------- | -------- |
| n个命令              | n次网络 + n次执行 | 1条命令    | 原子性   |
| 1次pipeline(n个命令) | 1次网络 + n次执行 | n条命令    | 非原子性 |





## 原生批量命令(mset, mget)与Pipeline对比

可以使用Pipeline模拟出批量操作的效果，但是在使用时要注意它与原生批量命令的区别，主要包括如下几点：

- 原生批量命令是原子的，Pipeline是非原子的；
- 原生批量命令是一个命令对应多个key，Pipeline支持多个命令；
- 原生批量命令是Redis服务端支持实现的，而Pipeline需要服务端和客户端的共同配合。

## 用法



## 参考链接

- [pipelining](https://redis.io/topics/pipelining)
- [Redis Pipeline使用](https://www.cnblogs.com/-wenli/p/12922089.html)

