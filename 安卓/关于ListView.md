## 关于ListView



[郭霖ListView工作原理]( https://blog.csdn.net/guolin_blog/article/details/44996879 )

[郭霖ListView异步加载图片闪动的问题](https://blog.csdn.net/guolin_blog/article/details/45586553 )



**ListView 异步加载图片出现乱序的原因**

ListView 在借助**RecycleBin机制**的帮助下，实现了一个**生产者消费者**的模式，不管有多少条数据需要显示，ListView中的子View其实来来回回就那么几个，移出屏幕的子View 很快会被移入屏幕的数据重新利用起来， 移出屏幕的view会进入到RecycleBin当中，而新进入屏幕的元素则会从RecycleBin中获取view ，这就是 RecycleBin 机制。我们来说说产生乱序的原因，每当有新元素进入界面的时候就会**回调 getView 方法**，如果我们是异步加载图片的话， 就会在 getView 方法中开启异步操作从网络上面获取图片，但是异步操作可能是比较耗时的，当我们快速滑动图片的时候可能会出现这样一种情况，某一位置上的元素进入屏幕之后开始从网络上请求图片，但是还没等图片加载到控件上的时候，它就已经被移出了屏幕，又根据 ListView 的工作机制，被移出屏幕的控件会很快被进入屏幕的元素重新利用起来，正是因为这个原因导致了图片乱序，因为如果这时候前面发起的网络请求正好得到回应的话， 首先会将刚才请求的图片显示到控件上面，虽然他们的位置不同，但是共用同一个显示图片的控件，这时候新移进来的元素如果正好也发起一个网络请求获取图片的话，在图片下载结束就会将图片显示到当前控件上面，因此就出现了先显示一张图片后又显示一张图片的情况了。

解决这个问题的方法有：在 getView 方法中对每个控件调用 **setTag** 方法，设置其标志，在需要用到控件的地方使用 **findViewWithTag** 获得控件，这种方式解决图片乱序的原理是：因为 ListView 中的控件都是可以重用的，当移出屏幕的控件重新被利用的时候会回调 getView 方法，在 getView 方法中会调用 setTag 方法为当前控件设置标志，那么这个标志将会覆盖掉原先设置在该控件上面的标志，这样每次使用 findViewWithTag 获取对应于旧的 Tag 的控件的时候就只能得到 null 了，而只有非 null 的时候我们才会显示图片到控件上面，这样就保证了不会显示旧的图片在控件上面的效果了。



**ListView 优化方案 **

关于 ListView 的优化总结几点

+ 重用 ConvertView，**ConvertView会把之前加载好的布局进行缓存，可以重用，就减少不必要 view 的创建**，因为 inflate 操作也是把 xml 实例化为相应的 View 实例，这个过程是属于 IO 操作，相对来说比较耗时。
+ **减少 findViewById 的次数**，因为即使使用了ConvertView以后不会重复加载布局了，但是每次getView方法回调的时候还是通过findViewById 重复创建控件的实例，那么就通过把 xml 文件中的涉及到的控件封装成内部类**ViewHolder**的方式， 通过 **View 的 setTag 和 getTag 方法将 当前布局View 和对应的 ViewHolder 对象绑定在一起**，减少不必要 的 findViewById 次数。
+ 如果涉及到加载图片之类比较耗时的操作，可以在用户快速滑动屏幕的时候禁止图片的加载，等到滑动停止的时候再去加载图片。
+  ListView 中 Item 的布局层次越简单越好，主要为了避免布局太深带来的重绘太复杂问题，还可以减少半透明元素的绘制，因为半透明更加的耗时。
+ 尽量保证 Adapter 适配器的 hasStables 方法返回 true，这样在调用 notifyDataSetChanged()方法的时候，如果 Item 内容没有发生变化的话，ListView 将不再重绘，以此达到优化。
+ **直接使用 RecycleView 代替 ListView**，每个 Item 自己发生变动，ListView 都会去调用 notifyDataSetChanged 方法更新所有的 Item，未免有点太浪费性能了，而 RecycleView 可以实现每个 Item 的局部刷新，不再需要刷新所有的 Item，同时引入了增加和删除的动画 效果，在性能和定制上有很大改善。
+ 尽量开启硬件加速功能。 



**ListView三级缓存**

+ 内存缓存
+ 磁盘缓存
+ 网络拉取





