## ANR



[**什么是ANR**](https://developer.android.com/topic/performance/vitals/anr)

Application Not Responding 应用程序没有响应。



**造成ANR的主要原因**

应用程序的响应性是由Activity Manager和WindowManage系统服务监视的。Android规定，Activity如果5s无法响应屏幕触摸事件就会出现ANR，而Broadcast如果10s之内还没有执行完操作也会出现ANR，Service 在 20s 内无法处理完成也会出现ANR。

+ Android4.0以后网络操作不能出现在主线程中，主线程执行IO操作被阻塞。
+ 主线程存在耗时的计算
  + Activity的所有生命周期回调都是执行在主线程中的。
  + Service是默认执行在主线程中的。
  + BroadcastReceiver的onReceive回调方法是执行在主线程的。
  + 没有使用子线程Looper的Handler的handleMessage方法是执行在主线程的。
  + AsyncTask的回调中除了doInBackground，其他都是执行在主线程的。

比如当前事件正在被处理却迟迟没有处理完成，导致用户界面一直收不到执行结果而一直等待；主线程中的 looper 因为某种原因阻塞等等。



**怎么解决ANR**

+ 使用AsyncTask处理耗时IO操作。
+ 使用Thread或者HandlerThread提高优先级
+ 使用Handler来处理工作线程的耗时任务
+ Activity的onCreate和onResume回调中尽量避免耗时的代码

实际开发中ANR很难从代码中发现问题，但是当一个进程出现了ANR以后，系统会在/data/anr目录下创建一个文件traces.txt，通过分析这个文件就能定位出ANR的原因。



**文章**

+ [探究ANR原理，是谁控制了ANR的触发时机](https://mp.weixin.qq.com/s/Wiek-NNXA654aqI-oqY2kg)
+ [Android ANR触发、监控、分析一网打尽](https://mp.weixin.qq.com/s/qQAPg0PwefYhScdN5bBPnA)