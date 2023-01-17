## Java内存模型



**Java内存模型**



**研究底层原理的重要性**

+ 从Java代码到CPU指令
  + JVM实现会带来不同的“翻译”，不同的CPU平台的机器指令又千差万别，无法保证并发安全的效果一致
  + 最开始，我们编写的Java代码，是*.java文件*
  + *在编译（javac命令）后，从刚才的*.java文件会变出一个新的Java字节码文件（*.class）*
  + JVM会执行刚才生成的字节码文件（*.class），并把字节码文件转化为机器指令
  + 机器指令可以直接在CPU上执运行，也就是最终的程序执行
+ 重点向下钻研
  + 转化过程的规范、原则



**JVM内存结构、Java内存模型、Java对象模型**

- JVM内存结构，和Java虚拟机的运行时区域有关。

- Java内存模型，和Java的并发编程有关。

- Java对象模型，和Java对象在虚拟机中的表现形式有关。

  Java对象自身的存储模型。JVM会给这个类创建一个instanceKlass ，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中 ，使用new创建一个对象的时候，JVM会创建一个instanceOopDesc对象，这个对象中包含了对象头以及实例数据。




### JMM是什么？重排序、可见性、原子性简介绍

**为什么需要JMM**

- C语言不存在内存模型的概念
- 依赖处理器，不同处理器结果不一样
- 无法保证并发安全
- 需要一个标准，让多线程运行的结果可预期

**是规范**

- Java Memory Model
- 是一组**规范**，需要各个JVM的实现来遵守JMM规范，以便于开发者可以**利用这些规范，更方便地开发多线程程序**。
- 如果没有这样的一个JMM内存模型来规范，那么很可能经过了不同JVM的不同规则的重排序之后，**导致不同的虚拟机上运行的结果不一样**，那是很大的问题。

**是工具类和关键字的原理**

- volatile、synchronized、Lock等的原理都是JMM
- 如果没有JMM，那就需要我们自己指定什么时候用内存栅栏等，那是相当麻烦的，幸好有了JMM，让我们只需要用同步工具和关键字就可以开发并发程序。

**最重要的3点内容**

- 重排序
- 内存可见性
- 原子性



### 重排序

**真正发生重排序**

+ 代码

  ```java
  /**
   * 描述：     演示重排序的现象 “直到达到某个条件才停止”，测试小概率事件
   */
  public class OutOfOrderExecution {
  
      private static int x = 0, y = 0;
      private static int a = 0, b = 0;
  
      public static void main(String[] args) throws InterruptedException {
          int i = 0;
          for (; ; ) {
              i++;
              x = 0;
              y = 0;
              a = 0;
              b = 0;
  
              CountDownLatch latch = new CountDownLatch(3);
  
              Thread one = new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          latch.countDown();
                          latch.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      a = 1;
                      x = b;
                  }
              });
              Thread two = new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          latch.countDown();
                          latch.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      b = 1;
                      y = a;
                  }
              });
              two.start();
              one.start();
              latch.countDown();
              one.join();
              two.join();
  
              String result = "第" + i + "次（" + x + "," + y + ")";
              if (x == 0 && y == 0) {
                  System.out.println(result);
                  break;
              } else {
                  System.out.println(result);
              }
          }
      }
  
  
  }
  
  ```

+ 出现x=0&&y=0，就是真正发生了重排序的情况，而不仅仅是线程交替执行。

  什么是重排序:在线程1内部的两行代码的实际执行顺序和代码在Java文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，它们的顺序被改变了， 这就是重排序，这里被颠倒的是y=a和b= 1这两行语句。


**重排序的好处和3种发生实际、重排序总结**

- 编译器优化

  JVM、JIT等等

- 指令重排序

  - CPU 的优化行为，和编译器优化很类似，是通过乱序执行的技术，来提高执行效率。所以就算编译器不发生重排，CPU 也可能对指令进行重排，所以我们开发中，一定要考虑到重排序带来的后果。

- 内存的“重排序”

  - 内存系统内不存在重排序，但是内存会带来看上去和重排序一样的效果，所以这里的“重排序”打了双引号。由于内存有缓存的存在，在JMM里表现为**主存和本地内存**，由于主存和本地内存的不一致，会使得程序表现出乱序的行为。在前面的例子中，假设没编译器重排和指令重排，但是如果发生了内存缓存不一致，也可能导致同样的情况：线程1 修改了 a 的值，但是修改后并没有写回主存，所以线程2是看不到刚才线程1对a的修改的，所以线程2看到a还是等于0。同理，线程2对b的赋值操作也可能由于没及时写回主存，导致线程1看不到刚才线程2的修改。

    

### 可见性

**可见性问题演示**

+ 代码

  ```java
  /**
   * 描述：     演示可见性带来的问题
   */
  public class FieldVisibility {
  
      volatile int a = 1;
      volatile int b = 2;
  
      private void change() {
          a = 3;
          b = a;
      }
  
  
      private void print() {
          System.out.println("b=" + b + ";a=" + a);
      }
  
      public static void main(String[] args) {
          while (true) {
              FieldVisibility test = new FieldVisibility();
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          Thread.sleep(1);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      test.change();
                  }
              }).start();
  
              new Thread(new Runnable() {
                  @Override
                  public void run() {
                      try {
                          Thread.sleep(1);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                      test.print();
                  }
              }).start();
          }
  
      }
  
  
  }
  
  ```

+ 不加volatile会出现打印a=3，b=2的情况

+ 使用volatile保证可见性

**为什么会有可见性问题**

- CPU缓存结构图

- CPU有多级缓存，导致读的数据过期

  - 高速缓存的容量比主内存小，但是速度仅次于寄存器，所以在CPU和主内存之间就多了Cache层
  - 线程间的对于共享变量的可见性问题不是直接由多核引起的，而是由多缓存引起的。
  - 如果所有个核心都只用一个缓存，那么也就不存在内存可见性问题了。
  - 每个核心都会将自己需要的数据读到独占缓存中，数据修改后也是写入到缓存中，然后等待刷入到主存中。所以会导致有些核心读取的值是一个过期的值。

**JMM的抽象：主内存和工作内存**

- 什么是主内存和本地内存

  - Java 作为高级语言，屏蔽了CPU cache等底层细节，用 JMM 定义了一套**读写内存数据的规范**，虽然我们不再需要关心一级缓存和二级缓存的问题，但是，JMM 抽象了主内存和本地内存的概念。
  - 这里说的本地内存并不是真的给每个线程分配了一块内存，而是JMM的一个抽象，是对于寄存器、一级缓存、二级缓存等的一个抽象。

- 主内存和本地内存的关系，JMM有以下规定：

  1. 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存中的拷贝。
  2. 线程不能直接读写主内存中的变量,而是只能操作自己工作内存中的变量，然后再同步到主内存中。
  3. 主内存是多个线程共享的，但线程间不共享工作内存,如果线程间需要通信，必须借助主内存中转来完成。

  所有的共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是通过本地内存交换的，所以才导致了可见性问题。



### happens-before原则

**happens-before原则**

- 什么是happens-before

  解决可见性问题的：在时间上，动作A发生在动作B之前，B保证能看见A，这就是happens-before。

- 什么不是happens- before

- 影响JVM重排序

  如果两个操作不具备happens-before，那么JVM是可以根据需要自由排序的，但是如果具备happens-before（比如新建线程时，run方法里面的语句一定发生在thread.start()之后），那么JVM也不能改变它们之间的顺序。

**Happens-Before规则有哪些？**

  - 单线程规则

  - **锁操作（synchronized和Lock）**

  - **volatile变量**

  - 线程启动

  - 线程join

  - 传递性

    如果hb(A, C) 而且hb(B, C)，那么可以推出hb(A, C)

  - 中断

    一个线程被其他线程interrupt时，那么检测中断（isInterrupted）或者抛出InterruptedException一定能看到。

  - 构造方法

    对象构造方法的最后一行指令 happens-before 于 finalize() 方法的第一行指令。

  - 工具类的Happens-Before原则

    1. 线程安全的容器get一定能看到在此之前的put等存入动作
    2. CountDownLatch
    3. Semaphore
    4. Future
    5. 线程池
    6. CyclicBarrier

**利用happens-before解决问题，代码案例：happens-before演示**

- 近朱者赤：只给b加了volatile，不仅b被影响,也可以实现轻量级同步。
- b之前的写入(对应代码b= a )对读取b后的代码( print b )都可见，所以在writerThread里对a的赋值，一 定会对readerThread里的读取可见，所以这里的a即使不加volatile，只要b读到是3 ,就可以由happens-before原则保证了读取到的都是3而不可能读取到1。



### volatile关键字

**volatile是什么**

- volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用volatile并不会发生上下文切换等开销很大的行为。
- 如果一个变量别修饰成volatile，那JVM就知道这个变量可能会被并发修改。
- 但是开销小，相应的能力也小，虽然说volatile是用来同步的保证线程安全的，但是volatile**做不到synchronized那样的原子保护**，volatile仅在很有限的场景下才能发挥作用。

**volatile的适用场合**

- 不适用组合操作：a++

- 适用场合1：boolean flag

  如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为赋值自身是有原子性的，而volatile又保证了可见性，所以就足以保证线程安全

- 适用场合2：作为刷新之前变量的触发器

  - 用了volatile int x后，可以保证读取x后，之前的所有变量都可见。
  - 一个实际生产中的例子

- volatile的作用：两点

  - 第一层：可见性

    读一个 volatile 变量之前，需要先使相应的本地缓存失效，这样就必须到主内存读取最新值，写一个 volatile 属性会立即刷入到主内存。

  - 第二层：禁止指令重排序优化

    解决单例双重锁乱序问题

**volatile和synchronized的关系？**

  - volatile在这方面可以看做是轻量版的synchronized：如果一个共享变量自始至终只被各个线程赋值，而没有其他的操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为**赋值自身是有原子性的**，而volatile又保证了可见性，所以就足以保证线程安全。

**学以致用：用volatile修正重排序问题**

OutOfOrderExecution类加了volatile后，用于不会出现(0, 0)的情况了。

**volatile 小结**

1. volatile 修饰符适用于以下场景：某个属性被多个线程共享，其中有一个线程修改了此属性，其他线程可以立即得到修改后的值，比如boolean flag。

2. volatile 属性的读写操作都是无锁的，它不能替代 synchronized，因为它**没有提供原子性和互斥性**。因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的。

3. volatile 只能作用于属性，我们用 volatile 修饰属性，这样 compilers 就不会对这个属性做指令重排序。

4. volatile 提供了可见性，任何一个线程对其的修改将立马对其他线程可见。volatile 属性不会被线程缓存，始终从主存中读取。

5. volatile 提供了 happens-before 保证，**对 volatile 变量 v 的写入 happens-before 所有其他线程后续对 v 的读操作**。

6. volatile 可以**使得 long 和 double 的赋值是原子的**。

   

### 保证可见性的措施

除了volatile可以让变量保证可见性外，synchronized、Lock、并发集合、Thread.join()和Thread.start()等都可以保证一定的可见性，具体看happens-before原则的规定。



### 对synchronized可见性的理解

- synchronized也可以达到同样的**happens-before**效果
- 这里关于synchronized有一个特别值得说的点，我们之前可能一致认为，使用了synchronized之后，**synchronized会帮我们设立临界区，这样在一个线程操作数据的时候，另一个线程无法进来同时操作，所以保证了线程安全**。其实这是不全面的，这种说法没有考虑到可见性问题。真正完整的说法是：synchronized不仅防止了一个线程在操作某对象时收到其他线程的干扰，同时**还保证了修改好之后，可以立即被其他线程所看到**。（因为如果其他线程看不到，那也会有线程安全问题）