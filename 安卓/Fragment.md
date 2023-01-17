## Fragment



**Fragment和Activity的关系**

+ Fragment是一种可以嵌入在Activity当中的UI片段，能够让程序更加合理充分的利用大屏幕空间，和Activity很像，可以理解成一个轻量型的Activity，即使视觉上看可能和Activity一样大。
+ Fragment 是 Android3.0 出现的，可以理解为碎片 Activity，它**依托于 Activity 存在**，由 Activity 的 FragmentManager 来管理，也就是它的生命周期受 Activity 影响，只有在 Activity 处于活动状态下才能进行 Fragment 各生命周期的变化，Activity 被销毁，绑定在它上的 Fragment 也会销毁。
+ Fragment 可以**解决多 Activity 的问题**，即可以将 Activity 的跳转，转换为是 Fragment 的切换。
+ Fragment 可以被**重用**，可以在不同的 Activity 中共用同一个 Fragment。
+ Activity 是间接继承自 Context，但是 Fragment**不继承于Context**，所以一个应用中 Context 的个数是不包括 Fragment 个数的。
+ Fragment 里面可以通过 **getActivity** 获得当前 Fragment 所在的 Activity 对 象，在 Activity 中可以通过 FragmentManager 查找它所包含的 Fragment。



**“安卓第五大组件”**

+ 效率更高，UI切换效果更好
+ 有自己的生命周期



**Fragment基本使用**

+ 静态加载：直接将Fragment添加到Activity的布局文件当中
  + 定义**Fragment的子类**；
  + Activity加载布局文件，在布局文件中通过**`<fragment>`**指定自定义Fragment，fragment加上id；
  + Activity需要继承自FragmentActivity；
  + 每个Fragment本质上都会生成一个**FrameLayout**，它加载的布局为其子布局。


+ 动态加载：Activity中代码动态管理

  使用**FragmentManager**和**FragmentTransaction**动态添加一个Fragment ，并且**提交事务**。

  
  + add(viewId, fragment)： 将fragment的视图添加为指定视图的子视图(加在原有子视图的后面)     
  + replace(viewId, fragment)：将fragment的视图添加为指定视图的子视图(先remove原有的子视图)
  + remove(fragment2)：将fragment的视图移除
  
  ```java
  public void showFragment2(View v) {
  		// 创建Fragment对象
  		fragment2 = new MyFragment2();
  		// 得到FragmentManager
  		FragmentManager manager = getSupportFragmentManager();
  		// 得到FragmentTransacation
  		FragmentTransaction transaction = manager.beginTransaction();
  		
  		//将当前操作添加到回退栈, 这样点击back回到上一个状态
  		transaction.addToBackStack(null);
  		
  		// 替换Fragment对象并提交
  		transaction.replace(R.id.ll_main_container, fragment2).commit();
  	}
  ```
  
  

**Fragment与Fragment Fragment与Activity之间的通信**

+ Fragment之间

  对处于同一个 Activity 中的 Fragment 可**借助于他们同在的 Activity 进行通信**，并且涉及到的 Activity 是继承自 **FragmentActivity** 的 Activity，不是平常的 Activity。

  例如如果是静态加载的Fragment之间，可以通过Activity获取FragmentManager然后通过id得到另一个Fragment的实例，去调用对方的方法。

+ Fragment与Activity

  Fragment内可通过 **getActivity** 获得当前 Fragment 所在的 Activity，在 Activity 中可以通过 **FragmentManager** 来管理当前 Activity 所用到的 Fragment。

+ 通用方式

  Handler、广播、EventBus、Bundle和fragment.setArguments(bundle)、接口回调等等。

   [掘金](https://juejin.im/entry/59746af7f265da6c34337d2e)

   [掘金](https://juejin.im/entry/58ca54332f301e007e38085b#comment)



**Fragment生命周期**

Fragment 依托于 Activity存在，**只有在 Activity 处于活动状态时才可进行 Fragment 各生命周期状态的转换**，Activity 一旦销毁它上面所附加的 Fragment 也将被销毁。 Fragment 生命周期所涉及到的方法有：**onAttach、onCreate、onCreateView、 onActivityCreated、 onStart、onResume、onPause、onStop、onDestroyView、onDestroy、 onDetach**。

onAttach 和 onDetach 是一对，表示 **Fragment 被添加到 Activity 中和 Fragment 被从 Activity 移出**；onCreateView 和 onDestroyView 是一对，**在创建 Fragment 视图（返回的View需要是Fragment布局的根视图）和移出 Fragment 视图**时调用；onActivityCreated是在 **Activity 的 onCreate 方法返回时调用**的。

通常在定义 Fragment 的时候都要重写 onCreateView 方法，这个方法的返回值就是我们想要显示的 View ，通常是通过 LayoutInflater 的 inflate 方法加载这个 View 的。

+ 添加Fragment对象显示  onAttach()-->onCreate()-->onCreateView()-->onActivityCreated()-->onstart()-->onResume()
+ home到桌面 ： onPause()-->onStop()
+ 从桌面回到应用 ：  onStart()-->onResume()
+ replace为其它Fragment ：  onPause()-->onStop()-->onDestroyView()
+ 返回到本身的Fragment  ： onCreateView()-->onActivityCreated()-->onstart()-->onResume()
+ 退出应用：   onPause()-->onstop()-->onDestroyView()-->onDestroy()-->onDetach()

**MainActivity()..**
**MainActivity onCreate()..**
**MainActivity onStart()..**
onAttach() Context
onAttach()
onCreate()
onCreateView()
onActivityCreated()
onStart()
**MainActivity onResume()..**
onResume()

**MainActivity onPause()..**
onPause()
**MainActivity onStop()..**
onStop()
**MainActivity onDestroy()..**
onDestroyView()
onDestroy()
onDetach()



**关于Fragment嵌套**



**关于ViewPager**

Fragment正常添加和Viewpager添加的区别？

fragment懒加载原理？

[安卓性能优化之懒加载（Fragment中数据的懒加载）](https://blog.csdn.net/vic6329063/article/details/82838430?tdsourcetag=s_pctim_aiomsg)

FragmentPagerAdapter与FragmentStatePagerAdapter的区别？

+ FragmentPagerAdapter适用于页面较少的情况，而FragmentStatePagerAdapter适于页面较多的情况。FragmentPagerAdapter在页面滑动过程中只是将Fragment移除；FragmentStatePagerAdapter在页面切换过程真正释放了Fragment，是回收内存的，更加节省内存。
