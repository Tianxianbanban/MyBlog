# Android平台剖析

## Activity是如何被显示在屏幕上的

> 作为开发者，我们通常就在Activity#onCreate生命周期节点，通过setContentView()设置布局文件，剩下的工作就是FrameWork层来完成了，但是有过调试经验的开发者就能知道，并不是setContentView()之后Activity页面就能够展示出来了，而是执行到Activity#onResume生命周期时Activity才能完全显示出来。
>
> 所以需要知道**onCreate**阶段和**onResume**阶段分别做了什么？同时Activity中很多信息都是在**attach**()阶段初始化的，也需要知道初始化了哪些和显示相关的信息。
>
> 知识储备：
>
> + Activity的ViewTree结构
> + 页面刷新机制
> + 结构和显示原理上，Activity、Dialog、PopupWindow之间的区别
> + 技术思考：如何让页面显示更良好高效，常见的导致页面卡顿的原因（主要就是主线程的耗时操作，或者绘制不合理，有过度绘制的情况，延误了绘制的时机。结合实际案例会更有说服力）



### setContentView

```Java
public class Activity extends ContextThemeWrapper,...{
    
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
    
    final void attach(Context context, ActivityThread aThread,...,Window window,...){
        //...
        
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(mWindowControllerCallback);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        //...
    }
}        
   
```

getWindow()获取Window的实例，然后再调用Window的setContentView方法。mWindow的初始化，可以从attach方法中看到，是一个PhoneWindow实例，PhoneWindow从字面意思看就是整个手机的窗口。

```java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
    
    ViewGroup mContentParent;
    private DecorView mDecor;
    
    @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor(); //!!!
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            //最后添加了我们设置的布局，可以理解为是系统布局中留出的一个位置
            mLayoutInflater.inflate(layoutResID, mContentParent); 
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
    
    private void installDecor() {
        if (mDecor == null) {
            //内部就是通过DecorView构造方法创建了一个实例，而DecorView就是FrameLayout的一个子类
            mDecor = generateDecor(-1); 
            //...
        }
        
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
            //...
        }
    }
    
    protected ViewGroup generateLayout(DecorView decor) {
        //...
        int layoutResource;
        if ((features ...) { //根据features选择系统布局
            layoutResource = ...;
        }
    }
}
```

从上可以看到mContentParent最后添加了我们设置的布局，可以理解为是系统布局中留出的一个位置，而DecorView则是系统布局的父View。



### attach

Activity实例创建好，并执行attach方法时，会为它创建一个**PhoneWindow**，每个Activity都会有一个自己的PhoneWindow。还为Activity创建了一个**WindowManager**，让它能够管理整个PhoneWindow（它对View进行管理）。

### onCreate

进入onCreate阶段，调用**setContentView**初始化时，如果还没有初始化ContentParent，说明是第一次进行setContentView，那么就会去初始化**DecorView**，还会给DecorView添加一个系统页面样式的子View，在这个子View中还可以**找到ContentParent**（用来添加自定义布局），再通过inflate方式就能把我们**自定义布局文件添加并转换成一个ViewTree**了。

### onResume

onResume生命周期是在`ActivityThread#handleResumeActivity`方法中触发的。在这个阶段会调用WindowManager的**addView**方法，添加DecorView。当WindowManager管理ViewTree时，会给ViewTree分配一个**ViewRootImpl**，用于管理ViewTree的绘制，包括显示、测量、同步刷新、事件分发等等，并与其他服务进行通信。最后通过**requestLayout（）**显示内容。

```Java
public final class ActivityThread extends ClientTransactionHandler
        implements ActivityThreadInternal {
    
    @Override
    public void handleResumeActivity(ActivityClientRecord r, boolean finalStateRequest,
            boolean isForward, String reason) {
        //触发resume回调
        if (!performResumeActivity(r, finalStateRequest, reason)) {
            return;
        }
        
        //Activity未关闭要显示的情况下，取出DecorView，并添加到WindowManager
        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            //回到Activity#attach方法，会发现每个Activity初始化的时候都会持有一个WindowManager，而且都是通过PhoneWindow获取的
      //((WindowManagerImpl)wm).createLocalWindowManager(this);
            ViewManager wm = a.getWindowManager();
            //...
            wm.addView(decor, l); //WindowManagerGlobal#addView
        }
        //...
        r.activity.makeVisible(); //把Activity设置为可见
        //其实这里还只是把DecorView设置为可见，并不足以支撑全部的绘制工作，只是触发了一次重绘
    }
}
```

WindowManagerGlobal#addView中持续调用到ViewRootImpl。

```java
public final class ViewRootImpl ... {
    final W mWindow; 
    //是一个继承自IWindow.Stub的静态内部类，作用就是和WMS进行通讯
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
        //...
        if (mView == null) {
            mView = view;
            //...
            requestLayout(); //触发布局和绘制
            //...
            //通知WMS添加窗口，其中mWindowSession也是来自WindowManagerGlobal的，同样是一个跨进程通信的句柄
            mWindowSession.addToDisplayAsUser(mWindow,...);
            //...
            view.assignParent(this); //decor的parent设置为ViewRootImpl
        }
    }
    
    @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }
    
    void scheduleTraversals() {
        //mTraversalScheduled看上去应该是一个防止重入的标记
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //根据字面意思，mChoreographer应该是和节奏有关
            //看上去是doTraversal后的一串逻辑包装传递给了mChoreographer，让它决定什么时候执行。
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null); //TraversalRunnable
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }
    
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }
    
    void doTraversal() {
        //mTraversalScheduled又置为false了
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals(); //!!!

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
    
    private void performTraversals() {
        final View host = mView;
        //...
        /*
        可以看到内部也是通过mWindowSession与WMS交互
        向WMS申请Surface。
        */
        relayoutWindow(params, viewVisibility, insetsPending);
        //...
        performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
        //...
        performLayout(lp, mWidth, mHeight);
        //...
        /*
        执行绘制，
        mSurface.lockCanvas(dirty);
        相当于通过Surface对象拿到了Buffer，作为绘制Bitmap的缓冲区。
        mView.draw(canvas);
        ...
        surface.unlockCanvasAndPost(canvas);
        */
        performDraw();
    }
}
```



### Dialog

```java
class Dialog {
    Dialog(@UiContext @NonNull Context context, @StyleRes int themeResId,
            boolean createContextThemeWrapper) {
        mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        //Dialog有属于自己的PhoneWindow
        final Window w = new PhoneWindow(mContext);
        mWindow = w;
    }
    
    public void setContentView(@LayoutRes int layoutResID) {
        //和Activity一样，都会调用到PhoneWindow的setContentView
        mWindow.setContentView(layoutResID);
    }
    
    public void show() {
        //...
        //可以看出Dialog同样有自己的DecorView
        mDecor = mWindow.getDecorView();
        //...
        mWindowManager.addView(mDecor, l);
    }
}
```



### PopupWindow

```java
public class PopupWindow {
    public PopupWindow(View contentView, int width, int height, boolean focusable) {
        //...
        //构造方法就调用了setContentView
        setContentView(contentView); //其中也获取了WindowManager
        //...
    }
    
    @UnsupportedAppUsage
    public void showAtLocation(IBinder token, int gravity, int x, int y) {
        //...
        preparePopup(p); //背景设置和DecorView创建
        //...
        invokePopup(p); //执行弹出，同样也是通过WindowManager#addView开启绘制
    }
}
```

可以看到PopupWindow是没有自己的PhoneWindow的，它必须要**依附于Activity**，但是它必须有自己的PopDecorView。



### requestLayout

接着看requestLayout内做了什么（源码在上方👆🏻）

+ 开始绘制
+ 结束绘制
+ post显示

**VSync信号**

+ 当整个屏幕刷新完毕，即一个垂直刷新周期完成，就会发出VSync信息

+ 以60Hz刷新率为前提，VSync信号的周期，应该是1000ms/60 = 16.666...ms。

  ```mermaid
  graph TD
    A(屏幕) -->|硬件VSync信号| B(SurfaceFlinger)
    B -->|软件VSync信号| C(ChoreoGrapher)
  ```

+ 加入执行了绘制，却最终没有被刷新到屏幕上，就是一种浪费资源的情况，并且由于这一段时间内屏幕显示内容没有变化，就会给用户卡顿的感觉。

  所以在有VSync信号时，最有效的绘制策略就是在上一个VSync信号周期开始就去绘制下一个VSync信号周期需要展示的图像。那么如果每次绘制都能在16.666ms内完成，图像显示就会很流畅了。



### invalidate、postInvalidate、requestLayout

> View中的方法。

+ [View的invalidate传递与View的绘制流程](https://github.com/Idtk/Blog/blob/master/Blog/9%E3%80%81Invalidate.md)
+ [View的requestLayout传递与测量、布局流程分析](https://github.com/Idtk/Blog/blob/master/Blog/10%E3%80%81RequestLayout.md)
+ https://www.cnblogs.com/normalandy/p/12408665.html
+ https://juejin.cn/post/6844903913196519431#heading-8



### View.post

+ [View.post() 为什么能够获取到 View 的宽高 ？](https://juejin.cn/post/6895735092438630407#comment)
+ [View.post() 不靠谱的地方你知道吗？](https://juejin.cn/post/6844903494214746120#comment)



## Parcelable为什么速度优于SeriaLizable?

知识储备：

+ Serializable的实现原理
+ Parcelable的实现原理
+ Intent中对这两种方式的处理
+ 技术思考：两种方式各自的缺点和适用场景



### 序列化

+ 将实例的状态转换为可以存储或者传输的形式（二进制流、字符串、可以被存储到文件磁盘、也可以通过各种协议传输，在Android系统中还可以将其进行跨进程传输）的过程。
+ 反过来将这种形式还原成实例，就是反序列化。



### Serializable

易用性

继承Serializable接口即可，不需要实现任何方法。

+ 用**反射**获取类的结构和属性信息，过程中会产生中间信息。
+ 有**缓存结构**，在解析相同类型的情况下，能够复用缓存。
+ 在性能可接受的范围内，易用性较好。
+ 先通过ObjectOutputStream，把对象转换成ByteArray，再通过writeByteArray写入Parcel。



### Parcelable

牺牲了易用性，避免了在反序列阶段大量使用反射的方式。

但是这种易用性的牺牲也是可以挽回的，比如Kotlin中就可以使用@Parcelize注解，减少样板代码的编写。

+ 对象自行实现出入口方法，避免对类结构的反射。
+ 二进制流存储在**连续内存**中，占用空间更小。
+ 牺牲了易用性（可以通过Parcelize弥补），换取极致性能。
+ 调用对象内部实现的writeToParcel方法中的逻辑，通过一些write方法直接写入Parcel。



### Intent传值场景

可以看到Intent和Bundle都是Parcelable的实现类。



### 不能抛开应用场景谈技术方案

+ 大多数场景下，Parcelable确实存在性能优势，Serializable的性能缺陷来自，需要反射构建ObjectStreamClass类型的描述信息。
+ 构建ObjectStreamClass类型的描述信息的过程，有缓存机制，能够大量复用缓存的使用场景下，Serializable反而会存在性能优势。
+ Parcelable原本在易用性上存在短板，但是Kotlin的Parcelize很好弥补了。



