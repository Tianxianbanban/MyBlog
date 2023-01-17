## 组件：Claymore的思想与实现AND解决了什么问题？



**Service Provider Interface**

+ [Android模块开发之SPI](https://www.jianshu.com/p/deeb39ccdc53)
+ [论面向接口编程](https://juejin.im/post/6844904016116187143#comment)



**文章**

+ [Claymore插件设计——安卓上能用的ServiceLoader](https://study-tech.bytedance.net/articles/2541)
+ [配置构建](https://developer.android.com/studio/build/index.html?hl=zh-cn#build-process)
+ [字节码插桩--你也可以轻松掌握](https://juejin.im/post/6844903795575504904#heading-13)



**相关概念**

+ 编译隔离

  相当于**编译期**让一些代码不可见，不暴露内部实现细节，也可以**避免不合理依赖**，可以通过runtimeonly引入依赖，**会打包到APK运行时使用，但不会添加到编译路径**，所以不会减小APK包体积。

+ 依赖隔离

  依赖隔离指的是implementation 引入依赖，这种依赖方式就是A依赖B，B依赖C，但是A不能直接使用C中的类。

  implementation引入依赖，会添加依赖到编译路径，并且会将依赖打包到输出（aar或apk），但是在编译时不会将依赖的实现暴露给其他module，也就是只有在运行时其他module才能访问这个依赖中的实现。使用这个配置，可以显著提升构建时间，因为它可以减少重新编译的module的数量。



**什么情况下会用到编译隔离？**

比如实现模块化的目标时，需要让模块尽量独立。



**Claymore解决的问题**

能够在编译隔离的情况下发现服务，让业务实现只依赖于service，代码结构更加清晰，却依然会在运行时产生service的实现类。



**Claymore的思想和实现**

+ 中心思想是客户端模块服务化，impl只能依赖service，通过服务发现的方式取对应的实现。
+ Claymore发现服务不是通过反射，而是直接在transform阶段生成构造代码，编译期插桩，生成构造器字节码，然后运行期选一个。

