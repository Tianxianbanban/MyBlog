## SurfaceView和普通View的区别、以及TextureView





**SurfaceView和普通View的区别**

首先，SurfaceView是在一个新开的子线程中可以重新绘制画面，而view必须在UI的主线程中更新画面。

出现 SurfaceView 的原因在于：虽然说通常情况下 View 已经可以满足大部分的绘图需求，但还是有缺陷，View 是通过刷新来重绘视图的，Android 系统通过发出 VSYNC 信号来进行屏幕的重绘，刷新的时间间隔是16ms，果在 16ms 内刷新完成的话是没有什么影响的，但是如果刷新的时候执行的操作逻辑太多，那么会出现卡顿的现象，将无法响应按键、触屏等消息。SurfaceView 就是解决这个问题的，使用SurfaceView由于是在子线程中更新画面所以不会阻塞UI主线程，但这也带来了另一个问题，就是事件同步，比如触屏了一下，需要在SurfaceView中的thread处理，一般就需要有一个event queue的设计来保存touchevent，这会稍稍复杂一点，因为涉及到线程安全。

View 主要用于主动更新的情况下，而 SurfaceView 主要用于被动更新，例如频繁的刷新； 
View 在主线程中对画面进行更新，而 SurfaceView 通常会通过一个子线程来进行更新； 
View 在绘图的时候是没有使用双缓冲机制的，而 SurfaceView 在底层实现中使用了双缓冲机制； 



**参考**

[Android自定义View之双缓冲机制和SurfaceView](https://juejin.im/post/5b6a4ba551882561ef446026)

[Android视图SurfaceView的实现原理分析](https://blog.csdn.net/luoshengyang/article/details/8661317/)

[反思：Google 为何把 SurfaceView 设计的这么难用？](https://juejin.cn/post/7140191497982312455#comment)



**SurfaceView和TextureView的区别**

[视频画面帧的展示控件SurfaceView及TextureView对比](https://mp.weixin.qq.com/s?__biz=MzI2OTQxMTM4OQ==&mid=2247484652&idx=1&sn=3cae1ac74a02b161a1f0d7e975edae3e&chksm=eae1f1bedd9678a8a98f1dfe3e079d559d744212ee45cf14add53549fd928f908fff7620ded9#rd)





