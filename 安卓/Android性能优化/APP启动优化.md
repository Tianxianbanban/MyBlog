## APP启动优化



**关于APP启动优化**

+ 启动分类

  + 冷启动：

    耗时最多，衡量标准

    Click Event -> IPC -> Process.start -> ActivityThread -> bindApplication -> LifeCycle -> ViewRootImpl

  + 热启动

    最快

    后台 -> 前台

  + 温启动

    较快

    LifeCycle 它只会重走Activity的声明周期，而不会再进行进程的创建、Application的创建等。

  冷启动之前

  + 启动APP

  + 加载空白Window

  + 创建进程

    冷启动之前的行为时系统的行为，我们是没有办法干预的。

  随后任务

  + 创建Application
  + 启动主线程
  + 创建MainActivity
  + 加载布局
  + 布置屏幕
  + 首帧绘制

+ 优化方向

  + Application和Activity生命周期



**启动时间测量方式**

+ adb命令

  adb shell am start -W packagename/（包名）首屏Activity

  相关输出：

  + ThisTime：最后一个Activity启动耗时

  + TotalTime：所有Activity启动耗时

  + WaitTime：AMS启动Activity的总耗时

  adb命令方式：线下使用方便，但是不能带到线上，而且非严谨精确时间。

+ 手动打点

  启动时埋点，启动结束埋点，二者差值。

  误区：onWindowFocusChanged只是首帧时间。

  正解：真实数据展示。

  手动打点方式：精确，可带到线上使用，注意避开误区选择数据真正显示作为测量时间点。addOnDrawListener。



**启动优化工具**

+ traceview

  图形形式展示执行时间、调用栈等等。信息全面，包含所有线程。

  Debug.startMethodTracing("文件名")；

  Debug.stopMethodTracing()；

  生成文件存放在sd卡：Android/data/packagename/files

  traceview工具运行时开销严重，整体都会变慢。可能会带偏优化方向。

+ systrace

  结合Android内核数据，声明html报告，在API18以上使用，推荐向下兼容的TraceCompact。

  轻量级，开销小；能直观反映CPU利用率。



**获取方法耗时**

+ 常规方式：侵入性强，工作量大。

+ AOP方式（Aspect Oriented Programming，面向切面编程）：针对同一类问题的统一处理。无侵入添加代码。

  AspectJ的使用。



**异步优化**

+ 优化技巧：Theme切换，感觉上的块。

+ 异步优化思想：子线程分担主线程任务，并行减少时间。

+ 注意事项：

  一部分代码不符合异步要求

  需要在某阶段完成

  区分CPU密集型和IO密集型任务。

  

**异步初始化**

+ 常规异步痛点

  代码不优雅

  场景不好处理（依赖关系）

  维护成本高

+ **启动器**

  核心思想：充分利用CPU多核，自动梳理任务顺序。

  启动流程：代码Task话，启动逻辑抽象为Task。根据所有任务的依赖关系，排序生成一个有向无环图。多线程按照排序后的优先级依次执行。



**延迟初始化方案**





**启动优化其他方案**

