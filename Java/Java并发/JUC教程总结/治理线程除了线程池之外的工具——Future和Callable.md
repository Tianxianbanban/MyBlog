## 治理线程除了线程池之外的工具——Future和Callable



**Runable的缺陷**

+ 没有返回值

  ```java
  public interface Runnable {
      public abstract void run();
  }
  ```

+ 也不能抛出checked Exception

+ 为什么有这样的缺陷呢？

  这是Runable定义的时候就已经决定了的。

+ Runnable为什么还要这样设计呢？

  抛出异常可能也没有机会被接收处理，不如直接在run方法当中直接处理了。

+ 针对无法抛出受检异常这个缺陷的不久措施？

  使用Callable接口



**Callable接口**

+ 类似于Runable，被其他线程执行的任务。

+ 需要实现call方法。

+ 有返回值

  ```java
  public interface Callable<V> {
      V call() throws Exception;
  }
  ```

  

**Future类**

+ Future的作用

  如果遇到耗时的方法，可以使用子线程去执行的同时先做其他的事情，直到想要获取执行结果的时候再去用Future去控制是否取消、获得结果等。

+ Callable和Future的关系

  我们可以用**Future.get**来获取Callable接口的返回的执行结果，还可以通过**Future.isDone()**来判断任务是否已经执行完了，以及**取消这个任务**，**限时获取任务的结果**等。

  在call()未执行完毕之前，调用get()的线程（假定此时是主线程）会被**阻塞**，知道call()方法返回结果后，此时future.get()才会得到该结果，然后主线程才会切换到runable的状态。

  所以Future是一个**存储器**，它**存储了call()方法这个任务的结果**，而这个任务的执行时间是无法提前确定的，因为这完全取决于call()方法执行的情况.

+ Future的5个主要方法

  + get方法：获取结果

    get方法的行为取决于Callable任务的状态，只有以下这5种情况：

    1. 任务正常完成：get方法会立刻返回结果
    2. 任务尚未完成（任务还未开始或进行中）：get将**阻塞**并直到任务完成。
    3. 任务执行过程中抛出**Exception**：get方法会抛出ExecutionException：这里的抛出异常，是call()执行时产生的按个异常，看到这个异常类型是java.util.concurrent.ExecutionException。无论call()执行时抛出的异常类型是什么，最后get方法抛出的异常类型都是ExecutionException。
    4. 任务被**取消**：get方法会抛出CancellationException，就可以据此重试或者采取其他手段。
    5. 任务**超时**：get方法有一个重载方法，是传入一个延迟时间的，如果时间到了还没有获得结果，get方法就会抛出TimeoutException。

  + get(long timeout, TimeUnit unit)：有超时的获取

    + 超时的需求很常见
    + 用get(long timeout, TimeUnit unit)方法时，如果call()在规定时间内完成了任务,那么就会正常获取到返回值；而如果指定时间内没有计算出结果，那么就会**抛出TimeoutException**。
    + 超时不获取，**任务需要取消**。

  + cancel方法：取消任务的执行

  + isDone()方法：判断线程**是否执行完毕**。但是不是代表执行成功或者失败。

  + isCancelled()方法：判断是否被取消。

  ```java
  public interface Future<V> {
  
      boolean cancel(boolean mayInterruptIfRunning);
      boolean isCancelled();
      boolean isDone();
      V get() throws InterruptedException, ExecutionException;
      V get(long timeout, TimeUnit unit)
          throws InterruptedException, ExecutionException, TimeoutException;
  }
  ```

+ Future的使用

  + get使用：线程池中submit方法返回Future对象

    首先，我们要给线程池提交我们的任务，提交时线程池会**立刻**返回给我们一个**空的Future容器**。 当线程的任务一旦执行完毕也就是当我们可以获取结果的时候，线程池便会把该**结果填入**到之前给我们的那个Future中去(而不是创建一个新的Future ) ，我们此时便可以**从该Future中获得任务执行的结果**。

    ```java
    public class OneFuture {
    
        public static void main(String[] args) {
            ExecutorService service = Executors.newFixedThreadPool(10);
            Future<Integer> future = service.submit(new CallableTask());
            try {
                System.out.println(future.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
            service.shutdown();
        }
    
        static class CallableTask implements Callable<Integer> {
    
            @Override
            public Integer call() throws Exception {
                Thread.sleep(3000);
                return new Random().nextInt();
            }
        }
        
    }
    ```

  + Callable的Lambda表达式形式

  + 多个任务的情况下，用Future数组来获取结果。

  + 任务执行过程中抛出Exception和isDone。

    get方法过程中抛出异常，并不是说一产生异常就抛出，直到我们**get执行时，才会抛出**。而且无论call()执行时抛出的异常类型是什么，最后get方法抛出的异常类型都是**ExecutionException**

    isDone只会反应是否执行结束，即是任务不是成功或是抛出异常。

  + 获取任务超时
  
  ```java
  
  /**
   * 演示get的超时方法，需要注意超时后处理，调用future.cancel()。
   * 演示cancel传入true和false的区别，代表是否中断正在执行的任务。
   */
  public class Timeout {
  
      private static final Ad DEFAULT_AD = new Ad("无网络时候的默认广告");
      private static final ExecutorService exec = Executors.newFixedThreadPool(10);
  
      static class Ad {
  
          String name;
  
          public Ad(String name) {
              this.name = name;
          }
  
          @Override
          public String toString() {
              return "Ad{" +
                      "name='" + name + '\'' +
                      '}';
          }
      }
  
  
      static class FetchAdTask implements Callable<Ad> {
  
          @Override
          public Ad call() throws Exception {
              try {
                  Thread.sleep(3000);
              } catch (InterruptedException e) {
                  System.out.println("sleep期间被中断了");
                  return new Ad("被中断时候的默认广告");
              }
              return new Ad("旅游订票哪家强？找某程");
          }
      }
  
  
      public void printAd() {
          Future<Ad> f = exec.submit(new FetchAdTask());
          Ad ad;
          try {
              ad = f.get(2000, TimeUnit.MILLISECONDS);
          } catch (InterruptedException e) {
              ad = new Ad("被中断时候的默认广告");
          } catch (ExecutionException e) {
              ad = new Ad("异常时候的默认广告");
          } catch (TimeoutException e) {
              ad = new Ad("超时时候的默认广告");
              System.out.println("超时，未获取到广告");
              //true和false的区别，代表是否中断正在执行的任务。
              boolean cancel = f.cancel(true);
              System.out.println("cancel的结果：" + cancel);
          }
          exec.shutdown();
          System.out.println(ad);
      }
  
      public static void main(String[] args) {
          Timeout timeout = new Timeout();
          timeout.printAd();
      }
  }
  
  /*
  true输出结果：
  超时，未获取到广告
  sleep期间被中断了
  cancel的结果：true
  Ad{name='超时时候的默认广告'}
  
  false输出结果：
  超时，未获取到广告
  cancel的结果：true
  Ad{name='超时时候的默认广告'}
   */
  ```
  
  



**线程池中submit方法返回Future对象**



**用FutureTask来创建Future**



**Future注意点**