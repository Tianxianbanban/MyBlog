##  Android 中 assets 文件夹与 raw 文件夹的区别 

相同点：两者目录下的文件都会原封不动打包在 apk 文件中，不会被编译成二进制文件。

不同点：

+ res/raw 中的文件会在 R.java 中生成 ID，但是 assets 文件夹中的内容不会在 R.java 中生成 ID；
+ 因为 res/raw 中的文件在 R.java 中有 ID，因此我们可以通过 ID 直接引用资源，但是对于 assets 中的文件只能通过 AssetManager 来处理；
+ res/raw 不可以有目录结构，而 assets 可以有目录结构，也就是可以在 assets 目录下建立文件夹； 

