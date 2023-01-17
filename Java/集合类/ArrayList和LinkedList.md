## ArrayList和LinkedList



### ArrayList

- 底层结构

	- 数组

- 主要操作

  - 初始化

    无参数直接初始化

    指定大小初始化

    指定初始数据初始化

  - 添加元素与扩容

    - 添加元素

      判断是否需要扩容，如果需要，就执行扩容操作；

      扩容完成之后，赋值是非常简单的，直接往数组上添加元素即可：elementData [size++] = e。也正是通过这种简单赋值，没有任何锁控制，所以这里的操作是**线程不安全**的。

      新增时，并没有对值进行严格的校验，所以 ArrayList 是**允许 null** 值的。

    - 扩容

      扩容的规则是原来容量大小 + 容量大小的一半，直白来说，扩容后的大小是原来容量的**1.5 倍**；

      ArrayList 中的数组的最大值是 **Integer.MAX_VALUE - 8**，超过这个值，JVM 就不会给数组分配内存空间了。

      本质上是，扩容会先新建一个符合我们预期容量的**新数组**，然后把老数组的数据拷贝过去，我们通过 **System.arraycopy** 方法进行拷贝，此方法是 native 的方法。

  - 删除

    - 删除方式

      - 批量删除
      - 根据值删除

        从源码中可以看出，某元素被删除后，为维护数组结构，会把数组后面的元素往前移动。

      - 数组索引删除

    - 注意

      新增的时候是没有对 null 进行校验的，所以删除的时候也是允许删除 null 值的；

      找到值在数组中的索引位置，是通过 **equals** 来判断的，如果数组元素不是基本类型，需要我们关注 equals 的具体实现。

  - 迭代器

    - java.util.Iterator 

      - 常用参数

        int cursor;// 迭代过程中，下一个元素的位置，默认从 0 开始。

        int lastRet = -1; // 新增场景：表示上一次迭代过程中，索引的位置；删除场景：为 -1。

        int expectedModCount = modCount;// expectedModCount 表示迭代过程中，期望的版本号；modCount 表示数组实际的版本号。

      - 常用方法

        hasNext 还有没有值可以迭代

        next 如果有值可以迭代，迭代的值是多少。从源码中可以看到，next 方法就干了两件事情，第一是检验能不能继续迭代，第二是找到迭代的值，并为下一次迭代做准备（cursor+1）。

        remove 删除当前迭代的值。lastRet = -1 的操作目的，是防止重复删除操作；删除元素成功，数组当前 modCount 就会发生变化，这里会把 **expectedModCount 重新赋值**，下次迭代时两者的值就会一致了。

  - 时间复杂度

    从新增或删除方法的源码解析，对数组元素的操作，只需要根据数组索引，直接新增和删除，所以时间复杂度是 O (1)。

  - 线程安全

    - 线程安全问题

      ArrayList 有线程安全问题的本质，是因为 ArrayList 自身的 elementData、size、modConut 在进行各种操作时，**都没有加锁**，而且这些变量的类型**并非是可见（volatile）**的，所以如果多个线程对这些变量进行操作时，可能会有值被覆盖的情况。

    - 改善解决

      类注释中推荐我们使用 **Collections#synchronizedList** 来保证线程安全，SynchronizedList 是通过在每个方法上面加上锁来实现，虽然实现了线程安全，但是性能大大降低。

      关于synchronized及其优化 http://cmsblogs.com/?p=2071

- 以上内容概括

	- 允许 put null 值，会自动扩容；
	- size、isEmpty、get、set、add 等方法时间复杂度都是 O (1)；
	- 是非线程安全的，多线程情况下，推荐使用线程安全类：Collections#synchronizedList；
	- 增强 for 循环，或者使用迭代器迭代过程中，如果**数组大小被改变，会快速失败**，抛出异常。

- 其他注意点

  - [增强for循环](https://juejin.im/post/5b84b858e51d45388a5b5d0d)

  - 扩容本质

- 总结

  ArrayList 其实就是围绕底层数组结构，各个 API 都是将**对数组操作**进行封装。



```java
//Java10
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    
    //省略。。。
    
    //默认容量大小
    private static final int DEFAULT_CAPACITY = 10;

    //空数组常量，有参构造方法传入0，会使用这个数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //默认空数组常量，无参构造方法会使用这个数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //存放元素的数组，从这可以发现ArrayList的底层实现就是一个Object数组
    transient Object[] elementData; 

    //数组中包含的元素个数
    private int size;
    
    //数组的最大上限
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    //这里为什么-8呢? 超过这个值，JVM 就不会给数组分配内存空间了。
    //注释里说某些VM在数组中保留一些标头字，尝试分配更大的数组可能会导致OutOfMemoryError：
    //请求的数组大小超出了VM限制
    //至于为什么一定是8这个数字，可能是关于代码编译以后在内存中运行的原理相关，或者是作者根据感觉写的
    
    /**
     * 构造一个初始容量为10的空列表。
     * 无参构造方法会使用上面的默认空数组常量。
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
    /**
     * 当我们指定了初始大小的时候，elementData的初始大小就是我们指定的初始大小。
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    
    public E get(int index) {
        Objects.checkIndex(index, size);
        return elementData(index);
        //没有越界，直接下标取值
    }
    
    E elementData(int index) {
        return (E) elementData[index];
    }
    
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
    
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;//添加到尾部
        size = s + 1;
    }
    
    private Object[] grow() {
        return grow(size + 1);
    }
    
    private Object[] grow(int minCapacity) {
        //调用Arrays.copyOf将元素拷贝到一个新的数组中去
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }
    
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍
        //而如果这个检查的过程中判断数组是用无参构造时使用的默认空数组，
        //就会使用默认的容量大小10来进行扩容
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
    
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        //调用一个native的复制方法，把index位置开始的元素都往后挪一位
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
    
    
    public E set(int index, E element) {
        Objects.checkIndex(index, size);
        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
    
    public E remove(int index) {
        Objects.checkIndex(index, size);
        final Object[] es = elementData;

        @SuppressWarnings("unchecked") E oldValue = (E) es[index];
        fastRemove(es, index);

        return oldValue;
    }
    
    private void fastRemove(Object[] es, int i) {
        modCount++;
        final int newSize;
        if ((newSize = size - 1) > i)
            System.arraycopy(es, i + 1, es, i, newSize - i);
        //也是利用System.arraycopy来移动元素
        es[size = newSize] = null;
    }
    
    public int size() {
        return size;//直接返回数组中元素个数，不是数组的大小
    }
    
    //返回第一个等于给定元素的值的下标，通过遍历比较数组中每个元素的值来查找
    //时间复杂度为O(n)
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    
    //和indexOf方法一样，只是它是从后往前找。
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

}
```



### LinkedList

- 整体架构

  - 链表每个节点我们叫做 Node：Node 有 prev 属性，代表前一个节点的位置；next 属性，代表后一个节点的位置。

  - first 是双向链表的头节点，它的前一个节点是 null。
  - last 是双向链表的尾节点，它的后一个节点是 null。
  - 当链表中没有数据时，first 和 last 是同一个节点，前后指向都是 null；
  - 因为是个**双向链表**，只要机器内存足够强大，是没有大小限制的。

- **主要操作**

  - 新增

    + 追加尾部（默认）

    + 追加头部
    + 指定位置

  - 删除

  	- 头部删除（默认）
  	- 尾部删除

  - 查询

    链表查询某一个节点比较慢，需要挨个循环查找。

    但是，从源码中可以发现，LinkedList 并没有采用从头循环到尾的做法，而是采取了**简单二分法**，首先确定 index 在链表的前半部分还是后半部分。前半部分则从头开始寻找，反之亦然。通过这种方式，使循环的次数至少降低了一半，提高了查找的性能。

- 方法对比

  LinkedList 实现了 **Queue** 接口，在新增、删除、查询等方面也增加了新的方法，这些方法容易混淆，在链表为空的情况下，返回值也不太一样。（与作为队列的主要方法对比）

  - 新增

    add（e）

    offer（e）

    底层实现相同

  - 删除

    remove（）

    poll（）

    链表为空时，remove 会抛出异常，poll 返回 null。

  - 查找

    element（）

    peek（）

    链表为空时，element 会抛出异常，peek 返回 null。

- 迭代器

  因为 LinkedList 要实现双向的迭代访问，所以我们使用 Iterator 接口肯定不行了，因为 **Iterator 只支持从头到尾的访问**。Java 新增了一个迭代接口，叫做：**ListIterator，这个接口提供了向前和向后的迭代方法**！（这并不是LinkedList独有）

  - 迭代访问

    从前往后

    从后往前

  - 迭代删除

    如果**迭代的同时进行非迭代删除，会快速失败**，抛出异常。

- 总结

  LinkedList 适用于有序、并且会按照顺序进行迭代的场景，主要是依赖于底层的**链表结构**。相比数组，链表的特点就是在指定位置插入删除元素的效率较高，但是查找的效率就不如数组那么高了。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
    
    //省略。。。
    
    //链表的结点个数
    transient int size = 0;

    //指向头结点的指针
    transient Node<E> first;

    //指向尾结点的指针
    transient Node<E> last;
    
    //结点结构
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    //添加元素
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    //在指定位置插入节点
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
    
    void linkLast(E e) {
        final Node<E> l = last;
        //当前结点的前趋指向尾结点，后继指向null
        final Node<E> newNode = new Node<>(l, e, null);
        //尾指针指向新的尾结点
        last = newNode;
        //如果原来有尾结点，则更新原来结点的后继指针，否则更新头指针
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;//指定结点的前趋
        //当前结点的前趋为指定结点的前趋，后继为指定的节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;//更新指定结点的前趋为当前结点
        //更新当前结点的后继
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        //size，以及版本号modCount都变化
        size++;
        modCount++;
    }
    
    public void addFirst(E e) {
        linkFirst(e);
    }
    
    private void linkFirst(E e) {
        final Node<E> f = first;
        //当前节点的前驱指向null，后继指针指向原来的头结点
        final Node<E> newNode = new Node<>(null, e, f);
        //头指针指向新的头结点
        first = newNode;
        //如果原来有头节点，则更新原来节点的前驱指针，否则更新尾指针
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
    
    public void addLast(E e) {
        linkLast(e);
    }
    
    public E remove() {
        return removeFirst();
    }
    
    //删除表头元素
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    
    //删除表头结点，返回表头元素的值
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;//头指针指向后一个结点
        if (next == null)
            last = null;
        else
            next.prev = null;//新头结点的前趋为null
        size--;
        modCount++;
        return element;
    }
    
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
    //删除表尾元素，返回表尾元素的值
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;//尾指针指向前一个节点
        if (prev == null)
            first = null;
        else
            prev.next = null;//新尾结点的后继为null
        size--;
        modCount++;
        return element;
    }
    
    //删除指定元素
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {//
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    //删除指定结点，返回指定元素的值
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;//当前结点的后继
        final Node<E> prev = x.prev;//当前结点的前趋

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;//更新前趋结点的后继为当前结点的后继
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;//更新后继结点的前趋为当前结点的前趋
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
    //获取指定下标元素
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    
    Node<E> node(int index) {
        // assert isElementIndex(index);
        
        //根据下标是否超过链表长度的一半，来选择从头开始遍历还是从尾部开始遍历
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)//从前往后找
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)//从后往前找
                x = x.prev;
            return x;
        }
    }
    
    //获取表头元素
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
    
    //获取表尾元素
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
    
    
    //替换指定下标的值
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
   
    
    
    public int size() {
        return size;
    }
    
}
```



### ArrayList 和 LinkedList 比较

+ 底层结构
+ 扩容
+ 增加、删除、修改、获取