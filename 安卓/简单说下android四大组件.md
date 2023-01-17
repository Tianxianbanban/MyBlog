## 简单说下android四大组件

首先Android系统四大组件分别是：Activity、Service、Broadcast Receiver、Content Provider。

其中**Activity**是所有应用程序的门面，凡是应用中看到的东西，都是放在Activity当中的。而**Service**，是用户无法看到的，它是在后台默默运行，即使用户退出了应用，Service也是可以继续运行的。**Broadcast Receiver**允许我们的应用接收来自各处的广播消息，比如说电话、短信等等，同时我们的应用也可以向外发送广播消息。**Content Provider**则为应用程序之间共享数据提供了可能，比如想要读取系统电话簿中的联系人，就需要通过内容提供器来实现。