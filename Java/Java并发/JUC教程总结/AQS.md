## AQS



#### **关于AQS**

AQS在并发包当中的应用非常多，设计思路很巧妙，是非常值得探 究和总结的一个知识点。



#### **为什么需要AQS**

+ **锁和协作类**的共同点：闸门。比如ReentrantLock和Semaphore就有很多的相似点，例如lock&acquire、tryLock&tryAcquire，支持中断与不支持中断的方法等等。不仅是ReentrantLock和Semaphore，包括CountDownLatch、ReentrantReadWriteLock都有这样的**类似的协作**（或者说是同步）功能，其实他们底层都是用了**一个共同的基类，这就是AQS**。
+ 因为那些协作类有**很多工作都是类似的**（如何安排线程、如何等待、陷入阻塞等），所以如果能够**提取出一个工具类**，那么就可以直接用，对于ReentrantLock和Semaphore而言就可以屏蔽很多细节，只关注它们自己的“业务逻辑”就可以了。

+ 和AQS的关系

  Semaphore内部有一个Sync类，Sync类继承了AQS；

  CountDownLatch也是一样的；等等，很多，只要利用了AQS，都是类似形式。




#### **AQS的作用**

+ 比喻：面试过程中无论群面还是单面，安排就坐、叫号、先来后到等HR的工作就是AQS做的工作，面试官不会关心两个面试者是不是号码相互冲突了，也不会去管面试者是否需要一个地方坐着休息，这些都是交给HR去做了。

  Semaphore：单面，一个人面完了，后一个人才能进来继续面试。CountDownLatch：群面，等待10人到齐。Semaphore、CountDownLatch等同步工具类，要做的就是写下自己“要人”的规则，比如是“出一个，进一个”，或者说是“凑齐10人，一起面试”。剩下的招呼面试者的杂活交给AQS来做。

+ 如果没有AQS

  + 就需要每个协作工具自己实现：同步状态的原子性管理、线程的阻塞与解除阻塞、队列的管理。
  + 在并发场景下，自己正确且高效地实现这些内容，都是相当有难度的，所以我们使用AQS来帮助搞定这些杂活，而使用的时候只用关注业务逻辑就行了。

+ 总结：

  AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。有了AQS以后，更多的协作工具，都可以很方便地被写出来。因为AQS解决了大量的细节问题，比如等待线程用先进先出的队列操作，以及一些标准来判断这些线程是等待还是不应等待，以及处理一些竞争问题，解决开销提高吞吐量等等。AQS的设计充分考虑了这些使用场景以及性能问题，所以使用AQS的并发工具类也同时拥有了这些优势。

  **有了AQS，构建线程协作类就容易多了。**



#### **AQS的重要性以及地位**

AbstractQueuedSynchronizer是Doug Lea写的，从JDK1.5加入的一个基于FIFO等待队列实现的一个用于实现同步器的基础框架。



#### **AQS内部原理解析**

+ AQS最核心的三个部分：

  + **state**

    state的具体含义，会根据具体实现类的不同而不同，比如在Semaphore里，它表示“剩余的许可证数量”，而在CountDownLatch里，它表示“还需要倒数的数量”。

    在ReentrantLock中，state表示“锁”的占有情况，包括可重入计数。state值为0时标识不被任何线程占有，如果是1则被线程持有了，它还会变成2、3、4，因为是可重入的，同一个线程多次获取这把锁。

    state是**volatile**修饰的，会被并发修改，所以修改state的方法都需要**保证线程安全**，比如getState、setState以及compareAndSetState操作来读取和更新这个状态。这些方法都依赖于juc.atomic包的支持。

  + 控制线程抢锁和配合的**FIFO队列**

    这个队列用来**存放"等待的线程”**， AQS就是 "==排队管理器==”，当多个线程争用同一把锁时，必须有排队机制将那些**没能拿到锁**的线程串在一起。当锁释放时，锁管理器就会挑选个合适的线程来占有这个刚刚释放的锁。

    AQS会维护一个等待的线程队列，把**线程都放到这个队列里**，这是一个双向链表形式的队列。

  + 期望协作工具类去实现的**获取/释放**等重要方法

    这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且含义各不相同。

    获取方法：获取操作会依赖state变量（它会查看这个变量的情况，比如ReentrantLock查看这个值，只要不是0就是被其他线程持有了，则进入阻塞状态），经常会阻塞(比如获取不到锁的时候)。在Semaphore中 ，获取就是acquire方法，作用是获取一个许可证，如果state的值是正数可以成功-1，就能获取到了；而在CountDownLatch里面， 获取就是await方法,作用是“等待，直到倒数结束”，去获取的时候只要state不为0就进入阻塞，直到其他方法把state减到0以后才消除阻塞状态被唤醒。
  
    释放方法：是获取方法的对立面，而且释放操作不会阻塞。在Semaphore中,释放就是release方法,作用是释放一个许可证，让state+1。CountDownLatch里面，释放就是countDown方法，作用是“倒数1个数”，让state-1。
  
    

#### **应用实例、源码解析**

+ AQS用法

  1. 写一个类，想好**协作的逻辑**，实现==获取/释放==方法。比如CountDownLatch就是有个门栓然后去等待、Semaphore则是每个线程去获取许可证……所以具体逻辑是不一样的。
  2. 内部写一个Sync类继承AbstractQueuedSynchronizer。
  3. 根据**独占or共享**，来重写**（独占）tryAcquire/tryRelease或者（共享）tryAcquireShared (int acquires)和tryReleaseShared(intreleases)等方法**，并在之前写的==获取/释放==方法中**调用**AQS的acquire/release或者xxxShared方法。
  
+ AQS在CountDownLatch的应用

  + 构造函数
  + getCount
  + countDown
  + await

  ```java
  //java10
  public class CountDownLatch {
      
      //构造方法,创建Sync实例,传入的count最终由state来代替，是要倒数的数量
      public CountDownLatch(int count) {
          if (count < 0) throw new IllegalArgumentException("count < 0");
          this.sync = new Sync(count);
      }
      
      //间接调用sync.getCount()方法
      //最终得到的是state的值
      public long getCount() {
          return sync.getCount();
      }
      
      //调用AQS的releaseShared，
      //当中会去调用Sync的tryReleaseShared方法，CAS减少state的数值
      //一旦state为0，tryReleaseShared会返回true
      //根据这个条件的成立，如唤醒队列当中的线程。
      public void countDown() {
          sync.releaseShared(1);
      }
      
      //进行等待，直到倒数结束
      //调用AQS的acquireSharedInterruptibly方法，
      //进而调用Sync的tryAcquireShared方法，
      //只要state数值减少到0了已经，tryAcquireShared方法就会返回1，
      //如果state是其他值就返回-1
      //所以如果state不为0，就进入等待队列，进入阻塞状态
      //state为0，就正常获得锁，而不需要等待了
      public void await() throws InterruptedException {
          sync.acquireSharedInterruptibly(1);
      }
      
      /**
       * Synchronization control For CountDownLatch.
       * Uses AQS state to represent count.
       */
      private static final class Sync extends AbstractQueuedSynchronizer {
          private static final long serialVersionUID = 4982264981922014374L;
  
          Sync(int count) {
              //设置state值，state的值设置成了count
              setState(count);
          }
  
          int getCount() {
              return getState();
          }
          
          public void await() throws InterruptedException {
              sync.acquireSharedInterruptibly(1);
          }
          
          public void countDown() {
          	sync.releaseShared(1);
      	}
  
          //state为0返回1，其他值（倒数还没结束）都返回-1
          protected int tryAcquireShared(int acquires) {
              return (getState() == 0) ? 1 : -1;
          }
  
          //
          protected boolean tryReleaseShared(int releases) {
              // Decrement count; signal when transition to zero
              //这里用一个循环在做CAS的自旋，尝试减少count计数值，
              //当减少到0的时候返回true
              for (;;) {
                  int c = getState();
                  //c == 0 表示已经释放过了，不需要反复释放。
                  if (c == 0) 
                      return false;
                  int nextc = c - 1;
                  if (compareAndSetState(c, nextc))
                      return nextc == 0;
              }
          }
          
      }
      
      //省略。。。
  }
  
  
  public abstract class AbstractQueuedSynchronizer
      extends AbstractOwnableSynchronizer
      implements java.io.Serializable {
      
      protected final int getState() {
          return state;
      }
     
      
      public final void acquireSharedInterruptibly(int arg)
              throws InterruptedException {
          if (Thread.interrupted())
              throw new InterruptedException();
          //tryAcquireShared看sync中的实现
          if (tryAcquireShared(arg) < 0)
              //也就是只要state不为0，就让当前线程入等待队列，阻塞
              doAcquireSharedInterruptibly(arg);
          //如果state是0了已经，就是正常获得锁，不需要等待了。
      }
      
      private void doAcquireSharedInterruptibly(int arg)
          throws InterruptedException {
          //首先把当前线程包装成一个Node结点，放入等待队列
          final Node node = addWaiter(Node.SHARED);
          try {
              for (;;) {
                  final Node p = node.predecessor();
                  if (p == head) {
                      int r = tryAcquireShared(arg);
                      if (r >= 0) {
                          setHeadAndPropagate(node, r);
                          p.next = null; // help GC
                          return;
                      }
                  }
                  //阻塞是在这里做的
                  //parkAndCheckInterrupt
                  if (shouldParkAfterFailedAcquire(p, node) &&
                      parkAndCheckInterrupt())
                      throw new InterruptedException();
              }
          } catch (Throwable t) {
              cancelAcquire(node);
              throw t;
        }
      }
      
      private final boolean parkAndCheckInterrupt() {
          //最终会调用Usafe的park方法，是个native方法
          //就是把当前线程挂起
          LockSupport.park(this);
          return Thread.interrupted();
      }
      
      public final boolean releaseShared(int arg) {
          //tryReleaseShared看Sync中的实现
          if (tryReleaseShared(arg)) {
              //在tryReleaseShared返回true也就是count为0了的情况下把阻塞的线程全部唤醒，
              //也就是count从1减到0的时候闸门打开,把等待的线程唤醒！
              doReleaseShared();
              return true;
          }
          return false;
      }
      
      //省略。。。
  }
  ```

  + AQS在CountDownLatch的总结

    调用CountDownLatch的await方法时，便会尝试获取"共享锁”，不过**一开始可能是获取不到该锁的，于是线程被阻塞**。
    
    而“共享锁”可获取到的条件，就是**"锁计数器”**（state）的值为0。而"锁计数器”的初始值为count ,每当一个线程调用该CountDownLatch对象的**countDown()**方法时，才将"锁计数器”**-1**。 count个线程调用countDown()之后，**“锁计数器”才为0，而前面提到的等待获取共享锁的线程才能被唤醒然后继续运行。**

+ AQS在Semaphore的应用

  在Semaphore中，state表示许可证的剩余数量。

  Semaphore主要操作就是释放和获取许可证。

  看tryAcquire方法 ，判断nonfairTryAcquireShared >= 0的话，代表成功。这里会先检查剩余许可证数量够不够这次需要的 ，用减法来计算。如果直接不够，那就返回负数，表示失败，当前线程没有拿到许可证进入阻塞状态；如果够了，就用自旋加compareAndSetState来改变state状态,直到改变成功就返回正数；或者是期间如果被其他人修改了导致剩余数量不够了， 那也返回负数代表获取失败。

  ```java
  //java10
  public class Semaphore implements java.io.Serializable {
      
      //调用AQS的acquireSharedInterruptibly
      //进而调用非公平的NonfairSync或者公平的FairSync，当中的tryAcquireShared
      //根据当中的tryAcquireShared的返回结果决定是否放入等待队列
      public void acquire(int permits) throws InterruptedException {
          if (permits < 0) throw new IllegalArgumentException();
          sync.acquireSharedInterruptibly(permits);
      }
      
      //调用AQS的releaseShared
      //进而调用Sync的tryReleaseShared，去进行增加许可证数量的CAS操作
      public void release() {
          sync.releaseShared(1);
      }
      
      //Sync
      abstract static class Sync extends AbstractQueuedSynchronizer {
          private static final long serialVersionUID = 1192457210091910933L;
  
          Sync(int permits) {
              setState(permits);
          }
  
          final int getPermits() {
              return getState();
          }
  
          //Sync默认是非公平的
          //返回值时正数则获取成功
          final int nonfairTryAcquireShared(int acquires) {
              for (;;) {
                  int available = getState();
                  int remaining = available - acquires;
                  if (remaining < 0 ||
                      compareAndSetState(available, remaining))
                      return remaining;
              }
          }
  
          //给许可证数量进行增加
          protected final boolean tryReleaseShared(int releases) {
              for (;;) {
                  int current = getState();
                  int next = current + releases;
                  if (next < current) // overflow
                      throw new Error("Maximum permit count exceeded");
                  if (compareAndSetState(current, next))
                      return true;
              }
          }
  
          final void reducePermits(int reductions) {
              for (;;) {
                  int current = getState();
                  int next = current - reductions;
                  if (next > current) // underflow
                      throw new Error("Permit count underflow");
                  if (compareAndSetState(current, next))
                      return;
              }
          }
  
          final int drainPermits() {
              for (;;) {
                  int current = getState();
                  if (current == 0 || compareAndSetState(current, 0))
                      return current;
              }
          }
      }
      
      
      //非公平
      static final class NonfairSync extends Sync {
          private static final long serialVersionUID = -2694183684443567898L;
  
          NonfairSync(int permits) {
              super(permits);
          }
  
          protected int tryAcquireShared(int acquires) {
              return nonfairTryAcquireShared(acquires);
              //nonfairTryAcquireShared在Sync当中
          }
      }
      
      //公平
      static final class FairSync extends Sync {
          private static final long serialVersionUID = 2014338818796000944L;
  
          FairSync(int permits) {
              super(permits);
          }
  
          //公平的tryAcquireShared会看看队列当中是不是有比当前线程等待更久的线程
          //如果有就返回-1
          //没有的话才进行获取许可证，减少许可证数量的CAS操作
          protected int tryAcquireShared(int acquires) {
              for (;;) {
                  if (hasQueuedPredecessors())
                      return -1;
                  int available = getState();
                  int remaining = available - acquires;
                  if (remaining < 0 ||
                      compareAndSetState(available, remaining))
                      return remaining;
              }
          }
      }
      
      //省略。。。
  }
  
  
  public abstract class AbstractQueuedSynchronizer
      extends AbstractOwnableSynchronizer
      implements java.io.Serializable {
      
      public final void acquireSharedInterruptibly(int arg)
              throws InterruptedException {
          if (Thread.interrupted())
              throw new InterruptedException();
          //tryAcquireShared在Semaphore中根据公平不公平有两种实现
          if (tryAcquireShared(arg) < 0)
              //tryAcquireShared返回值小于0,说明是许可证不够用，
              //或者是公平版本Sync子类里面从队列中看，已经有线程早已等待
              //就放入等待队列
              doAcquireSharedInterruptibly(arg);
      }
      
      public final boolean releaseShared(int arg) {
          //许可证成功释放后，就可以从队列中
          if (tryReleaseShared(arg)) {
              doReleaseShared();
              return true;
          }
          return false;
      }
      
      //省略。。。
  }
  ```

+ AQS在ReentrantLock的应用

  ReentrantLock最重要的操作是lock & unLock。

  释放锁的方法：到tryRelease，由于是可重入的，所以**state代表重入的次数**，每次释放锁，先判断是不是当前持有锁的线程释放的，如果不是就抛异常；如果是的话，重入次数就减一。如果减到了0 ，就说明完全释放了，于是free就是true，并且把state设置为0（这把锁当前没有被任何线程持有）。

  加锁的方法：去判断当前state是不是等于0，也会去判断当前线程是不是持有锁的线程，如果都不是，代表目前拿不到这把锁，就放到队列中去，并在以后合适的时机唤醒。
  
  ```java
  //Java10
  public class ReentrantLock implements Lock, java.io.Serializable {
      
      
      abstract static class Sync extends AbstractQueuedSynchronizer {
          private static final long serialVersionUID = -5179523762034025860L;
          
          //释放锁
          //调用AQS的release方法
          //进而调用Sync的tryRelease
          public void unlock() {
          	sync.release(1);
      	}
          
          //加锁
          //调用AQS的acquire
          //进而调用公平或者非公平Sync子类的tryAcquire方法
          //根据tryAcquire返回结果决定，获得锁返回true
          public void lock() {
         	 	sync.acquire(1);
     	 	}
  
          /**
           * Performs non-fair tryLock.  tryAcquire is implemented in
           * subclasses, but both need nonfair try for trylock method.
           */
          //非公平获得锁
          @ReservedStackAccess
          final boolean nonfairTryAcquire(int acquires) {
              final Thread current = Thread.currentThread();
              int c = getState();
              //只要state为0，就CAS方式更换State值，尝试获得锁
              if (c == 0) {
                  if (compareAndSetState(0, acquires)) {
                      //这把锁被current线程持有
                      setExclusiveOwnerThread(current);
                      return true;
                  }
              }
              else if (current == getExclusiveOwnerThread()) {
                  //否则如果当前线程已经持有了锁，那么就只要更新state的值就可以了
                  int nextc = c + acquires;
                  if (nextc < 0) // overflow
                      throw new Error("Maximum lock count exceeded");
                  setState(nextc);
                  return true;
              }
              return false;
          }
  
          //返回true的情况是锁已经被释放
          @ReservedStackAccess
          protected final boolean tryRelease(int releases) {
              //state是已经重入的次数
              int c = getState() - releases;
              //判断当前线程是否持有锁，只有持有锁才能解锁，否则抛出异常
              if (Thread.currentThread() != getExclusiveOwnerThread())
                  throw new IllegalMonitorStateException();
              boolean free = false;
              //1. 重入很多次，此次只-1并不代表释放这把锁； 
              //2. 只有当state为0的时候才释放锁
              if (c == 0) {
                  //让当前这把锁恢复自由状态，不被任何线程持有
                  free = true;
                  //设置当前持有这把锁的线程为null
                  setExclusiveOwnerThread(null);
              }
              setState(c);
              return free;
          }
  
          protected final boolean isHeldExclusively() {
              // While we must in general read state before owner,
              // we don't need to do so to check if current thread is owner
              return getExclusiveOwnerThread() == Thread.currentThread();
          }
  
          final ConditionObject newCondition() {
              return new ConditionObject();
          }
  
          // Methods relayed from outer class
  
          final Thread getOwner() {
              return getState() == 0 ? null : getExclusiveOwnerThread();
          }
  
          final int getHoldCount() {
              return isHeldExclusively() ? getState() : 0;
          }
  
          final boolean isLocked() {
              return getState() != 0;
          }
  
          /**
           * Reconstitutes the instance from a stream (that is, deserializes it).
           */
          private void readObject(java.io.ObjectInputStream s)
              throws java.io.IOException, ClassNotFoundException {
              s.defaultReadObject();
              setState(0); // reset to unlocked state
          }
      }
      
      static final class NonfairSync extends Sync {
          private static final long serialVersionUID = 7316153563782823691L;
          protected final boolean tryAcquire(int acquires) {
              return nonfairTryAcquire(acquires);
          }
      }
      
      //公平
      static final class FairSync extends Sync {
          private static final long serialVersionUID = -3000897897090466540L;
          /**
           * Fair version of tryAcquire.  Don't grant access unless
           * recursive call or no waiters or is first.
           */
          //获得锁
          @ReservedStackAccess
          protected final boolean tryAcquire(int acquires) {
              final Thread current = Thread.currentThread();
              int c = getState();
              if (c == 0) {
                  //查询是否有线程在等待队列中等待
                  //队列中没有线程的话CAS更换state值
                  //然后设置当前线程为锁的拥有者
                  if (!hasQueuedPredecessors() &&
                      compareAndSetState(0, acquires)) {
                      setExclusiveOwnerThread(current);
                      return true;
                  }
              }
              else if (current == getExclusiveOwnerThread()) {
                  int nextc = c + acquires;
                  if (nextc < 0)
                      throw new Error("Maximum lock count exceeded");
                  setState(nextc);
                  return true;
              }
              //如果c不为0，就回去查看当前线程是否是锁的拥有者
              //如果是就给state的值加1，并且返回true
              
              return false;//如果都不是上面的情况，那就是返回false了
          }
      }
  
      //省略。。。
  }
  
  public abstract class AbstractQueuedSynchronizer
      extends AbstractOwnableSynchronizer
      implements java.io.Serializable {
      
      public final boolean release(int arg) {
          //调用Sync的tryRelease
          //如果tryRelease返回true，代表这把锁已经被真正释放掉了
          //就会从等待结点中唤醒线程，去获取这把锁
          if (tryRelease(arg)) {
              Node h = head;
              if (h != null && h.waitStatus != 0)
                  unparkSuccessor(h);//把后面的节点unPark唤醒
              return true;
          }
          return false;
      }
      
      
      public final void acquire(int arg) {
          //EXCLUSIVE表示是一个把互斥锁
          //acquireQueued是节点有必要的时候就进行等待，如果有机会获取锁了就尝试获取锁
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }
      
      //省略。。。
  }
  ```
  
  



#### **使用AQS实现一个自己的Latch门闩**

简易版CountDownLatch，一次性门闩（一次释放，统一出发，不设置倒数功能了）。

```java
import java.util.concurrent.locks.AbstractQueuedSynchronizer;

/**
 * 描述：     自己用AQS实现一个简单的线程协作器
 * 此处非独占，可以多个线程同时等待。那么应该实现tryAcquireShared和tryReleaseShared
 */
public class OneShotLatch {

    private final Sync sync = new Sync();

    //释放锁
    public void signal() {
        sync.releaseShared(0);
    }
    
    //尝试获取锁
    public void await() {
        sync.acquireShared(0);
    }

    private class Sync extends AbstractQueuedSynchronizer {

        @Override
        protected int tryAcquireShared(int arg) {
            return (getState() == 1) ? 1 : -1;
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
           setState(1);

           return true;
        }
    }


    public static void main(String[] args) throws InterruptedException {
        OneShotLatch oneShotLatch = new OneShotLatch();
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName()+"尝试获取latch，获取失败那就等待");
                    oneShotLatch.await();
                    System.out.println("开闸放行"+Thread.currentThread().getName()+"继续运行");
                }
            }).start();
        }
        Thread.sleep(5000);
        oneShotLatch.signal();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()+"尝试获取latch，获取失败那就等待");
                oneShotLatch.await();
                System.out.println("开闸放行"+Thread.currentThread().getName()+"继续运行");
            }
        }).start();
    }
}

```





#### **相关设计模式——模板方法模式**





#### **参考资料**

[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

[打通Java任督二脉一并发数据结构的基石](https://juejin.im/post/5c11d6376fb9a049e82b6253)

[一行一行源码分析清楚AbstractQueuedSynchronizer](https://www.javadoop.com/post/AbstractQueuedSynchronizer)

[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

[英文论文的中文翻译](https://www.cnblogs.com/dennyzhangdd/p/7218510.html)



#### 总结

+ 调用CountDownLatch的await方法时，会尝试获取“共享锁”，不过一开始是获取不到该锁的，于是线程被阻塞。而“共享锁”可获取到的条件就是“锁计数器”的值为0。而“锁计数器”的初始值为count，每当一个线程调用该CountDownLatch对象的countDown()方法时，才将“锁计数器”-1。count个线程调用countDown()之后，“锁计数器”才为0，而前面提到的等待获取共享锁的线程才能继续运行。
+ 在Semaphore中，state表示许可证的剩余数量。看tryAcquire方法，判断nonfairTryAcquireShared > 0 的话代表成功。这里会先检查剩余许可证数量够不够这次需要的，用减法来计算，如果直接不够了，就返回负数，表示失败，如果够了，就用自旋加CAS操作改变State状态，直到改变成功就返回正数；或者期间如果被其他线程修改了导致剩余数量不够了，那也返回负数代表获取失败。
+ 释放锁的方法tryRelease，由于是可重入的，所以**state代表重入的次数**，每次释放锁，先判断是不是当前持有锁的线程释放的，如果不是就抛异常；如果是的话，重入次数就减一。如果减到了0 ，就说明完全释放了，于是free就是true，并且把state设置为0（这把锁当前没有被任何线程持有）。加锁的方法：去判断当前state是不是等于0，也会去判断当前线程是不是持有锁的线程，如果都不是，代表目前拿不到这把锁，就放到队列中去，并在以后合适的时机唤醒。
