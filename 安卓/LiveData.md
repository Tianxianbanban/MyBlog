## LiveData



#### 文章

+ [官方-LiveData概览](https://developer.android.google.cn/topic/libraries/architecture/livedata)
+ [实习-LiveData数据流设计入门](https://bytedance.feishu.cn/wiki/wikcnrsRSrIt6X1mhcf8QYv27bg)



#### 个人总结



**LiveData解决了什么问题，有什么优势？**

+ LiveData具有**生命周期感知能力**，不需要手动处理生命周期。那么它就可以只更新处于活跃生命周期状态的应用组件的观察者；而如果观察者的生命周期处于不活跃状态，它也不会接受任何LiveData事件。它遵循了**观察者模式**，LiveData通过通知Observer对象中来更新界面。
+ 观察者会绑定到Lifecycle对象，然后可以在关联对象的生命周期进入销毁状态时进行自我清理，因此可以**避免内存泄漏**的发生。
+ **数据始终保持最新状态**。即使组件的生命周期为不活跃状体，也会在进入活跃状态时接收最新数据；以及因为配置更改而重建的组件也会立即接收最新的可用数据。
+ 可以以单例模式去使用LiveData对象。



**LiveData的用法？**

+ 创建LiveData

  通过 LiveData实例来存储某种类型的数据，并且这通常在ViewModel类中完成

  ```kotlin
  class NameViewModel : ViewModel() {
  
      // 创建了一个存储String类型数据的LiveData对象
      val currentName: MutableLiveData<String> by lazy {
          MutableLiveData<String>()
      }
    
      // Rest of the ViewModel...
  }
  ```

+ 观察LiveData

  创建Observer对象，它可以控制当LiveData对象存储的数据更改时会执行的内容。当更新存储在LiveData对象中的值时，会触发所有已注册的Observer。

  ```java
  class MainActivity2 : AppCompatActivity() {
  
      private lateinit var model: NameViewModel
  
      private lateinit var nameTextView: TextView
      private lateinit var button: Button
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main2)
  
          model = ViewModelProviders.of(this).get(NameViewModel::class.java)
  
          nameTextView = findViewById(R.id.content)
          button= findViewById(R.id.bt)
  
          // 创建Observer对象
          val nameObserver = Observer<String> { newName ->
              nameTextView.text = newName
          }
        
        
         //将Observer注册到LiveData上，
          model.currentName.observe(this, nameObserver)
            
            //更新LiveData对象
          button.setOnClickListener {
              val anotherName = "John Doe"
              model.currentName.setValue(anotherName)
          }
      }
  }
  ```

  + observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer)

  + observeForever(@NonNull Observer<? super T> observer)

    使用起来与**LiveData#observe(LifecycleOwner, Observer)**没有太大差别，主要是当LiveData包装的数据发生变化时，无论页面处于什么状态，observeForever()都能收到通知。所以在使用完后，一定要记得调用**removeObserver**方法来停止对LiveData的观察，否则LiveData会一直处于激活状态，你的Activity永远不会被系统自动回收。

+ 更新LiveData

  如上代码的形式，然后会触发所有观察者。

  + setValue(T)

    从主线程更新LiveData对象。

  + postValue(T)

    在子线程中更新LiveData对象。

+ 更多用法

  + LiveData和Room

    Room数据库可以返回LiveData对象进行可观察查询。

  + [将协程和LiveData一起使用](https://developer.android.google.cn/topic/libraries/architecture/coroutines)

  + 扩展LiveData

    多个 Activity、Fragment 和 Service 之间可以共享LiveData对象。

  + 转换LiveData

    + 需要在LiveData对象分派给观察者之前对存储在其中的值进行更改，或者需要根据另一个实例的值返回不同的LiveData实例，就可以使用**Transformations**类。
      + Transformations.map()
      + Transformations.switchMap()
    + Repository
    + MediatorLiveData监听LiveData对象并处理它们发出的事件。

  + 合并多个LiveData源

    仍然可以使用**MediatorLiveData**。
    
  + 事件总线LiveDataBus

    [Android消息总线的演进之路：用LiveDataBus替代RxBus、EventBus](https://tech.meituan.com/2018/07/26/android-livedatabus.html)

    + 不需要调用反注册方法就可以避免内存泄漏，减少代码量。
    + 可观察性和生命周期感知能力，非常适合作为Android通信总线的基础构件。
    + 减小包大小，依赖更少。

  

**LiveData数据流设计**

+ 为什么要设计LiveData数据流

  项目当中涉及的数据其实可以抽象出一个流向，这个流向指的是数据之间相互影响的关系，一个良好的数据流意味着更加清晰数据关系，更加有迹可循的数据变化，并且尽量减少对数据更新的范围，其实这也就是尽可能缩小错误情况发生时影响的范围。

+ 如何设计LiveData数据流

  根据数据的流向，尽可能缩小某个数据源变更时影响的范围，必要的时候可以把表示数据源的结构再进行细分。



**LiveData原理**

![image-20210328202717097](D:\云盘备份\学习总结\安卓\LiveData.assets\image-20210328202717097.png)

