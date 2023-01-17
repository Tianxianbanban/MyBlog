## Java双重检验的单列模式

```JAVA
/**
 * 双重检查的单例模式
 */
public class SingletonDoubleCheck {
    /**
     * volatile：保证线程线程可见性，与禁止指令重排序
     * 多线程环境下使用单例模式，如果一个线程已经创建了对象，另外线程不可见的话就会重复创建对象；
     * 另外，对象的创建并不是原子性的，分为创建对象、调用构造方法、以及引用赋值三步，如果不禁止指令重	   * 排序，
     * 会出现的问题就是多线程环境下，一个线程创建了对象以后还没有来得及通过构造方法初始化，
     * 而其他线程就开始使用这个没有赋初值的对象，就会在使用过程中遇到空指针等问题。
     */
    private volatile static SingletonDoubleCheck instance;

    private SingletonDoubleCheck() {

    }

    public static SingletonDoubleCheck getInstance() {
        /**
         * 双重检查机制：
         * 第一层检查是检查单例对象是否创建；
         * 但是在多线程环境下，如果不同线程同时执行到这里，都检测到instance为空，就会一齐往下执行，			 * 重复创建对象，
         * 所以需要通过synchronized解决这个问题，这样第二个线程再进入到这个synchronized的时候，
         * 由于java内存模型的hapens_before原则，后进入的线程会看到之前线程的执行结果，
         * 就能够通过第二次判断，确定instance是否创建对象，防止对象重复创建。
         * 
         * 另外，这种方式比直接给getInstance方法设置synchronized效率高很多。
         */
        if (instance == null) {
            synchronized (SingletonDoubleCheck.class) {
                if (instance == null) {
                    instance = new SingletonDoubleCheck();
                }
            }
        }
        return instance;
    }
}

```

