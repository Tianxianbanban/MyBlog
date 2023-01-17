## RecyclerView和ListView的区别





### ListView

[Android ListView工作原理完全解析，带你从源码的角度彻底理解](https://blog.csdn.net/guolin_blog/article/details/44996879)

[郭霖ListView异步加载图片闪动的问题](https://blog.csdn.net/guolin_blog/article/details/45586553 )



**ListView缓存机制**

RecycleBin机制，RecycleBin定义在AbsListView当中。其中使用View[] mActiveViews存储View，是屏幕当中正在使用的View，mActiveViews中存储的View只能被获取一次；ArrayList<View>[] mScrapViews和ArrayList<View> mCurrentScrap则是用于存储移动出屏幕后被废弃的View。当getView方法在适配器中被调用的时候，其中传入的convertView如果不为空，就是从废弃的View当中获得的。



**ListView 异步加载图片出现乱序的原因**

ListView 在借助**RecycleBin机制**的帮助下，实现了一个**生产者消费者**的模式，不管有多少条数据需要显示，ListView中的子View其实来来回回就那么几个，移出屏幕的子View 很快会被移入屏幕的数据重新利用起来， 移出屏幕的view会进入到RecycleBin当中，而新进入屏幕的元素则会从RecycleBin中获取view ，这就是 RecycleBin 机制。我们来说说产生乱序的原因，每当有新元素进入界面的时候就会**回调 getView 方法**，如果我们是异步加载图片的话， 就会在 getView 方法中开启异步操作从网络上面获取图片，但是异步操作可能是比较耗时的，当我们快速滑动图片的时候可能会出现这样一种情况，某一位置上的元素进入屏幕之后开始从网络上请求图片，但是还没等图片加载到控件上的时候，它就已经被移出了屏幕，又根据 ListView 的工作机制，被移出屏幕的控件会很快被进入屏幕的元素重新利用起来，正是因为这个原因导致了图片乱序，因为如果这时候前面发起的网络请求正好得到回应的话， 首先会将刚才请求的图片显示到控件上面，虽然他们的位置不同，但是共用同一个显示图片的控件，这时候新移进来的元素如果正好也发起一个网络请求获取图片的话，在图片下载结束就会将图片显示到当前控件上面，因此就出现了先显示一张图片后又显示一张图片的情况了。

解决这个问题的方法有：在 getView 方法中对每个控件调用 **setTag** 方法，设置其标志，在需要用到控件的地方使用 **findViewWithTag** 获得控件，这种方式解决图片乱序的原理是：因为 ListView 中的控件都是可以重用的，当移出屏幕的控件重新被利用的时候会回调 getView 方法，在 getView 方法中会调用 setTag 方法为当前控件设置标志，那么这个标志将会覆盖掉原先设置在该控件上面的标志，这样每次使用 findViewWithTag 获取对应于旧的 Tag 的控件的时候就只能得到 null 了，而只有非 null 的时候我们才会显示图片到控件上面，这样就保证了不会显示旧的图片在控件上面的效果了。



**ListView 优化方案 **

关于 ListView 的优化总结几点

+ 重用 ConvertView，**ConvertView会把之前加载好的布局进行缓存，可以重用，就减少不必要 view 的创建**，因为 inflate 操作也是把 xml 实例化为相应的 View 实例，这个过程是属于 IO 操作，相对来说比较耗时。

+ **减少 findViewById 的次数**，因为即使使用了ConvertView以后不会重复加载布局了，但是每次getView方法回调的时候还是通过findViewById 重复创建控件的实例，那么就通过把 xml 文件中的涉及到的控件封装成内部类**ViewHolder**的方式， 通过 **View 的 setTag 和 getTag 方法将 当前布局View 和对应的 ViewHolder 对象绑定在一起**，减少不必要 的 findViewById 次数。

+ 如果涉及到加载图片之类比较耗时的操作，可以在用户快速滑动屏幕的时候禁止图片的加载，等到滑动停止的时候再去加载图片。

+ ListView 中 Item 的布局层次越简单越好，主要为了避免布局太深带来的重绘太复杂问题，还可以减少半透明元素的绘制，因为半透明更加的耗时。

+ 尽量保证 Adapter 适配器的 hasStables 方法返回 true，这样在调用 notifyDataSetChanged()方法的时候，如果 Item 内容没有发生变化的话，ListView 将不再重绘，以此达到优化。

+ **直接使用 RecycleView 代替 ListView**，每个 Item 自己发生变动，ListView 都会去调用 notifyDataSetChanged 方法更新所有的 Item，未免有点太浪费性能了，而 RecycleView 可以实现每个 Item 的局部刷新，不再需要刷新所有的 Item，同时引入了增加和删除的动画 效果，在性能和定制上有很大改善。

+ 尽量开启硬件加速功能。 

+ 当数据量比较大的时候，可以使用分段加载或者分页加载。

  分段加载。有些情况下需要加载网络中的数据，显示到ListView，而且往往此时都是数据量比较多的一种情况，如果数据有1000条，没有优化过的ListView都是会一次性把数据全部加载出来的，很显然需要一段时间才能加载出来，我们不可能让用户面对着空白的屏幕等好几分钟，那么这时我们可以使用分段加载，比如先设置每次加载数据10条，当用户滑动ListView到底部的时候，我们再加载20条数据出来，然后使用Adapter刷新ListView，这样用户只需要等待10条数据的加载时间，这样也可以缓解一次性加载大量数据而导致OOM崩溃的情况。

  分页加载。上面分段加载也不能完全解决OOM崩溃的情况，因为虽然我们在分段中一次只增加10条数据到List集合中，然后再刷新到ListView中去，假如有10万条数据，如果我们顺利读到最后这个List集合中还是会累积海量条数的数据，还是可能会造成OOM崩溃的情况，这时候我们就需要用到分页，比如说我们将这10万条数据分为1000页，每一页100条数据，每一页加载时都覆盖掉上一页中List集合中的内容，然后每一页内再使用分批加载，这样用户的体验就会相对好一些。



**ListView三级缓存**

+ 内存缓存
+ 磁盘缓存
+ 网络拉取

----





### RecyclerView

[RecyclerView 源码解析](https://juejin.im/entry/586a12c5128fe10057037fba#comment)

[把 RecyclerView 撸成马蜂窝](https://juejin.im/entry/57df94a067f3560056b1b88b#comment)

[RecyclerView缓存机制（咋复用？）](https://juejin.im/post/5c696ba9e51d457f136d24ff#comment)

[深入理解 RecyclerView 的缓存机制](https://juejin.im/post/5eae33a26fb9a043586c7f19#heading-0)



**RecyclerView缓存机制**

Scrap和Cache分别是通过position去找ViewHolder可以直接复用；

在RecyclerView当中也并不是每次都重新创建ViewHolder对象，不是每次都重新绑定ViewHolder数据，而是通过Recycler来获得下一个ViewHolder。

RecyclerView使用Recycler管理缓存ViewHolder，对于不同状态的ViewHolder存储在了不同的集合中，RecyclerView有四级缓存，分别是ArrayList<ViewHolder> **mAttachedScrap**、ArrayList<ViewHolder> **mCachedViews**、**ViewCacheExtension** mViewCacheExtension 和 **RecycledViewPool** mRecyclerPool，缓存的对象是**ViewHolder**。

而从RecycledViewPool 中复用的ViewHolder需要重新绑定数据，并且复用的ViewHolder只能复用于ViewType相同的表项，因为RecycledViewPool 对ViewHolder是按照ViewType分类存储的，RecycledViewPool通过type来获取ViewHolder，获取的ViewHolder是个全新，需要重新绑定数据；ViewCacheExtension自定义缓存，目前来说应用场景比较少却需慎用；从mCachedViews中复用ViewHolder只能复用于指定位置的item；而从mAttachedScrap复用的ViewHolder不需要重新创建也不需要重新绑定数据。

如果四个层级的缓存都没有命中，才会重新创建ViewHolder对象并且绑定。



----



### ListView和RecyclerView的对比



**基础使用**

ListView的适配器最终是继承BaseAdapter类重写方法，自定义**ViewHolder**和**ConvertView**一起完成复用优化工作。RecyclerView 同样也是需要继承重写 RecyclerView.Adapter 和 强制使用RecyclerView.ViewHolder ，以及使用到了LayoutManager。

所以这样看来，在RecyclerView当中与ListView不同之处在于：

+ 对ViewHolder的编写进行了规范化。另外，RecyclerView对 Item的复用 ，关于使用到ViewHolder对控件实例进行缓存的工作已经封装好，不需要像 ListView 那样调用 setTag 。
+ 但是 RecyclerView 多出了 LayoutManager 的设置工作 。



**布局效果**

+ ListView 做到了数据和视图的分离，布局排列是自身去管理。
+ 而RecycleView 将视图和布局进一步分离， 因而出现了 LayoutManager，**RecycleView 只负责管理视图的重复利用，然后将布局的管理全权交给了 LayoutManager**， 不像 ListView 那样被限制在垂直滚动布局样式 。LayoutManager中制定了一套可扩展的布局排列接口，子类只要按照接口规范来实现，就能定制出各种不同排列方式的布局了。 LayoutManager 是一个抽象类，系统已经为我们提供了三个相关的实现类 LinearLayoutManager（线性布局效果）**、**GridLayoutManager（网格布局效果）**、**StaggeredGridLayoutManager（瀑布流布局效果）。  RecyclerView 默认就能支持 **线性布局**、**网格布局**、**瀑布流布局** 三种 ，通过配置和切换 LayoutManager 就可以获得不同的布局效果。所以如果想自定义出更多布局效果，可以继承重写自己的LayoutManager，通过LayoutManager还可以设置滚动方向、获取Item的位置等等。



**缓存机制**

+ ListView的RecycleBin机制
+ RecyclerView的多级别缓存



**事件监听**

+ ListView中提供了setOnItemClickListener、 setOnItemLongClickListener、setOnItemSelectedListener 等几个用于设置监听器，但是**这些只能注册的是子项的点击事件**，具体到子项里面的某个控件的点击事件的处理会比较麻烦，而且添加了 HeaderView 和 FooterView ，ListView 会把 HeaderView 和 FooterView 算入 position 内，代码有可能会报数组越界的错误 。 
+ 但是**RecyclerView ，它并没有像 ListView 提供太多关于 Item 的某种事件监听，需要我们自己通过ViewHolder去给子项具体的View去注册点击事件，或者就是通过 addOnItemTouchListener  和GestureDetector手势判断来实现**。



**空数据的处理**

+ ListView 提供了 **setEmptyView** 这个 API 来让我们处理 Adapter 中数据为空的情况。
+ 但是 RecyclerView 并没有提供这类 API ，所以需要自己处理。



**HeaderView 和 FooterView**

+ 在 ListView 的设计中，addFooterView 、 addFooterView  可以添加HeaderView 和 FooterView 两种类型的视图 。
+ RecyclerView 并没有提供这类 API。



**局部刷新**

+ 更新了 ListView 的数据源后需要通过 Adapter 的 **notifyDataSetChanged** 方法来通知视图更新变化，调用简单，但它**会重绘每个 Item**，实际上却并不是每个 Item 都需要重绘。 

+ 而RecyclerView.Adapter 提供了 **notifyItemChanged** 用于更新单个 Item View 的刷新，我们可以省去自己写局部更新的工作。 



**动画效果**

RecyclerView 支持子项目层次的动画效果，是通过 ItemAnimation 接口（确定一下）实现的， 可以在 Adapter 中的数据发生变化的时候，通过调用 Adapter 的相关方法激活动画的产生，就是RecyclerView 在做局部刷新等等的时候有一个渐变的动画效果 。 



**嵌套滚动机制**

[掘金](https://juejin.im/entry/57a7f5228ac247005f366109#comment)



