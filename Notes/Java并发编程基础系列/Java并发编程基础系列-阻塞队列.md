# Java并发编程基础系列-阻塞队列
## 概述
    阻塞队列的特点就是阻塞效果，队列满时阻塞添加操作（put），队列空时阻塞获取操作（take）
    阻塞队列的作用就是用于生产者消费者模式。
    BlockingQueue继承自接口Queue，这两个接口中定义了阻塞队列的基本方法：

| 序号 | 方法                                                         | 作用                                                         |     来源      |
| :--: | ------------------------------------------------------------ | ------------------------------------------------------------ | :-----------: |
|  1   | boolean add(E e)                                             | 往队列添加元素e，成功返回true，队列已满抛出IllegalStateException |     Queue     |
|  2   | boolean offer(E e)                                           | 往队列添加元素e，成功返回true，失败返回false，优于add方法    |     Queue     |
|  3   | E remove()                                                   | 查找并移除头元素，成功返回移除的元素，队列为空则抛出NoSuchElementException |     Queue     |
|  4   | E poll()                                                     | 查找并移除头元素，成功返回移除的元素，队列为空返回null       |     Queue     |
|  5   | E element()                                                  | 查找头元素，成功返回头元素，队列为空则抛出NoSuchElementException |     Queue     |
|  6   | E peek()                                                     | 查找头元素，成功返回头元素，队列为空返回null                 |     Queue     |
|  7   | void **put**(E e) throws InterruptedException                | 往队列添加元素e，如果队列已满，则阻塞                        | BlockingQueue |
|  8   | boolean offer(E e, long timeout, TimeUnit unit)throws InterruptedException | 往队列添加元素e，成功返回true，如果队列已满则等待指定时间，若队列还是满则返回false | BlockingQueue |
|  9   | E **take**() throws InterruptedException                     | 查找并移除头元素，成功返回移除的元素，队列为空则阻塞         | BlockingQueue |
|  10  | E poll(long timeout, TimeUnit unit)throws InterruptedException | 查找并移除头元素，成功返回移除的元素，如果队列为空则等待指定时间，如果还是为空则返回null | BlockingQueue |

## 阻塞队列
### 有界队列
​	有界队列指的是存在边界的队列，也就是可能出现满员的队列
#### ArrayBlockingQueue
​	ArrayBlockingQueue是底层使用数组实现的阻塞队列，一旦创建，大小不再变化，支持先进先出。
##### 构造器
它有三个构造器用于创建实例，涉及的参数包括：
- int capacity：初始化数组容量
- boolean fair：是否公平队列（取决于使用的锁是不是公平锁）
- Collection<? extends E> c：最初要包含的元素的集合
> 引申：公平
  	公平的意思是阻塞的线程可以按照阻塞的先后顺序来访问队列，而不公平则是阻塞的线程会通过竞争来决定谁来访问队列，与阻塞顺序无关，默认情况下的ArrayBlockingQueue是非公平的阻塞队列，但是如果我们将fair传true，则会创建一个公平的阻塞队列。
##### 简单原理
​	往数组队列添加元素的时候，是按照顺序从数组下标0开始直到下标最大值，使用put操作添加元素，如果当前数组元素数量count与数组容量相同，那么就阻塞等待，直到出现空位，然后将新元素添加到空位，在ArrayBlockingQueue中有两个字段putIndex和takeIndex，分别对应要添加和移除的数组位下标，当添加完一个新元素putIndex就会自增指向下一个新元素要添加的数组位下标，当移除一个老元素takeIndex就会自增指向下一个要溢出的元素的数组位下标，如果数组满员，那么可以说明takeIndex一定和putIndex指向的同一个下标，当下一个元素被移除takeIndex自增，指向数组下一位，新元素就能添加到空出来的位，而putIndex也自增，再次和takeIndex重合。
	putIndex和takeIndex在到达数组最后位后会折返回来再次从0开始，如此往复。
	在操作元素的时候是通过加锁实现的，使用的锁是通过参数fair指定的锁（公平锁与非公平锁），默认为非公平锁。
##### 使用
​	一般我们使用单参数构造器（指定底层数组容量）来创建数组队列，这是个默认的不公平队列。
	数组队列的先天优势是查询快速，劣势是增删较慢（需要操作之后的所有元素）。
#### LinkedBlockingQueue
​	LinkedBlockingQueue是底层使用链表实现的阻塞队列，链表拥有天生的自由扩展的容量，当然我们也可以通过执行容量来进行防止其随意增长。
##### 构造器
​	LinkedBlockingQueue同样有三个构造器：无参构造器、两个单参构造器，涉及到参数为：
- int capacity：初始化链表容量
- Collection<? extends E> c：最初要包含的元素集合
##### 简单原理
​	链表节点为Node。
	LinkedBlockingQueue中有两个锁，分别对应于put生产操作和take消费操作，称为生产锁和消费锁。如此就表明，在链表队列中生产操作和消费操作是可以同时进行的。
	如此一来针对同一个元素的生产和消费同时进行该如何处理呢，这里有一个公共字段count保存链表中元素个数，这是个AutoInteger类型的值，它的更新会在锁内部进行，而其又是判断是否执行生产和消费操作的条件，这里就拿count来进行控制。
	针对首个元素，当生产线程添加完新元素到链表后，紧接着会执行count自增操作，结束后释放生产锁，消费线程消费的前提是在count不为0的情况下才会执行消费，添加之前count=0，添加之后count=1,count是在添加操作完成后自增为1，而且自增操作为原子操作，这时消费者线程可能已经循环等待了很久，在等待count变化为大于0的数，这边count刚刚自增为1，尚未释放生产锁的时候，那边消费者线程就会发现count不再为0，开始进行元素消费，完成后再把count自减（自减也是原子操作），但如果在此之前又有新生产线程添加新元素，导致count自增,那么他会变成2，这会不会与消费线程里面的count=1冲突呢？当然不会，count是AutoInteger类型，针对其的自增自减操作均是原子的，如果第二个自增先执行，那么count就会变成2，下面消费线程自减，就又变成了1。此时链表中有1个元素
	首节点为头节点，新增的元素位于链表尾部。
##### 使用
​	一般我们使用初始化容量的构造器创建链表队列，这样创建的是一个有界的阻塞队列。
	链表队列的先天优势是增删块速，劣势是查询较慢（需要从头遍历）
#### SynchronousQueue
​	SynchronousQueue是同步队列，其内部无法存储元素，他只用来转移元素，一个生产必须有一个消费，且必须同时存在，仅有生产线程会等待消费线程，反之亦然。
	由于无法存储元素，所以是有界队列，而且界限为0。
	同步队列就像是一个连接器，用于将一个生产线程和一个消费线程对接起来，数据直接在二者之间传递，不会经过队列。
	同步队列可以有两种方式，公平与非公平，公平同步队列采用双队列TransferQueue实现，非公平同步队列采用双协议栈TransferStack实现。至于元素的转移就是靠这两个内部类中的transfer方法完成的。
	每一个生产线程都在等待一个消费线程与其配对，否则无法再生产元素（添加元素）。
##### TransferQueue

##### TransferStack

#### LinkedBlockingDeque
​	Deque是“double ended queue”的简写形式，表示双端队列。
### 无界队列
​	无界队列指的是没有边界的队列，也就是可以无限存储元素的队列
#### PriorityBlockingQueue
​	PriorityBlockingQueue是支持优先级的无界阻塞队列。

#### DelayQueue

#### LinkedTransferQueue