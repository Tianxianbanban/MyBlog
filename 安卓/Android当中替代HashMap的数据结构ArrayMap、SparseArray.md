## Android当中替代HashMap的数据结构ArrayMap、SparseArray



**HashMap的缺点**

在Android系统中，内存有限，而HashMap占用的内存空间较大，插入键值对需要创建新的结点，而且需要对插入的键值进行自动装箱。因此可以采用时间换空间的方式使用一些更加轻量的数据结构例如ArrayMap或者SparseArray等。



**ArrayMap**

+ [ArrayMap源码解析](https://juejin.im/post/593f5aa8128fe1006afbfc4f#heading-12)

+ [如何通过 ArrayMap 和 SparseArray 优化 Android App](https://github.com/xitu/gold-miner/blob/master/TODO/android-app-optimization-using-arraymap-and-sparsearray.md)

+ 总结

  ArrayMap 采用两个数组来实现，一个数组用于存储 key 值 **hash 之后的顺序列表**，另一个数组存储按 key 的顺序记录的 key-value 值，当你想要获得某个 value 值的时候，ArrayMap 会计算输入 key 转换之后的 hash 值，然后通过二分查找寻找到这个 hash 值在 hash 值数组中的位置 index，有了这个 index 我们便可以在另外一个数组中直接访问到需要的键值对了，如果第二个数组键值对中的 key 和当前输入的 key 不一致则发生了冲突，可以该 key 为中心，上下去查找比对，直到找到匹配的值为止，也就是**开放定址法**。



**SparseArray**

+ [SparseArray源码解析](https://juejin.im/post/59f029e5f265da430e4e5d62#heading-14)

+ 总结

  对于 SparseArray 来说，他的高效性体现在他避免了对 key 和 value 的自动装箱操作，并且避免了装箱之后的解箱操作。

