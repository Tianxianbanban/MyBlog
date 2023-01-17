## CAS



**什么是CAS**

它是用在并发场合用来实现线程安全的一种算法，进行不可分割的**原子操作**。基本思想是：我认为value的值应该是A，如果是的话那我就把它改成B，如果不是A就说明已经被别人修改过了，那么我就不修改了，这就避免了并发情况下多人修改导致出错。

CAS有三个操作数：内存值V、预期值A、要求改的值B。**当且仅当预期值A和内存值V相同时，才将内存值修改为B**，否则什么都不做。最后返回现在的V值。

CPU的特殊指令：CAS实际上是要**利用CPU的特殊指令**，这些指令由CPU保证了他们的原子性，一个指令就可以做好几件事情，也不会出现线程安全问题。



**案例**

理解CAS的等价代码：

```java
public class SimulatedCAS {
    private volatile int value;

    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue) {
            value = newValue;
        }
        return oldValue;
    }
}
```





**应用场景**

CAS在juc包中的应用是很多的，既能保证安全性，又能提高性能，不需要去获取互斥同步锁。CAS的第一个应用就是**乐观锁**，还有**并发容器**，以及**原子类**。



**以AtomicInteger为例，分析在Java中是如何利用CAS实现原子操作的**

关于Java中是如何利用CAS实现原子操作：

+ AtomicInteger加载**Unsafe**工具，用来**直接操作内存**数据。

+ 所以实际上是通过Unsafe来实现底层操作。

  关于Unsafe类：

  Unsafe是CAS的核心类。Java无法直接访问底层操作系统，而是通过本地native方法来访问。不过尽管如此，JVM还是开启了一个后门，JDK中的一个类Unsafe，提供了**硬件级别的原子操作**。

  Unsafe代码中的objectFieldOffset方法获得的VALUE表示的是变量值在内存中的偏移地址，因为**Unsafe就是根据内存偏移地址获取数据的原值**的，这样我们就能通过Unsafe来实现CAS了。

+ 并且还需要volatile修饰value字段，保证**可见性**。

+ getAndAddInt方法分析

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
    private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
    //这个静态变量会在最初被加载，
    //VALUE则是用Unsafe的objectFieldOffset取得的，拿到的是AtomicInteger这个类中value字段的地址。
    //而value正是用volatile修饰的
    private volatile int value;
    
    //省略。。。
    
    //分析getAndAdd方法
    public final int getAndAdd(int delta) {
        //调用的是Unsafe的getAndAddInt方法
        return U.getAndAddInt(this, VALUE, delta);
    }
}


class Unsafe{
    //省略。。。
    
    @HotSpotIntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        //可以看到上面是一个dowhile循环
        //而循环条件就调用了weakCompareAndSetInt，正是CAS的体现
        return v;
    }
    
    
    @HotSpotIntrinsicCandidate
    public final boolean weakCompareAndSetInt(Object o, long offset,
                                              int expected,
                                              int x) {
        return compareAndSetInt(o, offset, expected, x);
    }
    
    //到这里已经是一个native方法了
    //关于这个本地方法，它的C++代码的思路是，利用偏移量拿到原值地址，然后进行相应的CAS操作
    @HotSpotIntrinsicCandidate
    public final native boolean compareAndSetInt(Object o, long offset,
                                                 int expected,
                                                 int x);
}
```

+ 总结

  Unsafe方法中的compareAndSetInt方法想办法拿到变量value在内存中的地址。然后通过C++代码实现**原子性比较和替换**。



**缺点**

+ ABA问题

  可以添加版本号解决。

+ 自旋时间过长