`java.util.concurrent.atomic`包下的类我们叫原子操作类。类似`i++`这种操作，本身就不是原子操作，很容易出现多线程的问题。当然可以借助悲观锁（如 synchronized）来实现，但是悲观锁并不是特别高效的一种解决方案。

atomic 包提供了一系列的操作简单，性能高效，并能保证线程安全的类去更新基本类型变量，数组元素，引用类型以及更新对象中的字段类型。atomic 包下的这些类都是采用的是乐观锁策略去原子更新数据，在 java 中则是使用 CAS 操作具体实现。

## atomic 包介绍

在 atomic 包里，一个有 17 个类(jdk1.8),其中有 16 个原子操作类，其中分为四种原子更新方式，分别为原子更新基本类型(boolean、int、long、double)、原子更新数组、原子更新引用和原子更新字段。atomic 包基本都是借助 Unsafe 类和 Varhandle (jdk9 后) 类的 CAS 来实现。

### 原子更新基本类型

用于通过原子的方式更新基本类型，包括以下几个类：

* AtomicBoolean

原子更新 Boolean 类型

* AtomicInteger

原子更新 Integer

* AtomicLong

原子更新 Long

* DoubleAccumulator

和 AtomicLong 类似，对 AtomicLong 的改进

* DoubleAdder

double 类型的累加器，DoubleAccumulator 的特例，只能用来计算加法，且从 0 开始计算。

* LongAccumulator

double 类型的聚合器，需要传入一个 double 类型的二元操作，可以用来计算各种聚合操作，包括加乘等。

* LongAdder

long 类型的累加器，对 AtomicLong 的改进，LongAccumulator 的特例，只能用来计算加法，且从 0 开始计算。

### 原子更新数组

* AtomicIntegerArray

原子更新整型数组里的元素

* AtomicLongArray

原子更新长整型数组里的元素

* AtomicReferenceArray

原子更新引用类型数组里的元素

### 原子更新引用

* AtomicReference

原子更新引用类型

* AtomicReferenceFieldUpdater

原子更新引用类型里的字段

* AtomicMarkableReference

原子更新带有标记位的引用类型

### 原子更新字段

* AtomicIntegerFieldUpdater

原子更新整型的字段的更新器

* AtomicLongFieldUpdater

原子更新长整型字段的更新器

* AtomicStampedReference

原子更新带有版本号的引用类型

## 总结

CAS 虽然带来了很好的性能，但是也会存在一些问题，例如 ABA 等。atomic 包下的类甚至可以说整个 JUC 都是基于 CAS 原理来实现的，atomic 包又能分为 原子更新基本类型、原子更新数组、原子更新引用和原子更新字段四大操作类。

