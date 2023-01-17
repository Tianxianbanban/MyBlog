## AIDL使用绑定启动远程Service出现Service Intent must be explicit: Intent 

![image-20200203175653055](C:\Users\CY\AppData\Roaming\Typora\typora-user-images\image-20200203175653055.png)

```java
Intent intent = new Intent();
intent.setAction("remote.MyRemoteService.Action");
```

使用AIDL调用远程Service通过隐式意图的方式，但是出现了

 Service Intent must be explicit: Intent { act=remote.MyRemoteService.Action }

抛出了一个异常，说意图不明确？



Android5.0以后绑定启动Service考虑到安全愿意，不允许隐式意图的方式启动，也就是说要给出一个明确的组件Service。`intent.setPackage(String packageName)`或者`intent.setComponent(ComponentName componentName)`都可以显示设置组件处理意图。那么下面这两种方式都能够解决：

```java
Intent intent = new Intent();
intent.setAction("remote.MyRemoteService.Action");//Service过滤器的内容
intent.setPackage("cy.review.servicetest");//待使用远程Service所属应用的包名
```

```java
Intent intent = new Intent();
intent.setAction("remote.MyRemoteService.Action");//Service过滤器的内容
//第一个参数待使用远程Service所属应用的包名，第二个参数所启动的远程Service完整类名
ComponentName componentName = new ComponentName(
        "cy.review.servicetest",
        "remote.MyRemoteService");
intent.setComponent(componentName);
```



参考： 

https://blog.csdn.net/l2show/article/details/47421961 

https://www.jianshu.com/p/52565022594e 

