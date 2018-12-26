# Java基础系列-ConcurrentHashMap 1.8
## 概述
    ConcurrentHashMap的HashMap的线程安全版本，当我们在多线程并发环境中编程时使用ConcurrentHashMap来代替HashMap。
    ConcurrentHashMap底层结构和实现原理基本与HashMap雷同，只是增加了针对并发的处理。
    ConcurrentHashMap通过对桶位数组值加锁的方式来保证并发下的操作安全性。注意这里不是对桶位加锁，而是对桶位上的元素进行加锁。
## 基础
### Java内存模型
具体查看[Java基础系列-Java内存模型]()
### volatile
volatile可以保证内存可见性和有序性，无法保证原子性，原子性需要依靠加锁来保证。
- 可加性：被volatile修饰的变量在被修改之后，会被立即更新到主内存中，读取volatile修饰的变量时需要直接从主内存读取，保证修改的值可见，保证线程安全。
- 有序性：被volatile修饰的变量的读写操作会被添加内存屏障，保证不会发生重排序，从而保证线程安全。volatile写之前的操作不能重排序到volatile之后，volatile读之后的操作不能重排序到volatile读之前，volatile先写后读的不能重排序

具体可查看[java基础系列--volatile关键字](https://www.cnblogs.com/V1haoge/p/7833881.html)
### CAS操作
CAS操作是直接调用计算机指令来完成操作，属于原子操作。
具体查看[Java基础系列-CAS操作]()
### 红黑树
红黑树是一种特制化的二叉查找树。
具体可查看[算法基础系列-红黑树]()
### 二进制操作
Java源码中涉及到了大量的二进制操作，总是让人云里雾里。
具体查看[Java基础系列-二进制操作]()
## 常量变量解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    private static final int MAXIMUM_CAPACITY = 1 << 30;// 桶数组的最大容量(2的30次方)
    private static final int DEFAULT_CAPACITY = 16;// 桶数组的默认容量为16(2的4次方)
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;// 
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;// 默认的并发等级，这个值一般等于桶数组容量，这个并发等级，其实就是可以同时支持的最大并发量
    private static final float LOAD_FACTOR = 0.75f;// 负载因子，一般不改动
    static final int TREEIFY_THRESHOLD = 8;// 树形化阈值，链表元素达到8个就尝试执行树形化
    static final int UNTREEIFY_THRESHOLD = 6;// 树退化阈值，树在扩容时分拆后树容量达到6时执行退化操作，转化为单向链表
    static final int MIN_TREEIFY_CAPACITY = 64;// 树形化容量阈值，只有在桶数组容量达到64之后才能执行树形化操作，否则会执行扩容
    private static final int MIN_TRANSFER_STRIDE = 16;// 数据迁移的最短步长，也就是分配给每个线程的迁移的区间最小值为16
    private static int RESIZE_STAMP_BITS = 16;// 用于生成扩容戳记sizeCtr的一个基础量
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;// 辅助扩容的线程的最大数量。1111111111111111  共16个1，整数是65535
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;// 
    
    static final int MOVED     = -1; // 表示正在进行元素迁移
    static final int TREEBIN   = -2; // 表示树形化已完成
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // 正常节点的hash值的可用位(共32位，除首位外均可用)
    static final int NCPU = Runtime.getRuntime().availableProcessors();// 当前服务器的CPU核心数
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("segments", Segment[].class),
        new ObjectStreamField("segmentMask", Integer.TYPE),
        new ObjectStreamField("segmentShift", Integer.TYPE)
    };
    // 以下几个变量属于公共变量全部加了volatile
    // volatile可以保证操作的有序性和可见性，无法保证操作的原子性
    transient volatile Node<K,V>[] table;// 桶数组
    private transient volatile Node<K,V>[] nextTable;// 扩容时的新桶数组
    private transient volatile long baseCount;// 
    private transient volatile int sizeCtl;// 容量控制器，用途很多，一般用于在改变桶数组容量时作为CAS锁。
    private transient volatile int transferIndex;// 
    private transient volatile int cellsBusy;// 
    private transient volatile CounterCell[] counterCells;// 
    // 键集合、值集合、Entry实体集合的视图缓存，用于快速访问
    private transient KeySetView<K,V> keySet;// 键集合缓存
    private transient ValuesView<K,V> values;// 值集合缓存
    private transient EntrySetView<K,V> entrySet;// 键值对集合缓存
    
    // 以下几个字段都是final的，一旦赋值就不变了，其赋值就在下面的静态块中
    private static final sun.misc.Unsafe U;
    private static final long SIZECTL;
    private static final long TRANSFERINDEX;
    private static final long BASECOUNT;
    private static final long CELLSBUSY;
    private static final long CELLVALUE;
    private static final long ABASE;
    private static final int ASHIFT;
    //...
}
```
## 静态块解析
### 描述
### 源码
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    static {
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentHashMap.class;
            SIZECTL = U.objectFieldOffset// 获取字段偏移量
                (k.getDeclaredField("sizeCtl"));
            TRANSFERINDEX = U.objectFieldOffset
                (k.getDeclaredField("transferIndex"));
            BASECOUNT = U.objectFieldOffset
                (k.getDeclaredField("baseCount"));
            CELLSBUSY = U.objectFieldOffset
                (k.getDeclaredField("cellsBusy"));
            Class<?> ck = CounterCell.class;
            CELLVALUE = U.objectFieldOffset
                (ck.getDeclaredField("value"));
            Class<?> ak = Node[].class;
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
    //...
}
```
## 构造器解析
### 构造器描述
    无参构造器初始容量采用默认的初始容量16，负载因子为默认的0.75，一旦使用带参数的构造器自定义了容量或负载因子、并发级别等参数，那么就会根据给定的值进行内部换算，得出最优的初始容量值。
    集合的实际初始容量和参数指定的容量一般不同，而是根据一定的规则计算出来的。有两种计算方法，分别对应2号和5号构造器中的算法。
    2号构造器中计算方法类似HashMap中方式，只是在进行二进制转换(调用tableSizeFor方法)之前还需要经过一些基础计算：给定容量*1.5+1。
    5号构造器中计算方法也类似HashMap,同样需要在进行二进制转换之前进行一些计算：给定容量/负载因子+1
### 源码解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    // 1-无参构造器
    public ConcurrentHashMap() {
    }
    // 2-指定初始容量的构造器
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }
    // 3-指定Map映射集的构造器，等于将给定的Map集合改造为线程安全的集合
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }
    // 4-指定初始容量和负载因子
    // 负载因子一般最好使用默认的0.75，这是一个通过检验的空间与时间消耗的折中值
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }
    // 5-指定初始容量、负载因子、并发等级
    // 这个并发等级，其实就是可以同时支持的最大并发量，ConcurrentHashMap采用在
    // 数组位元素加锁的方式来防止并发，这种加锁方式保证针对不同数组位的操作是可以
    // 同时进行的，不存在线程不安全情况，那么也就是说可以同时支持最多数组容量个线
    // 程并发执行，如果给定的并发等级大于初始容量，必然导致出错，必须将容量设置
    // 成大于等于并发等级的值
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;//sizeCtl初始化为与容量值一样
    }
    //...
}
```
## 功能解析
### 添加元素操作
#### 功能描述
    ConcurrentHashMap添加新元素与HashMap添加新元素的整体流程是相似的，只是多了针对多线程的处理，同时在hash算法上也做了修改
#### 源码解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable {
    //...
    public V put(K key, V value) {
        return putVal(key, value, false);
    }
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());// 首先通过特定hash算法得出key的hash值
        int binCount = 0;// 链表长度
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
				// 首次添加元素时，先进行桶数组初始化操作
                tab = initTable();
            // 原子的获取下标i处的节点元素
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
				// 如果桶位下标i处没有元素，则直接将新元素置于该桶下标位
				// tab表示要操作的数组，i为要操作的数组下标，null为原来的下标位元素，
				// 最后一个参数为新的下标位元素，这里有种乐观锁的概念
				// 为防止多线程插值导致问题，这里采用CAS来进行原子插入操作
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
			// 如果指定桶位下标存在元素且其hash值为-1，则表示有线程正在进行扩容-元素迁移，
            }else if ((fh = f.hash) == MOVED)
                // 进行辅助迁移，扩容完成之后，还要继续进行元素的添加操作
                tab = helpTransfer(tab, f);
            else {
                // 针对桶位存在链表或者树的情况
                V oldVal = null;
				// 这里对f加锁，f是桶位元素，链表头元素、树根元素
                synchronized (f) {
					// 二次校验桶位元素还是不是f
                    if (tabAt(tab, i) == f) {
						// 如果hash值大于等于0，则说明是链表
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
									// 找到相同的key则执行更新value操作
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
									// 尾插法插入新元素到链表尾部
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
						// 针对红黑树结构
                        }else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
							// 执行红黑树新增节点操作
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
					// 如果链表长度达到树化阈值8，执行链表树化操作
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 元素添加完成之后的操作，增加元素数量
        addCount(1L, binCount);
        return null;
    }
    
    // 0x7fffffff的二进制为：0111 1111 1111 1111 1111 1111 1111 1111，除第一位为0全为1
    // 这里表示为1的位置为可用位，因为1的与操作拥有保留原值的功能。
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    
	// 对key的hashCode值进行高低位相异或，这个与HashMap一样
	// 然后在和0x7fffffff（首位为0,其余为1,共32位）相与，1具有保留原值效果，
	// 所以其实最后结果并未变动。即使是容量的最大值1<<<30,它也空出了首位只用了后31位
	// 相较HashMap的hash算法，多了一步与操作
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
    
    // addCount方法的主要功能就是多线程更新容器元素个数baseCount
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        // 原子更新baseCount的值
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            // 如果check小于等于1，那么仅仅校验是否存在竞争，即是否存在多个线程
            if (check <= 1)
                return;
            s = sumCount();
        }
        // 如果check大于等于0，则需要校验是否需要进行扩容
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                // sc是sizeCtl的值，如果其小于0，那么桶数组要么正在初始化，要么正在扩容，
                // 在上面排除了table为null的情况，那么这里只能是在进行扩容，然后，当前线程开始参与扩容
                if (sc < 0) {
                    // 如果所有区段都已经分配完毕，则不再对新的线程进行扩容分配
                    // 条件1：(sc >>> RESIZE_STAMP_SHIFT) != rs ==> 
                    // sizeCtr在扩容初始值是(rs << RESIZE_STAMP_SHIFT) + 2)，而在分配多线程进行扩容的时候，
                    // 又会对sizeCtr进行操作，每多一个线程参与扩容，sizeCtr就是加1，每有一个线程完成扩容任务
                    // 退出扩容队列时sizeCtr又会减1，当所有的线程都完成扩容之后，sizeCtr的值又恢复成初始值了
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    // 当前线程参与扩容迁移操作
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 否则sc大于等于0，这是sc其实是当前的扩容阈值。
                // 尝试将sizeCtl的值更新为(rs << RESIZE_STAMP_SHIFT) + 2)，
                // 更新成功的话，则由当前线程开启扩容操作，这里sizeCtl的值为一个负值
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
    final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }    
    static final int resizeStamp(int n) {
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }    
    //...
}
```
### 初始化操作
#### 功能描述
    与HashMap一样，ConcurrentHashMap的桶数组初始化也是在首次添加元素的时候完成的，但是后者比前者多了一个控制参数sizeCtl。
    控制参数sizeCtl在这里的作用相当于锁，只有获得锁的线程才能执行桶数组的初始化，其他线程只能将自己挂起，并不是阻塞，而是从运行状态变成可运行状态（Thread.yield()）。
    sizeCtl默认为0，如果构造器指定了容量则为实际容量，执行初始化时会被修改为-1，初始化完成之后会被设置为扩容阈值，即容量的0.75（亦即容量*负载因子）注意们这里计算sizeCtl的时候是写死的，即使负载因子被自定为其他值，这里也还是0.75
#### 源码解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                // 其他线程正在进行桶数组初始化，sc会被设置为-1，本线程坐等初始化完成。
                Thread.yield(); // lost initialization race; just spin
            // 否则尝试将sizeCtl的值原子的更新为-1，然后进行初始化操作
            // 当有多个线程同时到达这里的时候，为避免重复初始化桶数组，这里才进行原子更新，
            // 只有抢到锁的线程才能执行桶数组初始化，而其他线程因为原子更新失败而开启下一循环，
            // 却发现sc<0，线程停止执行，变成可运行状态，等待CPU分配时间进行执行。
            // 当我们用无参构造器创建ConcurrentHashMap实例时，sizeCtl应为是0，
            // 当我们指定容量或者负载因子时，sizeCtl为计算出的实际容量值
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    // 如果sc==0，那么是使用无参构造器创建的ConcurrentHashMap，
                    // 这时需要使用默认容量16,否则就要使用通过给定容量计算出来的实际容量sc
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);// 最后将sc设置为n的四分之三值即0.75倍的容量，这相当于扩容阈值
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
    //...
}
```
### 扩容操作
#### 功能描述
##### 引发扩容的因素
    1-容器某一个桶位的元素数量达到8个但是容器总元素数量不足64个的情况，优先触发扩容操作
    2-容器新增元素之后，元素数量达到扩容阈值的情况下，要进行扩容
> 正是上述两处位置会触发扩容，正对应于tryPresize方法和addCount方法。而这两个方法里面拥有几乎相同的逻辑来触发扩容（transfer(tab, null)），第二个参数传null，代表当前线程是首个触发扩容的线程，如果第二个参数不是null，则当前线程为辅助扩容线程。
##### 基础准备
    准备1：扩容是由sizeCtr来控制的，在扩容的过程中，sizeCtr代表扩容的线程数量。
    准备2：rs和sc多次出现，其中rs是resizeStamp(n)的值，是一个极大的负值，扩容开启时初始化sizeCtr的时候就是使用这个负值左移16位后再加上2的结果，而sc指的就是sizeCtr的当前值。
    准备3：ForwardingNode是一个临时节点，表示的是完成迁移的桶数组位节点
    准备4：advanced值表示的是区间分配的循环条件，当advanced为false则结束循环，结束循环的因素有很多，包括，当期区段未转移完成或者所有元素均转移完成，否则所有元素的迁移都分配完成，否则完成了一次新的区间分配
    准备5：transferIndex是一个公共变量，保存的是多个线程分配区间后剩余未分配区间的最大数组位（倒序分配），只要它大于0，则还有待分配的区间需要进行元素转移，这个区间可能分配给新加入的线程，也可能分配给完成了一次区间元素迁移的老线程
    准备6：stride是区间分配步长：单核CPU的情况下,stride直接取旧数组长度n（也就是只分一个区间），如果n比16还小，则stride值设为16，多核CPU情况下，stride需要通过计算(n >>> 3) / NCPU（就将n除以8再除以CPU数量的结果），如果得出的结果小于16,那么还是以16为步长，也就是说，stride最小值为16
##### 扩容逻辑
    当第一个线程发现容器需要扩容，首先会CAS原子更新sizeCtr的值为(rs << RESIZE_STAMP_SHIFT) + 2)，这是一个负值，然后调用transfer(tab，null)方法开启扩容流程，由于该线程属于触发扩容的第一个线程，新桶数组并未创建，所以此处传null。
    进入到transfer方法之后，开始进行元素迁移操作，由于是开启扩容的线程，nextTab=null，所以需要先进行新桶数组的创建，如果是辅助迁移的线程，那么nextTab已经存在，直接传值即可（其为公共变量）
> 新桶数组<br/>
    创建新桶数组最主要的就是确认桶数组的容量，这里新桶数组直接就是旧桶数组容量的2倍，如果创建失败，则将sizeCtr置为Integer的最大值，将sizeCtr置为最大值后将不会再触发扩容。<br/>

    然后，为当前线程分配区间，分配区间的原理其实就是以bound来划分数组区段，bound指的是当前分配区间的最小桶位下标,transferIndex指向的是剩余未分配区间的最大位+1，其实和最后一次区间分配的线程的bound值一样，i指的是当前分配区间的最大桶位下标,如此一来i和bound就将每个线程的区间给固定了，通过--i来倒序循环迁移每一位的元素，而transferIndex面对的是所有的线程，属于公共变量，每个线程分配区间时，都会更新transferIndex的值。transferIndex的值逐渐减小，直到变成0代表旧桶数组的所有位均被分配完毕。
    区间分配好之后，然后就是针对区间内的元素进行迁移了，ConcurrentHashMap采用的是倒序迁移（--i）：
> 迁移逻辑<br/>
    迁移的情况分为两种，一种是针对单向链表，另一种是针对红黑树<br/>
    单向链表：类似于HashMap,将长链表分拆成为两个小链表，分别迁移到新数组的原位（低位）与原位+旧桶数组容量n（高位）的位置。<br/>
    红黑树：也类似于HashMap,将大红黑树拆成两个小树，如果小树中元素数量达到6个以下，则将其退化为单项链表，然后再将两个小树（或链表）分别迁移到新数组的原位（低位）和原位+旧桶数组容量n（高位）的位置。<br/>
    具体分拆逻辑可以参照源码注释

    i会逐渐减小，直到其等于bound为止，代表当前区间的元素迁移完毕（或者说当前线程的前一任务完毕），这时候要检查是否还有剩余的区间需要分配，如果transferIndex>0,则还有未分配的区间，那么为当前线程再分配一段区间，如果transferIndex<=0，则所有区间均已分配完毕，中断当前线程。
    等到所有的扩容线程均完成了自己所分配的迁移任务之后，参与扩容的线程逐步减少，每减少一个线程，sizeCtr就会减1，最后一个线程完成扩容之后，最后sizeCtr恢复了扩容前的初始值，然后将finishing置为true，i置为n。
    然后执行扩容后结束操作，主要内容就是置空nextTable变量，表示退出扩容期，重置table和sizeCtl，sizeCtl在非扩容期间为扩容阈值（桶数组容量*0.75）
> 辅助迁移<br/>
    所谓的辅助迁移就是添加元素的线程发现容器正处于扩容期间，则暂停添加操作，先辅助扩容，待扩容完成，再执行添加操作。
#### 源码解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    // 扩容操作
    private final void tryPresize(int size) {
        // c表示扩容后的实际容量，当给定的加倍的容量size大于等于最大容量的1/2，
        // 直接将实际容量设置为最大容量，否则通过之前的公式计算实际容量
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            // 如果桶数组还未初始化，则初始化桶数组
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;// n取sc和c中的较大值
                // 然后将sizeCtl设置为-1，进行桶数组初始化
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            sc = n - (n >>> 2);// 计算sc=当前桶数组容量*0.75
                        }
                    } finally {
                        // 最后将sizeCtl置为扩容阈值（默认的）
                        sizeCtl = sc;
                    }
                }
            }
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                // 如果计算得到的c(新桶数组容量)还没有sc（旧桶数组容量）大，或者旧桶数组
                // 的容量就已经达到或超过桶数组允许的最大值的话，这里不再进行扩容操作
                break;
            // 只有真正的扩容时才会执行以下操作
            else if (tab == table) {
                int rs = resizeStamp(n);// 我们只要知道这里得到的rs是一个负值
                // sc何时小于0：
                //  1-初始化桶数组的时候值为-1
                //  2-当正在进行扩容的时候，sc的值小于0
                if (sc < 0) {
                    // 这里的sc小于0,表示已经开始扩容，这种情况下，
                    // 在还存在待扩容区间的情况下，当前线程也加入扩容行列
                    Node<K,V>[] nt;
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        // 这种情况是在不存在扩容区间的情况下，不再让新线程加入扩容行列，直接中断
                        break;
                    // sizeCtl+1表示增加一个扩容线程，这里调用transfer时必然存在nexttable
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 否则的话当前线程就是第一个发起扩容的线程，表示nextTable还不存在，
                // 所以这里讲sizeCtl初始化为(rs << RESIZE_STAMP_SHIFT) + 2)，
                // 并且调用transfer的时候nextTable传null
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
    // 扩容后元素迁移操作
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        // n表示旧数组的长度，stride为处理步长
        int n = tab.length, stride;
        // 单线程情况下步长stride=n，
        // 多线程情况下，stride=(n >>> 3) / NCPU.(即：n/8/cpu数量)，这个值最小为16
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        // 发起迁移的第一个线程调用该方法时，nextTab会传null，
        // 剩余不会，针对首次传null的情况，这里进行处理：新建扩容数组
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];// 数组加倍扩容
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            // transferIndex为迁移位置的控制量，起初其值为桶数组最高位置下标+1（即数组size）
            transferIndex = n;
        } 
        int nextn = nextTab.length;// 新数组的长度
        // 创建一个标记节点fwd，将hash置为-1（MOVED），表示的是处理完成的节点,后面要用。
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;// advance表示的是是否结束区间分配循环，false表示结束循环
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            // 下面的这个while循环的作用就是划分区域，其中i指向transferIndex,
            // bound指向transferIndex-stride，那么这两个值就代表了一个小区间，
            // 这个小区间就是当前线程需要进行处理迁移的区间
            while (advance) {
                int nextIndex, nextBound;
                // 1-bound指向的是旧桶数组的一个下标位，它和i共同决定了一个区间，
                // 区间大小为stride(最小为16)且，指向高位，bound指向低位
                // 2-如果当前线程的区间处理完毕，而还未完成所有元素迁移的情况下，则再次进行区间分配，
                // 如果所有元素都迁移完毕finishing=true，则结束区间分配
                // 3-这里的--i是控制外层for循环的条件，表示逐个桶位元素的迁移，从后到前
                if (--i >= bound || finishing)
                    // 如果--i >= bound，那么结束while循环，开始--i桶位元素迁移操作
                    // --i >= bound表示区间从后向前的下一位未到达边界bound，还可以继续进行元素迁移
                    // finishing表示旧桶数组所有元素均迁移完毕的情况
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    // transferIndex是公共变量，如果它小于等于0，则表示迁移工作已全部完成，
                    // 结束内部循环并将i置为-1，下一步走下面的结束逻辑
                    i = -1;
                    advance = false;
                }
                // TRANSFERINDEX位置保存的值是上一线程分区的bound值，
                // 即上一个区间的bound值，bound值要比i值小stride
                // 这里是真正的区间分配的逻辑，分配完成advance置为false，while循环得以结束
                // 这里有个三元操作符，作用是当剩余的元素不足16时，直接将剩余全部元素作为一个
                // 区间分配给当前线程，将transferIndex更新为0
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            // 当旧桶数组的所有节点的元素都迁移完毕之后，进行如下结束逻辑
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    // 扩容结束的具体逻辑：nextTable置null，将新的桶数组作为
                    // 当前桶数组table,再计算新的阈值，赋值给sizeCtr，为桶数组容量*0.75
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);// 0.75*n
                    return;
                }
                // 原子更新sizeCtl，减1，表示当前线程已完成迁移任务，表示进行迁移的线程数量减少了一个
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // 迁移元素的线程逐个减少，这里sizeCtr的值逐渐减小，直到其值等于之前赋的初始值
                    // （resizeStamp(n) << RESIZE_STAMP_SHIFT-2）的时候，表示所有扩容的线程都完
                    // 成了扩容，亦即扩容结束，随将finishing置为true，将i=n，然后再次for循环的时
                    // 候进入while循环后，因为finishing=true，第一个判断就结束了，然后再次进入结束
                    // 逻辑，因为i=n而进入逻辑，因为finishing=true而执行扩容结束的具体逻辑
                    // 所有扩容线程中只有最后一个完成扩容的线程可以保留来进行扩容结束的逻辑，之前的
                    // 所有线程都在下面的这个return结束了
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                // 每一个旧桶位的元素迁移完成后，都需要将旧桶数组位置null，
                // 这里找到这个null桶位之后，将其更新为ForwardingNode节点，表示已完成迁移
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                // 如果当前节点是ForwardingNode节点，则说明该节点数据已迁移完毕，不再进行处理，跳过。
                advance = true; // already processed（已处理）
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            // 链表元素迁移
                            // 迁移逻辑：先将原链表分拆成两个小链表，然后将两个小链表分别置于
                            // 新桶数组的对应桶位，然后将原桶位置为fwd节点表示迁移结束。
                            // 分拆原理：n为旧桶容量，我们知道所有的桶容量就是2的次幂，二进制表示
                            // 的话就是1个1，N个0的二进制值，将fh(首结点的hash值)与n进行与操作，
                            // 结果只会有两种，一种就是0，一种就是n，分别对应新桶数组的原桶位（新桶低位）
                            // 和原桶位+旧桶容量位（新桶高位），分别用ln和hn表示
                            int runBit = fh & n;// n为旧桶容量
                            Node<K,V> lastRun = f;
                            // 下面这个循环的目的是找出链表最后一个“与结果”变化的链表节点lastRun，
                            // runBit表示最后的变化的“与结果”，可能是0可能是n，如果为0，则将最后这
                            // 一变化节点赋值给ln表示低位，否则赋值给hn表示高位，这样该节点之后的
                            // 节点将不用再参与后面的分拆，因为它们的与结果与lastRun的一样，要么全
                            // 是0要么全是n。所以下面的这个for循环和if-else其实是为了简化分拆链表
                            // 的操作，当然如果入到极端情况，最后还是发生变化的情况，那么分拆的时候
                            // 还是需要全部判断的。
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 组装两个新的小链表
                            // 组装链表的时候使用的构造器最后一个参数是当前节点的后置节点，
                            // 之前找出的ln和hn正好用于此处。
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            // 最后将组装好的小链表定位到新数组的指定位置，再将原数组的指定位置置为fwd表示迁移完毕。
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            // 红黑树元素迁移
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                // 下面的if-else其实是在组装高位与低位的两个双向链表
                                // 如果当前节点的hash值h与n的与结果为0，则当前节点位于低位
                                if ((h & n) == 0) {// 低位
                                    if ((p.prev = loTail) == null)
                                        // 针对第一个与结果为0的节点，将其置为低位首节点
                                        lo = p;// lo代表低位首节点
                                    else
                                        // 针对非首个与结果为0的节点进行处理
                                        loTail.next = p;
                                    loTail = p;// 循环的推进器
                                    ++lc;// 低位容量
                                }
                                else {// 高位
                                    if ((p.prev = hiTail) == null)
                                        // 针对第一个与结果为1的节点，将其置为高位首节点
                                        hi = p;// hi代表高位首节点
                                    else
                                        // 针对非首个与结果为1的节点进行处理
                                        hiTail.next = p;
                                    hiTail = p;// 循环的推进器
                                    ++hc;// 高位容量
                                }
                            }
                            // 将准备好的两个双向链表进行树化，当然如果该位节点数量为6个及以下，则结构会退化为单向链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            // 最后将组装好的高位与低位的结构置于新桶数组的指定位置，再将旧桶数组指定位置置为fwd表示迁移结束
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
    // 辅助迁移操作，当一个线程准备执行添加元素的操作时发现正在扩容，那么它就会停止添加操作，并且去辅助扩容操作。
    // 其实所谓的辅助扩容就是为当前线程分配一段桶位区间进行元素迁移操作
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        // 
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                // sizeCtl值原子加1，表示执行扩容的线程由多了一个，那么sizeCtl在线程扩容期间表示的就是执行扩容的线程的数量
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
    // 该方法的结果是一个负值
    static final int resizeStamp(int n) {
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
    //...
}
```
### 获取元素操作
#### 功能描述
    获取指定key的值，需要先对key进行hash,找到对应的桶位
#### 源码解析
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    //...
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {// e为桶位元素
            // 针对桶位元素就是目标元素的情况
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            // 如果头结点的 hash 小于 0，说明 正在扩容，或者该位置是红黑树
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 针对链表进行处理，获取指定的值
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
    static class Node<K,V> implements Map.Entry<K,V> {
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                // 遍历操作与链表一样，是因为红黑树结构其实还是一个双向链表
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
    //...
}
```
### 红黑树
#### 树形化操作
##### 功能描述

##### 源码解析


## 总结
### sizeCtr

### hash

