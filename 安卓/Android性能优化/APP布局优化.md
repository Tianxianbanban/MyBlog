## APP布局优化



**Android绘制原理及工具**

+ CPU负责计算显示内容，比如视图创建、布局计算、图片解码、文本绘制等等。
+ GPU负责栅格化操作（UI元素绘制到屏幕上），所谓栅格化就是将一些组件比如Bitmap拆分成不同的像素进行显示，然后完成绘制，这些操作非常耗时，所以引入GPU来加快栅格化操作。
+ Android16ms发出VSync信号触发UI渲染，也就每一帧都要求在这个时间内完成，才能保证视图效果平滑。
+ 大多数的Android设备屏幕刷新频率：60hz，60帧每秒是一个人感知的极限。

优化用具

+ Systrace

  + 关注Frames
  + 正常：绿色圆点，丢帧：黄色或者红色。
  + Alert栏

+ Layout Inspector

  + Android Studio自带工具
  + 可查看视图层次结构

+ Choreographer

  + 获取FPS，线上使用，具备实时性
  + API 16 之后使用
  + Choreographer.getInstance().postFrameCallback

  

**Android布局加载原理**

+ 布局加载流程

  setContentView -> LayoutInflater -> inflate -> getLayout -> createViewFromTag -> Factory -> createView -> 反射

+ 性能瓶颈

  + 布局文件解析：I/O过程
  + 创建View对象：反射（比new慢3倍）

+ LayoutInflater.Factory

  + LayoutInflater创建View的一个Hook。
  + 定制创建View的过程：全局替换自定义TextView等

+ Factory与Factory2

  + Factory2继承与Factory
  + 多了一个参数：parent



**优雅获取界面布局耗时**

+ 常规方式

  + 覆写方法，手动埋点

+ AOP/ArtHook

  + 切Activity的setContentView

+ 获取任一控件耗时

  + 低侵入性
  + LayoutInflater.Factory

+ 解决思路

  + 根本性解决：不使用反射或者减少IO操作

  + 侧面缓解：

    AsyncInflater：异步Inflate，在工作线程加载布局；回调主线程，主线程直接使用加载好的View对象；这样可以节约主线程的时间。AsyncLayoutInflater使用可以侧面缓解，但是不能设置LayoutInflater.Factory（自定义解决），而且注意View中不能有依赖于主线程的操作。



**异步inflate实践**





**布局加载优化实践**

+ Java代码写布局

  能够从本质上解决布局加载的性能问题，减少反射操作与IO操作；

  但是也引入了新的问题，不便于开发、可维护性差。

+ X2C

  保留了XML的优点，解决了它的性能问题

  开发人员写XML，加载Java代码

  原理：通过APT编译器翻译XML为Java代码

  一些问题：部分属性Java不支持，失去了系统的兼容（AppCompact），这需要修改X2C源码解决

  

**视图绘制优化**

视图绘制：

测量（确定大小）、布局（确定位置）、绘制（绘制视图）

绘制过程存在的性能瓶颈：

每个阶段都比较耗时；自顶向下的遍历；触发多次。

+ 优化布局层级以及复杂度

  + 减少View树层级

  + 布局尽量宽而浅，避免窄而深

  + ConstraintLayout

    实现几乎完全扁平化布局

    构建复杂布局性能更高

    具有RelativeLayout和LinearLayout特性

  + 不嵌套使用RelativeLayout，以及其他尽量少嵌套

  + 不在嵌套LinearLayout中使用weight

  + merge标签：减少一个层级，只能用于跟View

+ 避免过度绘制

  + 一个像素最好只被绘制一次
  + 调试GPU过度绘制
  + 去掉多余背景色，减少复杂shape的使用
  + 避免层级叠加
  + 自定义View使用clipRect屏蔽被遮盖View绘制

+ 其它

  + ViewStub：高效占位符、延迟初始化
  + onDraw中避免：创建大对象、耗时操作
  + TextView优化



**布局优化相关问题**

+ 布局优化过程中用过的工具
+ 布局为什么卡顿，如何优化
  + IO、反射、遍历、重绘
+ 做完布局优化以后的成果



**相关文章**

+ [Android性能优化第（十）篇---布局优化](https://www.jianshu.com/p/c0e0cca14162)
+ [Android性能优化第（四）篇---Android渲染机制](https://www.jianshu.com/p/9ac245657127)