## Android的消息机制



## 概述

Handler是Android中消息机制的上层接口，所以开发过程中只需要和Handler交互即可。通过Handler就可以将一个任务切换到**Handler所在线程**中执行。其中的一个应用场景就是在子线程中执行耗时操作例如读取文件访问网络等以后可以通过handler将更新UI（UI非线程安全，android当中不允许在子线程中更新；而耗时操作如果在主线程进行会造成ANR）的操作切换回主线程中执行。

Android中的消息机制主要是指Handler的运行机制。**Handler创建时会采用当前线程的Looper来构建内部消息循环系统**，然后可以通过Handler的一系列post方法（内部也是通过send方法完成的）将一个Runnable对象投递到Handler内部的Looper中去处理，或者通过Handler的一系列send方法发送一个消息到Looper中去处理，它会调用MessageQueue的enqueueMessage方法将这个消息放入消息队列待Looper处理。



## 分析

### **ThreadLocal的作用**

#### 使用场景

定义：ThreadLocal是一个**线程内部的数据存储类**，通过它可以在指定的线程中存储数据，数据存储以后只能在指定的线程中获取到存储的数据，其他线程无法获取。

使用场景：（1）当某些数据是**以线程为作用域并且不同线程具有不同的数据副本**时就可以采用ThreadLocal。（2）**复杂逻辑下的对象传递**，比如监听器的传递，有的时候一个线程中的任务过于复杂，可能表现为函数调用栈比较深以及代码入口的多样性，我们又需要监听器能够贯穿整个线程的执行过程，这个时候可以采用ThreadLocal。

+ 典型场景1：每个线程需要一个独享的对象（比如一些工具类，由于本身不是线程安全，如果多个线程共享同一个静态工具类的话有一定风险，就使用ThreadLocal给每个线程制造一个独享的对象，线程之间持有不同的实例就不会互相影响了，典型需要使用的类有SimpleDateFormat和Random）。**让某个需要用到的对象在线程间隔离**。
+ 典型场景2：每个线程内需要保存全局变量（比如一些常用业务内容），以便让不同方法直接使用，避免繁琐的传参。这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的。使用ThreadLocal的话，无需使用synchronized关键字，可以在不影响性能的情况下，无需层层传递参数，就可以达到保存当前线程可以使用的信息的目的。在线程的生命周期内，通过静态ThreadLocal实例的get()方法取得自己set过的对象。**在任何方法中都可以轻松获取到该对象。**
+ 好处：
  1. 达到线程安全
  2. 不需加锁，提高执行效率
  3. 更高效利用内存、节省开销：相比于每个任务都新建一个实例，使用ThreadLocal可以节省内存和开销.（比如100个任务提交给拥有10个线程的线程池，只要保证10个线程各自拥有一个处理任务的工具类实例就行，而不需要每个任务创建一个）
  4. 免去传参的繁琐：使代码耦合度更低、更优雅。

#### **原理**

工作过程：**不同的线程访问同一个ThreadLocal**的get方法，ThreadLocal内部会从**各自的线程中取出一个ThreadLocalMap**，然后再从ThreadLocalMap中以**当前ThreadLocal作为key**去查找对应的value值，不同线程中的ThreadLocalMap是不同的，所以可以通过ThreadLocal在不同线程中维护一套数据的副本而互不干扰。

内部实现：

+ ThreadLocal是一个泛型类public class ThreadLocal<T>

+ initialValue()

  + 该方法会返回当前线程对应的“初始值”，是一个延迟加载的方法，只有调用get的时候，才会触发。除非线程先前调用set方法(在这种情况下，不会为线程调用本initialValue方法)。
  + 通常每个线程最多调用一次此方法，但如果已经调用了remove后，再调用get方法，则可以再次调用此方法。
  + 如果不重写此方法，默认返回null。一般使用匿名内部类来重写initialValue方法，以便在后续使用中可以初始化副本对象。
  
+ ```java
  public void set(T value) {
          Thread t = Thread.currentThread();
      	//取出当前线程的ThreadLocalMap
      	//ThreadLocalMap不是Map，但是可以理解为Map
          ThreadLocalMap map = getMap(t);
          if (map != null)
              map.set(this, value);//key是ThreadLocal当前变量，value是我们需要的值
          else
              createMap(t, value);
      }
  
  ThreadLocalMap getMap(Thread t) {
          return t.threadLocals;
      }
  
  //Thread内定义了成员threadLocals
  //设置到ThreadLocal中的数据也就是写入了threadLocals这个ThreadLocalMap
  ThreadLocal.ThreadLocalMap threadLocals = null;
  //threadLocals本身就保存了当前线程所有“局部变量”，也就是一个ThreadLocal变量的集合。
  ```
  
  
  
+ ```java
  public T get() {
            Thread t = Thread.currentThread();
        //进行get操作的时候也就是把当前线程的ThreadLocalMap取出
            ThreadLocalMap map = getMap(t);
            if (map != null) {
                //将ThreadLocal自己本身作为key去取出内部的实际数据
                ThreadLocalMap.Entry e = map.getEntry(this);
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    T result = (T)e.value;
                    return result;
                }
            }
            return setInitialValue();
        }
  ```

  这个map以及map中的key和value都是保存在线程中的，而不是保存在ThreadLocal中。
  
+ remove 删除对应这个线程的值

+ 从set/get方法的实现可知，这些变量是维护在Thread类内部的ThreadLocalMap当中，只要**线程不退出，对象的引用将一直存在**，而当线程退出时，Thread类做的清理工作就包括了清理ThreadLocalMap。

  但如果使用线程池，线程未必会退出，如果将一些大的对象设置到ThreadLocal中，可能会出现**内存泄漏**，所以如果希望及时回收对象，可以使用**ThreadLocal.remove()**方法将变量移除。当然将ThreadLocal赋为null也可以加速回收，原因与[**ThreadLocalMap的实现使用了弱引用**有关](https://developer.aliyun.com/article/689656)，内部由一系列的Entry构成，static class Entry extends WeakReference<ThreadLocal<?>>。
  
+ ThreadLocalMap是Thread类中的变量，其中最重要的是一个键值对数组Entry[] table，可以认为是一个map（使用线性探测法处理哈希冲突），键值对：

  + 键：这个ThreadLocal
  + 值：实际需要的成员变量
  

#### ThreadLocal注意点

+ 内存泄露（某个对象不再有用，但是占用的内存却不能被回收）==对下面这堆解释目前不理解不赞同==

  + key是弱引用，弱引用不会阻止GC
  + value是强引用

  > 正常情况下，当线程终止，保存在ThreadLocal中的value会被垃圾回收，因为没有任何强引用了。但是如果线程不终止，key对应的value就不能被回收，因为存在调用链：Thread——ThreadLocalMap——Entry（key为null）——value。也即value无法回收，就可能会出现OOM。
  >
  > JDK已经考虑到这个问题，所以在set、remove、rehash方法中会扫描key为null的Entry，并将对应的value设置为null，这样value对象就可以被回收。
  >
  > 但是如果一个ThreadLocal不被使用，那么实际上set、remove、rehash方法也不会被调用，如果同时线程又不停止，那么调用链就一直存在，就会导致value的内存泄露。
  >
  > 避免内存泄露（阿里规约）：调用remove方法，就会删除对应的Entry对象，可以避免内存泄露，所以使用完ThreadLocal之后，应该调用remove方法。

+ 空指针异常问题

  在没有初始化的情况下get只会返回null。如果出现空指针异常可能是代码原因（比如使用获取的null值进行拆箱操作等）。

+ 共享对象

  如果在每个线程中ThreadLocal.set进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得的还是这个共享对象本身，还是有并发访问问题。

+ 如果不使用ThreadLocal就可以解决问题，不需要强行使用

  例如任务数很少的时候，在局部变量中新建对象就可以解决问题，就不需要使用到ThreadLocal。

+ 优先使用框架支持，而不是自己创造

  例如在Spring中，如果可以使用RequestContextHolder，就不需要自己去维护一个ThreadLocal，以免因为忘记调用remove方法等，造成内存泄露。



### **Message**

+ 消息对象

  Message包括了 int what（消息类别）、long when（消息触发时间）、int arg1、int arg2（参数2）、Object obj（消息内容）、Handler target（消息响应方）、Runable callback（回调方法）。

+ 消息池

  静态变量`sPool`的数据类型为Message，通过next成员变量，维护一个消息池；静态变量`MAX_POOL_SIZE`代表消息池的可用大小；消息池的默认大小为50。

  **Message.obtain()**，从消息池取Message，都是把消息池表头的Message取走，再把表头指向next。
  
  **Message#recycle()**，将Message加入到消息池的过程，都是把Message加到链表的表头。



### **消息队列的工作原理**

+ MessageQueue通过**单链表**数据结构来维护消息列表；

+ boolean enqueueMessage(Message msg, long when) **插入消息**，也就是单链表的插入；

+ Message next() **取出一条消息并从消息队列中删除**，next是一个**无限循环**的方法，如果消息队列中没有消息就会**阻塞**在这里，而有消息到来就会返回这条消息并从单链表中删除。

+ 源码梳理（==暂未求证==）

  ```java
  //MessageQueue.java
  public final class MessageQueue {
      //...
      
      boolean enqueueMessage(Message msg, long when) {
          if (msg.target == null) {
              throw new IllegalArgumentException("Message must have a target.");
          }
  
          synchronized (this) {
              if (msg.isInUse()) {
                  throw new IllegalStateException(msg + " This message is already in use.");
              }
  
              if (mQuitting) {
                  IllegalStateException e = new IllegalStateException(
                          msg.target + " sending message to a Handler on a dead thread");
                  Log.w(TAG, e.getMessage(), e);
                  msg.recycle();
                  return false;
              }
  
              msg.markInUse();
              msg.when = when;
              // 队头
              Message p = mMessages;
              boolean needWake;
              if (p == null || when == 0 || when < p.when) {
                  // 将消息放入队头位置
                  // New head, wake up the event queue if blocked.
                  msg.next = p;
                  mMessages = msg;
                  needWake = mBlocked;
              } else {
                  // 将新消息插入到队列中，同步消息根据when来确定队列位置，
                  //如果是同步屏障，同步屏障消息target==null，且新消息是异步的，则needWake为true，则计划唤醒队列。
                  // Inserted within the middle of the queue.  Usually we don't have to wake
                  // up the event queue unless there is a barrier at the head of the queue
                  // and the message is the earliest asynchronous message in the queue.
                  needWake = mBlocked && p.target == null && msg.isAsynchronous();
                  Message prev;
                  for (;;) {
                      prev = p;
                      p = p.next;
                      if (p == null || when < p.when) {
                          break;
                      }
                      // 如果队头的一下个消息也是异步消息，且未到唤醒时间，
                      //则将needWake置为false，不唤醒队列
                      if (needWake && p.isAsynchronous()) {
                          needWake = false;
                      }
                  }
                  msg.next = p; // invariant: p == prev.next
                  prev.next = msg;
              }
  
              // We can assume mPtr != 0 because mQuitting is false.
              if (needWake) {
                  nativeWake(mPtr);
              }
          }
          return true;
      }
      
      @UnsupportedAppUsage
      Message next() {
          // Return here if the message loop has already quit and been disposed.
          // This can happen if the application tries to restart a looper after quit
          // which is not supported.
          final long ptr = mPtr;
          if (ptr == 0) {
              return null;
          }
  
          int pendingIdleHandlerCount = -1; // -1 only during first iteration
          int nextPollTimeoutMillis = 0;
          for (;;) {
              if (nextPollTimeoutMillis != 0) {
                  Binder.flushPendingCommands();
              }
              
              //nextPollTimeoutMillis为-1时（消息队列为空）该方法阻塞直到有新消息为止，
              //或者等待指定的超时时间
              nativePollOnce(ptr, nextPollTimeoutMillis);
              //如果阻塞操作结束，则去获取消息
              synchronized (this) {
                  // Try to retrieve the next message.  Return if found.
                  final long now = SystemClock.uptimeMillis();
                  Message prevMsg = null;
                  Message msg = mMessages;
                  // 同步屏障，优先处理异步消息，此时同步消息不会被处理
                  //当消息的Handler为空时，则查询异步消息
                  if (msg != null && msg.target == null) {
                      // Stalled by a barrier.  Find the next asynchronous message in the queue.
                      do {
                          prevMsg = msg;
                          msg = msg.next;
                      } while (msg != null && !msg.isAsynchronous());
                  }
                  if (msg != null) {
                      // 消息未到触发时间，设置新的唤醒时间
                      //当异步消息触发时间大于当前时间，则设置下一次轮询的超时时长
                      if (now < msg.when) {
                          // Next message is not ready.  Set a timeout to wake up when it is ready.
                          nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                      } else {
                          // 获取到消息，同时重组新的队列，取走的消息从队列中移除
                          // Got a message.
                          mBlocked = false;
                          if (prevMsg != null) {
                              prevMsg.next = msg.next;
                          } else {
                              mMessages = msg.next;
                          }
                          msg.next = null;
                          if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                          msg.markInUse();
                          return msg;
                      }
                  } else {
                      //没有消息
                      // No more messages.
                      nextPollTimeoutMillis = -1;
                  }
  
                  // Process the quit message now that all pending messages have been handled.
                  if (mQuitting) {
                      dispose();
                      return null;
                  }
  
                  // If first time idle, then get the number of idlers to run.
                  // Idle handles only run if the queue is empty or if the first message
                  // in the queue (possibly a barrier) is due to be handled in the future.
                  if (pendingIdleHandlerCount < 0
                          && (mMessages == null || now < mMessages.when)) {
                      //如果消息队列为空或者消息执行时间还未到，则获取IdleHandler队列的大小，下面需要用到
                      pendingIdleHandlerCount = mIdleHandlers.size();
                  }
                  if (pendingIdleHandlerCount <= 0) {
                      // No idle handlers to run.  Loop and wait some more.
                      mBlocked = true;
                      continue;
                  }
  
                  if (mPendingIdleHandlers == null) {
                      mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                  }
                  //将IdleHandler列表转为数组
                  mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
              }
  
              // Run the idle handlers.
              // We only ever reach this code block during the first iteration.
              for (int i = 0; i < pendingIdleHandlerCount; i++) {//开始顺序执行所有IdleHandler
                  final IdleHandler idler = mPendingIdleHandlers[i];
                  mPendingIdleHandlers[i] = null; // release the reference to the handler
  
                  boolean keep = false;
                  try {
                      keep = idler.queueIdle();//具体执行
                  } catch (Throwable t) {
                      Log.wtf(TAG, "IdleHandler threw exception", t);
                  }
  
                  if (!keep) {
                      //根据queueIdle()方法返回值决定是否移除该IdleHandler
                      synchronized (this) {
                          mIdleHandlers.remove(idler);
                      }
                  }
              }
  
              // Reset the idle handler count to 0 so we do not run them again.
              pendingIdleHandlerCount = 0;
  
              // While calling an idle handler, a new message could have been delivered
              // so go back and look again for a pending message without waiting.
              nextPollTimeoutMillis = 0;
          }
      }
      
  }
  ```

  ```c++
  //nativePollOnce最终执行到的函数
  int Looper::pollInner(int timeoutMillis) {
         ...
      // Poll.
      int result = POLL_WAKE;
      mResponses.clear();
      mResponseIndex = 0;
      mPolling = true;//即将处于idle状态
  
      struct epoll_event eventItems[EPOLL_MAX_EVENTS];
      int eventCount = epoll_wait(mEpollFd.get(), eventItems, EPOLL_MAX_EVENTS, timeoutMillis);//等待事件发生或者超时，在nativeWake()方法，向管道写端写入字符，则该方法会返回；
      ...
      return result;
  }
  ```

  + 针对同步消息：将**时间小的消息放在靠近队头的位置，时间长的消息靠近队尾**。 从队头开始，找到新消息在队列中合适的位置，然后将新消息插入到队列中，如果队列为null或新消息的等待时间小于队头消息的等待时间，则直接将新消息放入到队列的头部。如果队头消息的时间小于新消息，则新消息将从队头消息开始依次和队列中的消息进行比较，直到找到合适的位置。

  + 在next方法中，当发现队列中**有同步屏障时，此时会优先返回队列中的异步消息**，从队列中获取异步消息，并将该异步消息从队列中移除，如果同步屏障不被移除，即使异步消息被处理完毕，同步消息也不会被处理，队列会进入阻塞状态。 如果队列中没有同步屏障，则从队列中获取同步消息，并将该同步消息从队列中移除。

  + 同步屏障：从名称上来看，就是**将同步消息屏蔽**的意思，当队列中包含同步屏障消息时，整个消息队列会进入优先处理异步消息的状态，同时同步消息会被屏蔽。 在说同步屏障时，需要区分清楚MessageQueue队列中的消息类型，方便理解同步屏障，MessageQueue队列中可以分为三种消息，**屏障消息，同步消息和异步消息**。 屏障消息: target为null的Message对象，即没有消费者的消息，该消息仅用来标记当前队列优先处理异步消息。 同步消息: target不为null的Message对象，即可以被消费的消息，且该消息的isAsynchronous方法返回false。 异步消息: target不为null的Message对象，和同步消息的区别是isAsynchronous方法返回true。 异步消息需要用户手动的将Message标记为异步，通过方法setAsynchronous(true)将消息标记为异步。

    如何使用同步屏障：在MessageQueue类中有两个方法，用来添加和删除同步屏障，分别为postSyncBarrier()和removeSyncBarrier()。 通过Handler -> Looper -> MessageQueue,然后使用其方法即可，其实这两个方法的使用场景基本不可见，因为这两个方法被声明为hide方法，相当于系统自己留的后门，优先处理**高优先级的任务**，所以我们在调用时会出错，结合源码看，该方法只在Android的ViewRootImpl中被调用了，主要用来处理绘图操作（后续再去仔细分析ViewRootImpl的源码）。

    同步屏障原理：上面分析的插入消息我们知道，同步消息和异步消息的入队均是通过when的大小来确定消息在队列中的位置，在读取消息时，队列通过消息的msg.target==null来判断同步屏障。如果队列有同步屏障消息，则优先处理异步消息（即msg.isAsynchronous()返回true），直到异步屏障被清理，异步消息处理顺序一样是通过when大小来确定。如果不包含同步屏障，则处理同步消息，根据when大小来确定消息的处理顺序。 这里声明下：when大小是不会相同的，因为消息的入队是顺序的，所以根据入队时的时间计算出的when大小是不同的。
    
    所以并不是所有的msg，target值都必须不为空，**（handler的同步屏障就是一个target为空的msg，用来优先执行异步方法的）**同步屏障有一个很重要的使用场所就是接受垂直同步Vsync信号，用来刷新页面view的（因为为了保证view的流畅度，所以每次刷新信号到来的时候，要把其他的任务先放一放，优先刷新页面）。
    
  + 可以看到，计算出nextPollTimeoutMillis后就调用nativiePollOnce这个native方法。这里的话大概可以猜到他的运行机制，因为他是根据执行时间进行排序的，那传入的这个nextPollTimeoutMillis应该就是休眠时间，类似于java的sleep(time)。休眠到下一次message的时候就执行。那如果我在这段时间又插入了一个新的message怎么办，所以**handler每次插入message都会唤醒线程，重新计算插入后，再走一次这个休眠流程**。
  
  + nativiePollOnce这个native方法可以通过名字知道，他用的是linux中的epoll机制。这个epoll和select一样都是linux的一个I/O多路复用机制，主要原理就不深入了，这里大概了解一下I/O多路复用机制和它与Select的区别就行。
  
    **Linux里的I/O多路复用机制**：举个例子就是我们钓鱼的时候，为了保证可以最短的时间钓到最多的鱼，我们同一时间摆放多个鱼竿，同时钓鱼。然后哪个鱼竿有鱼儿咬钩了，我们就把哪个鱼竿上面的鱼钓起来。这里就是把这些全部message放到这个机制里面，那个time到了，就执行那个message。
  
    **epoll与select的区别**：epoll获取事件的时候采用空间换时间的方式，类似与事件驱动，有哪个事件要执行，就通知epoll，所以获取的时间复杂度是O（1），select的话则是只知道有事件发生了，要通过O（n）的事件去轮询找到这个事件。



### **Looper的工作原理**

消息循环：会不停地从MessageQueue中查看是否有新消息，如果有新消息就会立刻处理，否则会**阻塞**。

+ **public static void prepare() 为当前线程创建Looper，保证Handler可以工作**，给主线程ActivityThread创建Looper还可以使用prepareMainLooper方法，getMainLooper可以在任何地方获取到主线程的Looper。
+ **private Looper(boolean quitAllowed) 构造方法中创建消息队列MessageQueue，并保存当前线程对象**；
+ **public static void loop() 调用了这个方法，消息系统才会真正起作用**。loop是个**死循环**，只有当messageQueue.next()返回null才退出循环，也就是**消息队列退出时**。
+ 在子线程中，如果手动创建了Looper，最好是在任务完成不再需要的时候调用**quit或者quitSafely方法终止消息循环**，否则这个子线程会一直处于等待状态，而如果退出Looper以后，这个线程就会立即终止。quit或者quitSafely方法被调用的时候，Looper也会调用**MessageQueue的quit或者quitSafely方法通知消息队列退出，next方法也会返回null**。
+ Looper处理一条消息，msg.target.dispatchMessage(msg)，也就是handler的dispatchMessage方法，Handler发送的消息最终又交给自己的方法来处理了，只是**dispatchMessage方法是在创建Handler时使用的Looper中执行的，这样就将代码逻辑切换到指定的线程中执行了。**

```java
public static void prepare() 为当前线程创建Looper，保证Handler可以工作；
private Looper(boolean quitAllowed) 构造方法中创建消息队列MessageQueue，并保存当前线程对象；
public static void loop() 调用了这个方法，消息系统才会真正起作用。loop是个死循环，只有当messageQueue.next()返回null才退出循环，也就是消息队列退出时。
```



### **Handler的工作原理**

Handler的**一系列post方法**（内部也是通过send方法完成的）将一个Runnable对象投递到Handler内部的Looper中去处理，或者通过Handler的**一系列send方法**发送一个消息到Looper中去处理。

handler发送消息的典型过程如下，可以看出这就是**向消息队列中插入了一条消息**，然后MessageQueue的next方法会返回这条消息给Looper处理，Looper处理消息会调用**msg.target.dispatchMessage(msg)**，最终交给Handler处理，也就是Handler的dispatchMessage方法会被调用，Handler就进入了处理消息的阶段。

```
public final boolean sendMessage(Message msg)
public final boolean sendMessageDelayed(Message msg, long delayMillis)
public final boolean post(@NonNull Runnable r)
public final boolean postDelayed(@NonNull Runnable r, long delayMillis)
public boolean sendMessageAtTime(Message msg, long uptimeMillis) 
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)
```

```
public void dispatchMessage(Message msg)
public void handleMessage(Message msg)//子类重写或者是用Callback内部的handleMessage方法
```

另外关于构造方法，handler内部的消息队列就是赋值了Looper当中的消息队列。

```
public Handler() 
public Handler(Callback callback, boolean async) //在这个方法内会去判断当前线程有无Looper，如果没有的话就会抛出异常。
```



#### **关于Handler的内存泄露问题**

+ 原因：

  首先Java中**非静态内部类** &**匿名内部类**都默认持有外部类的引用 ，另外主线程中Looper的生命周期和应用的声明周期是一致的。那么也就是Handler 的生命周期和 Activity 是不一致的（**Looper持有Message的引用，Message持有Handler的引用**），所以经常会带来内存泄漏的问题。

  比如这样的情况：在 Activity 中定义了一个继承自**Handler 的非静态内部类**，并通过它发送了一条消息，该消息会在 10 分钟之后返回当前时间，接着将 Activity 退出，但是此时 Activity 并不会被 GC 回收掉的，因为我们的**消息任务还在 MessageQueue 中排队**，Message当中定义了Handler target，也就是**含有Handler的引用**，那么 Handler 是无法释放的，而 **Handler本身又持有外部类 Activity 的引用**，那么也就导致了 Activity 不能被回收释放了，造成内存泄漏。

+ 解决：**静态内部类+弱引用**，将非静态内部类设置成 static 类型的，同时为了高效率的回收，我们可以将所**引用的外部类的实例在内部类中设置成 WeakReference 类型，也就是弱引用类型**。此外为了防止 Looper对象的内存泄漏，我们可以在Activity销毁的时候调用removeCallbackAndMessages 方法，移出 MessageQueue 里面的所有消息。

  例如：

  ```java
  private Handler handler = new Handler() {
      public void handleMessage(Message msg) {//在主线程执行
          if (msg.what == 1) {
              //4. 在handleMessage()中处理消息
              String result = (String) msg.obj;
              et_handler1_result.setText(result);
              pb_handler1_loading.setVisibility(View.INVISIBLE);
          }
      }
  };
  
  	//可以修改为：
  	//通过使用Static和弱引用WeakReference结合解决，
      Handler handler=new MyHandler(this);
  
      private static class MyHandler extends Handler {
          private final WeakReference<HandlerTestActivity> activity;
  
          private MyHandler(HandlerTestActivity activity) {
              this.activity = new WeakReference<>(activity);
          }
  
          @Override
          public void handleMessage(Message msg) {
              super.handleMessage(msg);
              HandlerTestActivity mActivity = activity.get();
              if (mActivity != null) {
                  mActivity.doHandleMessage(msg);
              }
          }
      }
  
  
      private void doHandleMessage(Message msg) {
          if (msg.what == 1) {
              //4. 在handleMessage()中处理消息
              String result = (String) msg.obj;
              et_handler1_result.setText(result);
              pb_handler1_loading.setVisibility(View.INVISIBLE);
          }
      }
  ```

+ 其它内存泄露分析：[重新理解为什么Handler可能导致内存泄露](https://mp.weixin.qq.com/s/8Emvbmbn1CgBPjddETEjMA)



----

#### **主线程的消息循环**

Android的主线程就是ActivityThread，主线程的入口方法为main，在main方法中系统会通过Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop（）来开启主线程的消息循环。主线程消息循环开始以后，ActivityThread还需要一个Handler来和消息队列进行交互，这个Handler就是ActivityThread.H，它内部定义了一组消息类型，主要包含了**四大组件的启动和停止等过程**。ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方式完成ActivityThread的请求后会回调ApplicationThread的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后会将ApplicationThread中的逻辑切换到ActivityThread中去执行，即切换到主线程中执行，这个过程就是主线程消息循环模型。



#### MessageQueue中没有消息时会ANR吗？

[Handler后传篇一: 为什么Looper中的Loop()方法不能导致主线程卡死?](https://juejin.cn/post/6844903774096457736#comment)==目前仅参考==

主线程的Looper为什么不会导致应用ANR？

> 是否了解ANR产生条件？
>
> 是否对Android App的进程运行机制有深入理解？
>
> 是否对Looper的消息机制有深刻的理解？
>
> 是否对IO多路复用有一定的认识？

ANR是怎么产生的？

ANR类型：

+ Service Timeout：

  + 前台服务20s
  + 后台服务200s

  与ActiveServices#scheduleServiceTimeoutLocked（ProcessRecord proc）有关

+ BroadcastQueue Timeout:

  + 前台广播10s
  + 后台广播60s

+ ContentProvider Timeout：10s

+ InputDispatching Timeout：5s

​	ActivityThread主线程中也调起了Looper循环。

**Looper的工作机制是什么？**

**Looper不会导致应用ANR的本质原因是什么？**

Looper和ANR的关系

Looper是一个整体进程上的概念，而ANR是执行到某个环节对开发者占用主线程耗时的监控。按照这个关系来看，Looper无法产生ANR。

**Looper为什么不会导致CPU占用率高？**

+ epoll_wait

  因此并没有占用CPU空转，而是阻塞（放弃CPU时间片）等待唤醒



#### Handler发送消息的Delay靠谱吗？

> 是否清除UI时间相关的任务如动画的设计实现原理
>
> 是否对Looper的消息机制深刻理解
>
> 是否做过UI过度绘制或者其他消息机制的优化

**不可靠**，原因有如下：

1. 发送的消息越多Looper负载越高，任务越容易积压，进而导致卡顿
2. 消息队列有一些消息处理非常耗时，导致后面的消息延时处理
3. 大于Handler Looper的周期时基本可靠（例如主线程>50ms）
4. 对于时间精确度要求较高，不要用handler的delay作为即时的依据

如何优化可靠性？

1. 精简消息：这几点主要确保处理的消息精简，避免资源浪费
   + 队列优化，重复消息过滤

     removeCallbacksAndMessages等

   + 互斥消息取消

   + 复用消息

     减少对象创建，减少GC造成的卡顿

2. 适当使用IdleHandler在空闲时处理任务

3. 使用独享Looper（HandlerThread）

#### Handler发送消息可能会丢失吗

+ 从MessageQueue#enqueueMessage源码可以看到，mQuitting == true时候不会继续插入消息，而会抛出异常。即MessageQueue调用quit退出时。
+ 

#### Handler其他用法

+ IdleHandler

  ```java
  MessageQueue.IdleHandler idleHandler = new MessageQueue.IdleHandler() {
              @Override
              public boolean queueIdle() {
                  //处理任务
                  return true;
              }
          };
  
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
              Looper.getMainLooper().getQueue().addIdleHandler(idleHandler);
          }
  ```

  [IdleHandler原理以及延迟初始化方案实现](https://juejin.cn/post/7055564669540368392)

  + 梳理原理可得，IdleHandler执行的条件是消息队列空闲：（1）需要执行的消息还未到时间；（2）消息队列为空



### 其他

#### 为什么非UI线程不能更新UI？

> 线程安全的概念
>
> UI线程的工作机制
>
> SurfaceView实现高帧率的原理

**UI线程是什么？**

Zygote —— App —— ActivityThread （主线程）

**主线程如何工作？**

Handler消息机制

**如果把UI设计成线程安全的**

UI为什么不设计成线程安全的

+ UI具有可变性，甚至是高频可变性。变化快，在此基础上如果使用加锁同步策略性能又会很差。
+ UI对响应时间的敏感性要求UI操作必须高效。
+ UI组件必须批量绘制来保证效率。

**非UI线程一定不能更新UI吗？**

+ 通过Handler等方式

+ SurfaceView（UI线程的其他存在形式）

  使用了非UI线程去刷新修改绘制UI内容，lockCanvas

  同时因为绘制的时间放到了非UI线程，也可以实现比较高的帧率。



自己实现一个简单的Handler-Looper框架？

> 是否对Looper的消息机制有深刻的理解？
>
> 是否对Java并发包中提供的队列有较为清除的认识？
>
> 是否能够运用所学知识设计出一套类似的框架？

Handler的核心能力

+ 线程间通信
+ 延迟任务执行

Looper的核心能力

+ 循环，从MessageQueue中取消息分发

MessageQueue的核心能力

+ 持有消息
+ 消息按照时间排序（优先级）
+ 队列为空时阻塞读取
+ 头节点有延时可以定时阻塞

使用DelayQueue





**文章**

[Android必知必会——消息机制](https://juejin.im/post/5e9d853ee51d4546d4398620#heading-20)

https://cloud.tencent.com/developer/article/1780768

https://juejin.cn/post/6893791473121280013#comment



临时问题：

- [x] Message.when 干啥