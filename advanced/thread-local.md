> [!TIP]
> - 线程之间的threadLocal变量是互不影响的
> - 使用private final static进行修饰，防止多实例时内存的泄露问题
> - 线程池环境下使用后将threadLocal变量remove掉或设置成一个初始值

## 描述

ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。通常使用静态的变量来维护 **ThreadLocal** 如:

```java
static ThreadLocal<String> userIdThreadLocal = new ThreadLocal<String>
```

通常用来保存 userId、TransactiuonId 等。spring 也很经常用 ThreadLocal 来保存一些内容。如用

```java
org.springframework.web.context.request.RequestContextHolder
```
来保存一些请求信息，方便整个线程调用。

ThreadLocal类提供的几个方法：

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```

`get()` 方法是用来获取ThreadLocal在当前线程中保存的变量副本，`set()` 用来设置当前线程中变量的副本，`remove()` 用来移除当前线程中变量的副本，`initialValue()` 是一个 protected 方法，一般是用来在使用时进行重写的，它是一个延迟加载方法，只有调用 `get()` 方法的时候才有可能调用

- demo

```java
import java.util.concurrent.TimeUnit;

public class ThreadLocalDemo {
    static ThreadLocal<String> local = new ThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        local.set("main");

        new Thread(()->{
            System.out.println("new thread:"+local.get());

        }).start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("main thread:"+local.get());
    }
}
```

- 输出结果

```shell
main thread:main
new thread:null
```

## 实现原理

- 在Thread中维护着 `ThreadLocal.ThreadLocalMap threadLocals` 成员变量
- 在 `ThreadLocal.ThreadLocalMap` 中以 `ThreadLocal.threadLocalHashCode` 来为基础算出 key 作为缓存相应的 value ，以区分不同 ThreadLocal 的变量。

> [!TIP]
> ThreadLocal 只是维护着变量的引用，引用的 key 是弱引用，弱引用只存在于key上，所以key会被回收，而 value 还存在着强引用，只有 thread 退出以后，value 的强引用链条才会断掉。

> [!NOTE]
> - ThreadLocal 需要注意的问题，每次执行完毕后，要使用 remove() 方法来清空对象，否则 ThreadLocal 存放大对象后，会出现OMM。
> - ThreadLocal 要使用 static 的，在其他地方可以直接用 get 和 set 方法。

## ThreadLocal内存泄漏

ThreadLocalMap 使用 ThreadLocal 的弱引用作为 key ，如果一个 ThreadLocal 没有外部强引用来引用它，那么系统 GC 的时候，这个 ThreadLocal 势必会被回收，这样一来，ThreadLocalMap 中就会出现 key 为 null 的 Entry，
就没有办法访问这些 key 为 null 的 Entry 的 value，如果当前线程再迟迟不结束的话，这些 key 为 null 的 Entry 的 value 就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value` 永远无法回收，造成内存泄漏。

其实，ThreadLocalMap 的设计中已经考虑到这种情况，也加上了一些防护措施：在 ThreadLocal 的 get()、set()、remove() 的时候都会清除线程 ThreadLocalMap 里所有 key 为 null 的 value。

但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用 static 的 ThreadLocal，延长了 ThreadLocal 的生命周期，可能导致的内存泄漏（参考 ThreadLocal 内存泄露的实例分析）。
- 分配使用了 ThreadLocal 又不再调用 get()、set()、remove() 方法，那么就会导致内存泄漏。

**为什么使用弱引用？**

- key 使用强引用：引用的 ThreadLocal 的对象被回收了，但是 ThreadLocalMap 还持有 ThreadLocal 的强引用，如果没有手动删除，ThreadLocal 不会被回收，导致 Entry 内存泄漏。
- key 使用弱引用：引用的 ThreadLocal 的对象被回收了，由于 ThreadLocalMap 持有 ThreadLocal 的弱引用，即使没有手动删除，ThreadLocal 也会被回收。value 在下一次 ThreadLocalMap 调用 set、get、remove的时候会被清除。


比较两种情况，我们可以发现：由于 ThreadLocalMap 的生命周期跟 Thread 一样长，如果都没有手动删除对应 key，都会导致内存泄漏，
但是使用弱引用可以多一层保障：弱引用 ThreadLocal 不会内存泄漏，对应的 value 在下一次 ThreadLocalMap 调用 set、get、remove 的时候会被清除。

因此，ThreadLocal 内存泄漏的根源是：由于 ThreadLocalMap 的生命周期跟 Thread 一样长，如果没有手动删除对应 key 就会导致内存泄漏，而不是因为弱引用。
综合上面的分析，我们可以理解 ThreadLocal 内存泄漏的前因后果，那么怎么避免内存泄漏呢？

- 每次使用完 ThreadLocal，都调用它的 remove() 方法，清除数据。

在使用线程池的情况下，没有及时清理 ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用 ThreadLocal 就跟加锁完要解锁一样，用完就清理。

## InheritableThreadLocal

与 ThreadLocal 不同的是 `InheritableThreadLocal` 是可以被基础的，`InheritableThreadLocal` 在 Thread 中同样是维护着一个成员变量 `inheritableThreadLocals`。

- demo如下：


```java
import java.util.concurrent.TimeUnit;

public class ThreadLocalDemo {
    static ThreadLocal<String> local = new InheritableThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {
        local.set("main");

        new Thread(()->{
            System.out.println("new thread:"+local.get());

        }).start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("main thread:"+local.get());
    }
}
```

- 结果


```shell
new thread:main
main thread:main
```

**原理**

我们在new一个线程的时候

```java
public Thread() {
    init(null, null, "Thread-" + nextThreadNum(), 0);
}
```

然后

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
    init(g, target, name, stackSize, null);
}
```

接下来

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
     ......
    if (parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;
    ......
    }
}
```

这时候有一句 `ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);` ，然后

```java
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
    return new ThreadLocalMap(parentMap);
}
```

```java
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                    table[h] = c;
                    size++;
                }
            }
        }
    }
```

当我们创建一个新的线程X的时候，X线程就会有 `ThreadLocalMap` 类型的 `inheritableThreadLocals`，因为它是 Thread 类的一个属性。这样就先得到当前线程存储的这些值，例如 `Entry[] parentTable = parentMap.table`; 。再通过一个 for 循环，不断的把当前线程的这些值复制到我们新创建的线程 X 的 `inheritableThreadLocals` 中。
