## Android的线程和线程池



### **主线程和子线程**

Android沿用Java的线程模型，分为主线程和子线程，其中主线程主要处理和界面相关的事情，也叫UI线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程的作用则是执行耗时任务，比如网络请求、IO操作等等。由于Android的特性，如果在主线程中去执行耗时操作那么会导致程序无法及时响应可能出现ANR现象，因此耗时操作必须放在子线程中去执行。



### **Android中的线程形态**

#### Thread

#### AsyncTask

封装了Thread+Handler，可以在线程池中执行任务，然后把执行进度和最终结果传递给主线程中更新UI。

（但是AsyncTask并不适合进行特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。）

+ 抽象泛型类

  + Params 参数的类型
  + Progress 后台任务执行进度的类型
  + Result 后台任务返回结果的类型

+ 四个核心方法

  + onPreExecute

    异步任务执行之前做些准备工作。

  + doInBackground

    在线程池中执行，执行异步任务；params参数是异步任务的输入参数，另外这个方法需要返回计算结果给onPostExecute方法。这个方法中可以通过publishProgress方法来更新任务进度，publishProgress方法会调用onProgressUpdate方法。

  + onProgressUpdate 

    在主线程中执行，当后台任务的执行进度发生改变时会调用它。

  + onPostExecute 

    主线程中执行，异步任务执行之后会调用这个方法，result参数是后台任务的返回值。
    
    当异步任务被取消时，onCancelled()方法会被调用，这个时候onPostExecute则不会被调用。

  ```java
  public abstract class AsyncTask<Params, Progress, Result> {
      //省略。。。
      
      protected void onPreExecute() {
      }
      
      protected abstract Result doInBackground(Params... params);
      
      protected void onProgressUpdate(Progress... values) {
      }
      
      protected void onPostExecute(Result result) {
      }
      
  }
  ```

+ 使用限制

  + AsyncTask的类必须在==主线程中加载==
  + AsyncTask对象必须在==主线程中创建==
  + execute方法必须在UI线程调用
  + 程序中别直接调用onPreExecute、doInBackground、onProgressUpdate、onPostExecute方法。
  + 一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常。
  + Android1.6之前，AsyncTask是串行执行任务的，Android1.6的时候AsyncTask开始采用线程池处理并行任务，但是从Android3.0开始，为了避免AsyncTask带来的并发错误，AsyncTask又采用了一个线程来串行执行任务。尽管如此，在Android3.0之后的版本中，我们仍然可以通过executeOnExecutor方法来并行执行任务。

+ AsyncTask的工作原理

  根据下面的源码分析，可以了解到Android3.0之后默认情况下AsyncTask是**串行执行**的。

  为了让AsyncTask可以在Android3.0以上的版本中并行执行，可以采用**executeOnExecutor**方法。

  ```java
  public abstract class AsyncTask<Params, Progress, Result> {
      
      //省略。。。
      
      public final AsyncTask<Params, Progress, Result> execute(Params... params) {
          return executeOnExecutor(sDefaultExecutor, params);
          //sDefaultExecutor实际上是一个串行线程池
          //一个进程中所有的AsyncTask全部在这个串行的线程池中排队执行
      }    
      
      public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
              Params... params) {
          if (mStatus != Status.PENDING) {
              switch (mStatus) {
                  case RUNNING:
                      throw new IllegalStateException("Cannot execute task:"
                              + " the task is already running.");
                  case FINISHED:
                      throw new IllegalStateException("Cannot execute task:"
                              + " the task has already been executed "
                              + "(a task can be executed only once)");
              }
          }
  
          mStatus = Status.RUNNING;
  
          onPreExecute();//最先执行
  
          mWorker.mParams = params;
          exec.execute(mFuture);//然后线程池开始执行
          /*
          把params参数封装为FutureTask对象，在这里充当Runable的作用，
          交给SerialExecutor的execute方法去处理。
          */
  
          return this;
      }
      
      public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
      private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
      
      //从SerialExecutor的实现可以分析AsyncTask的排队执行的过程。
      private static class SerialExecutor implements Executor {
          final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
          Runnable mActive;
  
          /*
          execute方法会先把FutureTask对象插入到任务队列mTasks中，
          如果这个时候没有正在活动的AsyncTask任务，那么就会调用SerialExecutor#scheduleNext方法
          来执行下一个AsyncTask任务。同时当一个AsyncTask任务执行完后，AsyncTask会继续执行其他任务
          直到所有任务都被执行为止。
          从这里可以看出，默认情况下AsyncTask都是串行执行的。
          */
          public synchronized void execute(final Runnable r) {
              mTasks.offer(new Runnable() {
                  public void run() {
                      try {
                          r.run();
                      } finally {
                          scheduleNext();
                      }
                  }
              });
              if (mActive == null) {
                  scheduleNext();
              }
          }
  
          protected synchronized void scheduleNext() {
              if ((mActive = mTasks.poll()) != null) {
                  THREAD_POOL_EXECUTOR.execute(mActive);//真正执行
              }
          }
      }
      
      
      /*
      AsyncTask有两个线程池（上面的SerialExecutor和THREAD_POOL_EXECUTOR）
      和一个Handler(InternalHandler)，
      其中SerialExecutor用于任务的排队，而线程池THREAD_POOL_EXECUTOR用于任务真正执行，
      InternalHandler用于将执行环境从线程池切换到主线程。
      */
      public static final Executor THREAD_POOL_EXECUTOR;
  
      static {
          ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                  CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                  sPoolWorkQueue, sThreadFactory);
          threadPoolExecutor.allowCoreThreadTimeOut(true);
          THREAD_POOL_EXECUTOR = threadPoolExecutor;
      }
      
      //其中一个构造方法
      public AsyncTask(@Nullable Looper callbackLooper) {
          //mHandler
          mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
              ? getMainHandler()
              : new Handler(callbackLooper);
  
          /*
          FutureTask的run方法会调用mWorker的call方法，
          因此mWorker的call方法最终会在线程池中执行。
          */
          mWorker = new WorkerRunnable<Params, Result>() {
              public Result call() throws Exception {
                  mTaskInvoked.set(true);//设置为true，表示当前任务已经被调用过了
                  Result result = null;
                  try {
                      Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                      //noinspection unchecked
                      result = doInBackground(mParams);//调用doInBackground
                      Binder.flushPendingCommands();
                  } catch (Throwable tr) {
                      mCancelled.set(true);
                      throw tr;
                  } finally {
                      postResult(result);//把doInBackground的返回值传给postResult方法
                      //postResult如下面
                  }
                  return result;
              }
          };
  
          mFuture = new FutureTask<Result>(mWorker) {
              @Override
              protected void done() {
                  try {
                      postResultIfNotInvoked(get());
                  } catch (InterruptedException e) {
                      android.util.Log.w(LOG_TAG, e);
                  } catch (ExecutionException e) {
                      throw new RuntimeException("An error occurred while executing doInBackground()",
                              e.getCause());
                  } catch (CancellationException e) {
                      postResultIfNotInvoked(null);
                  }
              }
          };
      }
      
      private Result postResult(Result result) {
          @SuppressWarnings("unchecked")
          Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                  new AsyncTaskResult<Result>(this, result));
          message.sendToTarget();
          //会通过mHandler发送一个MESSAGE_POST_RESULT消息
          return result;
      }
      
      private Handler getHandler() {
          return mHandler;
          //从上面构造方法梳理，mHandler返回的可能是sHandler
      }
      
      /*
      可见sHandler是一个静态的Handler对象，
      为了能够将执行环境切换到主线程，就要求sHandler这个对象必须在主线程中创建。
      由于静态成员会在加载类的时候进行初始化，因此就变相要求AsyncTask类必须在主线程中加载，
      否则同一个进程中的AsyncTask都将无法正常工作。
      sHandler收到MESSAGE_POST_RESULT消息后会调用AsyncTask的finish方法（如下所示）。
      */
      private static InternalHandler sHandler;
      
      private static Handler getMainHandler() {
          synchronized (AsyncTask.class) {
              if (sHandler == null) {
                  sHandler = new InternalHandler(Looper.getMainLooper());
              }
              return sHandler;
          }
      }
      
      private static class InternalHandler extends Handler {
          public InternalHandler(Looper looper) {
              super(looper);
          }
  
          @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
          @Override
          public void handleMessage(Message msg) {
              AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
              switch (msg.what) {
                  case MESSAGE_POST_RESULT:
                      // There is only one result
                      result.mTask.finish(result.mData[0]);
                      break;
                  case MESSAGE_POST_PROGRESS:
                      result.mTask.onProgressUpdate(result.mData);
                      break;
              }
          }
      }
      
      
      private void finish(Result result) {
          if (isCancelled()) {
              //如果被取消执行，就调用onCancelled方法
              onCancelled(result);
          } else {
              //否则调用onPostExecute方法
              onPostExecute(result);
          }
          mStatus = Status.FINISHED;
      }
  }
  ```

  AsyncTask工作原理总结：
  
  通常使用 AsyncTask 是创建 AsyncTask 对象后执行 execute 方法，创建 AsyncTask 对象时会同时创建一个 **WorkerRunnable** 对象，并以 WorkerRunnable 对象为参数创建一个 **FutureTask** 对象， AsyncTask 的工作原理从 execute 方法开始，execute 方法首先会执行 executeOnExecutor 方法，并传入一个 SerialExecutor 类型的对象，**SerialExecutor 是一个串行线程池，一个线程里面的所有 AsyncTask 全部都在这个串行的线程池中排队执行**，executeOnExecutor 中先会执行 onPreExecute 方法，这是在我们创建 AsyncTask 对象的时候自己实现的， 运行在主线程中，可以在这个方法里面进行任务开始的提示性操作，接着**线程池开始执行，也就是从这一步开始切换到了子线程中**，传入的对象就是我们创建 AsyncTask 对象的时候生成的 FutureTask 对象，在 SerialExecutor 线程池的 execute 方法中会先把当前 FutureTask 对象插入到任务队列中，如果当前任务队列中没有正在活动的 AsyncTask 任务的话，就会执行 scheduleNext 方法从队列中取得一个 AsyncTask 任务，同时当一个 AsyncTask任务执行结束之后会在 finally 块中调用 scheduleNext 方法执行任务队列中的下一个 AsyncTask 任务，从这里也看出来默认情况下 AsyncTask 是串行执行的，而真正的执行操作就是在 scheduleNext 方法里面使用线程池 THREAD_POOL_EXECUTOR 执行任务，前面的 SerialExecutor 线程池主要是用来任务排队的，保证默认情况下的串行执行而已，而 THREAD_POOL_EXECUTOR 才是真正的任务执行者，此外在 AsyncTask 里面还有一个 InternalHandler 对象，其实他就是一个 Handler 对象而已，在WorkerRunnable 的 call 方法中调用到的 postResult 方法中会使用到，作用就是为了从子线程切换到主线程，便于在子线程执行的过程中进行一些与界面元素的交互过程，比如下载进度条的更新等，那也就必须要求 InternalHandler 对象在主线程创建了，看源码发现 InternalHandler 对象是 static 的，由于静态成员会在加载类的时候进行初始化，因此就变相要求AsyncTask类必须在主线程中加载，要保证 AsyncTask 对象在主线程中创建，而执行任务时是 THREAD_POOL_EXECUTOR 执行 execute 方法，内部实际上执行的是 FutureTask 的 run 方法，而 FutureTask 的 run 方法实际上执行的是创建 FutureTask 对象时传入参数WorkerRunnable对象的 call 方法，call 方法内可以看到执行了 doInBackground 方法，该方法也是需要我们在创建 AsyncTask 对象的时候实现的，可在这个方法里面执行一些耗时操作， 它运行在子线程中，在该方法中我们可以通过 publishProgress 发送耗时任务处理的进度信息，该方法运行在子线程中，内部会通过 InternalHandler 将进度消息发送出去，在 InternalHandler 的 handleMessage 会发现是通过 onProgressUpdate 进行消息处理的，该方法运行在主线程中，可以进行更新进度条的一些操作，在 doInBackground 方法执行结束后会将返回结果作为参数传递给 postResult 方法， 它同样会通过 InternalHandler 发送消息，后在InternalHandler 里面的 handleMessage 里面处理该消息，调用 finish 方法，也就是切换到了主线程中了，在 finish 方法中会根据主线程有没有被暂停来执行 onCancelled 或者 onPostExecute 方法，这两个方法是运行在主线程的，到这里 AsyncTask 的执行结束了。

#### HandlerThread

HandlerThread继承于Thread，是一个具有消息循环的线程，在它的内部可以使用Handler。 

它的实现，就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际的使用中就允许HandlerThread中创建Handler了。

与普通Thread不同的是，普通Thread主要用于在run方法中执行一个耗时任务，而HandlerThread在内部创建了消息队列，外界需要通过Handler的消息方式来通知HandlerThread执行一个具体任务。它在Android中**一个具体的使用场景就是IntentService**。另外，由于HandlerThread的run方法是一个无限循环，因此当明确不需要使用HandlerThread的时候，可以通过quit或者quitSafely方法来终止线程的执行。

  ```java
  public class HandlerThread extends Thread {
      //省略。。。
      
      @Override
      public void run() {
          mTid = Process.myTid();
          Looper.prepare();
          synchronized (this) {
              mLooper = Looper.myLooper();
              notifyAll();
          }
          Process.setThreadPriority(mPriority);
          onLooperPrepared();
          Looper.loop();
          mTid = -1;
      }
  }
  ```

#### IntentService

IntentService是一种特殊的Service，继承了Service并且是一个抽象类。内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出。从任务执行角度看，像是一个后台线程，但是**IntentService是一种Service，优先级比单纯的线程高许多，不容易被系统杀死从而可以尽量保证任务的执行**，而如果是一个后台线程，由于这个时候进程中没有活动的四大组件，那么这个线程的优先级会非常低，会很容易被系统杀死，这就是IntentService的优点。

```java
public abstract class IntentService extends Service {
    //省略。。。
    
    //IntentService第一次被启动的时候onCreate方法会被调用
    @Override
    public void onCreate() {
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();
        //会使用HandlerThread的Looper创建一个Handler对象mServiceHandler
        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
        /*
        这样，通过mServiceHandler发送的消息都会在HandlerThread中执行，
        从这个角度上看，IntentService也可以用于执行后台任务。
        */
    }

    //每次启动IntentService，它的onStartCommand方法就会调用一次，
    //onStartCommand中处理每个后台任务的Intent。
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);//调用onStart方法
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
    
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
        /*
        也就是，IntentService仅仅通过mServiceHandler发送一个消息，这个消息会在HandlerThread中被处理。
        mServiceHandler收到消息后，会把Intent对象传递给onHandlerIntent方法去处理
        */
    }
    
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);//会等待所有消息都处理完毕后才终止服务
        }
    }
    
    /*
    要我们在子类中实现
    它的作用是从Intent参数中区分具体任务并执行这些任务。
    */
    protected abstract void onHandleIntent(@Nullable Intent intent);
    
    /*
    另外，由于每执行一个后台任务就必须启动一次IntentService，
    而IntentService内部则通过消息的方式向HandlerThread请求执行任务，
    Handler中的Looper是顺序处理消息的，这就意味着IntentService也是顺序执行后台任务的，
    当有多个后台任务同时存在时，这些后台任务会按照外界发起的顺序排队执行。
    */
}
```

IntentService原理总结：

HandlerThread 其实就是在子线程中减少自己创建 Looper 以及运转 Looper，而IntentService 其实封装的更巧妙，使用 HandlerThread 的时候我们还需要创建 Handler 对象出来，使用 IntentService 连 Handler 对象也不用创建了。 IntentService 是一个抽象类，因而在使用的时候需要创建一个实现类，里面仅有一个抽象方法是 onHandleIntent ，可以在这个方法里面做一些处理 Intent 的操作了，作为特殊Service，IntentService 自然也是在后台执行的，也是通过 startService 启动，他的优先级要高于一般的线程，那么 IntentService 有什么用处呢？适合于执行一些高优先级的后台耗时任务，高优先级的后台任务是 Service 的特点，但由于 Service 是处于主线程的，他不适合处理耗时任务，但 IntentService 却可以，原因在于 IntentService 在创建的时候就会开启一个线程出来，耗时任务是在子线程中进行的，具体说这里的线程其实就是HandlerThread，在耗时任务处理结束之后该Service 会自动停止；IntentService 封装了 Handler 来处理耗时任务的，在构造方法里面会看到创建了 一个 HandlerThread 线程出来，并且调用了他的start方法启动线程，HandlerThread中会在该线程的run方法里创建 Looper 对象并且调用 loop 将 Looper 运转起来，接着会通过创建的 Looper 对象创建 一个 ServiceHandler 出来，其实就是 Handler 对象而已，该对象里面有 handleMessage 方法，在我们通过 startService 方法启动 IntentService 的时候会回调 onStartCommand 方法，该方法会执行 IntentService 的 onStart 方法，而正是在 onStart 方法里面会将我们 startService 传入的 intent 对象封装成 Message 对象通过在构造函数中创建的 ServiceHandler 类型 handler 对象的 sendMessage 方法发送出去，紧接着就会回调 ServiceHandler 的 handleMessage 方法，里面实际上执行的是 onHandleIntent 方法，也就是我们在实现 IntentService 抽象类的时候需要实现的方法， 具体实现对 Intent 的操作，操作结束之后 handleMessage 方法会执行 stopSelf 方法结束当前 IntentService。 



### **Android中的线程池**

+ ThreadPoolExecutor
+ 线程池的分类