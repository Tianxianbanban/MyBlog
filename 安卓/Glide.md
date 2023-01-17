## Glide



**基本使用**

```java
Glide.with(this).load(url).into(imageView);

Glide.with() 传入的实例会决定Glide加载图片的生命周期 ;
load()用于指定待加载的图片资源，可以加载本地图片、应用资源、二进制流、Uri对象等等;
into()方法传入View的实例，让图片显示在这个空间上面。

添加占位图:
Glide.with(this)
    .load(url)
    .placeholder(R.drawable.loading)//
    .into(imageView);

异常占位图:
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)//
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);

指定图片格式:
Glide.with(this)
     .load(url)
     .asBitmap()//
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);

Glide.with(this)
     .load(url)
     .asGif()//
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);

指定图片大小:设置图片加载成多大尺寸无论ImageView的大小
Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .override(100, 100)//
     .into(imageView);

禁用内存缓存:
skipMemoryCache(true);//

磁盘缓存相关设置:
diskCacheStrategy(DiskCacheStrategy.NONE) ;//禁用
DiskCacheStrategy.NONE： 表示不缓存任何内容。
DiskCacheStrategy.SOURCE： 表示只缓存原始图片。
DiskCacheStrategy.RESULT： 表示只缓存转换过后的图片（默认选项）。
DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。
```





**使用Glide的好处**

+ 基本的处理：图片压缩、缓存机制、内存管理等。
+ 除此之外，**Glide对Bitmap的管理是跟随生命周期去发生改变的**。其它的框架基本都是用LRU算法，当Activity销毁的时候不会释放之前加载图片占用的内存。而Glide的优势就是当Activity销毁的时候，之前加载的所有图片的内存都释放了。



**从源码中看基本流程**

+ with() 

  Glide会根据我们传入with()方法的参数来确定图片加载的生命周期。如果在Glide.with()方法中传入的是一个Application对象，Glide就不需要做什么特殊的处理，它自动就是和应用程序的生命周期是同步的，如果应用程序关闭的话，Glide的加载也会同时终止；如果传入非Application参数的情况，则会向当前的Activity当中添加一个隐藏的Fragment。

  因为Glide需要知道加载的生命周期，可是Glide并没有办法知道Activity的生命周期，于是Glide就使用了**添加隐藏Fragment**的这种小技巧，因为Fragment的生命周期和Activity是同步的，如果Activity被销毁了，Fragment是可以监听到的，这样Glide就可以**捕获这个事件并停止图片加载**了。但也不是无法知道Activity的生命周期，因为看Glide的实现上来看，通过注入一个监听器来观察生命周期的，在Activity中也能实现，而是**Glide要在Activity与Fragment中都能使用，Fragemnt作为共性部分才被作为监听生命周期的**，同时我觉得Glide也因为如果监听了Activity 势必要新建一个BaseActivity让用户继承，这样代码的侵入性就多了，所以不适合。

  Glide就是通过ReqeustManager监听到Fragment的生命周期，从而根据生命周期管理让Request做出相对应的请求。

  （with方法返回的是一个RequestManager对象）

+ load()

  Glide是支持图片URL字符串、图片本地路径等等加载形式的，因此RequestManager中也有很多个load()方法的重载。DrawableTypeRequest的父类DrawableRequestBuilder当中定义的就是我们常用的一些方法比如placeholder()、error()、diskCacheStrategy()、override()等等，以及into()方法。

  （load方法返回一个DrawableTypeRequest对象）

+ into()

  内部调用的buildRequest方法用来构造用来发出加载图片请求的Request对象的，而它调用的buildRequestRecursive方法主要是处理缩略图，buildRequestRecursive内部继续调用obtainRequest方法，这个方法继续调用GenericRequest的obtain()方法，需要传入非常多的参数，其中很多的参数比如placeholderId、errorPlaceholder、diskCacheStrategy等等是使用Glide常用的方法相关，也就是说load()方法中调用的所有API，其实都是在这里组装到Request对象当中的。这一个步骤是构建Request对象。

  而Request对象的执行是下一个步骤requestTracker.runRequest(request)方法的执行，其中调用request.begin()会处理一些占位的图的加载等，而对于我们目的图片的加载，如果前面使用了override() API为图片指定了一个固定的宽高就会调用onSizeReady()方法；如果没指定的话就会调用target.getSize()方法根据ImageView的layout_width和layout_height值做一系列的计算，来算出图片应该的宽高，当然了在计算完之后，它也会调用onSizeReady()方法。

  （into方法返回的是一个Target对象）



**缓存机制**

+ 内存缓存
+ 磁盘缓存



**相关设计模式**

+ 单例模式

  with方法中RequestManagerRetriever 对象的获得。

+ 工厂模式

  使用Build的相关方法进行配置创建对象。

+ 建造者模式

+ 外观模式

+ 策略模式



**参考文章**

+ [Glide生命周期如何实现](https://juejin.im/post/5e61b44451882549052f500c#heading-0)
+ [郭霖_Glide源码解析](https://blog.csdn.net/guolin_blog/article/details/53939176)