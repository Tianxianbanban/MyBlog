## Android数据持久化



### **Android存储数据的几种方式**

+ 使用 SharedPreferences 存储：底层实现原理是基于 XML 文件存储的 key-value 键值对。
+ 文件存储：Context 提供了两个方法来打开数据文件里面的 IO 流 FileInputStream openFileInput(String name) 和 FileOutputStream  openFileOutput(String name,int mode)，可以用这两个方法将数据存储到文件中(记得关闭流)  。
+ 使用 SQLite 存储数据，作为轻量级嵌入式数据库，当然可以进行数据的存储了， 但是 SQLite 缺点在于只能在同一程序中共享存储的数据。
+ 如果想在跨进程之间共享存储的数据该，就可以使用 ContentProvider：因为 ContentProvider 的底层实现是 Binder，所以是适合进程间数据共享的，ContentProvider 内部是通过表格的方式来组织数据，有点类似于SQLite 数据库，但是 ContentProvider 对底层数据存储方式没有任何要求，每个应用程序对外都会提供一个公共的 URI 对象，如果某个应用程序有数据需要共享的时候，首先在该应用程序中会为这些数据定义一个 URI，如果另外一个应用程序想要拿到这个应用程序的共享数据的话，就会通过 ContentProvider 传入这个 URI 来获取到数据，具体是怎么获取数据的就是通过 ContentResolver 对象来进行 Insert、Update 等等的操作了。 
+ 网络数据存储。（前面四种都是本地数据存储方式） 



### **SharedPreferences**

[请不要滥用SharedPreference](https://cloud.tencent.com/developer/article/1578441)

[一眼看穿 SharedPreferences](https://juejin.im/post/5c34615bf265da614171bf8a#comment)

[全面剖析SharedPreferences](http://gityuan.com/2017/06/18/SharedPreferences/)



### **文件存储**

**参考资料**

+ [Android存储用例和最佳做法](https://developer.android.com/training/data-storage/use-cases?hl=zh-cn)
+ [浅谈Android文件存储](https://juejin.cn/post/6844903552683343880#comment)

**内部存储**

+ 内部存储目录**/data/data/PackageName**

  + /data/data/PackageName/shared_prefs/

    存储SP文件

  + /data/data/PackageName/databases/

    存储数据库文件

  + /data/data/PackageName/xxxwebviewcachexxx（系统版本不同名字和位置也不同）

    存储应用内置webview产生的cache和cookies等。

  + 其他

+ 内部存储的特点

  存储空间较小。仅对应用可见，适合存储一些私密信息比如登录信息等。应用卸载，这部分数据也会随之删除。但是应用对这部分内容的读取速度会高于外部存储。并且，这部分数据仅应用本身可以访问，也就**不需要向用户申请读写权限**。

**外部存储**

+ 外部存储中的文件可以被用户和所有应用程序访问和修改，因此这部分空间的读写系统要求用户给予权限。
  + **Manifest.permission.READ_EXTERNAL_STORAGE**
  + **Manifest.permission.WRITE_EXTERNAL_STORAGE**

+ 文件分区

  + 公共文件目录

    用来存放所有APP可能会用到的文件，一般可以和其他应用共享的文件就可以存在在这里了，`Environment.getExternalStoragePublicDirectory(int type)`可以根据type返回对应文件目录。其中type类型有：

    + Environment.DIRECTORY_DCIM

      外部存储根文件夹下的DCIM文件夹，也就是手机存储照片的地方。

    + Environment.DIRECTORY_DOWNLOADS

    + Environment.DIRECTORY_MUSIC

    + Environment.DIRECTORY_MOVIES

  + 应用专属文件目录 ，也会随着应用卸载删除数据**/storage/emulated/0/Android/data/PackageName/**，文件目录和内部存储有些相似，可以使用的API也是相似的。

    + `Context.getExternalFilesDir(String type)`：data/PackageName/files文件夹

      type和Environment.getExternalStoragePublicDirectory(int type)用法类似，但是可以为空，getContext().getExternalFilesDir(null)就是files根文件夹 data/packageName/files。也可以传一个值： getContext().getExternalFilesDir("apple")：此时就是files文件夹下的子文件夹：Android/data/packageName/files/apple。

    + `Context.getExternalCacheDir()`：data/PackageName/cache文件夹 

  + 其他存储设备，例如SD卡存储/sdcard

**常用API**

+ [快速理解Android文件存储路径](https://juejin.cn/post/6844903778227847182)

**分区存储**

Android10对存储机制作出了一定的修改，如果要在Android10以及更高的版本上使用原始文件路径的API，必须在 AndroidManifest 文件中添加`requestLegacyExternalStorage=true`。



### **SQLite**



### **ContentProvider**



### **网络存储**