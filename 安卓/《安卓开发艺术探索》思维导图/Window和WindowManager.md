# Window和WindowManager

Android中所有的视图都是通过Window来呈现的，不管是Activity、Dialog还是Toast，它们的视图实际上都是附加在Window上的，因此Window实际上是View的直接管理者。



**Window和WindowManager**

Window只是一个窗口的概念，Window是一个抽象类，具体的实现是PhoneWindow。

创建一个Window需要通过WindowManager（接口，继承自ViewManager，真正的实现时WindowManagerImpl）完成，所以WindowManager是外界访问Window的入口。

```java
ViewManager#addView(View view, ViewGroup.LayoutParams params)
WindowManager.LayoutParams中有flag和type两个比较重要的参数：
    Flag参数表示Window的属性，Type表示Window的类型。
```

Window的具体实现定位于WindowManagerService当中，**WindowManager和WindowManagerService的交互是一个IPC过程**。



**Window的内部机制**

Window是一个抽象的概念，每个Window都对应了一个View和一个ViewRootImpl，**Window和View是通过ViewRootImpl来建立联系**，因此Window并不是实际存在的，它是以View的形式存在，从WindowManager中的三个主要接口方法可以看出**View才是Window存在的实体**。

Window的内部机制可以从Window的添加、删除、更新来看，WindowManagerImpl中Window的三大操作都是通过**桥接模式**委托给WindowManagerGlobal来处理的，WindowManagerGlobal以工厂的形式向外提供自己的实例。（1）addView：首先会检查参数是否合法，如果是子Window的话还会调整一些布局参数；然后创建ViewRootImpl并将View添加到列表中；接着通过ViewRootImpl来更新界面并完成Window的添加过程，而且Window添加过程是一次**IPC调用**，在Session（是一个Binder对象）的内部会通过WindowManagerService来实现Window的添加，它的内部会为每一个应用保留一个单独的Session。（2）removeView：Window的删除过程和添加过程一样，都是通过WindowManagerImpl进一步委托WindowManagerGlobal实现的。首先是通过findViewLocked来查找待删除的View的索引，然后再调用removeViewLocked来做进一步的删除，它是通过ViewRootImpl来完成删除操作的，首先获取ViewRootImpl，然后从ViewRootImpl中获取View，将其添加到mDyingViews列表中待删除，无论是异步还是同步删除，真正删除View的逻辑都是在dispatchDetachedFromWindow方法的内部实现。（3）updateViewLayout：WindowManagerGlobal在updateViewLayout中更新View的LayoutParams替换掉旧的LayoutParams，然后再更新ViewRootImpl的LayoutParams，继续在ViewRootImpl中会通过scheduleTraversals方法来对View进行重新布局，包括测量、布局、绘制三个过程。ViewRootImpl还会通过IPC过程调用WindowManagerService的热layoutWindow（）来更新Window视图。



**Window的创建过程**

View不能单独存在，必须依附在Window这个抽象的概念上，因此有视图的地方就有Window，所以Activity、Dialog、Toast等视图都对应一个Window。

+ Activity的Window创建

  在Activity（实现了Window.Callback）的attch方法中创建Activity所属Window对象并为其设置回调接口。

  ```java
  PolicyManager.makeNewWindow(this);//返回一个PhoneWindow
  给PhoneWindow创建DecorView；
  将View添加到DecorView的mContentParent（DecorView中的一个ViewGroup）中，也就是例如setContentView(R.layout.activity_main);
  回调Activity的onContentChanged方法通知Activity视图已经发生了改变。
  ```

  

+ Didlog的Window创建

+ Toast的Window创建