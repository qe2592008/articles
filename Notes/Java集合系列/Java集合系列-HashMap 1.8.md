# Java集合系列-HashMap 1.8
## 一、概述
    HashMap是基于哈希实现的映射集合。
    HashMap可以拥有null键和null值，但是null键只能有一个，null值不做限制。HashTable是不允许null键和值的。
    HashMap是非线程安全的集合，HashTable是添加了同步功能的HashMap，是线程安全的。
    HashMap是无序的，并不能保证其内部键值对的顺序。
    HashMap提供了常量级复杂度的元素获取和添加操作（当然是在hash分散均匀的情况下）。
    HashMap有两个影响功能的因素：初始容量与负载因子，当集合中的元素数量超过了初始容量和负载因子的乘积值时，会触发resize扩容
    HashMap默认的初始容量是16，负载因子是0.75
    HashMap在链表添加元素是采用尾插法，之前的版本采用头插法，因为会导致循环链表的问题，改成了尾插法，并添加了红黑树来优化链表过长的情况下查询慢的问题
    HashMap底层结构为数组+链表/红黑树
    HashMap底层链表的元素达到8个的情况下，如果HasnMap内部桶个数（即桶容量）达到64个则进行树形化，否则进行resize扩容
## 二、常量/变量解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    // 默认的初始容量，值为16，如果自定义也必须为2的次幂
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量值为2的30次幂，就是0100 0000 0000 0000 0000 0000 0000 0000 共32位
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的负载因子，值为0.75，可自定义，必须小于1
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 链表转树结构的元素数量界限值，当某个hash值下的链表元素个数达到8个，
    // 则将其改为树结构
    static final int TREEIFY_THRESHOLD = 8;
    // 红黑树反转链表的元素数量界限值，当数量小于6的时候才会反转
    static final int UNTREEIFY_THRESHOLD = 6;
    // 树形阈值，这个阈值针对的是整个Map中桶的数量，表示只有在所拥有的桶数量
    // 达到64时才能执行树形化，否则先去扩容去吧，可见在桶数小于64时，优先执行扩容
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 表示桶数组
    transient Node<K,V>[] table;
    // 缓存
    transient Set<Map.Entry<K,V>> entrySet;
    // 集合中元素的数量，
    transient int size;
    // 集合结构的修改次数，包括集合元素的增删，和集合结构的变化，仅仅更改已有
    // 元素的值并不会增加该值，主要用于Iterator的快速失败
    transient int modCount;
    // 集合的扩容阈值
    int threshold;
    // 集合的负载因子，默认0.75，时间与空间的折中，增加负载因子，
    // 能增加元素容纳量，减小空间消耗，却增加的查询的时间消耗，
    // 减小负载因子，能减少元素容纳量，减少查询时间消耗，但却要及早的去扩容，
    // 增加了空间消耗
    final float loadFactor;
    // ...
}
```
## 三、功能解析
### 3.1 添加元素操作
#### 3.1.1 功能描述：
添加新的映射元素(newKey，newValue)，首先通过特定的hash算法计算newKey的hash值（newHash）。
> Hash算法：获取newKey的hashCode值，然后进行高低位相异或。
> hashCode值的获取方法在Object类中已有定义，当然也有某些类进行了重写，总的来说有以下几种：
> - String类型的hashCode：自定义算法较复杂
> - 包装类型的hashCode：当前值
> - 其他类型的hashCode：类名+@+内存位置的16进制表示

如果是首次添加元素，那么就意味着桶尚未初始化，所以这里会先执行初始化操作（resize），如果初始化成功或者非首次添加元素，那么开始定位元素的桶位。
> 桶定位算法：用之前hash算法的结果newHash与桶的个数-1进行与操作
> 该算法的本意是保留newHash值的后几进制位来确定桶位，如何保留后几位呢？我们知道二进制算法中1的与操作具有保留原值的效果
> 这里正是使用这种原理来实现，假设桶位数为16位，16的二进制位10000，16-1=15，15的二进制位就是1111，末四位全是1，通过1的
> 保留原值的作用，当拿它与newHash值的二进制值进行与操作后，结果就是newHash保留后4位的结果，其余位置0。而4位正好在桶位0-15之内。
> 而这也就是桶位数必须是2的次幂的原因，因为2的次幂的数字的二进制值全部是首位为1，其后全是0的值，当其-1之后就会变成首位
> 变0，其后全是1的值,而桶的下标是从0开始，最高位正好是-1之后的值。

查看确认桶位是否已有元素，如果没有，直接存放新元素到该桶位，如果桶位已有元素存在，那么就是出现hash碰撞，这时的解决办法就是使用链表或者红黑树来存储，如果该桶位存储的数据结构已经是红黑树，那么执行红黑树添加元素操作，否则执行链表的尾插法，将新元素插入到链表的末尾。
> 尾插法：1.7之前的版本全是头插法，将新元素作为表头元素，1.8之后改成尾插法，将新元素作为表尾元素，至于原因就是为了避免多线程扩容导致循环链表出现
> 在执行尾插法的时候需要遍历链表，查找是否存在相同key的元素，若存在则直接用newValue替换旧值，不再执行插入操作。

新元素插入完成之后，校验Map中总元素个数是否达到了阈值(这的个阈值是桶容量和负载因子乘积)，如果超过阈值则进行扩容。
#### 3.1.2 源码解析：
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    public V put(K key, V value) {
        // 首先通过hash方法计算hash值，然后执行存值操作
        return putVal(hash(key), key, value, false, true);
    }
    // hash算法：首先获取key的hashCode，然后将其高低16位相异或，全员参与（hashcode值的所有二进制位都能参与hash运算）
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            // 首次添加值执行初始化
            n = (tab = resize()).length;
        // 定位桶下标，n的值为2的次幂，同时也是桶的数量，hash是之前通过hash算法得出的结果，n-1之后末几位全部是1，
        // 再和hash与运算，等于保留hash的后几位作为结果。比如：(1111)&(01010101)的结果为0101，保留了后四位	
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 存在相同key的情况（桶位置）
                e = p;
            else if (p instanceof TreeNode)
                // 红黑树的情况
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 链表的情况
                for (int binCount = 0; ; ++binCount) {
                    // 遍历链表采用尾插法添加新元素
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 存在相同key的情况（链表元素位置）
                        break;
                    p = e;
                }
            }
            // 针对存在相同key的情况进行统一处理：替换value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();// 扩容两倍
        afterNodeInsertion(evict);
        return null;
    }    
    //...
    }
```
> 注意：桶容量为2的次幂的原因正是因为便于元素通过位运算实现定位。
### 3.2 初始化/扩容操作
#### 3.2.1 功能描述
执行扩容方法的原因主要是集合中元素数量达到阈值或者是集合桶数组某个桶位置的元素数量达到8个，
但集合桶容量未超过64的情况下，特殊的情况是首次添加元素时的初始化操作也走这个方法。
##### 3.2.1.1 初始化
只会计算初始化容量和初始化阈值然后创建一个初始桶数组并返回结果。
> 对于使用了带参构造器的情况，会定制初始容量和负载因子，如果只定制了初始容量则使用默认负载因子，
> 构造器会通过一个进制运算根据自定义的容量算出一个大于等于自定义容量值的最小的2的次幂值作为真正的容量
> 比如：自定义容量为10，则计算容量值为16，然后再根据这个容量计算阈值为12。

##### 3.2.1.2 扩容
首先校验旧容量是否已经达到或者大于容量最大值MAXIMUM_CAPACITY，如果是则不再进行扩容操作，还在原桶数组中保存元素，
并将阈值设置为Integer的最大值，设置为最大值之后就不会再触发扩容操作(因为Map中元素的总个数最大也就是Integer的最大值了，不可能比之更大)，
然后校验容量加倍后的新容量是否超过容量最大值MAXIMUM_CAPACITY，如果没有的话则将阈值加倍。
新容量和新阈值都有了，然后创建新的桶数组，在之后就是元素迁移了。

##### 3.2.1.3 元素迁移
遍历旧桶数组，校验每个桶位的元素结构，
如果只有一个元素，直接在新桶数组进行重定位，定位方式不变，
如果是红黑树，走树结构迁移逻辑，
否则就是链表，进行链表迁移，链表迁移进行了平衡优化，由于新桶数组和旧数组的两倍容量，
我们简单的将新容量分成相等的两半，称之为低位区与高位区，低位区下标与旧数组相同，
高位区下标为旧数组下标+旧数组容量。
链表迁移时，会根据该链表中元素的键的hash值与旧容量进行与运算，这就会有两个结果，为0或者不为0。
> 旧容量也是2的次幂，高位为1，其后全是0，比如10000（表示容量为16），将其和hash结果相与，
> 只会保留旧容量二进制为1的那一位对应的hash值的那一位，其余位全变成0，如果hash值的那一
> 位为0结果就是0，hash值那一位为1结果就是10000。

根据相与的结果来进行链表分拆，将结果为0的链表元素还定位到相同的桶下标位，即新桶数组的低位区，将结果为1的链表元素定位到原下标+旧桶容量的位置，即高位区。

这两个链表会先组装链表结构，然后将链表表头元素定位到低位区或高位区。
#### 3.2.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;// 旧容量
        int oldThr = threshold;// 旧阈值
        int newCap, newThr = 0;// 新容量、新阈值
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
				// 如果旧容量已经达到最大容量值，将阈值设置为最大值，返回旧桶数组
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
			    // 如果加倍后的新容量没有超过最大容量，且旧容量大于等于16，则新阈值加倍
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 只在通过带参数的构造器（初始容量和负载因子）
            // 创建的容器首次添加元素进行桶数组初始化时会走这里
            newCap = oldThr; 
        else {               // zero initial threshold signifies using defaults
			// 初始化容量和阈值，这就是首次添加元素时执行的初始化逻辑
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {// 计算新阈值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];// 创建桶数组
        table = newTab;
        // 如果是初始化操作，此处oldTab为null,会直接返回新建桶数组，否则执行元素迁移
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
						// 针对桶位置只有一个元素的情况，直接重定位元素，定位模式一致
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
						// 针对红黑树的情况
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order 针对链表的情况
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {// 遍历旧桶位的旧链表
                            next = e.next;
							// 这个判断的结果取决于在于hash值在于oldCap的1所在进制位对应的进制位是1还是0，
							// 由于oldCap只有这一位为1，那么hash的该位将保留原值，其余位全部得0，增加这么
							// 一个貌似随机的判断，用于进一步分散元素到不同的桶。
							// 其实就是将旧桶第i位桶的链表元素分散到新桶的第i和第i+oldCap桶位上，为0还是为1随机
							// 在循环中形成两个小链表，然后将首个元素赋值给新桶的对应桶位即可。
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
						// 定位两个小链表的首元素
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
    //...
}
```
### 3.3 获取元素操作
#### 3.3.1 操作描述
简单看代码
#### 3.3.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    public V get(Object key) {
        Node<K,V> e;
        // 计算key的hash值用作桶定位的基础
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 只有一个元素、链表头元素、树根元素要单独校验
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 存在链表或者树结构的情况
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    // 执行树结构获取元素逻辑
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 否则就是链表结构，遍历链表寻找匹配的元素
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }    
    //...
}
```
### 3.4 移除元素操作
#### 3.4.1 操作描述
简单看源码
#### 3.4.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    public V remove(Object key) {
        Node<K,V> e;
        // 计算key的hash值，用作后面定位元素桶位的基础
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        // 先进行桶定位，定位方式不变
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            // 针对只有一个元素、链表头元素、树根元素进行处理
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            // 如果第一个元素不是，针对链表和树结构进行后续元素处理
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    // 执行树结构获取元素逻辑
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    // 链表循环获取等key的元素
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    // 执行树结构删除元素操作
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    // 针对链表头和树根元素的情况
                    tab[index] = node.next;
                else
                    // 针对链表内元素进行删除
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }    
    //...
}
```
### 3.5 红黑树
#### 3.5.1 树形化操作
##### 3.5.1.1 操作描述
参照源码
##### 3.5.1.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    // 树形化准备
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            // 对于触发了树形化操作，但是桶容量还没达到64的情况下优先去做扩容处理，扩容也会分拆链表
            resize();
        // 定位要做树形下的桶位置，获取桶位元素e
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            // 循环遍历链表中的元素，将其改造成为双向链表结构，表头元素为hd
            do {
                // 将e元素封装成为树节点TreeNode
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
                // 执行树形化
                hd.treeify(tab);
        }
    }
    // 将Node节点封装成树节点
    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
    }
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        //...
        // 树形化操作
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;//代表根节点
            // 此处循环将this赋值给x,this代表的是当前树节点，这个类是HashMap的内部类用于标识树节点，
            // this就是当前类的实例，也就是一个树节点，但是是哪个树节点，就要依靠之间的代码上下文来判
            // 断了，看看调用该方法的地方有这样的代码：hd.treeify(tab);这就表示当前节点就是那额hd节
            // 点，而这个hd节点就是之前改造好的双向链表的表头结点
            // 这里循环的是双向链表中的元素
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) {
                    // root == null的情况是链表头结点的时候才会出现，这时候将这个头结点作为树根节点
                    x.parent = null;//根节点无父节点
                    x.red = false;//黑色
                    root = x;//赋值
                }
                else {
                    // 这里只有非链表头节点才能进来
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    // 此处循环的是已构建的红黑树的节点，从根节点开始，遍历比较当前链表节点与当前红黑树节点的
                    // hash值，dir用于保存比较结果，如果当前链表节点小，则dir为-1，否则为1，实际情况却是，能
                    // 拨到同一个桶位的所有元素的hash值那是一样的呀，所以dir的值是无法依靠hash值比较得出结果
                    // 的，那么重点就靠最后一个条件判断来得出结果了，
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);// 最后需要依靠这个方法来决定dir的值

                        TreeNode<K,V> xp = p;
                        // 根据dir的值来决定将当前链表节点保存到当前树节点的左边还是右边，
                        // 或者当前链表节点需要与当前树节点的左节点还是右节点接着比较
                        // 主要寻找子节点为null的情况，将节点保存到null位置
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                // dir<=0，将链表节点保存到当前树节点的左边子节点位置
                                xp.left = x;
                            else
                                // dir<=0，将链表节点保存到当前树节点的右边子节点位置
                                xp.right = x;
                            // 一旦添加的一个新节点，就要进行树平衡操作，以此保证红黑树结构
                            // 树的平衡操作依靠的就是其左右旋转操作
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            // 最后将组装好的树的根节点保存到桶下标位
            moveRootToFront(tab, root);
        }
        static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
            int n;
            if (root != null && tab != null && (n = tab.length) > 0) {
                // 首先定位桶下标位
                int index = (n - 1) & root.hash;
                TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
                // 校验当前桶下标位的值是否为根节点的值，可能会存在不同的原因是树的平衡操作将原本的根节点挪移了
                // 如果相同，那么不作任何处理，如果不同，就需要替换桶位元素为树根节点元素，然后改变双向链表结构
                // 将root根节点作为双向链表表头元素,为何要替换呢，因为在判断桶位元素类型时会对链表进行遍历，如
                // 果桶位置放的不是链表头或者尾元素，遍历将变得非常麻烦
                if (root != first) {
                    Node<K,V> rn;
                    tab[index] = root;
                    TreeNode<K,V> rp = root.prev;
                    if ((rn = root.next) != null)
                        ((TreeNode<K,V>)rn).prev = rp;
                    if (rp != null)
                        rp.next = rn;
                    if (first != null)
                        first.prev = root;
                    root.next = first;
                    root.prev = null;
                }
                // 校验链表和树的结构
                assert checkInvariants(root);
            }
        }        
        //...
    }
    //...
}
```
#### 3.5.2 红黑树分拆操作
##### 3.5.2.1 操作描述
很简单，看源码
##### 3.5.2.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        //...
        // 将一颗大树分拆为两颗小树，如果树太小，退化为单向链表
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            // this代表当前节点，也就是树的根节点，桶位节点
            // map代表当前集合
            // tab代表新桶数组
            // index代表当前节点的桶位下标
            // bit为旧桶容量
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;// lc表示低位树容量，hc表示高位树容量
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                // 分拆树节点的依据，结果为0的一组（低位组），结果不为0的一组（高位组）
                if ((e.hash & bit) == 0) {
                    // 组装低位组双向链表
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    // 组装高位组双向链表
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }
            // 针对低位组进行树形化处理，如果该组元素数量少于6个则退化为单向链表
            if (loHead != null) {
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            // 针对高位组进行树形化处理，如果该组元素少于6个则退化为单向链表
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }        
        //...
    }
    //...
}
```
#### 3.5.3 红黑树添加元素操作
##### 3.5.3.1 操作描述
参照源码
##### 3.5.3.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        //...
        // 红黑树的添加元素，map为当前HashMap，tab为当前桶数组，h为新增元素的key的hash值，k为新增元素的key,v为新增元素的value
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            // 当前节点是已定位的桶位元素，其实就是树结构的根节点元素
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                // dir代表当前树节点与待添加节点的hash比较结果
                // ph代表当前树节点的hash值
                // pk代表当前树节点的key
                // 由于一个桶位的所有元素hash值相等，所以最后得出结果需要依靠
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    // 如果当前节点的hash值大，dir为-1
                    dir = -1;
                else if (ph < h)
                    // 如果当前节点的hash值小，dir为1
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    // hash值相等的情况下，如果key也一样直接返回当前节点，返回去之后会执行value的替换操作
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            // 这个找到的q也是与待添加元素key相同的元素，执行替换
                            return q;
                    }
                    // 最终需要依靠这个方法来得出dir值的结果
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                // 根据dir的值来决定是当前节点的左侧还是右侧，如果该侧右子节点则继续循环寻找位置，否则直接将新元素添加到该侧子节点位置
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);//封装树节点
                    if (dir <= 0)
                        // dir<=0，将新节点添加到当前节点左侧
                        xp.left = x;
                    else
                        // 否则将新节点添加到当前节点右侧
                        xp.right = x;
                    // 设置新节点的链表位置，将其作为xp的下级节点
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        // 如果xp节点原本有下级节点xpn，则要将新节点插入到xp和xpn之间（指双向链表中）
                        ((TreeNode<K,V>)xpn).prev = x;
                    // 插入了新节点之后，要进行树平衡操作，平衡操作完成，将根节点设置为桶位节点
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }    
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
        // 一般我们在HashMap中保存的键值对的类型都是不变的，这一般用泛型控制，
        // 那么就意味着，两个元素的key的类型时一样的，所以才需要靠其hashCode来决定大小
        // System.identityHashCode(parameter)是本地方法，用于获取和hashCode一样的结果，
        // 这里的hashCode指的是默认的hashCode方法，与某些类重写的无关
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }
        //...
    }
    //...
}
```
#### 3.5.4 红黑树添加元素平衡操作
##### 3.5.4.1 操作描述
###### 3.5.4.1.1 左旋操作描述
绕A节点左旋，等于将A的右子节点B甩上来替换自己的位置，而自己顺势下沉成为其左子节点，这时你会发现，B有三个子节点，明显结构不对，将B的原来的左子节点C转移到下沉的A上，成为其右子节点，旋转结束
其实，要保证左子树节点值小于其根节点，右子树节点值大于其根节点，那么在替换AB节点之后，C节点的值就出现了问题，只有将其挪到A节点右边才能继续保证上面的结构。
首先我们知道B节点为A的右节点，那么B>A，而C为B的左节点，则C<B,而C又位于A的右子树，则C>A,因此：A<C<B。要保证这个式子永远成立，就必须依靠挪移节点来完成。
现在B为最顶部节点且为最大值，那么A和C必须位于其左子树，而C>A则，C必须位于A的右子树，再看看之前的情况，如果A为顶点节点，那么BC均应位于其右子树，而B>C，那么要么B为C的右节点，要么C为B的左节点
###### 3.5.4.1.2 右旋操作描述
绕A几点右旋，等于将A的左子节点B甩上来替换自己的位置，而自己顺势下沉成为其右子节点，这是你会发现，B有三个子节点，明显结构不对，将B的原来的右子节点C转移到下沉的A上，成为其左子节点，旋转结束
首先我们知道B为A的左子节点，所以B<A,再者C为B的右子节点，那么C>B，而C又位于A的左子树，则C<A,最后：A>C>B。要保证这个结果成立，那么再B替换A的位置之后，A下沉为B的右子节点，因为A>B,所以往右走，
这时C和A均位于B的右侧，比较二者发现C<A，那么将C放到A的左侧成为其左子节点
###### 3.5.4.1.3 添加平衡操作描述
新增节点全部初始化为红色节点，然后分以下几种情况：
- 新增节点为根节点：颜色置黑；
- 新增节点父节点为黑色节点或者父节点是根节点（原本为黑色）：不操作；
- 新增节点x的父节点为其父节点（x祖节点）的左子节点：
    - x祖父节点的右子节点存在并为红色（那么x祖父节点一定是黑色节点）：将x的祖父节点置为红色，x的父节点和其兄弟节点置为黑色，然后以x的祖父节点为新的x执行循环；
    - x祖父节点无右子节点或为黑色节点：
        - 如果x是其父节点的右子节点：执行以x父节点xp为基准的左旋操作，x被甩上来替换xp的位置，并置黑，原x祖父节点（现x节点父节点）置红，然后以该祖父节点右旋，之后x节点再次被甩上来替换了祖父节点xpp的位置，然后以xp为新的x执行循环
- 新增节点x的父节点为其父节点的右子节点：
    - x祖父节点的左子节点存在并为红色（那么x祖父节点一定为黑色节点）：将x的祖父节点置为红色，x的父节点和其兄弟节点置为黑色，然后以x的祖父节点为新的x执行循环；
    - x祖父节点无左子节点或为黑色节点：
        - 如果x是其父节点的左子节点：执行以x父节点xp为基准的右旋操作，x被甩上来替换xp的位置，并置黑，原x祖父节点（现x节点父节点）置红，然后以该祖父节点右旋，之后x节点再次被甩上来替换了祖父节点xpp的位置，然后以xp为新的x执行循环
##### 3.5.4.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        //...
        // 左旋操作，其中root为根节点，p为当前节点，r为p的右节点，rl为r的左节点，pp为p的父节点
        // 左旋之后，r替换p的位置，rl挪到p的右节点
        // 节点位置变换之后，既要改变其父节点的left/right值，也要改变当前节点中parent的值，
        // 改变是双向的，父子均有指向，改变之后均要修改
        static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                              TreeNode<K,V> p) {
            TreeNode<K,V> r, pp, rl;
            if (p != null && (r = p.right) != null) {
                // 首先将r节点的左子节点(rl)送给p当其右子节点
                if ((rl = p.right = r.left) != null)
                    rl.parent = p;//变换rl的父节点为p,原来为r
                if ((pp = r.parent = p.parent) == null)
                    // 原p节点为根节点的情况，r替换之后，需要重新着色为黑色，保证根节点为黑色
                    (root = r).red = false;
                else if (pp.left == p)
                    // 原p节点为其父节点pp的左子节点的情况，r替换后，需要修改pp节点的left指向r节点
                    pp.left = r;
                else
                    // 原p节点为其父节点pp的右子节点的情况，r替换后，需要修改pp节点的right指向r节点
                    pp.right = r;
                //然后将p节点作为r节点的左子节点，即为p节点顺势下沉为r的左子节点
                r.left = p;
                p.parent = r;//变换p的父节点为r
            }
            return root;
        }
        // 右旋操作，嘿，那就是左旋的反向操作罢了
        // root为根节点，p为当前节点，l为其左子节点，lr为l的右子节点，pp为p的父节点
        static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                               TreeNode<K,V> p) {
            TreeNode<K,V> l, pp, lr;
            if (p != null && (l = p.left) != null) {
                // 首先将l的右子节点lr挪给p
                if ((lr = p.left = l.right) != null)
                    lr.parent = p;//变换lr的父节点为p,原来为l
                if ((pp = l.parent = p.parent) == null)
                    // 如果p节点是根节点，替换为l之后，l便成为新的根节点，需要重新着色为黑色，保证红黑树结构
                    (root = l).red = false;
                else if (pp.right == p)
                    // 如果原p节点是其父节点pp的右子节点，那么需要将其右子节点改成l
                    pp.right = l;
                else
                    // 如果原p节点是其父节点pp的左子节点，那么需要将其左子节点改成l
                    pp.left = l;
                // 最后将原p节点置为l节点的右子节点，并修改p的父节点为l
                l.right = p;
                p.parent = l;
            }
            return root;
        }
        // 平衡操作,x为新增节点，root为根节点
        static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                    TreeNode<K,V> x) {
            x.red = true;// 新增节点全部为红色节点
            for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
                if ((xp = x.parent) == null) {
                    // 1 x为根节点的情况，将其重新着色为黑色
                    x.red = false;
                    return x;
                }
                else if (!xp.red || (xpp = xp.parent) == null)
                    // 2 如果x节点的父节点为黑色，又或者x的父节点是根节点，没有影响，不操作
                    return root;
                if (xp == (xppl = xpp.left)) {
                    // 3 如果x节点的父节点是其父节点（x的祖父节点）的左子节点
                    if ((xppr = xpp.right) != null && xppr.red) {
                        // 3-1 再如果x的祖父节点的右子节点存在且为红色，则将这个节点和x的父节点统统改成黑色，
                        // 再把x的祖父节点改成红色，将x祖父节点作为新的x节点执行循环
                        xppr.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        // 3-2 否则的情况
                        if (x == xp.right) {
                            // 3-2-1 如果x节点是其父节点的右子节点，则执行以x父节点为基准的左旋操作，
                            // 左旋之后新增节点x替了其原父节点xp，将原xp节点当做现在的x节点，原来的x
                            // 节点是现在x节点的父节点xp,原来的x节点的祖父节还是现在x的祖父节点
                            root = rotateLeft(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {// xp为原来的x节点
                            // 将xp节点置为黑色
                            xp.red = false;
                            if (xpp != null) {// xpp还是之前的xpp
                                // 将xpp节点置为红色，然后执行右旋,右旋可以将xpp节点用xp节点替换，红黑交换
                                xpp.red = true;
                                root = rotateRight(root, xpp);
                            }
                        }
                    }
                }
                else {
                    // 4 如果x节点的父节点是其父节点（x的祖父节点）的右子节点
                    if (xppl != null && xppl.red) {
                        // 4-1 再如果x的祖父节点的左子节点存在并且为红色，则将该节点置为黑色，
                        // 将x的父节点置为黑色，祖父节点置为红色，然后把xpp祖父节点作为新的x节点
                        xppl.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        // 4-2 否则的情况
                        if (x == xp.left) {
                            // 4-2-1 如果x节点是其父节点的左子节点的情况，先以x父节点进行右旋，
                            // 右旋之后原来的xp节点被新的x节点替换，原来的xp节点作为新xp节点的右子节点，
                            // 现在看作为x,然后重新定义xpp，其实xpp位置不变
                            root = rotateRight(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            // 将现在的xp节点置为黑色
                            xp.red = false;
                            if (xpp != null) {
                                // 将祖父节点置为红色。然后执行左旋，左旋之后，原来的xp节点接替了xpp节点的位置，xpp变成原来xp的左子节点
                                xpp.red = true;
                                root = rotateLeft(root, xpp);
                            }
                        }
                    }
                }
            }
        }        
        //...
    }
    //...
}
```
#### 3.5.5 红黑树删除元素操作
##### 3.5.5.1 操作描述
红黑树的节点删除操作主要分为这么三种：

- 待删节点没有子节点
- 待删节点有一个子节点
- 待删节点有两个子节点

针对第一种情况，真的好简单，待删节点即为叶子节点，直接删除即可；

针对第二种情况，也不难，将那个子节点替换待删节点即可；

至于第三种情况，那就麻烦了，但通过变换，可以将其转化为第一种或者第二种情况：处理方式是，找到待删节点的右子树中的最左节点（或者左子树中的最右节点），将其与待删节点互换位置，然后就将情况转换为第一种或者第二种了。

针对第三种情况转换方法的解析：为什么要找到待删节点的右子树最左节点呢，因为红黑树是二叉搜索树，这个二叉搜索树中满足"左子节点<其父节点<其父节点的右子节点"的规则，那么找到的右子树的最左节点，就是整颗树中大于待删节点值的最小值节点了，为了保证二叉搜索树的搜索结构（也就是刚刚那个公式），我们只能找最接近待删节点值的节点值来接替它的位置，如此能保证二叉搜索的结构，但是可能会破坏红黑树的结构，因为如果待删节点为红色，而替换节点为黑色的话，那岂不是在待删节点分支多加了一个黑色节点嘛，还有其他各种情况，种种，需要进行删除节点后的树平衡操作来保证红黑树的结构完整。

下面重点说说删除后的平衡问题：

其实只要待删节点是黑色节点，一旦删除必然会导致分支中黑色节点缺一（红黑树不再平衡），具体情况又分为以下几种：（基础条件：待删节点p为黑色，其只有一个子节点x，操作在待删节点被删除之后，子节点替换其位置之后）
- 如果子节点x为红色节点，那么只需要将其置黑即可;
- 如果子节点x为黑色节点，为保证本分支黑色节点不会再变少，那么只能求助于其兄弟节点分支了：
    - x为左子节点：
        - x无兄弟节点xpr:以x的父节点xp为基准进行循环；
        - x有兄弟节点xpr：
            - xpr为红色节点(那么xp必然为黑色节点)：将xp置红，xpr置黑，以xp为基准左旋；<br/>
                解析：开始情况是x分支删除了一个黑色节点，即x分支缺少一个黑色几点，而x的兄弟节点xpr为红色节点，xp为黑色节点，我们将xp和xpr颜色互换，那么在xpr分支黑色节点数量是不变的（只是位置变了），然后我么以红色的xp为基准执行左旋,将黑色的xpr甩上去替换xp的位置，xp作为xpr的左子节点，那么x端分支便多出了xpr这个黑色节点来补足不够的数量，而兄弟分支黑色节点数量还是不变的。
            - xpr为黑色节点(那么xp颜色不明)：
                - 兄弟节点的左子节点和右子节点全null或全黑或一黑一null：将兄弟节点置红，然后以xp为基准进行循环；
                - 兄弟节点的左子节点和右子节点全红或一红一null或一红一黑：
                    - 兄弟节点的右子节点为黑色或null，即兄弟节点的左子节点为红色：将兄弟节点与其做自己节点交换颜色，兄弟节点置红，左子节点置黑，然后以兄弟节点为基准执行右旋操作，将其黑色的左子节点甩上去做自己的父节点，自己做其右子节点，然后将新的兄弟节点xpr(原来的xprl)的颜色置为与xp一致（不明，非黑即白），新的sr(即原来的xpr)置黑（这个置黑的原因是因为右旋操作之前执行了颜色替换，兄弟节点右侧分支少了一个黑色节点，右旋之后变为黑色的sl补充了这个黑色节点，但是现在我们要用sl[新xpr]来替换xp节点[置为xp节点的颜色]，那么右侧分支原本用来补充之前缺少的黑色节点又消失了，所以将已知的红色节点sr置为黑色来进行补充），xp置黑，以xp左旋，xpr被甩上来替换xp的位置，xp则是补充给x分支的黑色节点，xpr与以前的xp颜色一致，所以兄弟分支黑色节点不变。<br/>
                        **解析**：新的sr(即原来的xpr)的原因是因为右旋操作之前执行了颜色替换，兄弟节点右侧分支少了一个黑色节点，右旋之后变为黑色的sl补充了这个黑色节点，但是现在我们要用sl[新xpr]来替换xp节点[置为xp节点的颜色]，那么右侧分支原本用来补充之前缺少的黑色节点又消失了，所以将已知的红色节点sr置为黑色来进行补充）
    - x为右子节点：与上面的情况正好对称（不再介绍）

貌似有点难...大家要看进去思考才能理解，光看没用！
##### 3.5.5.2 源码解析
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    //...
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        //...
        // 当前节点即为要删除的节点，map为当前集合，tab为当前桶数组
        final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                                  boolean movable) {
            int n;
            if (tab == null || (n = tab.length) == 0)
                return;
            // 定位待删节点的桶位下标index
            int index = (n - 1) & hash;
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
            TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
            if (pred == null)
                // 如果当前节点是双向链表头节点/树根节点,则将链表第二元素置为桶位元素，即删除该节点
                tab[index] = first = succ;
            else
                // 如果当前节点不是双向链表头节点，则将其后置节点赋给其前置节点作为后置节点，即删除该节点
                pred.next = succ;
            if (succ != null)
                // 修改后置节点中prev指向新的前置元素节点
                succ.prev = pred;
            if (first == null)
                return;
            if (root.parent != null)
                root = root.root();
            if (root == null || root.right == null ||
                (rl = root.left) == null || rl.left == null) {
                // 退化为链表机构
                tab[index] = first.untreeify(map);  // too small
                return;
            }
            //// 之前的操作是在双向链表中删除当前节点的痕迹，下面是在树结构中删除的操作
            // p为待删节点（即当前节点），pl为p的左子节点，pr为p的右子节点，
            TreeNode<K,V> p = this, pl = left, pr = right, replacement;
            if (pl != null && pr != null) {
                // 当前节点同时拥有左右子节点的情况,sl表示当前节点的右子树的最左节点
                // 要删除当前节点，需要找到与当前节点值最靠近的左右两侧的节点之一，这
                // 里找的是右侧的，即找的是整个树中大于当前节点值的最小值节点，将找到
                // 的节点与待删节点互换，互换之后再删除节点，如果原来的那个最左节点还
                // 有右子节点，则将该右子节点替换其父节点（待删节点）
                TreeNode<K,V> s = pr, sl;
                while ((sl = s.left) != null) // find successor
                    s = sl;// 找到右子树的最左节点
                boolean c = s.red; s.red = p.red; p.red = c; // swap colors 首先互换颜色
                TreeNode<K,V> sr = s.right;// s为最左节点，那么它不可能有左子节点，最多有右子节点
                TreeNode<K,V> pp = p.parent;
                if (s == pr) { // p was s's direct parent
                    // 如果找到的s即为待删节点的直接右子节点（说明s无左子节点），那么直接替换这两个节点
                    p.parent = s;
                    s.right = p;
                }
                else {
                    // 否则的情况，先找到s的父节点sp,将其设置为p的父节点，
                    TreeNode<K,V> sp = s.parent;
                    if ((p.parent = sp) != null) {
                        if (s == sp.left)
                            // 将p作为原来s的父节点的左子节点（即替换p和s的位置）
                            sp.left = p;
                        else
                            // TODO 这里是什么意思呢?找的就是sp的最左节点，这里怎么跑到右节点上去了呢，虽然p是要删除的节点
                            sp.right = p;
                    }
                    // 把p的右子节点pr置为s的右子节点
                    if ((s.right = pr) != null)
                        // 把s置为pr的父节点
                        pr.parent = s;
                }
                // 替换之后p是无左子节点的，（即原来的s是最左节点，无左子节点）
                p.left = null;
                // 把s的右子节点sr置为p的右子节点
                if ((p.right = sr) != null)
                    // 把sr的父节点设置为p
                    sr.parent = p;
                if ((s.left = pl) != null)
                    // 将p的左子节点置为s的左子节点
                    pl.parent = s;
                // 把p的父节点设置为s的父节点
                if ((s.parent = pp) == null)
                    // 如果p没有父节点，将s节点设置为根节点
                    root = s;
                // 否则如果p是其父节点pp的左子节点
                else if (p == pp.left)
                    // 现在将s设置为pp的左子节点
                    pp.left = s;
                else
                    // 否则如果p是其父节点的右子节点，则将s设置为pp的右子节点
                    pp.right = s;
                if (sr != null)
                    // 如果s存在右子节点，则将其置为replacement,现在和待删节点只有右子节点的情况一样
                    replacement = sr;
                else
                    // 否则将p置为replacement,至此第一种情况替换结束，现在和待删节点没子节点的情况一样
                    replacement = p;
            }
            else if (pl != null)
                // 待删节点只有左子节点的情况，将其左子节点置为replacement
                replacement = pl;
            else if (pr != null)
                // 当前节点只有右子节点的情况，将其右子节点置为replacement
                replacement = pr;
            else
                // 待删节点没有子节点的情况，直接将其设置为replacement
                replacement = p;
            if (replacement != p) {// 如果待删节点有子节点replacement的情况
                // 准备替换replacement节点和p节点
                TreeNode<K,V> pp = replacement.parent = p.parent;
                if (pp == null)
                    // 待删节点p为根节点的情况,将replacement设置为根节点即可
                    root = replacement;
                else if (p == pp.left)
                    // p是作为其父节点pp的左子节点，则将replacement设置为pp的左子节点
                    pp.left = replacement;
                else
                    // 否则p是作为其父节点pp的右子节点，则将replacement设置为pp的右子节点
                    pp.right = replacement;
                // 最后将p节点的所有关系置空
                p.left = p.right = p.parent = null;
            }
            // 如果待删节点是红色节点则不影响平衡，无需执行树平衡操作，否则需要进行树平衡操作
            TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);
            // 如果p节点没有任何子节点的情况
            if (replacement == p) {  // detach
                // 根据实际情况置空p节点的各属性
                TreeNode<K,V> pp = p.parent;
                p.parent = null;
                if (pp != null) {
                    if (p == pp.left)
                        pp.left = null;
                    else if (p == pp.right)
                        pp.right = null;
                }
            }
            if (movable)
                // 如果可移动，那么将根节点设置为桶位节点
                moveRootToFront(tab, r);
        }
        // 删除节点后的平衡操作，root为根节点，x为上面提到的replacement节点，该节点其实为替换p节点，为其子节点
        static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                                   TreeNode<K,V> x) {
            for (TreeNode<K,V> xp, xpl, xpr;;)  {
                if (x == null || x == root)
                    // 如果x不存在或者其本来就是根节点，字节返回根节点
                    return root;
                else if ((xp = x.parent) == null) {
                    // 如果x节点不存在父节点，那么则其为根节点，但是还未设置为root节点，那么直接置黑色，并将其设置为root节点
                    x.red = false;
                    return x;
                }
                else if (x.red) {
                    // 如果x是红色节点，则将其置黑，因为如果x为红色，那么其父节点p必然为黑色，
                    // 删掉之后，会导致黑色节点减少，正好x为红色拿来补充黑色节点，使黑色节点数不变，
                    // 如果x是黑色，那么就会导致当前分支黑色节点减少，需要使用其他方法进行平衡
                    x.red = false;
                    return root;
                }
                // 如果x为其父节点左子节点（删除后的结果）
                else if ((xpl = xp.left) == x) {
                    // 如果x存在兄弟节点（其父节点的右子节点），且为红色，那么xp必定为黑色
                    // 那么就表明x节点分支的黑色节点少了一个，也就是其兄弟节点多一个（其他所有分支都多一个），
                    if ((xpr = xp.right) != null && xpr.red) {
                        // 那么将兄弟节点置黑，父节点置红，这是x分支还是少一个黑节点，兄弟分支黑节点不变
                        xpr.red = false;
                        xp.red = true;
                        // 再然后执行xp节点的左旋，将其右子节点（即x的兄弟节点）甩上去，
                        // xp作为其左子节点，如此一来将兄弟节点这一黑色几点变成两份之共享，
                        // 无形之中使得x分支黑色节点加1，从而达到平衡
                        root = rotateLeft(root, xp);
                        xpr = (xp = x.parent) == null ? null : xp.right;
                    }
                    if (xpr == null)
                        x = xp;// 如果x没有兄弟节点，那么循环以x的父节点为基准再次进行平衡
                    else {
                        // 否则，那就是x存在兄弟节点，且为黑色的情况，则其父节点颜色不明，x颜色不明
                        // sl为兄弟节点的左子节点，sr为兄弟节点右子节点
                        TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                        // 如果sr和sl中，全部为null,或者全部为黑色节点，或者有一个为黑色节点，另一个是null
                        if ((sr == null || !sr.red) &&
                            (sl == null || !sl.red)) {
                            // 将兄弟节点置为红色
                            xpr.red = true;
                            x = xp;// 为了循环
                        }
                        // 否则，就是sl和sr全为红色或者一红一null，或者一红一黑
                        else {
                            // 如果sr为null(则sl必为红色)，或者sr为黑色（则sl必为红色）
                            if (sr == null || !sr.red) {
                                if (sl != null)
                                    // 将sl置为黑色
                                    sl.red = false;
                                // 兄弟节点置红
                                xpr.red = true;
                                // 然后右旋兄弟节点，将其左子节点甩上去做自己的父节点，这时兄弟分支黑节点数量不变
                                root = rotateRight(root, xpr);
                                xpr = (xp = x.parent) == null ?
                                    null : xp.right;// 更新xpr的指向,将其指向旋转之后新的兄弟节点，即原来的sl(黑色)
                            }
                            if (xpr != null) {
                                // 将xp和xpr颜色弄一致，因为如果xp是黑色，左旋之后兄弟分支会少一个黑节点，
                                // 这样xpr就会补充这个黑色，如果xp是红色，那么xpr也是红色，左旋之后xpr落座
                                // xp的位置，还是原来的颜色，而左侧确多出了xp这个黑色节点。
                                xpr.red = (xp == null) ? false : xp.red;
                                if ((sr = xpr.right) != null)
                                    // 将sr置黑即把原来的xpr置黑
                                    sr.red = false;
                            }
                            if (xp != null) {
                                // 将xp置黑
                                xp.red = false;
                                // 然后在执行xp左旋，等于将sl甩到了xp的位置，
                                // 而且这个sl必然为黑色，是为了补充x分支缺少的那一个黑节点
                                root = rotateLeft(root, xp);
                            }
                            x = root;
                        }
                    }
                }
                // 否则如果x是其父节点的右子节点的话
                else { // symmetric （对称的）
                    if (xpl != null && xpl.red) {
                        xpl.red = false;
                        xp.red = true;
                        root = rotateRight(root, xp);
                        xpl = (xp = x.parent) == null ? null : xp.left;
                    }
                    if (xpl == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                        if ((sl == null || !sl.red) &&
                            (sr == null || !sr.red)) {
                            xpl.red = true;
                            x = xp;
                        }
                        else {
                            if (sl == null || !sl.red) {
                                if (sr != null)
                                    sr.red = false;
                                xpl.red = true;
                                root = rotateLeft(root, xpl);
                                xpl = (xp = x.parent) == null ?
                                    null : xp.left;
                            }
                            if (xpl != null) {
                                xpl.red = (xp == null) ? false : xp.red;
                                if ((sl = xpl.left) != null)
                                    sl.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateRight(root, xp);
                            }
                            x = root;
                        }
                    }
                }
            }
        }        
        //...
    }
    //...
}
```
## 四、总结
HashMap中涉及到了数组操作，单向链表操作，双向链表操作，红黑树操作：
> 数组操作:<br/>
> - HashMap中的数组指的就是桶数组，用于承载链表（的头结点）和红黑树（根节点）<br/>
> - 数组的创建在第一次往集合中添加元素的时候进行，默认的数组长度为16，也可以手动指定，系统会自动计算一个大于等于给定容量的最小的2的次幂的容量值作为数组初始容量，数组的最大容量为不超过两倍的Integer的最大限值。数组还有一个负载因子，默认为0.75，这个值可算是一个时间空间的折中了，一般不会手动修改，但也能手动指定，如果设置过大，查询消耗增加，如果设置过小，空间消耗增加。<br/>
> - 当向集合中添加一个新元素的时候，通过元素的key的hash算法结果来定位其在数组中的位置,这个hash算法要选择的足够好来使得元素能够尽量平均的散布在数组的各个位置，而不是堆积在几处。
> - 数组的容量是不可变的，所以一旦原数组容量受限，一般是创建新的数组来替代，这叫数组的扩容，扩容需要满足一定的条件，HashMap中有一个阈值用于作为扩容的条件，这个值是当前容量和负载因子的乘积，只要当前的集合的元素数量达到了阈值，就要执行扩容操作，当然还有一种情况也要扩容，那就是在单个数组位上的元素数量达到8个时，但数组容量未达到64个时，优先执行数组扩容。数组扩容后新数组为原数组的2倍容量。<br/>

> 单向链表操作：<br/>
> - hashMap是依靠hash来存储元素的，hash存储总是难以避免碰撞的出现，HashMap使用单向链表来保存发生碰撞的元素。新元素会保存到链表的尾部，如果新元素的key已经存在，那么将会是一个更新操作，不会有新元素增加。
> - 数组扩容的时候需要进行元素的迁移，这里就是链表的迁移，链表迁移的时候会触发链表分拆，将一个完整链表分拆成为两个链表，我们成为低位链表和高位链表，低位链表的数组位置同旧数组，而高位链表的数组位置为低位链表数组位+旧数组容量。可见通过链表分拆也是可以降低链表中元素数量的。
> - 在Jdk1.8以前的版本中在高并发的情况下，HashMap数组扩容的时候可能会出现死循环，这是因为两个线程同时进行扩容和元素迁移时，由于再次对链表元素进行循环头插法存储元素，极可能就会导致出现了循环引用，Jdk1.8中已经修复这一Bug，采用的是将头插法改为尾插法，而且迁移元素的方式也发生了变化，但是HashMap仍然是线程不安全的集合，多线程环境中最好使用ConcurrentHashMap。

> 双向链表操作：<br/>
> - hashMap中存在双向链表，他是在单向链表元素达到8个，且数组容量达到64位之后执行树形化时转换的，也就是说，HashMap中的红黑树同时也是一个双向链表。这个双向链表的作用是在某些特殊情况下（树太小的时候），在将红黑树退化为单向链表结构时使用的。正因为如此，在红黑树的增删节点、平衡节点的时候还需要保证双向链表的结构。

> 红黑树操作：<br/>
> - 这一部分可以算是hashMap中最最复杂难懂的东西了
> - 红黑树的树形化操作
> - 红黑树的增加元素操作
> - 红黑树的增加元素平衡操作
> - 红黑树的树分拆操作
> - 红黑树的删除元素操作
> - 红黑树的删除元素平衡操作
> - 红黑树的退化单项链表操作，有两处退化，一处是在扩容时，树分拆之后，子树内元素容量少于6个时，执行退化操作，还有就是在移除树中元素之后，如果树结构满足某些条件则执行退化操作
> - 其实平衡的目的就是为了恢复被添加和移除元素操作破坏的红黑树结构罢了，使用的手段无非变色和旋转。

参考：<br/>
[【Java入门提高篇】Day25 史上最详细的HashMap红黑树解析](http://www.cnblogs.com/mfrank/p/9227097.html)<br/>
[红黑树(一)之 原理和算法详细介绍](http://www.cnblogs.com/skywang12345/p/3245399.html)