# Jetpack

## [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn)

> ViewModel的一个重要作用就是可以帮助Activity分担一部分工作，它是专门**用于存放与界面相关的数据**的。也就是说，只要是界面上能看得到的数据，它的相关变量都应该存放在ViewModel中，而不是Activity中，这样可以在一定程度上减少Activity中的逻辑。
>
> ViewModel还有一个非常重要的特性。我们都知道，当手机发生横竖屏旋转的时候，Activity会被重新创建，同时存放在Activity中的数据也会丢失。而ViewModel的生命周期和Activity不同，它可以保证在手机屏幕发生旋转的时候不会被重新创建，只有当==Activity退出的时候才会跟着Activity一起销毁==（❓语句不够严谨，手机屏幕旋转的时候，Activity会回调onDestroy，即使重新回调onCreate，ViewMode也确实没有重新创建实例，这里的销毁指的是什么呀，从活动栈出栈吗）。因此，将与界面相关的变量存放在ViewModel当中，这样即使旋转手机屏幕，界面上显示的数据也不会丢失。

### ViewModel使用

#### 创建ViewModel实例

```kotlin
ViewModelProvider(this).get(MainViewModel::class.java)

//ViewModelProvider.java
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) { //ViewModelStoreOwner
    this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
            ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
            : NewInstanceFactory.getInstance());
}
```



#### 传入构造参数创建实例

所有ViewModel的实例都通过`ViewModelProvider`来获取。也可以借助`ViewModelProvider.Factory`。

```kotlin
//Factory
class MainViewModelFactory(private val countReserved: Int): ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return MainViewModel(countReserved) as T
    }
}

//创建实例
viewModel = ViewModelProvider(this, MainViewModelFactory(countReserved)).get(MainViewModel::class.java)
```

> 因为Factory#create()方法的执行时机和Activity的生命周期无关，所以也不会每次调用都创建新的实例。



### ViewModel生命周期

#### **实例化ViewModel不用new？**

ViewModel有独立的生命周期，并且生命周期要长于Activity。是获取ViewModel时传递给ViewModelProvider的Lifecycle决定的。从首次请求ViewModel直到Lifecycle完成并销毁：Activity完成时，或者Fragment分离时。

通常在系统首次调用 Activity 对象的 `onCreate()` 方法时请求 `ViewModel`。系统可能会在 activity 的整个生命周期内多次调用 `onCreate()`，如在旋转设备屏幕时。当前Activity的生命周期不断变化，经历了销毁重建，但是ViewModel的生命周期没有变化。==`ViewModel` 存在的时间范围是从首次请求 `ViewModel` 直到 activity 完成并销毁==(❓)。

==所以如果用new方式去创建ViewModel实例，那么在Activity每次onCreate()时，都创建一个新的实例，当手机屏幕发生旋转时，就无法保留其中的数据了。==

<img src="./images/Jetpack.assets/(null)" alt="img" style="zoom:40%;" />



**为什么使用ViewModelProvider去创建ViewModel实例就能在Activity销毁重建时保留数据？**

**为什么不同Fragment之间都能监听到数据变化，是因为他们创建ViewModel是同一个对象吗？**

不同的Fragment之间如何实现共享数据：

- 两个Fragment可以使用其Activity范围共享ViewModel来处理这种通信。
- 通过以这种方式共享ViewModel，Fragment之间不需要互相了解，Activity也不需要执行任何操作来促进通信。
- 下面梳理可以看出只要ViewModelProvider里上传的是同一个Activity/Fragment，通过owner.getViewModelStore()就能拿到Activity/Fragment中唯一的mViewModelStore，mViewModelStore通过一个hashmap来保存了ViewModel对象，在get方法中如果hashmap里之前就已经保存了要获取的ViewModel就直接返回，如果是第一次调用，就存储hashmap中，后面获取的就是当前这个对象了。

### 对架构的贡献

在“生命周期管理一致性”的基础上，解决了：

+ 把**托管的状态**（ViewModel），与生命周期对齐，不受Activity实例回收的影响。

+ 让Fragment和Activity共享作用域。

### 实现原理

#### 获取ViewModel实例

```Kotlin
ViewModelProvider(this).get(MainViewModel::class.java)
```

```kotlin
//ViewModelProvider.kt
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
    this(owner.getViewModelStore(), 
         owner instanceof HasDefaultViewModelProviderFactory
            ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
            : NewInstanceFactory.getInstance());
}

public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
    mFactory = factory;
    mViewModelStore = store;
}
```

- 创建ViewModelStore对象
- ViewModelStore中通过hashmap保存每个Activity/Fragment中的ViewModel对象，直接通过get方法获取我们需要的ViewModel即可。



##### **ViewModelStore的获取**

<img src="./images/Jetpack.assets/(null)-20221217174915871.(null)" alt="img" style="zoom:40%;" />

<img src="/Users/chenying/Library/Mobile Documents/com~apple~CloudDocs/MyTyporaContent/学习笔记/安卓/images/Jetpack.assets/(null)-20221217174934154.(null)" alt="img" style="zoom:40%;" />

代码梳理如下：

1. 👆🏻上面的ViewModelProvider的构造方法中，会初始化一个mFactory和一个mViewModel对象，mFactory主要用于后面get方法创建ViewModel对象，mViewModelStore主要用于存储当前Activity/Fragment生命周期中所有ViewModel对象。

   1. 如果没有传入Factory对象，会默认创建一个SavedStateViewModelFactory对象，它是ViewModelProvider.KeyedFactory的子类，在它的构造方法中，会定义一个ViewModelProvider.AndroidViewModelFactory对象，用于后面创建ViewModel对象。
   2. 传入的owner，从中获取ViewModelStore，后续getLastNonConfigurationInstance()方法用来获取前一个被销毁的Activity，从中获取ViewModelStore。
   3. 从mViewModelStore中获取ViewModel对象，如果不为空就返回。否则通过Factory创建一个ViewModel，这里会判断mFactory是否为KeyedFactory，是代表系统默认创建的factory创建对象，否则用自定义的Factory创建ViewModel对象，最后将其存储到mViewModelStore中。

2. 其中mFactory的创建过程如下：

   ```Kotlin
   public ViewModelProvider.Factory getDefaultViewModelProviderFactory() {
       if (getApplication() == null) {
           throw new IllegalStateException("Your activity is not yet attached to the "
                   + "Application instance. You can't request ViewModel before onCreate call.");
       }
       if (mDefaultFactory == null) {
           mDefaultFactory = new SavedStateViewModelFactory(//SavedStateViewModelFactory
                   getApplication(),
                   this,
                   getIntent() != null ? getIntent().getExtras() : null);
       }
       return mDefaultFactory;
   }
   ```

   ```Kotlin
   //SavedStateViewModelFactory.kt
   public SavedStateViewModelFactory(@Nullable Application application,
           @NonNull SavedStateRegistryOwner owner,
           @Nullable Bundle defaultArgs) {
       mSavedStateRegistry = owner.getSavedStateRegistry();
       mLifecycle = owner.getLifecycle();
       mDefaultArgs = defaultArgs;
       mApplication = application;
       mFactory = application != null
               ? ViewModelProvider.AndroidViewModelFactory.getInstance(application) //!!!
               : ViewModelProvider.NewInstanceFactory.getInstance();
   }
   ```

3. ViewModelStore的获取过程如下：

   ```Kotlin
   //ComponentActivity.java
   public ViewModelStore getViewModelStore() {
       if (getApplication() == null) {
           throw new IllegalStateException("Your activity is not yet attached to the "
                   + "Application instance. You can't request ViewModel before onCreate call.");
       }
       ensureViewModelStore();
       return mViewModelStore;
   }
   
   void ensureViewModelStore() {
       if (mViewModelStore == null) {
           NonConfigurationInstances nc =
                   (NonConfigurationInstances) getLastNonConfigurationInstance();//!!!☀️
           if (nc != null) {
               // Restore the ViewModelStore from NonConfigurationInstances
               mViewModelStore = nc.viewModelStore;//!!!
           }
           if (mViewModelStore == null) {
               mViewModelStore = new ViewModelStore();
           }
       }
   }
   ```

4. get方法获取ViewModel实例

   ```Kotlin
   //ViewModelProvider.java
   public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
       String canonicalName = modelClass.getCanonicalName();
       if (canonicalName == null) {
           throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
       }
       return get(DEFAULT_KEY + ":" + canonicalName, modelClass); //!!!
   }
   
   public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
       ViewModel viewModel = mViewModelStore.get(key); //!!!☀️
   
       if (modelClass.isInstance(viewModel)) {
           if (mFactory instanceof OnRequeryFactory) {
               ((OnRequeryFactory) mFactory).onRequery(viewModel);
           }
           return (T) viewModel; //!!!
       } else {
           //noinspection StatementWithEmptyBody
           if (viewModel != null) {
               // TODO: log a warning.
           }
       }
       if (mFactory instanceof KeyedFactory) {
           viewModel = ((KeyedFactory) mFactory).create(key, modelClass); //!!!
       } else {
           viewModel = mFactory.create(modelClass);
       }
       mViewModelStore.put(key, viewModel); //!!!
       return (T) viewModel;
   }
   ```

5. ViewModelStore里面就是一个HashMap实现吗，如下

   ```Kotlin
   public class ViewModelStore {
   
       private final HashMap<String, ViewModel> mMap = new HashMap<>();
   
       final void put(String key, ViewModel viewModel) {
           ViewModel oldViewModel = mMap.put(key, viewModel);
           if (oldViewModel != null) {
               oldViewModel.onCleared();
           }
       }
   
       final ViewModel get(String key) {
           return mMap.get(key);
       }
   
       Set<String> keys() {
           return new HashSet<>(mMap.keySet());
       }
   
       /**
        *  Clears internal storage and notifies ViewModels that they are no longer used.
        */
       public final void clear() {
           for (ViewModel vm : mMap.values()) {
               vm.clear();
           }
           mMap.clear();
       }
   }
   ```



##### **Activity.NonConfigurationInstances对象是怎么被保存的**

Activity.NonConfigurationInstances能保存ViewModelStore，原因如下：

```Kotlin
//ComponentActivity.java
void ensureViewModelStore() {
    if (mViewModelStore == null) {
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance(); //!!!☀️
        if (nc != null) {
            //从NonConfigurationInstances中恢复ViewModelStore
            // Restore the ViewModelStore from NonConfigurationInstances
            mViewModelStore = nc.viewModelStore; //!!!☀️
        }
        //如果mViewModelStore依然为null，则创建新的ViewModelStore对象
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
    }
}

//Activity.java
public Object getLastNonConfigurationInstance() {
    return mLastNonConfigurationInstances != null
            ? mLastNonConfigurationInstances.activity : null; //mLastNonConfigurationInstances
}
```

推出：当上一个Activity销毁前，如有保存viewModelStore会存在mLastNonConfigurationInstances.activity中，它是一个==NonConfigurationInstances==类型的对象，如何保存ViewModel数据，如下：

<img src="./images/Jetpack.assets/(null)-20221217175246184.(null)" alt="img" style="zoom:40%;" />

从代码上看得的话，NonConfigurationInstances保存mViewModelStore的整个过程：

当Activity因为配置信息发生变化销毁然后重建，调用`ActivityThread#handleRelaunchActivity`方法

```Kotlin
handleRelaunchActivityInner(r, configChanges, tmp.pendingResults, tmp.pendingIntents,
        pendingActions, tmp.startsNotResumed, tmp.overrideConfig, "handleRelaunchActivity");
```

```Kotlin
//ActivityThread.java
private void handleRelaunchActivityInner(ActivityClientRecord r, int configChanges,
        List<ResultInfo> pendingResults, List<ReferrerIntent> pendingIntents,
        PendingTransactionActions pendingActions, boolean startsNotResumed,
        Configuration overrideConfig, String reason) {
        //...
        handleDestroyActivity(r, false, configChanges, true, reason); //true
        //...
}

public void handleDestroyActivity(ActivityClientRecord r, boolean finishing, int configChanges,
        boolean getNonConfigInstance, String reason) { //getNonConfigInstance
        performDestroyActivity(r, finishing, configChanges, getNonConfigInstance, reason);
}

void performDestroyActivity(ActivityClientRecord r, boolean finishing,
        int configChanges, boolean getNonConfigInstance, String reason) {
        //...
        if (getNonConfigInstance) {
            try {
                r.lastNonConfigurationInstances = r.activity.retainNonConfigurationInstances(); //!!!
        } catch (Exception e) {
            //...    
        }
        //...
}
```

当`getNonConfigInstance == true`时，会为`ActivityClientRecord.lastNonConfigurationInstances`赋值，这里就是Activity在重新创建时获取的上一个Activity的ViewModelStore的地方。

上一个Activity的ViewModelStore怎么保存在lastNonConfigurationInstances的呢？mViewModelStore保存的结构：

<img src="./images/Jetpack.assets/(null)-20221217175404838.(null)" alt="img" style="zoom:25%;" />

```Kotlin
//Activity.java
NonConfigurationInstances retainNonConfigurationInstances() {
    Object activity = onRetainNonConfigurationInstance(); //!!!
    HashMap<String, Object> children = onRetainNonConfigurationChildInstances();
    FragmentManagerNonConfig fragments = mFragments.retainNestedNonConfig();

    // We're already stopped but we've been asked to retain.
    // Our fragments are taken care of but we need to mark the loaders for retention.
    // In order to do this correctly we need to restart the loaders first before
    // handing them off to the next activity.
    mFragments.doLoaderStart();
    mFragments.doLoaderStop(true);
    ArrayMap<String, LoaderManager> loaders = mFragments.retainLoaderNonConfig();

    if (activity == null && children == null && fragments == null && loaders == null
            && mVoiceInteractor == null) {
        return null;
    }
    //正常返回一个NonConfigurationInstances对象
    NonConfigurationInstances nci = new NonConfigurationInstances(); //!!!
    nci.activity = activity;
    nci.children = children;
    nci.fragments = fragments;
    nci.loaders = loaders;
    if (mVoiceInteractor != null) {
        mVoiceInteractor.retainInstance();
        nci.voiceInteractor = mVoiceInteractor;
    }
    return nci;
}
```

获取Activity.NonConfigurationInstances的activity字段要保存的信息。

ComponentActivity重写了Activity的onRetainNonConfigurationInstance方法：

```Kotlin
//ComponentActivity.java
public final Object onRetainNonConfigurationInstance() {
    // Maintain backward compatibility.
    Object custom = onRetainCustomNonConfigurationInstance();
    //获取当前的ViewModelStore对象
    ViewModelStore viewModelStore = mViewModelStore; //!!!☀️
    if (viewModelStore == null) {
        // No one called getViewModelStore(), so see if there was an existing
        // ViewModelStore from our last NonConfigurationInstance
        NonConfigurationInstances nc =
                (NonConfigurationInstances) getLastNonConfigurationInstance();
        if (nc != null) {
            viewModelStore = nc.viewModelStore;
        }
    }

    if (viewModelStore == null && custom == null) {
        return null;
    }
    //正常返回一个ComponentActivity.NonConfigurationInstances对象
    NonConfigurationInstances nci = new NonConfigurationInstances();
    nci.custom = custom;
    nci.viewModelStore = viewModelStore;
    return nci;
}
```

正常返回一个ComponentActivity.NonConfigurationInstances对象，里面包含了ViewModel对象。

回到ActivityThread.performDestroyActivity()，最终是将一个Activity.NonConfigurationInstances对象保存到了ActivityClientRecord中。



##### **保存的数据如何恢复呢**

当==重新创建Activity时==，ActivityThread会调用performLaunchActivity方法。

```Kotlin
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {//r
    //...
    activity.attach(appContext, this, getInstrumentation(), r.token,
        r.ident, app, r.intent, r.activityInfo, title, r.parent,
        r.embeddedID, r.lastNonConfigurationInstances, config, //r.lastNonConfigurationInstances
        r.referrer, r.voiceInteractor, window, r.configCallback,
        r.assistToken, r.shareableActivityToken);
    //...
}
```

调用了Activity.attach方法，并且传入了我们销毁上一个Activity时保存在ActivityClienRecord中的lastNonConfigurationInstances对象。

```Kotlin
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances, //NonConfigurationInstances
        Configuration config, String referrer, IVoiceInteractor voiceInteractor,
        Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
        IBinder shareableActivityToken) {
        attachBaseContext(context);
        //...
        //为当前Activity实例的mLastNonConfigurationInstances赋值
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        //...
}
```

Activity#attach方法在Activity的onCreate方法之前被调用，方法内部为当前Activity实例的mLastNonConfigurationInstances赋值。



##### **ActivityClientRecord生命周期范围，如何保证能在下一次创建后复用呢？**

ActivityThread中有一个成员变量mActivities，通过map形式保存一个activity的ActivityClientRecord。

```Kotlin
final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
```

==一个Activity创建时，会调用stratActivityNow，创建一个ActivityClientRecord()，针对每个Activity都会生成一个自己的token，标识唯一的Activity==，在调用performLaunchActivity时会向mActivities保存当前Activity的ActivityClientRecord。

```Kotlin
//ActivityThread.java
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P, trackingBug = 115609023)
public final Activity startActivityNow(Activity parent, String id,
        Intent intent, ActivityInfo activityInfo, IBinder token, Bundle state,
        Activity.NonConfigurationInstances lastNonConfigurationInstances, IBinder assistToken,
        IBinder shareableActivityToken) {
    ActivityClientRecord r = new ActivityClientRecord();
        r.token = token;
        r.assistToken = assistToken;
        r.shareableActivityToken = shareableActivityToken;
        r.ident = 0;
        r.intent = intent;
        r.state = state;
        r.parent = parent;
        r.embeddedID = id;
        r.activityInfo = activityInfo;
        r.lastNonConfigurationInstances = lastNonConfigurationInstances;
    if (localLOGV) {
        ComponentName compname = intent.getComponent();
        String name;
        if (compname != null) {
            name = compname.toShortString();
        } else {
            name = "(Intent " + intent + ").getComponent() returned null";
        }
        Slog.v(TAG, "Performing launch: action=" + intent.getAction()
                + ", comp=" + name
                + ", token=" + token);
    }
    // TODO(lifecycler): Can't switch to use #handleLaunchActivity() because it will try to
    // call #reportSizeConfigurations(), but the server might not know anything about the
    // activity if it was launched from LocalAcvitivyManager.
    return performLaunchActivity(r, null /* customIntent */); //!!!
}

private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    //...
    synchronized (mResourcesManager) {
        mActivities.put(r.token, r); //!!!
    }
    //...
}
```

后面在ActivityClientRecord的取值时，都是调用mActivities.get()，通过activity的token来获取的ActivityClientRecord。

```kotlin
```

在Activity销毁或者屏幕发生旋转时，会先把上一个Activity的ActivityClientRecord取出来，为lastNonConfigurationInstance赋值后，再从mActivities中remove掉当前Activity。

```kotlin
```



## Lifecycles

> 如要在非Activity的类中去感知Activity的生命周期。可通过在Activity中**嵌入一个隐藏的Fragment**来进行感知，或者通过**手写监听器**的方式来进行感知。
>
> 或借助Lifecycles组件，它可让任何一个类都能轻松感知到Activity的生命周期，同时又不需要在Activity中编写大量的逻辑处理。
>
> 为了给外界提供生命周期状态，经常会把Activity作为参数传给其他部分，代码难以维护，有内存泄露风险。

### LifeCycle使用

```Kotlin
//创建观察者
class MyObserver: LifecycleObserver {

    companion object {
        private const val TAG = "MainActivity MyObserver"
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START) //!!!
    fun activityStart() {
        Log.d(TAG, "activityStart: ")
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop() {
        Log.d(TAG, "activityStop: ")
    }
}

//监听Activity的生命周期
lifecycle.addObserver(MyObserver())

//主动获知当前生命周期状态
lifecycle.currentState
```

1. 方法上使用注解，传入生命周期事件。

2. 获取lifecycle对象，观察生命周期。

   只要Activity是继承自AppCompatActivity的，或者Fragment是继承自androidx.fragment.app.Fragment的，那么本身就是一个LifecycleOwner的实例，这部分工作已经由AndroidX库自动帮我们完成了。

### 对架构的贡献

+ 通过**模板方法模式**和**观察者模式**，解决“生命周期管理一致性”问题。

### 原理

#### LifeCycle的生命周期状态事件和状态

```java
public abstract class Lifecycle {
    @MainThread
    public abstract void addObserver(@NonNull LifecycleObserver observer);
    
    @MainThread
    public abstract void removeObserver(@NonNull LifecycleObserver observer);
    
    @MainThread
    @NonNull
    public abstract State getCurrentState();
    
    @MainThread
    @NonNull
    public abstract State getCurrentState();
    
    @SuppressWarnings("WeakerAccess")
    public enum Event {
        /**
         * Constant for onCreate event of the {@link LifecycleOwner}.
         */
        ON_CREATE,
        /**
         * Constant for onStart event of the {@link LifecycleOwner}.
         */
        ON_START,
        /**
         * Constant for onResume event of the {@link LifecycleOwner}.
         */
        ON_RESUME,
        /**
         * Constant for onPause event of the {@link LifecycleOwner}.
         */
        ON_PAUSE,
        /**
         * Constant for onStop event of the {@link LifecycleOwner}.
         */
        ON_STOP,
        /**
         * Constant for onDestroy event of the {@link LifecycleOwner}.
         */
        ON_DESTROY,
        /**
         * An {@link Event Event} constant that can be used to match all events.
         */
        ON_ANY;

        /**
         * Returns the {@link Lifecycle.Event} that will be reported by a {@link Lifecycle}
         * leaving the specified {@link Lifecycle.State} to a lower state, or {@code null}
         * if there is no valid event that can move down from the given state.
         *
         * @param state the higher state that the returned event will transition down from
         * @return the event moving down the lifecycle phases from state
         */
        @Nullable
        public static Event downFrom(@NonNull State state) {
            switch (state) {
                case CREATED:
                    return ON_DESTROY;
                case STARTED:
                    return ON_STOP;
                case RESUMED:
                    return ON_PAUSE;
                default:
                    return null;
            }
        }

        /**
         * Returns the {@link Lifecycle.Event} that will be reported by a {@link Lifecycle}
         * entering the specified {@link Lifecycle.State} from a higher state, or {@code null}
         * if there is no valid event that can move down to the given state.
         *
         * @param state the lower state that the returned event will transition down to
         * @return the event moving down the lifecycle phases to state
         */
        @Nullable
        public static Event downTo(@NonNull State state) {
            switch (state) {
                case DESTROYED:
                    return ON_DESTROY;
                case CREATED:
                    return ON_STOP;
                case STARTED:
                    return ON_PAUSE;
                default:
                    return null;
            }
        }

        /**
         * Returns the {@link Lifecycle.Event} that will be reported by a {@link Lifecycle}
         * leaving the specified {@link Lifecycle.State} to a higher state, or {@code null}
         * if there is no valid event that can move up from the given state.
         *
         * @param state the lower state that the returned event will transition up from
         * @return the event moving up the lifecycle phases from state
         */
        @Nullable
        public static Event upFrom(@NonNull State state) {
            switch (state) {
                case INITIALIZED:
                    return ON_CREATE;
                case CREATED:
                    return ON_START;
                case STARTED:
                    return ON_RESUME;
                default:
                    return null;
            }
        }

        /**
         * Returns the {@link Lifecycle.Event} that will be reported by a {@link Lifecycle}
         * entering the specified {@link Lifecycle.State} from a lower state, or {@code null}
         * if there is no valid event that can move up to the given state.
         *
         * @param state the higher state that the returned event will transition up to
         * @return the event moving up the lifecycle phases to state
         */
        @Nullable
        public static Event upTo(@NonNull State state) {
            switch (state) {
                case CREATED:
                    return ON_CREATE;
                case STARTED:
                    return ON_START;
                case RESUMED:
                    return ON_RESUME;
                default:
                    return null;
            }
        }

        /**
         * Returns the new {@link Lifecycle.State} of a {@link Lifecycle} that just reported
         * this {@link Lifecycle.Event}.
         *
         * Throws {@link IllegalArgumentException} if called on {@link #ON_ANY}, as it is a special
         * value used by {@link OnLifecycleEvent} and not a real lifecycle event.
         *
         * @return the state that will result from this event
         */
        @NonNull
        public State getTargetState() {
            switch (this) {
                case ON_CREATE:
                case ON_STOP:
                    return State.CREATED;
                case ON_START:
                case ON_PAUSE:
                    return State.STARTED;
                case ON_RESUME:
                    return State.RESUMED;
                case ON_DESTROY:
                    return State.DESTROYED;
                case ON_ANY:
                    break;
            }
            throw new IllegalArgumentException(this + " has no target state");
        }
    }
}
```

LifeCycle使用了Event、State两个枚举来跟踪其关联组件的生命周期状态。

#### LifeCycle如何观察Activity和Fragment的生命周期

```java
```





## LiveData

> LiveData是Jetpack提供的一种**响应式编程组件**，它可以包含任何类型的数据，并在**数据发生变化的时候通知给观察者**。它具有生命周期感知能力，意指它遵循其他应用组件（如 activity、fragment 或 service）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。
>
> 这对于activity和Service特别有用，意味着可以安全观察LiveData对象而不用担心内存泄露问题（可以做到 何时通知、何时解绑，从而做到安全无泄漏）。开发者也不需要在onPause或者onDestroy方法中解除对LiveData的订阅，需要注意的是：一旦观察者重新恢复Resumed状态，将会重新收到LiveData的最新数据。
>
> 也特别适合与ViewModel结合在一起使用。**单独使用ViewModel，并不能将数据的变化主动通知给Activity**。但是也==一定不能把Activity的实例传给ViewModel，ViewModel的生命周期是长于Activity的，如果把Activity的实例传给ViewModel，就很有可能会因为Activity无法释放而造成内存泄漏==，这是一种非常错误的做法。
>
> 

### 基本用法

```Kotlin
//使用LiveData包装需要获知的数据
class MainViewModel(countReserved: Int): ViewModel() {

    var counter = MutableLiveData<Int>() //LiveData

    init {
        counter.value = countReserved
    }

    fun plusOne() {
        val count = counter.value ?: 0
        counter.value = count + 1
    }

    fun clear() {
        counter.value = 0
    }

}

//观察数据，例如在Activity中
viewModel.counter.observe(this, Observer { count -> //observe
    infoText.text = count.toString()
})
```

1. MutableLiveData是一种可变的LiveData。getValue()、setValue()和postValue()方法可以进行读取。
   1. getValue()方法用于获取LiveData中包含的数据；
   2. setValue()方法用于给LiveData设置数据，但是只能在==主线程==中调用；postValue()方法用于在==非主线程==中给LiveData设置数据，否则会崩溃。
2. Activity本身就是一个LifecycleOwner对象，因此直接传this就好；第二个参数是一个Observer接口，当counter中包含的数据发生变化时，就会回调到这里，因此我们在这里将最新的计数更新到界面上即可。

### 对架构的贡献

可观察的生命周期感知的数据持有者

+ 遵循从**唯一的、可信的源**，完成数据分发的原则，与之相反的是“**一次请求得到一个结果**”的回调（Callback）。
+ 感知到页面退出后，不再分发和接收数据。
+ 利用映射、组合等操作，提升灵活性。

是理想的MVVM中ViewModel中的数据成员（被托管的数据和状态）

+ 是**唯一的、可信的源**，降低了数据绑定代码的复杂度（不用多次创建，不用考虑时序性）
+ 可**观察**改变，便于双向绑定
+ **生命周期感知**，适合以页面为主的功能里使用
+ 可**映射**，支持**调停模式**，灵活性高。

### map()和switchMap()

想要在LiveData对象分发给观察者之前对其中存储的值进行更改，可以使用map和switchMap操作。

1. 不想将某个类型的LiveData全部暴露给外部，可以用map方式将其转换成其他数据类型。

   ```Kotlin
   private val userLiveData = MutableLiveData<User>()
   val userName = Transformations.map(userLiveData) { user ->
       "${user.firstName}${user.lastName}"
   }
   ```

   当userLiveData的数据发生变化时，map()方法会监听到变化并执行转换函数中的逻辑，然后再将转换之后的数据通知给userName的观察者。

2. 如果ViewModel中的某个LiveData对象是调用另外的方法获取的，**一些每次监听不同LiveData对象的场景**，那么我们就可以借助switchMap()方法，将这个LiveData对象转换成另外一个可观察的LiveData对象。

   ```Kotlin
   //每次获取不同LiveData对象
   object Repository {
   
       fun getUser(userId: String): LiveData<User> {
           val liveData = MutableLiveData<User>()
           liveData.value = User(userId, userId, 0)
           return liveData
       }
   
   }
   
   //转换成已经可观察的LiveData对象
   private val userIdLiveData = MutableLiveData<String>()
   val user = Transformations.switchMap(userIdLiveData) { userId ->
       Repository.getUser(userId)
   }
   ```

​		LiveData内部不会判断设置的数据和原有数据是否相同，只要==调用了setValue()或postValue()方法就一定会触发数据变化==事件。   

3. 那看上去，map和switchMap操作很相似，有什么==区别==呢？

   

### LiveData的实现原理

> 它只会通知处于active状态的观察者。

#### LiveData如何观察组件生命周期变化

#### LiveData#observe回调

```java
public abstract class LiveData<T> {
    @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        assertMainThread("observe");
        //如果被观察者当前的状态是DESTROYED，就直接return，这个状态的组件就不允许注册
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
        // 对观察者的包装
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
        // 将观察者添加到缓存中，如果存在则跳过
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
        // 将LifecycleBoundObserver添加到Lifecycle订阅列表中,赋予生命周期订阅
        owner.getLifecycle().addObserver(wrapper);
    }
    
   
    class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {
        @NonNull
        final LifecycleOwner mOwner;

        LifecycleBoundObserver(@NonNull LifecycleOwner owner, Observer<? super T> observer) {
            super(observer);
            mOwner = owner;
        }

        //继承了ObserverWrapper，重写了shouldBeActive，用于判断当前传入的组件状态是否Active状态
        //Active状态：STARTED、RESUMED
        @Override
        boolean shouldBeActive() {
            return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
        }

        //组件状态发生变化时调用onStateChanged
        @Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
            Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
            //那么当组件处于DESTROYED时，就会使用removeObserver移除Observer
            if (currentState == DESTROYED) {
                removeObserver(mObserver);
                return;
            }
            Lifecycle.State prevState = null;
            while (prevState != currentState) {
                prevState = currentState;
                //activeStateChanged方法定义在ObserverWrapper（Observer的包装类）中
                activeStateChanged(shouldBeActive());
                currentState = mOwner.getLifecycle().getCurrentState();
            }
        }

        @Override
        boolean isAttachedTo(LifecycleOwner owner) {
            return mOwner == owner;
        }

        @Override
        void detachObserver() {
            mOwner.getLifecycle().removeObserver(this);
        }
    }
    
    
    private abstract class ObserverWrapper {
        final Observer<? super T> mObserver;
        boolean mActive;
        int mLastVersion = START_VERSION;

        ObserverWrapper(Observer<? super T> observer) {
            mObserver = observer;
        }

        abstract boolean shouldBeActive();

        boolean isAttachedTo(LifecycleOwner owner) {
            return false;
        }

        void detachObserver() {
        }

        /*
        会根据Active状态和处于Active状态的组件数量，对onActive方法和onInactive方法进行回调
        （这两个方法用于扩展LiveData对象）
        */
        void activeStateChanged(boolean newActive) {
            if (newActive == mActive) {
                return;
            }
            // immediately set active state, so we'd never dispatch anything to inactive
            // owner
            mActive = newActive;
            changeActiveCounter(mActive ? 1 : -1);
            //！！！
            if (mActive) {
                dispatchingValue(this);
            }
        }
    }
    
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
        //具体看下面内容👇🏻
    }
}
```

> 如果生命周期变为非活跃状态，它会在再次变为活跃状态时接收最新的数据。例如，曾经在后台的 Activity 会在返回前台后立即接收最新的数据。

总结下：1. 注册观察者时，将LifecycleOwner和Observer包装成LifecycleBoundObserver放入Map保存；且这一阶段，如果组件状态处于DESTROYED，则不允许注册；2. 之后数据更新时，会回调LifecycleBoundObserver#onStateChanged方法，最终回调Observer#onChanged，完成通知。

#### postValue/setValue

```java
public abstract class LiveData<T> {
    
    private final Runnable mPostValueRunnable = new Runnable() {
        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                // 获取最新待设置的数据
                newValue = mPendingData;
                // 重置待同步的数据为默认
                mPendingData = NOT_SET;
            }
            setValue((T) newValue); //！！！
        }
    };
    
    protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            // 数据是否set过
            postTask = mPendingData == NOT_SET;
            // 待同步的数据
            mPendingData = value;
        }
        // 当前正在postValue，忽略本次操作
        if (!postTask) {
            return;
        }
        // 将任务发送到主线程执行，实际就是把setValue方法切换到主线程调用。
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
    
    
    /**
    * 1. 设置数据
    * 2. 分发数据
    * 3. 通知观察者
    */
    @MainThread
    protected void setValue(T value) {
        assertMainThread("setValue");
        // 版本号++
        mVersion++;
        // 同步数据
        mData = value;
        // 分发数据！！！ 因此，无论是主线程还是子线程更新数据，都会调用dispatchingValue
        dispatchingValue(null); 
    }
    
    @SuppressWarnings("WeakerAccess") /* synthetic access */
    void dispatchingValue(@Nullable ObserverWrapper initiator) {
        //正处于分发状态
        if (mDispatchingValue) {//mDispatchingValue用于标记是否处于分发状态中
            mDispatchInvalidated = true; //分发无效
            return;
        }
        mDispatchingValue = true;
        do { //分发有效
            mDispatchInvalidated = false;
            // 不为null时证明是lifecycle状态变为活跃
            // 主动修改LiveData数据时，initiator是null
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
                //setValue时触发, 轮训观察者列表去更新
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                     // 如果分发失效,直接跳出(非关键点)
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;//标记不处于分发状态
    }
    
    @SuppressWarnings("unchecked")
    private void considerNotify(ObserverWrapper observer) {
        // 当前观察者非Active时直接return
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
        // 再次check观察者最新状态,即检查lifecycle对应的状态
        if (!observer.shouldBeActive()) {
            // 如果非活跃状态,通知观察者当前非活跃状态
            observer.activeStateChanged(false);
            return;
        }
        // 版本检测,如果当前观察者持有的版本>=当前的版本,即证明已经更新过了
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        // 更新观察者当前的版本
        observer.mLastVersion = mVersion;
        // 执行数据通知
        observer.mObserver.onChanged((T) mData);
    }
}
```

+ postValue的实现很巧妙，内部会先判断当前是否正在更新数据(即数据是否为默认)，然后将我们要设置的数据保存起来，如果正在更新，则跳过本次任务发送，否则将本次更新任务发送到主线程去执行(不难猜测内部也是handler执行?)，在具体的 runable 中，会直接去取最新待同步的值,然后将其置为默认值，最后执行setValue();**不过需要注意的，多线程下调用，可能会丢失某次的通知。**

+ setValue() 时，内部会对当前 LiveData 持有的版本号 version 进行自增，然后调用**dispatchingValue()** 去分发本次数据，然后会去遍历当前的观察者列表，然后判断观察者是否是活跃状态，即是否是 onStrat() - onPause() 之间，如果是并且==当前观察者的版本号小于 LiveData 维护的版本号，由此证明当前观察者尚未通知过，从而触发通知==。

### 一些实践问题

+ [Activity销毁重建导致LiveData数据倒灌](https://juejin.cn/post/6986895858239275015#comment)



   临时记录问题：

- [ ] 导致ViewModel也销毁的时机是什么？

- [x]  setValue中dispatchingValue参数为null的情况
- [ ] LiveData为什么不会引起内存泄露？
- [ ] Sticky EventBus 和LiveData什么关系啊？
- [ ] 不主动saveinstanceState，那么重建后会有数据吗
- [ ] Fragment如何作为LifecycleOwner
- [ ] 图片裁剪怎么实现的，项目里借助了SDK吗

   
