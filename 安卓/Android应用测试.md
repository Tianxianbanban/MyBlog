# Android应用测试

> 在对自己代码有一定自信的前提下对一些易于出错的地方进行完整的测试覆盖，不要盲目地追求测试覆盖率。

### 单元测试环境构建

### 测试哪些内容

#### 边界条件

+ 一致性

  值是否和预期一致。可以理解为当输入并不是预期的标准数据时，被测试方法是否可以正确输出预期结果或抛出异常。例如要实现一个加法功能，有两个EditText分别输入两个整型数字，但当用户输入的不是整型而是文字时如何处理。

+ 有序性

  值是否像期望的那样是无序或有序的。

+ 区间性

  值是否位于合理的最小值和最大值之间，例如，圆的角度为1～360°，当用户设置进来的角度为400°时你如何处理。

+ 依赖性

  代码是否引用了一些不在代码本身控制范围之内的外部资源，当这些外部资源存在或不存在时代码是否可以产生相应的预期结果。例如需要将图片缓存到SD卡中，如果SD卡被移除或者没有SD卡我们做何处理。

+ 存在性

  值是否存在。测试方法是否可以处理值不存在的情况，例如对象为null的情况下得到的结果是什么。

+ 基数性

  是否恰好有足够的值。这里的基数指的是计数，测试方法是否可以正确计数，并检查最后的计数值。

+ 时间性

  所有事情的发生是否是有序的、是否在正确的时刻、是否恰好及时。与时间相关问题有：相对时间（时间上的顺序）、绝对时间（消耗的时间和时钟上的时间）、并发问题。例如，方法调用的时间顺序、代码超时、不同的本地时间、多线程同步等。

#### 覆盖执行路径

> 如果没有单元测试，必须手动单击应用进行测试，这样不仅效率低下、浪费时间，也不能保证你每次都会执行完整的路径覆盖，这样就有可能存在Bug。

### 模拟所需的功能模块-Mock对象

手动Mock对象

> 通过Mock对象的方式避免了一些**耗时的、无法使用**的功能，能够提高测试效率，也能**避免依赖一些特定的环境和功能**，在一定程度上使得开发过程成为**并行**的模式。

使用Mockito库

+ 验证某些行为
+ 做一些测试桩（Stub）
+ 参数匹配器
+ 验证函数的确切调用次数、最少调用、从未调用
+ 确保交互操作没有执行在Mock对象上
+ 简化Mock对象的创建
+ 为连续的调用做测试桩（stub）
+  为回调做测试桩
+ doReturn()、doThrow()、doAnswer()、doNothing()和doCallRealMethod()系列方法的运用
+ 监控真实对象
+ 为下一步的断言捕获参数



### [Android中测试](https://developer.android.com/training/testing/fundamentals?hl=zh-cn)

#### 基本单元测试

在androidTest目录下创建单元测试类，继承自TestCase。

#### 使用Instrumentation测试

+ 需要Context的测试用例

  在Android开发中，Context是一个极为重要的类型，启动Activity、获取资源、获取系统服务等都需要它。我们很多工具类可能也会需要Context进行相关的操作，如获取资源等。Android提供了一个AndroidTestCase来运行与Android平台相关的测试类，它有一个mContext字段，可以通过这个Context来完成测试工作。

+ 测试Activity

+ 测试Service

+ 测试ContentProvider

插桩单元测试



[测试框架](https://developer.android.com/training/testing/unit-testing/local-unit-tests?hl=zh-cn)

+ JUnit：最受欢迎且应用最广泛的 Java 单元测试框架。
+ Robolectric：与多个 Android 框架依赖项互动或以复杂的方式与这些依赖项互动时使用。
  + Robolectric 在本地 JVM 或真实设备上执行真实的 Android 框架代码和原生框架代码的虚假对象。
  + https://juejin.cn/post/6844903982062632967#comment
+ Mockito：
+ [Espresso](https://developer.android.com/training/testing/espresso/recipes?hl=zh-cn)：运行插桩中型测试时使用
+ [Truth](https://google.github.io/truth/) 
+ [UI Automator](https://developer.android.com/training/testing/ui-testing/uiautomator-testing?hl=zh-cn) 

 [Android 平台上的测试驱动型开发](https://www.youtube.com/watch?v=pK7W5npkhho&%3Bstart=111&hl=zh-cn)

 [Espresso](https://developer.android.com/training/testing/espresso?hl=zh-cn) 





### 测试驱动开发（TDD）

TDD的全称叫做Test-Driven Development，也就是测试驱动开发。它是一种软件开发的流程，其由敏捷的“极限编程”引入。它的原理就是先写测试用例，然后再写功能代码，确保所有对外暴露的代码都可测试，并且通过测试。







相关文章

+ [Android单元测试研究与实践](https://tech.meituan.com/2015/12/24/android-unit-test.html)
+ [测试基础知识-官方文档](https://developer.android.com/training/testing/fundamentals)
+ [测试一直都是Android程序员忽视的重要一环](https://mp.weixin.qq.com/s/soDfw_AHSCghpD4mdQO7-w)

相关书籍

+ https://book.douban.com/subject/30675326/
+ https://book.douban.com/subject/25863881/
+ https://book.douban.com/subject/25742200/
+ https://book.douban.com/subject/35079613/