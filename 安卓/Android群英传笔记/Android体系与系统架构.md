## Android体系与系统架构



**Google生态系统**



**Android系统架构**

+ 最新的系统架构图(添加了硬件抽象层HAL)

  ![Android系统架构](Android%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84.png)

+ Linux

  Linux层，Android最低层最核心的部分。Linux层包含了Android系统的核心服务，包括硬件驱动、进程管理、安全系统，等等。

+ HAL

  硬件抽象层 (HAL)提供标准接口，向更高级别的 Java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个接口，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。

+ Dalvik与ART

  Dalvik包含了一整套的Android运行环境虚拟机，每个App都会分配Dalvik虚拟机来保证互相之间不受干扰，并保持独立。它的特点是在**运行时编译**。打个比方，就好比你买了一辆可折叠的自行车，平时是折叠的，只有骑的时候，才需要组装起来用。而在Android 5.X版本开始，ART模式已经取代了Dalvik, ART采用的是**安装时就进行编译**，以后运行时就不用编译了，这就好比你买了辆组装好了的自行车，装好就可以骑了。当然，对在其虚拟机环境中运行的大部分App来说，它们都运行着同样的代码。

+ Framework

+ Standard libraries

+ Application



**Android App组件架构**

+ Android四大组件如何协同工作

+ 应用运行上下文对象

  Android系统的上下文对象，即在Context中，为我们封装了这样一个“语境”。Activity、Service、Application都是继承自Context。Android应用程序会在如下所示的几个时间点创建应用上下文Context，如创建Application、创建Activity、创建Service。创建Context的时机就是在创建Context的实现类的时候。当应用程序第一次启动时，Android系统都会创建一个Application对象，同时创建Application Context，所有的组件都共同拥有这样一个Context对象，这个应用上下文对象贯穿整个应用进程的生命周期，为应用全局提供了功能和环境支持。而创建Activity和Service组件时，系统也会给它们提供运行的上下文环境，即创建Activity实例、Service实例的Context对象。编码时在Activity中获取Context对象时，可以直接使用this，而在匿名内部类中，就必须指定XXXXActivity.this才可以获得该Activity的Context对象。也可以通过getApplicationContext()方法来获取整个App的Context，但是通过getApplicationContext()方法获得的是整个应用的上下文引用，这与某个组件的上下文引用，在某些时候还是有区别的。

  我们知道Android程序和 java程序最大的区别就在于，在 java中，你只需要一个main 函数作为入口就可以跑起来了，Android 程序是需要一个完整的工程环境的，并且 Activity、Service、Broadcast 等系统组件并不是像普通的 java 对象一样随便 new 一下就能创建实例的，而是需要各自的运行环境上下文，即这里的 Context，可以说Context 是维持 Android 程序中个组件能够正常工作的核心功能类，包含了应用程序的环境信息， 通过他我们可以获得应用程序的资源和类，Context 有相应的继承结构图。

  其中getApplication()和 getApplicationContext()区别：我们可在 Activity 或 Service 中通过 getApplication()获得 Application 实例，但是在 BroadcastReceiver 中却不行，原因在于 getApplication 方法是 Activity 和 Service 自己添加的，并不是 Context 中本身就有的，如果我们想在 BroadcastReceiver 中获得 Application 实例，可通过 getApplicationContext 方法，也就说 getApplication 适用范围不如 getApplicationContext；使用 Application 的时候需要注意的是，它本身已经是全局唯一的了，这点是由系统来保证的，如果我们想要自己实现一个类似于单例模式的 Application 类，不仅没必要， 而且经常会出错，原因在于你写单例的时候通常是将构造函数写成 private 类型，并且对外提供一个 public 方法来获得这个实例，而实例的获得是通过 new 的方式实现的，前面已说过Android中并不能随便 new一个上下文对象的子类就能运行程序来，他是需要Android 环境支撑的，你只是 new 了一个实例出来，但这个实例其实是跟 Android 没什么关系的，而系统在创建 Application 的时候是会有当前环境信息的，这需要注意。

  

**Android系统源代码目录与系统目录**

+ Android系统源代码目录
+ Android系统目录
+ Android APP文件目录