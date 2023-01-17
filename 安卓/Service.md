## Service

**Service的应用场景，以及和Thread区别**

**开启Service的两种方式以及区别**



#### **Service基础**

+ **Service是什么？**

  Service（服务）是一个**可以在后台长时间运行而没有用户界面**的应用组件。可以由其他的应用组件比如Activity、Broadcast启动，Service一旦被启动后就会一直在后台运行，即使启动它的Activity或者Broadcast都已经被销毁Service也不会受到影响。另外还可以把Service绑定到Activity，可以让Service和Activity进行数据交互，甚至Service和Activity可以是在**不同的进程**，这时通过进程间通信来进行数据传输。其中Service和广播Broadcast都**运行在主线程**当中，内部都（默认）**不能做耗时操作**！

+ **Service和Thread的区别**

  首先，Service和Thread没有关系，是两个概念。

  + 定义

    Thread是程序执行的最小单元，是分配CPU的基本单位，可以用来执行一些异步操作等等；而Service它是**Android当中的一种机制**，当它运行的时候，如果是本地服务Local Service那么就是**运行在主线程上**，绝对**不能做耗时操作**，也就是说Thread的运行是相对独立的，而Service的运行是依托于它所在的主线程上，相比于Thread，Service的独立性没有那么强了。其实**两者没有什么关系**，我们不能把后台和子线程的概念搞混，服务和后台也是不同的概念，Android当中的后台指的是它的运行不依赖于UI，即使Activity被销毁了或者程序关闭了，这个服务进程仍然存在，它会一直在后台进行一些计算啊或者数据统计相关的内容。如果一定要在Service里面添加耗时操作也一定要**开启子线程去处理耗时操作**。

    那为什么既然Service处理耗时操作需要开启线程为什么不直接在Activity当中开启子线程呢？因为**Activity很难对Thread进行控制**，当Activity被销毁了就很难有办法获取已经开启的子线程的实例的引用！而Service来处理后台任务，Activity就可以很放心地被销毁，不用担心对后台任务的控制了。

  + 实际开发

    在Android当中，线程一般指的是工作线程，也就是后台线程，做一些耗时操作；而主线程只是负责处理一些UI绘制，为了及时响应绝对不能够做耗时操作。

    而Service是Android四大组件之一，一般情况下是运行在主线程当中，不能够做耗时操作，否则系统会报ANR异常，之所以称Service为后台服务就是因为它没有UI界面，用户无法感知，但是它是在后台运行了一些数据，所以说如果一定要在Service当中进行耗时任务就一定要开启子线程。

  + 应用场景
  
    而Service需要长时间在后台运行而且不需要交互，比如在后台播放音乐或者数据统计等等。当一定要处理网络或者IO操作等等耗时操作的时候都应该开启子线程的方式保证UI线程不被占用，不影响用户体验。



**两种启动Service的方式**

+ **startService**

  就是通过Activity调用startService（intent）启动服务，**一旦服务开启就一直在后台运行**，即是启动它的Activity被销毁了也对Service的运行没有影响，除非是手动的关闭这个Service。

  1. 定义一个类继承Service
  2. 在Manifest.xml文件中注册该Service
  3. 使用Context的startService（Intent）方法启动该Service
  4. 不再使用时，调用stopService（Intent）方法停止该服务。

  ```java
  MyService()
  MyService onCreate()//只会在创建Service时调用一次
  MyService onStartCommand()
  MyService onDestroy()
  ```

  

+ **bindService**

  启动Service以后和Activity属于绑定状态，绑定服务后会提供给客户端一个服务端接口，就相当于Activity和Service交互的接口，Activity和Service进行数据交互，发送请求获取结果等等操作，甚至Service和Activity在不同进程中时，可以进行**进程间通信**来传输数据。这些是需要在Service和Activity绑定之后才能使用的，同时，**多个Activity可以绑定同一个Service，绑定全部取消以后，服务就会自动被销毁**，并不一定要像startService的方式一样使用stopService才能被销毁。

  1. 创建BindService服务端，继承自Service并在类中创建**一个实现IBinder接口的实例对象**,它可以提供公共方法给客户端调用。
  2. 从onBind（）回调方法返回这个IBinder实例。
  3. 然后在客户端绑定服务bindService(intent, connection, Context.BIND_AUTO_CREATE)，绑定成功后就可以从ServiceConnection#onServiceConnected（）回调方法接收Binder对象，**通过这个Binder对象就可以使用Service当中提供的方法**了。
  
  ```java
  MyService()
  MyService onCreate()
  MyService onBind() //返回实现IBinder接口对象的实例
  ServiceConnection onServiceConnected()
  MyService onUnbind()
  MyService onDestroy()
  ```
  
  



**Service回调方法**

onBind（）在bindService时才会被调用。

onCreate（）Service首次创建，在调用onStartCommand或者onBind之前，会被调用一次，如果已经运行了就不会再被调用了。

onStartCommand（）通过startService启动Service的时候会回调，执行了这个方法，Service就正式开启了。

onDestory（）Service被销毁时回调，可以进行一些资源的清理，比如说子线程或者是监听器进行回收等等。