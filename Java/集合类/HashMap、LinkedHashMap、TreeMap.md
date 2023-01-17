## HashMap、LinkedHashMap、TreeMap



### HashMap

#### 底层结构

- 数组

  把数组中的每一个节点称为一个桶，哈希表中的每个节点都用来存储一个键值对。在插入元素时，如果发生冲突（即多个键值对映射到同一个桶上）的话，就会通过链表的形式来解决冲突。因为一个桶上可能存在多个键值对，所以在查找的时候，会先通过 key 的哈希值先定位到桶，再遍历桶上的所有键值对，找出 key 相等的键值对，从而来获取 value。

- 链表

  当链表的长度 >= **8** 时，链表会转化成红黑树；

- 红黑树

  当红黑树的大小 <= **6** 时，红黑树会转化成链表。

#### 常见属性

其中，影响 HashMap 性能的两个重要参数：“initial capacity”（初始化容量）和”load factor“（负载因子）。简单来说，容量就是哈希表桶的个数，负载因子就是键值对个数与哈希表长度的一个比值，当比值超过负载因子之后，HashMap 就会进行 rehash操作来进行扩容。

#### 主要操作

- 查找

- 散列

  https://www.bbsmax.com/A/VGzlaMD8Jb/

- 新增

  - **链表的新增**

    链表的新增比较简单，就是把当前节点追加到链表的尾部，和 LinkedList 的追加实现一样的。

    链表长度大于等于8时转化成红黑树（空间换时间的改进方式）。

    链表查询的时间复杂度是 O (n)，红黑树的查询复杂度是 O (log (n))。在链表数据不多的时候，使用链表进行遍历也比较快，只有当链表数据比较多的时候，才会转化成红黑树，但红黑树需要的占用空间是链表的 2 倍，考虑到转化时间和空间损耗，所以我们需要定义出转化的边界值。**==考虑设计 8 这个值的时候，参考了泊松分布概率函数==**，由泊松分布中得出结论，参考链表各个长度的命中概率，尤其是当链表的长度是 8 的时候，出现的概率是 0.00000006，不到千万分之一。所以说正常情况下，链表的长度不可能到达 8 ，而**一旦到达 8 时，肯定是 hash 算法出了问题**，所以在这种情况下，为了让 HashMap 仍然有较高的查询性能，所以让链表转化成红黑树。

  - **红黑树新增结点过程**

    1. 首先判断新增的节点在红黑树上是不是已经存在，判断手段有如下两种：
       + 1.1. 如果节点没有实现 Comparable 接口，使用 equals 进行判断；
       + 1.2. 如果节点自己实现了 Comparable 接口，使用 compareTo 进行判断。

    2. 新增的节点如果已经在红黑树上，直接返回；不在的话，判断新增节点是在当前节点的左边还是右边，左边值小，右边值大；

    3. 自旋递归 1 和 2 步，直到当前节点的左边或者右边的节点为空时，停止自旋，当前节点即为我们新增节点的父节点；

    4. 把新增节点放到当前节点的左边或右边为空的地方，并于当前节点建立父子节点关系；

    5. 进行着色和旋转，结束。

    总结

    红黑树的新增，要求我们对红黑树的数据结构有一定的了解。但我们要清楚着色指的是给红黑树的节点着上红色或黑色，旋转是为了让红黑树更加平衡，提高查询的效率，总的来说都是为了满足红黑树的 5 个原则：（1）节点是红色或黑色；（2）根是黑色；（3）所有叶子都是黑色；（4）从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点；（5）从每个叶子到根的所有路径上不能有两个连续的红色节点；（6）空间换时间。

  - **扩容**

    if (++size > threshold)   resize()；如果 HashMap**==键值对个数==**大于扩容的门槛，就开始扩容。

    - 计算扩容后大小：如果数组容量超过最大容量，则无法进行扩容；没有超过则扩大为原来的两倍。

    - 设置新的扩容阈值。
    - 创建新的哈希表：遍历旧哈希表的每个桶，重新计算桶里元素的新位置。如果桶上只有一个键值对，就直接插入；如果是通过红黑树来处理冲突的，则调用相关方法把树分离开；如果采用链式处理冲突，也是通过二次哈希的方式计算节点新位置。

- 引申注意点

  - 哈希算法

  - equals方法和hashCode方法

    Object类当中hashcode方法的初衷是为了散列，而equals方法原本是通过**比较内存地址**来比较对象是否相等。如果为了比较对象逻辑上是否相同，才需要重写equals方法。而**重写equals方法，如果不重写hashcode方法，逻辑上相同的对象会有不同的哈希值，会造成两个方法的矛盾**。

    为什么重写equals还要重写hashCode：因为在某些容器比如Map中，对于Map来说管理键值对时判断两个键是否相同是**既要考虑hash码是否相同，又要考虑equals方法比较是否返回为true**，所以我们在写类的时候如果要重写equals方法就说明我们自己制定了一种规则来比较两个对象是否相等，为了让Map可以按照我们的规则来管理键值对，我们需要重写hashcode方法，让hashcode与equals方法逻辑保持一致。

    Map为什么既要比较hash又要使用equals方法比较呢：比如在HashMap当中插入键值对操作的时候会先判断是否有重复的键值，就是先比较hash值的再通过==比较内存地址或者equals方法比较，因为hash值在之前已经计算出了，在这个地方先比较这个值是否相等速度更快，如果hash值不等，根据&&的计算规则，后面的判断也就不用进行了；另外，hash值相等，equals比较还不一定相等。

    [重写 equal() 时为什么也得重写 hashCode() 之深度解读 equal 方法与 hashCode 方法渊源](https://juejin.im/entry/57aae2042e958a0066c99216?tdsourcetag=s_pctim_aiomsg)

  - jdk1.7和jdk1.8的区别

    JDK1.8加入了红黑树，为了提高查找效率；因为在JDK1.7时只有链表，存放的数据一多就容易导致链表变长。

    JDK1.7的时候，HashMap存放数据采用的是头插法，因为作者认为后来的值被查找的可能性更大一些，可以提升查找的效率，但是容易出现逆序且环形链表死循环问题；JDK1.8改为尾插法。

- 概括

  - 允许 null 值，不同于 HashTable ，是线程不安全的；
  - load factor（影响因子） 默认值是 0.75， 是均衡了时间和空间损耗算出来的值，较高的值会减少空间开销（扩容减少，数组大小增长速度变慢），但增加了查找成本（hash 冲突增加，链表长度变长）。
  - 如果有很多数据需要储存到 HashMap 中，建议 HashMap 的容量一开始就设置成足够的大小，这样可以防止在其过程中不断的扩容，影响性能；
  - HashMap 是非线程安全的，我们可以自己在外部加锁，或者通过 Collections#synchronizedMap 来实现线程安全，它的实现是在每个方法上加上了 synchronized 锁；
  - 在迭代过程中，如果 HashMap 的结构被修改，会**快速失败**。

- 总结

  - HashMap 的内容较多，但大多数 api 是对数组 + 链表 + 红黑树这种数据结构进行封装。

#### 源代码

```java
//Java8
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
  
  //默认的初始容量为 16 
	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
 
	//最大的容量上限为 2^30 
	static final int MAXIMUM_CAPACITY = 1 << 30; 
	 
	//默认的负载因子为 0.75 
	static final float DEFAULT_LOAD_FACTOR = 0.75f; 

 
	//变成树型结构的临界值为 8 ，桶上的链表长度 >= 8时，链表转化成红黑树
	static final int TREEIFY_THRESHOLD = 8; 
 
	//恢复链式结构的临界值为 6 ，桶上的红黑树大小 <= 6时，红黑树转化成链表
	static final int UNTREEIFY_THRESHOLD = 6; 
 
	//哈希表 
	transient Node<K,V>[] table; 
 
	//哈希表中键值对的个数 
	transient int size; 
 
	//哈希表被修改的次数 
	transient int modCount; 
	 
	//它是通过 capacity * load factor 计算出来的，当 size 到达这个值时，就会进行扩容操作 
	int threshold; 
 
	//负载因子 
	final float loadFactor; 
 
  //当数组容量大于这个阈值 64 时，链表才会转化成红黑树，否则仅采取扩容来尝试减少冲突
	static final int MIN_TREEIFY_CAPACITY = 64;
    
   //键值对的个数，HashMap 的实际大小，可能不准(因为当你拿到这个值的时候，可能又发生了变化)
  transient int size;
    
    transient Node<K,V>[] table;//哈希表，存放数据的数组
    
    
    //链表结点的定义和红黑树结点的定义都有
    /*
    链表结点Node 类的定义，它是 HashMap 中的一个静态内部类，哈希表中的每一个节点都是 Node 类型。
    可以看到，Node 类中有 4 个属性，其中除了 key 和 value 之外，还有 hash 和 next 两个属性
    hash 是用来存储 key 的哈希值的，next 是在构建链表时用来指向后继节点的。 
    */
    static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;

        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final String toString() {
            return key + "=" + valu e;
        }


        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(val ue);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;
                if (Objects.equals(key, e.getKey()) &&
                        Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
    
    
    /*
    下面是一些重要方法
    */
    
    
    //get 方法主要调用的是 getNode 方法
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //如果哈希表不为空 && key 对应的桶上不为空 
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
            //是否直接命中 
            if (first.hash == hash && // always check first node 
                    ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
            //判断是否有后续节点 
            if ((e = first.next) != null) {
                //如果当前的桶是采用红黑树处理冲突，则调用红黑树的 get 方法去获取节点
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode (hash, key);
                //不是红黑树的话，那就是传统的链式结构了，通过循环的方法判断链中是否存在该 key
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
    /*实现步骤大致如下： 
    1、通过 hash 值获取该 key 映射到的桶。 
    2、桶上的 key 就是要查找的 key，则直接命中。 
    3、桶上的 key 不是要查找的 key，则查看后续节点：  
    （1）如果后续节点是红黑树节点，通过调用红黑树的方法查找该 key。  
    （2）如果后续节点是链式节点，则通过遍历链查找该 key。
    */
    
    
    
    //put方法的具体实现也是在 putVal 方法中
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIf Absent,
                   boolean evict) {
        Node<K, V>[] tab;
        Node<K, V> p;
        int n, i;
        //如果哈希表为空，则先创建一个哈希表 
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;//扩容
        //如果当前桶没有碰撞冲突，则直接把键值对插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K, V> e;
            K k;
            //如果桶上节点的 key 与当前 key 重复，是要找的结点
            了
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
            //如果是采用红黑树的方式处理冲突，则通过红黑树的 putTreeVal 方法去插入这个键值对 
            else if (p instanceof TreeNode)
                e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
                //否则就是传统的链式结构 
            else {
                //采用循环遍历的方式，判断链中是否有重复的 key 
                for (int binCount = 0; ; ++binCount) {
                    //到了链尾还没找到重复的 key，则说明 HashMap 没有包含该键
                    if ((e = p.next) == null) {
                        //创建一个新节点插入到尾部 
                        p.next = newNode(hash, key, value, nul l);

                        //如果链的长度大于 TREEIFY_THRESHOLD 这个临界值，则把链变为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st 
                            treeifyBin(tab, hash);
                        break;
                    }
                    //找到了重复的 key 
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //这里表示在上面的操作中找到了重复的键，所以这里把该键的值替换为新值
            if (e != null) { // existing mapping for key 
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //判断是否需要进行扩容 
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
    /* put 方法比较复杂，实现步骤大致如下： 
   1、先通过 hash 值计算出 key 映射到哪个桶。 
   2、如果桶上没有碰撞冲突，则直接插入。 
   3、如果出现碰撞冲突了，则需要处理冲突：  
   （1）如果该桶使用红黑树处理冲突，则调用红黑树的方法插入。  
   （2）否则采用传统的链式方法插入。如果链的长度到达临界值，则把链转变为红黑树。
   4、如果桶中存在重复的键，则为该键替换新值。 
   5、如果 size 大于阈值，则进行扩容。
   */
    
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //当数组容量大于这个阈值MIN_TREEIFY_CAPACITY 64 时，
        //链表才会转化成红黑树，否则仅采取扩容来尝试减少冲突
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
    
    
    //remove 方法的具体实现在 removeNode 方法中
    public V remove(Object key) {
        Node<K, V> e;
        return (e = removeNode(hash(key), key, null, false, tru e)) == null ?
                null : e.value;
    }

    final Node<K, V> removeNode(int hash, Object key, Object va lue,
                                boolean matchValue, boolean movabl e) {
        Node<K, V>[] tab;
        Node<K, V> p;
        int n, index;
        //如果当前 key 映射到的桶不为空 
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (p = tab[index = (n - 1) & hash]) != null) {
            Node<K, V> node = null, e;
            K k;
            V v;
            //如果桶上的节点就是要找的 key，则直接命中 
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                //如果是以红黑树处理冲突，则构建一个树节点 
                if (p instanceof TreeNode)
                    node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
                    //如果是以链式的方式处理冲突，则通过遍历链表来寻找节点 
                else {
                    do {
                        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //比对找到的 key 的 value 跟要删除的是否匹配 
            if (node != null && (!matchValue || (v = node.value) == value ||
                    (value != null && value.equals(v)))) {
                //通过调用红黑树的方法来删除节点 
                if (node instanceof TreeNode)
                    ((TreeNode<K, V>) node).removeTreeNode(this, t ab, movable);
                    //使用链表的操作来删除节点 
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
    
   
    //在get方法和put方法中都需要先计算key映射到哪个桶上，然后才进行之后的操作
    //映射到哪个桶上就是通过(n - 1) & hash， n 指的是哈希表的大小，hash 指的是 key 的哈希值
    //上面所得到的hash值都是用下面这个方法获得的
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
        //采用了二次哈希的方式，其中 key 的 hashCode 方法是一个 native 方法
    }
    /*
    hash方法先通过 key 的 hashCode 方法获取哈希值，再拿哈希值与它的高 16 位内容做一个异或操作来得到最后的哈希值。
    注释中是这样解释的：
    如果当 n 很小，假设为 64 的话，那么 n-1 即为 63（2^6 - 1），
    这样的值跟 hashCode()直接做与操作，实际上只使用了哈希值的后 6 位。
    如果当哈希值的“高位变化很大，低位变化很小”的情况，这样算很容易造成冲突，
    所以这里“把高低位都利用起来”，从而解决了这个问题。 
    */
    
    
    //扩容的时机：
    //1. 插入的时候，哈希表为空；
  	//2. 或者是超过了扩容阈值;
    //3. 或者链表转成红黑树的过程，判断出数组容量没有超过转红黑树的阈值64，则优先扩容。
    final Node<K, V>[] resize() {
        Node<K, V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //计算扩容后的大小 
        if (oldCap > 0) {
            //如果当前容量超过最大容量，则无法进行扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //没超过最大值则扩为原来的两倍，MAXIMUM_CAPACITY = 1 << 30，DEFAULT_INITIAL_CAPACITY = 1 << 4
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
            oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold 
        } else if (oldThr > 0) // initial capacity was placed in threshold 
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults 
            //无参构造HashMap,第一次put时，就会到这里设置数组容量为默认的16，threshold = 16 * 0.75
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                    (int) ft : Integer.MAX_VALUE);
        }
        //新的 resize 阈值 
        threshold = newThr;
        //创建新的哈希表 
        @SuppressWarnings({"rawtypes", "unchecked"})
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            //遍历旧哈希表的每个桶，重新计算桶里元素的新位置 
            for (int j = 0; j < oldCap; ++j) {
                Node<K, V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //如果桶上只有一个键值对，则直接插入 
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //如果是通过红黑树来处理冲突的，则调用相关方法把树分离开 
                else if (e instanceof TreeNode)
                        ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                //如果采用链式处理冲突 
                else { // preserve order 
                        Node<K, V> loHead = null, loTail = null;
                        Node<K, V> hiHead = null, hiTail = null;
                        Node<K, V> next;
                        //通过上面讲的方法来计算节点的新位置 
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            } else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
    /* HashMap 在进行扩容时，使用的 rehash 方式非常巧妙，
   因为每次扩容都是翻倍，与原来计算（n-1）&hash 的结果相比，只是多了一个 bit 位，
   所以节点要么就在原来位置，要么就被分配到“原位置+旧容量”这个位置。
   正是因为这样巧妙的 rehash 方式，保证了 rehash 之后每个桶上的节点数必定小于等于原来桶上的节点数，
   即保证了 rehash 之后不会出现更严重的冲突。
   */
    //这些操作，也决定了HashMap的大小只能够是2的幂次方。

    
    
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    /*
    返回大于输入参数且最近的2的整数次幂的数：
    如果不是2的幂次方，即是创建HashMap的时候指定了初始大小，
    HashMap在构建的时候也会调用tableSizeFor（int cap）来调整大小，
    这个方法实际作用也就是把cap变成一个等于2的幂次方的数。
    */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
    
}
```



-----

### LinkedHashMap

#### 底层结构

从源代码注释中，我们可以先了解到 LinkedHashMap 是通过**哈希表**（继承了 HashMap，拥有了 HashMap 那一套高效的操作）和**链表**实现的，它通过维护一个链表来保证对哈希表迭代时的有序性，而这个有序是指**键值对插入的顺序**。另外，当向哈希表中重复插入某个键的时候，不会影响到原来的有序性。也就是说，假设你插入的键的顺序为 1、2、3、4，后来再次插入 2，迭代时的顺序还是 1、2、 3、4，而不会因为后来插入的 2 变成 1、3、4、2（但其实我们可以改变它的规则，使它变成 1、3、4、2）。

#### 属性

#### 方法

```java
//HashMap中定义了下面三个空方法，
//它们在 LinkedHashMap 中有各自的实现。正是通过重写这三个方法来保证LinkedHashMap的链表的插入、删除的有序性。
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

#### 源代码

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V> {
  	
    /**
    * 链表节点继承了 HashMap 节点，而且包含了前指针和后指针，所以这里可以看出它是一个双向链表.
    */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
    //头指针
    transient LinkedHashMap.Entry<K,V> head;
    
    //尾指针
    transient LinkedHashMap.Entry<K,V> tail;
    
    //默认为false。为true时，表示链表中键值对的顺序与插入顺序一致，重复插入键也会更新顺序。
    //例如，为false时，就是上面例子所指的 1、2、3、4 的情况；为true时，就是 1、3、4、2 的情况。
    final boolean accessOrder;
    
    
    /**
     * 就是把当前节点 e 移至链表的尾部。因为使用的是双向链表，所以在尾部插入可以 O（1）的时间复杂度来完成。
     * 并且只有当 accessOrder设置为 true 时，才会执行这个操作。
     * 在 HashMap 的 putVal 方法中，就调用了这个方法。
     */
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        //当 accessOrder 的值为 true，且 e 不是尾节点
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
    
    
    /**
     * afterNodeInsertion 方法是在哈希表中插入了一个新节点时调用的.
     * 它会把链表的头节点删除掉，删除的方式是通过调用 HashMap 的 removeNode 方法。
     * 想想通过afterNodeInsertion 方法和 afterNodeAccess 方法，是不是就可以简单的实现一个基于LRU的淘汰策略了？
     * 当然，我们还要重写 removeEldestEntry 方法，因为它默认返回的是 false。
     */
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
    
    /**
     * 这个方法是当 HashMap 删除一个键值对时调用的，
     * 它会把在 HashMap 中删除的那个键值对一并从链表中删除，保证了哈希表和链表的一致性。
     */
    void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }
    
    /**
     * get方法很简单，因为它调用的是 HashMap 的 getNode 方法来获取结果的。
     * 并且，如果 accessOrder 设置为 true，那在获取到值之后，还会调用 afterNodeAccess 方法。
     * 这样也能保证一个 LRU 的算法。
     */
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
    
    //...
  
}
```

#### **文章补充**

+ [HashMap](https://juejin.im/post/5e6e0ac9e51d4526fb5df679#heading-0)
+ [LruCache里为什么用LinkedHashMap，HashMap可以吗？](https://juejin.im/post/5e86daef6fb9a03c4c5bba01#comment)



-----

###  TreeMap

#### **关于TreeMap的排序**

+ key需要实现Comparable接口；如果key没有实现Comparable接口，就需要通过设定比较器Comparator，实现compare方法来自定规则排序。
+ TreeMap`不使用`equals()`和`hashCode()，TreeMap在比较两个Key是否相等时，依赖Key的`compareTo()`方法或者`Comparator.compare()`方法。

```java
class Test{
    public void test(){
        //key已经实现了Comparable接口
        Map<String, Integer> map = new TreeMap<>();
        map.put("orange", 1);
        map.put("apple", 2);
        map.put("pear", 3);
        
        //作为Key的class没有实现Comparable接口，那必须在创建TreeMap时同时指定一个自定义排序算法
        Map<Person, Integer> map1 = new TreeMap<>(new Comparator<Person>() {
            public int compare(Person p1, Person p2) {
                return p1.name.compareTo(p2.name);
            }
        });
        map1.put(new Person("Tom"), 1);
        map1.put(new Person("Bob"), 2);
        map1.put(new Person("Lily"), 3);
        
        //TreeMap在比较两个Key是否相等时，
        //依赖Key的compareTo()方法或者Comparator.compare()方法
        Map<Student, Integer> map2 = new TreeMap<>(new Comparator<Student>() {
            public int compare(Student p1, Student p2) {
                return p1.score > p2.score ? -1 : 1;
            }
        });
        map2.put(new Student("Tom", 77), 1);
        map2.put(new Student("Bob", 66), 2);
        map2.put(new Student("Lily", 99), 3);
        System.out.println(map2.get(new Student("Bob", 66))); // 输出null，
        //因为上面的比较当中没有==这个比较
    }
}

class Person {
    String name;
    public Person(String name) {
        this.name = name;
    }
}

class Student extends Person {
    int score;
    public Student(String name, int score) {
        super(name);
        this.score = score;
    }
}

```

#### **文章补充**

[使用TreeMap](https://www.liaoxuefeng.com/wiki/1252599548343744/1265117109276544)

[深入理解HashMap和TreeMap的区别](https://juejin.im/post/5eacb271e51d454da36cf0df#heading-1)

#### **TreeMap和HashMap的区别**

+ 存储结构

  HashMap的存储是通过数组、链表、红黑树实现。

  TreeMap的实现就是红黑树。

+ 排序

  HashMap的输出结果不定。

  TreeMap输出的结果是排好序的。

+ null值处理

  HashMap允许key为null。

  TreeMap也允许key为null，但是要在Comparator中处理好规则。

+ 性能

  HashMap由于通过散列映射到桶，所以在添加、查找、删除会比较快。

  TreeMap在添加、删除过程中会进行排序，对性能有一定影响。

+ 其他

  HashMap的key需要重写equals方法和hashCode方法。
  
  TreeMap的key不需要。
  



----

### HashTable

[HashMap和Hashtable的详细区别](https://juejin.cn/post/6844903925460500487#heading-18)