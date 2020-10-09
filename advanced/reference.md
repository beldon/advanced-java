
## 强引用(StrongReference)

## 软引用(SoftReference)

## 弱引用(WeakReference)

ThreadLocal、ThreadLocal 内存泄露
ThreadLocalMap.Entry 是个弱引用， ThreadLocal作为若引用
ThreadLocal 内存泄露：https://www.jianshu.com/p/c07b1192611c

## 虚引用(PhantomReference)

查看 Cleaner 类代码
用于监控、对象被回收后处理些内容等
引用队列(ReferenceQueue)
灵活使用强引用与其他引用的组合，若强应用与其他引用一起时，当强引用去掉后，就会变成其他引用