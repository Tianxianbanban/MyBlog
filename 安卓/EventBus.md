## EventBus



**事件总线设计**

为了简化并且更加高质量地在Activity、Fragment、Thread和Service等之间通信，同时解决组件之间高耦合的同时仍然继续高效通信。

**EventBus**

一款针对Android优化的发布—订阅事件总线。简化了各组件间、组件与后台线程间的通信。开销更小，代码更加优雅，以及将发送者和订阅者解耦。



#### **基本使用**

+ 添加依赖

+ **注册（同时记得解注册）**并且**订阅**事件

+ 自定义一个**事件类**，就是构造发送消息的类

+ **发送**事件

+ 在订阅事件的地方处理事件。如果在内部**取消事件分发**，后面的方法就接收不到这个消息了，因为一个事件可能被不只一个方法订阅了。如果是**黏性事件**（ 不管是在事件发送之前注册的事件接收者还是在事件发送之后注册的事件接收者都能够收到事件 ），可以在发送事件之后再订阅也能接收到该事件，与黏性广播类似。

+ 关于混淆规则：https://greenrobot.org/eventbus/documentation/proguard/

  

#### **EventBus三要素和四种ThreadMode**

+ Event：事件。可以是任意类型的对象
+ Subscriber：事件订阅者。在EventBus3.0之后，事件处理的方法可以任意取名，只需要添加一个**注解**`@Subscriber`，并且指定**线程模型**（默认为POSTING）。
+ Publisher：事件发布者。可以自己实例化EventBus对象，但是一般使用EventBus.getDefault()就可。根据post函数参数类型，自动调用订阅相应类型事件的函数。
+ ThreadMode：线程模型， 指事件订阅者所在线程和发布事件者所在线程的关系 
  + POSTING（默认）：**事件在哪个线程发布出来的，事件处理函数就会在哪个线程中运行**，发布事件和接收事件在同一个线程中。在这个线程模型的事件处理函数中尽量避免执行耗时操作，因为会阻塞事件传递，甚至引起ANR。
  + MAIN：事件的处理会**在UI线程中执行**。事件处理的时间不能太长，长了会导致ANR。
  + BACKGROUND：如果事件在UI线程中发出，事件处理函数就会在新的线程中运行；如果事件本来就是子线程中发布的，事件处理函数就直接在发布事件的线程中执行。此**事件处理函数中禁止UI操作**。
  + ASYNC：无论事件在哪个线程中发布，事件处理函数都会**在新建的线程中**执行，同样禁止进行UI操作。



#### **源码分析**

+ 构造方法

  获取EventBus实例，会调用EventBus.getDefault()方法， 是一个**单例模式**，采用了**双重检查**模式 （DCL） 

  ```java
  static volatile EventBus defaultInstance;
  
  public static EventBus getDefault() {
          EventBus instance = defaultInstance;
          if (instance == null) {
              synchronized (EventBus.class) {
                  instance = EventBus.defaultInstance;
                  if (instance == null) {
                      instance = EventBus.defaultInstance = new EventBus();
                  }
              }
          }
          return instance;
      }
  ```

  在构造方法中，构造一个EventBusBuilder在构造方法中对EventBus进行配置，这里采用了**建造者模式**。

  ```java
  public EventBus() {
          this(DEFAULT_BUILDER);//DEFAULT_BUILDER是默认的EventBusBuilder，用来构造EventBus
      }
  
  //上面会调用另外一个构造方法
  //通过构造一个EventBusBuilder在构造方法中对EventBus进行配置，这里采用了建造者模式。
  EventBus(EventBusBuilder builder) {
          logger = builder.getLogger();
          subscriptionsByEventType = new HashMap<>();
          typesBySubscriber = new HashMap<>();
          stickyEvents = new ConcurrentHashMap<>();
          mainThreadSupport = builder.getMainThreadSupport();
          mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
          backgroundPoster = new BackgroundPoster(this);
          asyncPoster = new AsyncPoster(this);
          indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
          subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
                  builder.strictMethodVerification, builder.ignoreGeneratedIndex);
          logSubscriberExceptions = builder.logSubscriberExceptions;
          logNoSubscriberMessages = builder.logNoSubscriberMessages;
          sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
          sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
          throwSubscriberException = builder.throwSubscriberException;
          eventInheritance = builder.eventInheritance;
          executorService = builder.executorService;
      }
  ```

  

+ 订阅者注册

  ```java
  //获取EventBus后，便可以将订阅者注册到EventBus中
  EventBus.getDefault().register(EventBusSendActivity.this);
  
  public void register(Object subscriber) {
          Class<?> subscriberClass = subscriber.getClass();
      //找出订阅者的所有订阅方法
          List<SubscriberMethod> subscriberMethods = 															subscriberMethodFinder.findSubscriberMethods(subscriberClass);
          synchronized (this) {
              for (SubscriberMethod subscriberMethod : subscriberMethods) {
                  subscribe(subscriber, subscriberMethod);
              }
          }
      }
  ```

  + 查找订阅者的订阅方法

    SubscriberMethod类中，主要用来**保存订阅方法的Method对象、线程模式、事件类型、优先级、是否是黏性事件等属性**。 findSubscriberMethods方法找出一个SubscriberMethod的集合，也就是传进来的订阅者的所有订阅方法，接下来遍历订阅者的订阅方法来完成订阅者的注册操作。  

    ```java
    private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();
    
    List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        //从缓存中查找是否有订阅方法的集合，如果找到了就立马返回。如果缓存中没有，则
        //根据 ignoreGeneratedIndex属性的值来选择采用何种方法来查找订阅方法的集合。
            List<SubscriberMethod> subscriberMethods = 																			METHOD_CACHE.get(subscriberClass);
            if (subscriberMethods != null) {
                return subscriberMethods;
            }
    
            if (ignoreGeneratedIndex) {
                subscriberMethods = findUsingReflection(subscriberClass);
            } else {
                subscriberMethods = findUsingInfo(subscriberClass);
            }
            if (subscriberMethods.isEmpty()) {
                throw new EventBusException("Subscriber " + subscriberClass
                        + " and its super classes have no public methods with the 										@Subscribe annotation");
            } else {
                //找到订阅方法的集合后，放入缓存，以免下次继续查找。
                METHOD_CACHE.put(subscriberClass, subscriberMethods);
                return subscriberMethods;
            }
        }
    
    
    
    private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
            FindState findState = prepareFindState();
            findState.initForSubscriber(subscriberClass);
            while (findState.clazz != null) {
                findState.subscriberInfo = getSubscriberInfo(findState);
                if (findState.subscriberInfo != null) {
                    SubscriberMethod[] array = 																		findState.subscriberInfo.getSubscriberMethods();
                    for (SubscriberMethod subscriberMethod : array) {
                        if (findState.checkAdd(subscriberMethod.method, 																subscriberMethod.eventType)) {
                            findState.subscriberMethods.add(subscriberMethod);
                        }
                    }
                } else {
                    findUsingReflectionInSingleClass(findState);
                }
                findState.moveToSuperclass();
            }
            return getMethodsAndRelease(findState);
        }
    ```

  + 订阅者的注册过程

    在查找完订阅者的订阅方法以后便开始对所有的订阅方法进行注册。 EventBus.getDefault().register(EventBusSendActivity.this)中 register方法中会遍历订阅对象当中的方法，依次调用了subscribe方法来对订阅者和订阅方法进行注册 。 并且在 register方法当中对黏性事件处理，在这里将黏性事件发送给订阅者。

    `Map<Class<?>, CopyOnWriteArrayList<Subscription>>`  subscriptionsByEventType 是保存eventType到subscriptions的映射，也就是**事件类型到<订阅者&订阅方法=订阅对象>们的映射**；

    `Map<Object, List<Class<?>>` typesBySubscriber是保存了订阅者subscriber到subscribedEvents的映射，也就是**订阅者到它所涉及的事件类型们的映射**；

    在subscribe方法当中：（1）首先是将**订阅者subscriber和订阅方法subscriberMethod构成的Subscription订阅对象**根据eventType封装到subscriptionsByEventType 中；（2）然后是将订阅方法的事件类型eventType添加进typesBySubscriber中，将subscribedEvents根据subscriber封装到typesBySubscriber中；（3）再是处理黏性事件。

    ```java
    // Must be called in synchronized block
        private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
            Class<?> eventType = subscriberMethod.eventType;
            //根据subscriber和subscriberMethod创建一Subscription（订阅对象）
            //订阅者和订阅方法形成订阅对象
            Subscription newSubscription = new Subscription(subscriber, 																			subscriberMethod);
            CopyOnWriteArrayList<Subscription> subscriptions = 																subscriptionsByEventType.get(eventType);
            if (subscriptions == null) {
                subscriptions = new CopyOnWriteArrayList<>();
                subscriptionsByEventType.put(eventType, subscriptions);
            } else {
                if (subscriptions.contains(newSubscription)) {
                    throw new EventBusException("Subscriber " + subscriber.getClass() 								+ " already registered to event "+ eventType);
                }
            }
            //上面就是用了一个Map，Map<Class<?>, CopyOnWriteArrayList<Subscription>>存储了
            //eventType和subscriptions的映射关系，
            //如果subscriptions不在map当中就要添加进去，重复添加报异常
    
            
            //按照订阅方法的优先级插入到订阅对象集合中，完成订阅方法的注册。
            int size = subscriptions.size();
            for (int i = 0; i <= size; i++) {
                if (i == size || subscriberMethod.priority > 														subscriptions.get(i).subscriberMethod.priority) {
                    subscriptions.add(i, newSubscription);
                    break;
                }
            }
    
            List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
            if (subscribedEvents == null) {
                subscribedEvents = new ArrayList<>();
                typesBySubscriber.put(subscriber, subscribedEvents);
            }
            subscribedEvents.add(eventType);
    
            //Map<Class<?>, Object> stickyEvents
            //黏性事件，从stickyEvents事件保存队列中取出该事件类型的事件发送给当前订阅者
            if (subscriberMethod.sticky) {
                if (eventInheritance) {
                    // Existing sticky events of all subclasses of eventType have to be considered.
                    // Note: Iterating over all events may be inefficient with lots of sticky events,
                    // thus data structure should be changed to allow a more efficient lookup
                    // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                    Set<Map.Entry<Class<?>, Object>> entries = 																				stickyEvents.entrySet();
                    for (Map.Entry<Class<?>, Object> entry : entries) {
                        Class<?> candidateEventType = entry.getKey();
                        if (eventType.isAssignableFrom(candidateEventType)) {
                            Object stickyEvent = entry.getValue();
                            checkPostStickyEventToSubscription(newSubscription, 																		stickyEvent);
                        }
                    }
                } else {
                    Object stickyEvent = stickyEvents.get(eventType);
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        }
    ```
  
+ 事件的发送

  EventBus.getDefault().post(new MessageEvent("数据"))， 获取EventBus对象以后，可以通过post方法来进行对事件的提交 。

  ```java
  /** Posts the given event to the event bus. */
      public void post(Object event) {
          //保存着事件队列和线程状态信息
          //ThreadLocal<PostingThreadState> currentPostingThreadState
          PostingThreadState postingState = currentPostingThreadState.get();
          List<Object> eventQueue = postingState.eventQueue;
          eventQueue.add(event);
          //首先从PostingThreadState对象中取出事件队列，然后再将当前的事件插入事件队列。
  
          if (!postingState.isPosting) {
              postingState.isMainThread = isMainThread();
              postingState.isPosting = true;
              if (postingState.canceled) {
                  throw new EventBusException("Internal error. Abort state was not 																				reset");
              }
              try {
                  while (!eventQueue.isEmpty()) {
                 		//将队列中的事件依次交由 postSingleEvent 方法进行处理，并移除该事件
                      postSingleEvent(eventQueue.remove(0), postingState);
                  }
              } finally {
                  postingState.isPosting = false;
                  postingState.isMainThread = false;
              }
          }
      }
  
  
  ```

  postSingleEvent(eventQueue.remove(0), postingState)方法中eventInheritance 表示是否向上查找事件的父类，它的默认值为 true，可以通过在EventBusBuilder中进行 配置。当eventInheritance为true时，则通过lookupAllEventTypes找到所有的父类事件并存在List中，然后通过 postSingleEventForEventType方法对事件逐一处理。 postSingleEventForEventType方法

  ```java
   private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
          Class<?> eventClass = event.getClass();
          boolean subscriptionFound = false;
       //eventInheritance 表示是否向上查找事件的父类，它的默认值为 true，
       //可以通过在EventBusBuilder中进行配置
          if (eventInheritance) {
              //当eventInheritance为true时，
              //则通过lookupAllEventTypes找到所有的父类事件并存在List中，
              //然后通过 postSingleEventForEventType方法对事件逐一处理。
              List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
              int countTypes = eventTypes.size();
              for (int h = 0; h < countTypes; h++) {
                  Class<?> clazz = eventTypes.get(h);
                  subscriptionFound |= postSingleEventForEventType(event, postingState, 																				clazz);
              }
          } else {
              subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
          }
          if (!subscriptionFound) {
              if (logNoSubscriberMessages) {
                  logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
              }
              if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                      eventClass != SubscriberExceptionEvent.class) {
                  post(new NoSubscriberEvent(this, event));
              }
          }
      }
  ```

  ```java
  private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
          CopyOnWriteArrayList<Subscription> subscriptions;
          synchronized (this) {
              subscriptions = subscriptionsByEventType.get(eventClass);
          }
          if (subscriptions != null && !subscriptions.isEmpty()) {
              //遍历Subscriptions，将事件 event 和对应的 Subscription（订阅对象）传递给 
              //postingState并调用postToSubscription方法对事件进行处理
              for (Subscription subscription : subscriptions) {
                  postingState.event = event;
                  postingState.subscription = subscription;
                  boolean aborted;
                  try {
                      postToSubscription(subscription, event, 																				postingState.isMainThread);
                      aborted = postingState.canceled;
                  } finally {
                      postingState.event = null;
                      postingState.subscription = null;
                      postingState.canceled = false;
                  }
                  if (aborted) {
                      break;
                  }
              }
              return true;
          }
          return false;
      }
  ```

  ```java
  private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
          switch (subscription.subscriberMethod.threadMode) {
              case POSTING:
                  invokeSubscriber(subscription, event);
                  break;
              case MAIN:
                  //如果threadMode是 MAIN，若提交事件的线程是主线程，
                  //则通过反射直接运行订阅的方法
                  if (isMainThread) {
                      invokeSubscriber(subscription, event);
                  } else {
                      //否则需要mainThreadPoster 将我们的订阅事件添加到主线程队列中
                      //mainThreadPoster 是HandlerPoster类型的，
                      //继承 自Handler，通过Handler将订阅方法切换到主线程执行
                      mainThreadPoster.enqueue(subscription, event);
                  }
                  break;
              case MAIN_ORDERED:
                  if (mainThreadPoster != null) {
                      mainThreadPoster.enqueue(subscription, event);
                  } else {
                      // temporary: technically not correct as poster not decoupled from subscriber
                      invokeSubscriber(subscription, event);
                  }
                  break;
              case BACKGROUND:
                  if (isMainThread) {
                      backgroundPoster.enqueue(subscription, event);
                  } else {
                      invokeSubscriber(subscription, event);
                  }
                  break;
              case ASYNC:
                  asyncPoster.enqueue(subscription, event);
                  break;
              default:
                  throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
          }
      }
  ```

  postToSubscription方法中，**取出订阅方法的threadMode（线程模型），之后根据threadMode来分别处理**。如果threadMode是 MAIN，若提交事件的线程是主线程，则通过反射直接运行订阅的方法；若其不是主线程，则需要 mainThreadPoster 将我们的订阅事件添加到主线程队列中。mainThreadPoster 是HandlerPoster类型的，继承 自Handler，通过Handler将订阅方法切换到主线程执行。 

+ 订阅者取消注册

  EventBus.getDefault().unregister(this) 

  ```java
  /** Unregisters the given subscriber from all event classes. */
      public synchronized void unregister(Object subscriber) {
          //通过subscriber找到subscribedTypes（事件类型集合）
          List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
          if (subscribedTypes != null) {
              //遍历 subscribedTypes，并调用 unsubscribeByEventType方
              for (Class<?> eventType : subscribedTypes) {
                  unsubscribeByEventType(subscriber, eventType);
              }
              //将subscriber对应的eventType从 typesBySubscriber中移除
              typesBySubscriber.remove(subscriber);
          } else {
              logger.log(Level.WARNING, "Subscriber to unregister was not registered 													before: " + subscriber.getClass());
          }
      }
  ```

  ```java
  /** Only updates subscriptionsByEventType, not typesBySubscriber! Caller must update typesBySubscriber. */
      private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
          //通过eventType来得到对应的Subscriptions（订阅对象集合），
          //并在for循环中判断如果Subscription （订阅对象）的subscriber（订阅者）属性等于
          //传进来的subscriber，则从Subscriptions中移除该Subscription。
          List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
          if (subscriptions != null) {
              int size = subscriptions.size();
              for (int i = 0; i < size; i++) {
                  Subscription subscription = subscriptions.get(i);
                  if (subscription.subscriber == subscriber) {
                      subscription.active = false;
                      subscriptions.remove(i);
                      i--;
                      size--;
                  }
              }
          }
      }
  ```



**使用优点**

+  使用发布者/订阅者模式进行松散耦合。`EventBus`使中央通信只需几行代码即可解耦类，从而简化了代码，消除了依赖关系并加快了应用程序的开发。 
+  组件之间的通信简化
+ 事件的发送者和接受者分离
+ 避免了因为复杂且容易出错的依赖性和生命周期造成的问题
+ 轻量



**使用注意点**

过多EventBus的代码会造成代码结构混乱，难以测试和追踪，违背了解耦的初衷。 




**涉及的设计模式**

+ 单例模式
+ 建造者模式



**涉及的集合类**

```
ConcurrentHashMap
CopyOnWriteArrayList
```