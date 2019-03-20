# Java并发编程系列-AbstractQueuedSynchronizer

## 一、概述
AbstractQueuedSynchronizer简称为AQS，是并发包中用于实现并发工具的基础类，非常明显，它是一个抽象类。

它提供了一个依赖于FIFO队列的框架用于实现各种阻塞锁与同步器。

它依赖于一个int值来表示状态，并定义了获取和修改该状态值的原子方法，具体的同步器需要实现该抽象类，并且使用它定义的这些原子方法来操作状态值。

它的实现类一般作为待实现的同步器的静态内部类而存在，用来提供一些方法来实现同步器的功能。

我们可以将其看作是基础的同步器，并不是具体的某一个同步器，而是同步器的一个抽象。
## 二、源码解析
### 2.1 继承体系解析
首先来看看其继承体系：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {}
```
可以看到它继承了AbstractOwnableSynchronizer抽象类，这个类很简单，我们可以整体来看看：
```java
// 就是一个简单的独占式同步器，持有被独占拥有的线程
public abstract class AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private static final long serialVersionUID = 3737899427754241961L;
    // 供子类调用的构造器
    protected AbstractOwnableSynchronizer() { }
    // 表示独占拥有的线程，下面是其get和set方法
    private transient Thread exclusiveOwnerThread;
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```
### 2.2 内部类解析
#### 2.2.1 Node
静态内部类Node用于将要加入同步队列的线程封装成为队列节点。这个队列采用双向链表实现，支持先进先出。
##### (1)修饰符:
```java
static final class Node {}
``` 
> 该静态内部类被final修饰，表明作者希望其不被继承修改。

##### (2)字段：
```java
static final class Node {
    // 两个节点标记，用于标识节点对应的线程获取锁的模式，是共享式获取，还是独享式获取
    static final Node SHARED = new Node();// 共享模式的节点标记
    static final Node EXCLUSIVE = null;// 独享模式的节点标记
    // 四个节点状态，其实还有一个状态为0-表示当前节点在同步队列中，等待着获取锁
    static final int CANCELLED =  1;// 表示当前节点封装的线程被中断或者超时
    static final int SIGNAL    = -1;// 表示当前节点的后继节点需要被唤醒（unpark）
    static final int CONDITION = -2;// 表示当前节点位于等待队列中，在等待条件满足
    static final int PROPAGATE = -3;// 表示当前场景下后续的acquireShared能够得以执行？？
    // 节点状态，其值就是上面定义的这四个状态值再加上0
    volatile int waitStatus;
    // 同步队列的节点指针
    volatile Node prev;// 双向链表中节点指向前节点的指针
    volatile Node next;// 双向链表中节点指向后节点的指针
    // 节点封装的执行线程
    volatile Thread thread;
    // 等待队列的节点指针
    Node nextWaiter;// 单向链表中节点指向后节点的指针
}
```
> 节点状态：
> - 0：默认状态，表示节点是同步队列中等待获取锁的线程的节点
> - 1：CANCELLED，表示节点被取消，原因可能是超时或者被中断，一旦置于该状态，则不再改变
> - -1：SIGNAL，表示当前节点的后继节点被阻塞（或即将被阻塞）（使用park），因此当前线程释放锁或者被取消执行时需要唤醒（unpark）后继节点
> - -2：CONDITION，表示当前节点位于等待队列中，当节点被转移到同步队列的时候，状态值会被更新为0
> - -3：PROPAGATE，表示持续的传播releaseShared操作

##### (3)构造器：
```java
static final class Node {
    Node() {}
    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }
    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```
> 三个构造器各有用处：
> - Node()：用户初始化头结点，或者创建共享标记SHARED
> - Node(Thread thread, Node mode)：给同步队列添加新节点时使用，用于构造新节点
> - Node(Thread thread, int waitStatus)：给等待队列添加新节点时使用，用于构造新节点
>
> 注意：上面的构造器中的mode（模式）属于Node类型，它有两种模式SHARED和EXCLUSIVE，分别表示共享模式和独享模式。而waitStatus表示的是节点状态。

##### (4)方法：
```java
static final class Node {
    // 校验当前节点是否是共享模式
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    // 获取前置节点，必须为非null
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
}
```
> 方法解析：
> 
> isShared方法主要用于校验当前节点的锁获取模式，是共享还是独享，实现方式采用nextWaiter与SHARED比较，参照上面的第二个构造器的实现，我们可以知道在新增一个节点的时候，会对节点的nextWaiter进行赋值，而所赋的值正好是新增节点的模式标记，可以说nextWaiter持有节点的模式标记，那么拿其来与SHARED进行比较就是很显然的事情了。
>
> predecessor方法用于获取前置节点，主要是在当前置节点不可为null时使用，这样当前置节点为null，就会抛出空指针。
#### 2.2.2 Condition
Condition并非AQS中的内部类，而是其内部类ConditionObject的父接口，为了后面的ConditionObject，我们提前了解下Condition。

Condition是一个接口，旨在定义一系列针对获取锁的线程的操作，实现类似于Object类中wait/notify的功能。我们通过其方法定义可以明显感觉到这一点。
```java
public interface Condition {
    // 使当前线程等待，知道被唤醒或者中断，注意需要在临界区使用，执行该方法之后该线程持有的锁将被释放，线程处于等待状态
    // 四种情况下会退出等待状态：被signal唤醒，被signalAll唤醒，被interrupt唤醒（需要当前线程可以响应中断），发生伪唤醒
    void await() throws InterruptedException;
    // 使当前线程等待，直到被唤醒（不响应中断），注意要在临界区使用，执行该方法之后该线程持有的锁将被释放，线程处于等待状态
    // 三种情况下会退出等待状态：被signal唤醒，被signalAll唤醒，发生伪唤醒
    void awaitUninterruptibly();
    // 使当前线程等待，知道被唤醒或者中断或者超时，注意需要在临界区使用，执行该方法之后该线程持有的锁将被释放，线程处于等待状态
    // 五种情况下会退出等待状态：被signal唤醒，被signalAll唤醒，被interrupt唤醒（需要当前线程可以响应中断），超时，发生伪唤醒
    // nanosTimeout表示当前线程要等待的时间长度
    // 该方法返回一个正数表示线程被提前唤醒，返回一个负数或0表示等待超时
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    // 同上，不同在于上面的只能传参为纳秒值，该方法可以通过单位随便传值
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    // 使当前线程等待，知道被唤醒或者中断或者过了截止日期，注意需要在临界区使用，执行该方法之后该线程持有的锁将被释放，线程处于等待状态
    // 退出等待状态的情况同上，只是这里传参为一个固定的时间点，线程等待到这个时间点将自动苏醒
    boolean awaitUntil(Date deadline) throws InterruptedException;
    // 唤醒等待队列中的一个线程，该线程从await返回时必须获取到锁
    void signal();
    // 唤醒等待队列中的所有线程，每个线程从await返回时必须获取到锁
    void signalAll();
}
```
#### 2.2.3 ConditionObject
ConditionObject是Condition的实现类，在AQS中以普通内部类的方式存在。

ConditionObject内部维护了一个单向链表实现的等待队列，队列的节点与AQS中同步队列的节点类型一致，均为上面的内部类Node类型。

下面我们来仔细看看这个类：
##### (1)体系结构
```java
public class ConditionObject implements Condition, java.io.Serializable {}
```
该类实现了Condition接口和Serializable接口，拥有序列化功能
##### (2)字段
```java
public class ConditionObject implements Condition, java.io.Serializable {
    // 序列化ID
    private static final long serialVersionUID = 1173984872572414699L;
    // 等待队列头结点指针
    private transient Node firstWaiter;
    // 等待队列尾节点指针
    private transient Node lastWaiter;
    // 中断模式
    private static final int REINTERRUPT =  1;// 退出等待队列时重新中断
    private static final int THROW_IE    = -1;// 退出等待队列时抛出InterruptedException异常
}
```
> 我们可以看到类的五个字段中除了三个静态字段之外，剩下的两个被transient修饰，也就是说虽然该类支持序列化，但是序列化无值。
##### (3)方法
ConditionObject中的公共方法其实就是对Condition接口中定义方法的实现，下面我们逐个分析：
> **await()**：
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     public final void await() throws InterruptedException {
>         // 1-响应中断，同时会清除中断标记
>         if (Thread.interrupted())
>             throw new InterruptedException();
>         // 2-将当前线程封装成Node节点并添加到等待队列尾部
>         Node node = addConditionWaiter();
>         // 3-释放当前线程所占用的lock，在释放的过程中会唤醒同步队列中的下一个节点
>         int savedState = fullyRelease(node);
>         int interruptMode = 0;
>         // 4-阻塞当前线程，直到被中断或者被唤醒
>         while (!isOnSyncQueue(node)) {// 校验当前线程是否被唤醒（是否被转移到同步队列），如果已唤醒则退出循环
>             LockSupport.park(this);// 阻塞当前线程
>             if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)// 校验当前线程是否被中断
>                 break;// 如果被中断则退出循环
>         }
>         // 5-自旋等待获取到同步状态（即获取到lock）
>         if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
>             interruptMode = REINTERRUPT;
>         if (node.nextWaiter != null) // clean up if cancelled
>             unlinkCancelledWaiters();
>         //  6-处理被中断的情况
>         if (interruptMode != 0)
>             reportInterruptAfterWait(interruptMode);
>     }
> }
> ```
> 方法解析：
> - 第一步：优先响应中断，首先校验当前线程是否被中断，如果被中断则抛出InterruptedException异常，否则下一步；
> - 第二步：调用addConditionWaiter()方法，目的是将当前线程封装成为Node节点并添加到等待队列的尾部，源码如下：
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     private Node addConditionWaiter() {
>         Node t = lastWaiter;// 保存尾节点
>         // If lastWaiter is cancelled, clean out.
>         // 如果尾节点线程被取消，则清除之
>         if (t != null && t.waitStatus != Node.CONDITION) {
>             unlinkCancelledWaiters();// 清除等待队列中所有的被取消的线程节点
>             t = lastWaiter;
>         }
>         // 将当前线程封装成为等待队列的Node节点
>         Node node = new Node(Thread.currentThread(), Node.CONDITION);
>         if (t == null)
>             // 如果等待队列为空，则将新节点作为头节点
>             firstWaiter = node;
>         else
>             // 否则将新节点作为新的尾节点添加到等待队列中
>             t.nextWaiter = node;
>         // 更新尾节点指针
>         lastWaiter = node;
>         return node;
>     }
> }
> ```
> > 这个方法里面除了封装节点和添加节点之外，还有针对等待队列进行清理的流程，主要是为了清理被取消的线程节点
> 
> - 第三步：调用fullyRelease(node)方法，用于释放当前线程所持有的锁并唤醒同步队列的下一节点，详情可见AQS方法解析部分；
> ```java
> public abstract class AbstractQueuedSynchronizer
>     extends AbstractOwnableSynchronizer
>     implements java.io.Serializable {
>     final int fullyRelease(Node node) {
>         boolean failed = true;
>         try {
>             int savedState = getState();// 获取同步状态state值
>             // 执行release方法，尝试释放当前线程持有的共享状态，并唤醒下一个线程
>             if (release(savedState)) {
>                 failed = false;
>                 return savedState;
>             } else {
>                 throw new IllegalMonitorStateException();
>             }
>         } finally {
>             if (failed)
>                 node.waitStatus = Node.CANCELLED;
>         }
>     }
> }
> ```
> - 第四步：调用LockSupport.park(this)阻塞当前线程，一但消除被中断后者线程被唤醒转移到同步队列，则退出循环，继续下一步；
> > 这里涉及到一个中断模式的问题。中断模式之前提到过，有两种：REINTERRUPT和THROW_IE，分别表示针对被中断的线程在退出等待队列时的处理方式，前者重新中断，后者则抛出异常。
> > 此处interruptMode表示的就是中断模式的值，初始赋值为0，然后通过checkInterruptWhileWaiting(node)方法不断的进行校验，其源码如下：
> > ```java
> > public class ConditionObject implements Condition, java.io.Serializable {
> >     private int checkInterruptWhileWaiting(Node node) {
> >         return Thread.interrupted() ?
> >             (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
> >             0;
> >     }
> > }
> > ```
> > 如果线程被中断则通过方法transferAfterCancelledWait(node)判断线程是否是在被唤醒之前被中断，如果是则返回true，否则返回false；如果返回true则采用THROW_IN模式，否则采用REINTERRUPT模式。无论是上面的哪一种模式都代表线程被中断了，那么此处interruptMode就不再是0，那么条件成立，break退出循环。除此之外transferAfterCancelledWait(node)方法无论返回true还是false，都会将现场节点转移到同步队列中
> - 第五步：当前线程已经被转移到同步队列中，然后开始自旋以获取同步状态，待其获取到同步状态（锁）之后，返回该线程是否被中断，如果被中断，再根据其中断模式进行整理，如何整理呢，主要就是如果当前中断模式是THROW_IE模式，则保持不变，否则一律修改成REINTERRUPT模式，之后会再次进行一次同步队列节点清理。
> - 第六步：最后针对不同的中断模式进行中断处理，如果是THROW_IN则抛出异常，如果是REINTERRUPT则再次进行中断。

> **awaitNanos(long)**:
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     public final long awaitNanos(long nanosTimeout)
>             throws InterruptedException {
>         // 1-优先响应中断
>         if (Thread.interrupted())
>             throw new InterruptedException();
>         // 2-将当前线程封装成Node节点并添加到等待队列尾部
>         Node node = addConditionWaiter();
>         // 3-释放当前线程所占用的lock，在释放的过程中会唤醒同步队列中的下一个节点
>         int savedState = fullyRelease(node);
>         final long deadline = System.nanoTime() + nanosTimeout;// 计算截止时间点
>         int interruptMode = 0;
>         // 4-阻塞当前线程，直到被中断或者被唤醒或者超时
>         // 4-1 校验当前线程是否被唤醒，如果没有进入循环体
>         while (!isOnSyncQueue(node)) {
>             // 4-2 如果超时时间小于等于0，则表示线程立即超时，然后进行线程节点转移处理，并结束循环
>             if (nanosTimeout <= 0L) {
>                 transferAfterCancelledWait(node);// 转移线程节点
>                 break;
>             }
>             // 4-3 如果超时设置时间nanosTimeout大于等于spinForTimeoutThreshold，则进行定时阻塞当前线程
>             if (nanosTimeout >= spinForTimeoutThreshold)
>                 LockSupport.parkNanos(this, nanosTimeout);
>             // 4-4 如果线程被中断，则转移线程到同步队列，并结束循环
>             if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
>                 break;
>             // 每次循环都会计算新的nanosTimeout值，然后在下次循环的时候设置阻塞的时限
>             nanosTimeout = deadline - System.nanoTime();
>         }
>         // 5-自旋等待获取到同步状态（即获取到lock）
>         if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
>             interruptMode = REINTERRUPT;
>         if (node.nextWaiter != null)
>             unlinkCancelledWaiters();
>         //  6-处理被中断的情况
>         if (interruptMode != 0)
>             reportInterruptAfterWait(interruptMode);
>         return deadline - System.nanoTime();
>     }
> }
> ```
> 方法解析：
> 
> 这个方法的流程与上面的await基本一致，只是在第4步中添加了关于超时判断的逻辑，这里就着重看一下这一部分，其余部分不再赘述。
> 
> 包括两个部分的内容，第一是开始的校验，如果设置的超时时间小于等于0，表示线程等待立即超时，然后立即转移到同步队列尾部，尝试获取锁；第二是如果设置的超时时间大于等于spinForTimeoutThreshold的值，则将当前线程阻塞指定的时间，这个时间会随着循环的次数不断的减小。

另外的两个等待方法awaitUntil(Date deadline)和await(long time, TimeUnit unit)就不再赘述了，原理完全一致，有一个不同的是awaitUninterruptibly()方法：
> **awaitUninterruptibly()**：
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     public final void awaitUninterruptibly() {
>         // 1-将当前线程封装成Node节点并添加到等待队列尾部
>         Node node = addConditionWaiter();
>         // 2-释放当前线程所占用的lock，在释放的过程中会唤醒同步队列中的下一个节点
>         int savedState = fullyRelease(node);
>         boolean interrupted = false;
>         // 3-阻塞当前线程，直到被唤醒
>         while (!isOnSyncQueue(node)) {
>             LockSupport.park(this);// 阻塞当前线程
>             if (Thread.interrupted())
>                 interrupted = true;
>         }
>         // 4-自旋尝试获取同步锁
>         if (acquireQueued(node, savedState) || interrupted)
>             selfInterrupt();
>     }
> }
> ```
> 其实就是不响应中断的等待方法，从源码中可以看出，虽然不响应中断，但是仍然保存着中断标志。

下面就来看看唤醒的方法：
> **signal()**：
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     public final void signal() {
>         // 1-校验当前线程时候独享式持有共享锁，如果不持有则抛出异常
>         if (!isHeldExclusively())
>             throw new IllegalMonitorStateException();
>         Node first = firstWaiter;// 保存等待队列首节点
>         // 2-如果队列不为空，则执行头节点唤醒操作
>         if (first != null)
>             doSignal(first);
>     }
>     private void doSignal(Node first) {
>         do {
>             // 3-如果等待队列只有一个节点，则将lastWaiter更新为null
>             if ( (firstWaiter = first.nextWaiter) == null)
>                 lastWaiter = null;
>             first.nextWaiter = null;
>             // 4-尝试将线程节点从等待队列转移到同步队列，如果成功则结束循环，如果失败则再次判断firstWaiter首节点是否为null，如果不是null，则再次循环，否则结束循环
>         } while (!transferForSignal(first) &&
>                  (first = firstWaiter) != null);
>     }
> }
> ```
> 方法解析：
> - 第一步：校验当前线程时候独享式持有共享锁，如果不持有则抛出异常
> - 第二步：如果队列不为空，则执行头节点唤醒操作
> - 第三步：如果等待队列只有一个节点（头节点），则将lastWaiter更新为null
> - 第四步：尝试将线程节点从等待队列转移到同步队列，如果成功则结束循环，如果失败则再次判断firstWaiter首节点是否为null，如果不是null，则再次循环，否则结束循环

> **signalAll()**：
> ```java
> public class ConditionObject implements Condition, java.io.Serializable {
>     public final void signalAll() {
>         // 1-校验当前线程时候独享式持有共享锁，如果不持有则抛出异常
>         if (!isHeldExclusively())
>             throw new IllegalMonitorStateException();
>         Node first = firstWaiter;
>         // 2-如果队列不为空，则执行节点唤醒操作
>         if (first != null)
>             doSignalAll(first);
>     }
>     private void doSignalAll(Node first) {
>         lastWaiter = firstWaiter = null;// 要唤醒所有线程节点，那么等待队列就是被清空，那么就将这两个指针置为null
>         // 3-针对等待队列中的节点一个一个进行唤醒操作
>         do {
>             Node next = first.nextWaiter;// 保存二节点
>             first.nextWaiter = null;
>             transferForSignal(first);// 将首节点转移到同步队列
>             first = next;// 重置首节点，将二节点作为新的首节点
>         } while (first != null);
>     }
> }
> ```
### 2.3 静态内容解析
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private static final Unsafe unsafe = Unsafe.getUnsafe();// 注入Unsafe实例
    private static final long stateOffset;// 同步状态偏移量
    private static final long headOffset;// 等待队列的头结点偏移量
    private static final long tailOffset;// 等待队列的尾节点偏移量
    private static final long waitStatusOffset;// 节点等待状态偏移量
    private static final long nextOffset;// 节点的下级节点偏移量
    static {
        try {
            // 获取这五个字段的内存偏移量并保存到各自的字段中
            stateOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
            headOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
            tailOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
            waitStatusOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("waitStatus"));
            nextOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("next"));
        } catch (Exception ex) { throw new Error(ex); }
    }
}
```
从这一部分内容可以看出来AQS底层和ConcurrentHashMap一样是使用CAS来实现原子操作的。

这一部分就是引入Unsafe来实现原子以上几个字段的原子更新。知道即可。
### 2.4 字段解析
AQS中字段不多，如下所示：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private transient volatile Node head;// 等待队列的头结点
    private transient volatile Node tail;// 等待队列的尾节点
    private volatile int state;// 同步状态，初始为0，获取锁时会加1，释放锁时减1，当重入锁时也会加1
    static final long spinForTimeoutThreshold = 1000L;// 自旋时限1000纳秒
}
```
这里的head和tail分别指向的是同步器的同步队列的头结点与尾节点。这个同步队列采用双向链表实现，其节点就是之前介绍的内部类中的Node类型。

state表示同步状态，初始为0，表示未被持有，当其被某线程持有时，就会增加1，而且这个也是实现重入的基础，当该线程再次获取当前锁时，只需要state加1即可，每释放一个锁，state-1，直到state等于0时，该同步锁为完全释放。

spinForTimeoutThreshold是一个内置的快速自旋时限，当设置的超时时间小于这个值的时候，无需再执行等待设置，直接进入快速自旋即可，原因在于 spinForTimeoutThreshold 已经非常小了，非常短的时间等待无法做到十分精确，如果这时再次进行超时等待，相反会让nanosTimeout 的超时从整体上面表现得不是那么精确，所以在超时非常短的场景中，AQS会进行无条件的快速自旋。
### 2.5 方法解析
AQS中的方法可以粗分为四类：获取同步状态方法、释放同步状态方法、队列检验方法、队列监控方法，我们罗列一个表格来简单介绍下这些方法：

|分类|序号|方法|说明|备注|
|:---|:---:|:---|:---|:---|
|获取同步状态方法|1|final void acquire(int arg)|独享获取同步状态，不响应中断||
|获取同步状态方法|2|final void acquireInterruptibly(int arg)|独享获取同步状态，响应中断||
|获取同步状态方法|3|final boolean tryAcquireNanos(int arg, long nanosTimeout)|独享获取同步状态，响应中断，响应超时||
|获取同步状态方法|4|final void acquireShared(int arg)|共享获取同步状态，不响应中断||
|获取同步状态方法|5|final void acquireSharedInterruptibly(int arg)|共享获取同步状态，响应中断||
|获取同步状态方法|6|final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)|共享获取同步状态，响应中断，响应超时||
|释放同步状态方法|7|final boolean release(int arg)|独享释放同步状态||
|释放同步状态方法|8|final void acquireShared(int arg)|共享释放同步状态||
|队列检验方法|9|final boolean hasQueuedThreads()|校验同步队列中是否有线程在等待获取同步状态||
|队列检验方法|10|final boolean hasContended()|校验是否有线程争用过此同步器（同步队列是否为空）||
|队列检验方法|11|final boolean isQueued(Thread thread)|校验给定线程是否在同步队列之上||
|队列检验方法|12|final boolean hasQueuedPredecessors()|校验是否有线程等待获取同步状态比当前线程时间长（同步队列中是都有前节点）||
|队列检验方法|13|final boolean owns(ConditionObject condition)|校验给定的condition是否是使用当前同步器作为锁||
|队列检验方法|14|final boolean hasWaiters(ConditionObject condition)|校验等待队列是否有等待线程||
|队列监控方法|15|final int getWaitQueueLength(ConditionObject condition)|获取等待队列中线程数量||
|队列监控方法|16|final Collection<Thread> getWaitingThreads(ConditionObject condition)|获取等待队列中等待线程的集合||
|队列监控方法|17|final Thread getFirstQueuedThread()|获取同步队列中的头节点线程||
|队列监控方法|18|final int getQueueLength()|获取同步队列中线程数量||
|队列监控方法|19|final Collection<Thread> getQueuedThreads()|获取同步队列中线程的集合||
|队列监控方法|20|final Collection<Thread> getExclusiveQueuedThreads()|获取同步队列中欲独享获取同步状态的线程集合||
|队列监控方法|21|final Collection<Thread> getSharedQueuedThreads()|获取同步队列中欲共享获取同步状态的线程集合||

这些方法中重点就是获取同步状态方法和释放同步状态方法，下面我们也重点就看下这些个方法的实现：
#### acquire(int)
该方法表示独享式获取同步状态，但不响应中断，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
}
```
该方法中调用了四个方法来完成功能，依次为：
- tryAcquire(int)：一个模板方法，授权子类来实现，主要用于尝试独享式获取同步状态。
- addWaiter(Node)：将当前线程封装成Node节点，添加到同步队列尾部
- acquireQueued(Node,int)：自旋获取锁，获取成功后返回等待过程中是否被中断过
- selfInterrupt()：进行中断处理

> 解析：首先尝试独享式获取同步状态，如果获取到了就结束，
如果未获取到则将线程封装成为Node节点并添加到同步队列尾部，然后自旋以获取同步状态，
一旦获取到同步状态，退出自旋，并返回当前线程在自旋期间是否被中断过，如果被中断过则再次自我中断，
为什么需要再次自我中断呢，这只是为了保留中断现场，因为在自旋结束进行中断校验时使用的是Thread.interrupted()，
该方法会导致中断状态被清除。

tryAcquire方法是一个模板方法，需要在AQS的子类中实现，默认的实现只是抛出了一个异常
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
}
```
addWaiter方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private Node addWaiter(Node mode) {
        // 将当前线程与同步状态获取模式封装成为Node节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        // 尝试快速进行一次enq操作，将新节点设置为同步地列尾节点，
        // 如果成功会结束方法但如果不成功，可以由下面的enq方法来执行，
        // 这个enq方法可以通过无限循环的方法直到执行成功
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 将新节点添加到同步队列中
        enq(node);
        return node;
    }
    // 将新节点添加到同步队列中
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                // 这一步主要是针对同步队列未初始化时进行的初始化操作，初始化完成后下次循环就会执行新节点的添加操作
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 将之前的尾节点设置为新节点的前节点，然后原子更新尾节点为新节点
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
}
```
> 解析：很明显上面的addWaiter方法中出现了添加新节点到同步队列的逻辑，而在之后的enq方法中再次出现，
主要目的就是为了能在执行enq方法之前可以先进行一次尝试，看能否一次执行成功，若成功，则皆大欢喜，
不必走下面的逻辑，若不成功，再走enq方法，来通过无限循环的方式强制执行成功。所以前面的逻辑可以看成是一次简单的enq操作。

acquireQueued方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;// 默认失败
        try {
            boolean interrupted = false;// 中断标记
            for (;;) {// 无限循环以自旋
                final Node p = node.predecessor();// 获取前置节点
                // 如果前节点是头节点，并且当前线程获取同步状态成功，则将当前节点置为头节点
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC，这里去除以前的节点对当前节点的引用，当前节点对象不再被使用后可以被GC清理
                    failed = false;// 表示成功
                    return interrupted;
                }
                // 如果前置节点不是头节点，或者当前节点线程未获取到同步状态，则将尝试将前置节点状态更新为SIGNAL，并阻塞当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```
> 解析：以无限循环的方法自旋，每次循环都尝试独享式获取同步状态，如果获取到了同步状态，
那么将当前节点置为头节点；如果前置节点不是头节点或者未获取到同步状态则尝试将前置节点的状态更新为SIGNAL，并阻塞当前线程（park），
这种情况下，当前线程需要被唤醒才能继续执行，当被唤醒之后可以再次循环，尝试获取同步状态，如果不成功，将会再次阻塞，等待再次被唤醒。

AbstractQueuedSynchronizer方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;// 获取前置节点的状态
        if (ws == Node.SIGNAL)
            // 表示后置线程节点（当前节点需要被唤醒）
            return true;
        if (ws > 0) {
            // 表示前置节点线程被取消，那么清理被取消的线程节点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            // 尝试将前置节点的状态置为SIGNAL，只有置为SIGNAL之后才能返回true.
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
}
```
> 解析：这个方法主要目的就是为了将前置节点状态置为SIGNAL，这个状态意思是它后面的那个节点被阻塞了，
需要被唤醒，可见这个状态就是一个标记，标记着后面节点需要被唤醒。

parkAndCheckInterrupt方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);// 阻塞当前线程
        return Thread.interrupted();
    }
}
```
> 解析：一旦线程执行到这一步，那么当前线程就会阻塞，后面的return暂时就不会执行。只有在被唤醒之后才能接着返回中断校验的结果。

> 总结：acquire方法首先尝试独享式获取同步状态（tryAcquire），获取失败的情况下需要将当前线程封装成为一个Node节点，
然后首先尝试将其设置为同步队列的为节点，如果失败，则自旋直到成功为止，然后进行自旋判断当前节点是否第二节点，如果是，
则尝试获取同步状态，如果成功，将当前节点置为头节点；否则如果当前节点不是第二节点，或者获取同步状态失败，
则将前置节点状态置为SIGNAL，然后阻塞(park)当前线程，等待被唤醒，唤醒之后会重复自旋，判断节点是否第二节点和尝试获取同步状态，
如果还不成功，那么就再次阻塞...
#### acquireInterruptibly(int)
该方法表示独享式获取同步状态，响应中断，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        // 中断校验，会清除中断状态
        if (Thread.interrupted())
            throw new InterruptedException();
        // 尝试独享式获取同步状态，如果失败则尝试中断的获取。
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
    // 中断的获取同步状态
    private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        // 首先将当前线程封装成为Node节点，并保存到同步队列尾部
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            // 自旋，逻辑桶acquire
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```
> 解析：一开始就进行中断校验，如果未被中断则尝试独享式获取同步状态，获取失败后则封装线程为Node节点并保存到同步队列，然后自旋，逻辑与acquire种的acquireQueued方法逻辑一致，不再赘述。
#### tryAcquireNanos(int, long)
该方法表示独享式获取同步状态，响应中断，响应超时，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final boolean tryAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        // 首先响应中断，进行中断校验，若被中断，抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        return tryAcquire(arg) ||
            doAcquireNanos(arg, nanosTimeout);// 超时获取
    }
    // 超时获取
    private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        // 如果超时时间小于等于0，则直接超时，返回false
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;// 计算截止时间点
        final Node node = addWaiter(Node.EXCLUSIVE);// 封装线程节点，并添加到同步队列
        boolean failed = true;
        try {
            for (;;) {// 自旋
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                nanosTimeout = deadline - System.nanoTime();// 计算剩余超时时间
                // 如果剩余超时时间小于等于0，这说明超时，返回false
                if (nanosTimeout <= 0L)
                    return false;
                if (shouldParkAfterFailedAcquire(p, node) &&// 将前置节点状态置为SIGNAL
                    nanosTimeout > spinForTimeoutThreshold)// 剩余超时时间大于快速自旋时限（1000纳秒）
                    LockSupport.parkNanos(this, nanosTimeout);// 限时阻塞当前线程，超时时间为剩余超时时间
                // 再次响应中断，进行中断校验，若被中断直接抛出异常
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```
> spinForTimeoutThreshold：这是系统内置的一个常量，设置为1000纳秒，这是一个很短的时间，如果要阻塞的剩余时间小于这个值，就没有必要再执行阻塞，直接进入快速自旋过程。

> 解析：整体逻辑基本与前面的两种类似，不同之处在于增加了针对超时时间处理的逻辑。
>
> 与acquireInterruptibly类似，一开始就进行中断校验，若被中断则抛出异常，否则尝试独享式获取同步状态，
> 获取成功，则返回true，如果获取失败，则将线程封装成Node节点保存到同步队列，然后计算截止时间点（当前时间+超时时间）,
> 然后开始自旋，自旋的逻辑中前半部分与之前相同，只有在前置节点不是头节点或者获取同步状态失败的情况下逻辑发生了改变，
> 先计算剩余超时时间nanosTimeout（截止时间点-当前时间）,然后将前置节点的状态置为SIGNAL，判断剩余超时时间是否大于
> spinForTimeoutThreshold，如果大于则限时阻塞当前线程，否则快速自旋即可。
#### acquireShared(int)
该方法表示共享式获取同步状态，不响应中断，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
}
```
> 解析：首先尝试共享式获取同步状态，如果获取失败（返回负值），则执行doAcquireShared方法。

tryAcquireShared方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
}
```
> 该方法是一个模板方法，需要子类来完善逻辑。但大致意义如下，如果获取失败返回负数（-1），如果是该同步状态被首次共享获取成功，返回0，非首次获取成功，则返回正数（1）

doAcquireShared方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private void doAcquireShared(int arg) {
        // 将线程封装成功节点，保存到同步队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {// 自旋
                final Node p = node.predecessor();// 获取前置节点
                if (p == head) {
                    // 如果前置节点为头节点
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        // 如果成功获取到同步状态，则将当前节点置为头节点，并进行传播唤醒
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                // 如果前置节点非头节点或者获取同步状态失败，则将前置节点设置为SIGNAL，然后阻塞当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // 预存原始头节点
        setHead(node);// 将当前节点置为头节点
        // propagate可为0或1，0表示同步状态被首次获取，1表示被多次获取
        // h为原始头节点
        // head为新头节点
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;// 获取下级节点s
            // 如果后继节点不存在或者后继节点是共享式的，则唤醒后继节点
            if (s == null || s.isShared())
                doReleaseShared();// 唤醒后继节点
        }
    }
}
```
> 解析：该方法的逻辑相对于acquireQueued只是稍有变动，大致意思是相同的。不同之处在于此处涉及到一个传播（Propagate）。
> 所谓的传播，其实是在当前节点共享式获取到同步状态之后，检查其后置节点是否也是在等待共享式获取同步状态，若是，则将唤醒其后置节点。

doReleaseShared源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    private void doReleaseShared() {
        for (;;) {// 自旋
            Node h = head;// 获取头节点
            if (h != null && h != tail) {// 如果队列中存在多个节点的话
                int ws = h.waitStatus;// 头节点状态ws
                // 如果头节点状态为SIGNAL，则将其
                if (ws == Node.SIGNAL) {// 说明其后继节点线程被阻塞，需要唤醒
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))// 首先将头节点状态重置为0
                        continue;// 如果重置头节点状态操作失败则重试
                    unparkSuccessor(h);// 然后进行后继节点唤醒
                }
                // 如果头节点状态为0，则将其状态更新为PROPAGATE
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;// 头节点更新操作失败则重试
            }
            if (h == head)
                break;// 头节点发生变化则退出自旋
        }
    }
    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;// 获取后继节点s
        if (s == null || s.waitStatus > 0) {
            // 如果s为null或者其状态为取消，则从后遍历队列节点，找到node节点之后的首个未被取消的节点t，赋给s
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);// 执行s节点线程的唤醒操作
    }
}
```
> 解析：doReleaseShared方法被两处调用，一为此处，另一为releaseShared方法，这个是用来共享式释放同步状态的方法。
> doReleaseShared方法的作用就是为了唤醒后继节点，主要逻辑如下：首先获取头节点的状态ws，如果ws是SIGNAL，
> 表示后继节点需要被唤醒，然后自旋将头节点状态更新为0，并执行后继节点唤醒操作，这里要确保唤醒的是头节点之后首个
> 未被取消的线程节点，唤醒之后，后继节点的线程开始继续执行，当前线程也继续执行；如果ws是0，则将头节点的状态更新为PROPAGATE，
> 来确保同步状态可以顺利传播（因为如果ws为SIGNAL，会自动唤醒下一个节点，而0则不会，所有将其更新为PROPAGATE，表示共享式获取的传播）
> 被唤醒的线程会重置头节点，一旦重置，当前线程在最后校验头节点那一步就会成功，然后执行break退出自旋。
>
> 一般来说这里唤醒的主要目的是为了唤醒一个共享式获取同步状态的线程节点，它会直接获取到同步状态；但也存在特殊情况，比如
> 这个节点线程被取消了，导致唤醒了一个独享式获取的线程节点，那么在这个线程被唤醒后尝试独享式获取同步状态的时候会获取不到
> （因为同步状态被共享式获取的线程持久，而且可能是多个）从而再次进入阻塞。
> 
> 其实唤醒的主要来源还是靠同步状态释放操作来发起的。
#### acquireSharedInterruptibly(int)
该方法表示共享式获取同步状态，响应中断，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        // 首先响应中断
        if (Thread.interrupted())
            throw new InterruptedException();
        // 尝试共享式获取同步状态，失败则执行doAcquireSharedInterruptibly方法
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
    // 可中断的共享式获取同步状态
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        // 首先封装线程节点，保存到同步队列尾部
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {// 自旋
                final Node p = node.predecessor();// 获取前置节点
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    // 如果发生了中断则抛出异常
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
}
```
> 解析：这个方法与acquireShared几乎一致，仅仅是在处理中断的问题上有点区别，所以不再赘述。
#### tryAcquireSharedNanos(int, long)
该方法表示共享式获取同步状态，响应中断，响应超时，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        return tryAcquireShared(arg) >= 0 ||
            doAcquireSharedNanos(arg, nanosTimeout);
    }
    private boolean doAcquireSharedNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        // 如果超时时间小于等于0,则直接超时，返回false
        if (nanosTimeout <= 0L)
            return false;
        // 计算超时截止时间点（当前时间+超时时间）
        final long deadline = System.nanoTime() + nanosTimeout;
        // 封装节点并保存队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {// 自旋
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return true;
                    }
                }
                // 计算剩余的超时时间
                nanosTimeout = deadline - System.nanoTime();
                // 如果剩余超时时间小于等于0，直接超时，返回false
                if (nanosTimeout <= 0L)
                    return false;
                // 将前置节点置为SIGNAL，然后校验剩余超时时间，如果不足spinForTimeoutThreshold，则进入快速自旋，否则执行阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                // 再次响应中断
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    }
```
> 解析：基本雷同，可以参考共享式获取同步状态的方法和独享式响应中断超时的获取方法。
#### release(int)
该方法表示独享式释放同步状态，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final boolean release(int arg) {
        // 首先尝试独享式释放同步状态
        if (tryRelease(arg)) {
            Node h = head;// 头节点
            // 头节点存在且状态不为0，则唤醒其后继节点
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        // 释放失败返回false
        return false;
    }
}
```
> 解析：首先调用tryRelease来尝试独享式释放同步状态，如果成功，则根据头节点的状态来决定是否唤醒后继节点，
> 头节点为0则不唤醒。唤醒操作通过调用unparkSuccessor方法来实现，具体逻辑之前已有描述，这里总结一下：
> 其实就是唤醒头节点之后的首个未被取消的节点线程，这个线程可能是独享式的也可能是共享式的。

tryRelease源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
}
```
> 解析：tryRelease方法是一个模板方法，同样需要子类来实现。
#### releaseShared(int)
该方法表示共享式释放同步状态，源码如下：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    public final boolean releaseShared(int arg) {
        // 尝试共享式释放同步状态，成功后唤醒后继节点
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
}
```
> 解析：很简单，其中的doReleaseShared方法我们也了解了。

tryReleaseShared源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }
}
```
> 解析：和前面的那几个模板方法一样，需要子类来实现。

剩下的方法都是一些校验和监控的方法，并不涉及重点逻辑，不再赘述，下面做一个总结

## 三、总结
> 总结：
>
> 1. AQS同步器内部维护了一个底层为双向链表的同步队列，用于保存那些获取同步状态失败的线程。每个AQS同步器还可以关联多个Condition，其中每个Condition内部维护了一个底层为单向链表的等待队列，用于保存那些基于特定条件而陷入等待的线程。
> 2. 内部类Node描述的是同步队列和等待队列中节点的类型。节点有两点需要注意，那就是节点的模式与状态
>     - 节点模式：
>         - EXCLUSIVE：独享式
>         - SHARED：共享式
>     - 节点状态：
>         - 0：初始状态，该状态下不会唤醒后继节点
>         - CANCELLED（1）：取消状态，节点线程被中断或超时
>         - SIGNAL（-1）：唤醒状态，表示该节点的后继节点被阻塞，需要唤醒
>         - CONDITION（-2）：表示当前节点位于等待队列中，在等待条件满足
>         - PROPAGATE（-3）：表示共享式获取同步状态的传播
> 3. 内部类ConditionObject是Condition的实现类，作为附着在同步器上的一个功能，可用可不用；它提供了一些方法来执行等待和唤醒操作：
>     - 等待操作：
>         - await()：响应中断
>         - awaitNanos(long)：响应中断，响应超时
>         - awaitUninterruptibly()：不响应中断，不响应超时
>     - 唤醒操作：
>         - signal()
>         - signalAll()
> 4. AQS同步器提供了多个方法从来辅助实现同步状态的获取与释放：
>     - 独享式获取：
>         - acquire(int)：不响应中断，不响应超时
>         - acquireInterruptibly(int)：响应中断
>         - tryAcquireNanos(int, long)：响应中断，响应超时
>     - 独享式释放：
>         - release(int)
>     - 共享式获取：
>         - acquireShared(int)：不响应中断，不响应超时
>         - acquireSharedInterruptibly(int)：响应中断
>         - tryAcquireSharedNanos(int, long)：响应中断，响应超时
>     - 共享式释放：
>         - releaseShared(int)

参考：

- [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
- [深入理解AbstractQueuedSynchronizer(AQS)](https://www.jianshu.com/p/cc308d82cc71)
- [深入理解AbstractQueuedSynchronizer（三）](https://www.jianshu.com/p/52b07c88605e)
- [详解Condition的await和signal等待/通知机制](https://www.jianshu.com/p/28387056eeb4)
- [【死磕Java并发】-----J.U.C之AQS：同步状态的获取与释放](https://www.jianshu.com/p/3b5ea60c5ad9)
- [【JUC】JDK1.8源码分析之AbstractQueuedSynchronizer（二）](https://www.cnblogs.com/leesf456/p/5350186.html)
- [再谈AbstractQueuedSynchronizer2：共享模式与基于Condition的等待/通知机制实现](https://www.cnblogs.com/xrq730/p/7067904.html)
