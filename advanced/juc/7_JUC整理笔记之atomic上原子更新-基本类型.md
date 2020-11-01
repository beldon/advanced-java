atomic 包下的原子更新基本类型的包括 AtomicBoolean、AtomicInteger、AtomicLong、DoubleAccumulator、DoubleAdder、LongAccumulator、LongAdder，分别是对 Boolean、Integer、Long、Double 的原子操作，例如 get、set、getAndSet、cas、increment、decrement 等操作。

## 分析说明

### 相同方法

* get & set

普通的 volatile (如 AtomicBoolean 的 volatile value) 值的 get 和 set 操作。

* lazySet
```java
//AtomicBoolean 
unsafe.putOrderedInt(this, valueOffset, v); //jdk1.8 及之前
VALUE.setRelease(this, (newValue ? 1 : 0)); //jdk9 及之后
```
该操作在 1.8 前是使用 Unsafe，jdk 9 后是用了 Unsafe 的 setRelease，其原理是一样的，都是保证操作前的 load 和 stroe 不会被重排序，但不保证操作后的 load 和 store 的重排序。
* getAndSet

获取 set 之前的值，并修改值。

```java
//jdk1.8
boolean prev;
do {
    prev = get();
} while (!compareAndSet(prev, newValue));
return prev;
//jdk9 及后
return (int)VALUE.getAndSet(this, (newValue ? 1 : 0)) != 0;
```
在 jdk1.8 的时候，getAndSet 是一个循环，不断 compareAndSet， 直到成功为止，这就是鼎鼎有名的自旋，JUC 下很多功能都是这样自旋的；在 jdk9 之后，则直接用 VarHandle 来操作了，也就是自旋的操作交给了 jdk 管了，不再用写在 java 代码里面。
* compareAndSet & weakCompareAndSet

这两者在 jdk1.8 里面的代码是一样的（后续的 jdk 版本会对其进行区分），都是对比并 set 值，但是 weakCompareAndSet 的描述里面，多了个**“Possibly”**，也就是说，尽管对比的值是一样也可能会修改失败。

在某些平台下，weakCompareAndSet 会更加高效，weakCompareAndSet 底层不会创建任何 happen-before 的保证，也就是不会对 volatile 字段操作的前后加入内存屏障。因此就无法保证多线程操作下对除了 weakCompareAndSet 操作的目标变量( 该目标变量一定是一个 volatile 变量 )之其他的变量读取和写入数据的正确性。

参考资料：[对 volatile、compareAndSet、weakCompareAndSet 的一些思考](https://www.jianshu.com/p/55a66113bc54)

在 jdk9 之后，weakCompareAndSet 标记为过期了，建议使用 weakCompareAndSetPlain 来替代，从 weakCompareAndSetPlain  这个名字就可以很容易理解了，compare 的时候还是用 volatile (value 是被 volatile 修饰)，set 的时候就是 plain 了。

### jdk9 后新增的方法

jdk9 后新增的方法有 getPlain、setPlain、getOpaque、setOpaque、getAcquire、setRelease、compareAndExchange、compareAndExchangeAcquire、compareAndExchangeRelease、weakCompareAndSetVolatile、weakCompareAndSetAcquire、weakCompareAndSetRelease，其使用说明和 Varhandle 相对应的方法是一致的。

### AtomicBoolean

AtomicBoolean 的实现并不是 boolean 值， 而是一个 int 的值，并且其值用 volatile 修饰，其中 value 为 1 时是 true，为 0 时是 false。

```java
private volatile int value;
```
在 jdk1.8 及以前，AtomicBoolean 是使用 Unsafe 类来操作的，在 jdk1.9 之后则使用了 VarHandle。在 AtomicBoolean 中，两者的使用原理是一样的，只是 jdk1.9 后更推荐使用 Varhandle，关于两者的区别，可以查看之前的文章。
### AtomicInteger

```java
private volatile int value;
```
* getAndIncrement、getAndDecrement、incrementAndGet、decrementAndGet

这几个方法都和 getAndAddInt 一样 ，只不过只是加/减 1，然后获取更新前/后的值。

* getAndUpdate、updateAndGet、getAndAccumulate、accumulateAndGet

getAndUpdate、updateAndGet 与 getAndAccumulate、accumulateAndGet 很相似，前两者结接收 IntUnaryOperator 参数，获取要更新的值，后两者接收 一个 int 和 IntBinaryOperator 参数，然会通过把 x 和当前值传给 IntBinaryOperator 以获取更新的值。这几个方法都是通过自旋方式，调用 weakCompareAndSetVolatile 更新。

### AtomicLong

```java
private volatile long value;
```
AtomicLong 的所拥有的方法和用法与 AtomicInteger 是一样的。
### LongAdder

LongAdder  的功能在 AtomicLong 中也存在，但是为什么还存在 LongAdder呢。

根据官方文档的介绍，LongAdder 在高并发的场景下会比 AtomicLong 具有更好的性能，代价是消耗更多的内存空间。

AtomicLong 里的方法是通过 CAS 方式，然后自旋更新值，当并发高的时候，自旋次数会增大，反而影响到性能。

LongAdder设计思路就是分散热点，将value值分散到一个数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样冲突的概率就小很多。如果要获取真正的long值，只要将各个槽中的变量值累加返回。

![图片](https://uploader.shimo.im/f/wRqeAzZusnleOTxN.png!thumbnail)

### LongAccumulator

LongAccumulator 可以说是 LongAdder 的加强版， LongAdder 只是对数值的加减，但是 LongAccumulator 通过 LongBinaryOperator 对更新前的值与传入的值作其他操作，例如乘除等。

### DoubleAdder、DoubleAccumulator

这两个类同 LongAdder、LongAccumulator

### 

