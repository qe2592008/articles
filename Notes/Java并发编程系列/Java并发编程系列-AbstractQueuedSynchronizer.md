# Java并发编程系列-AbstractQueuedSynchronizer

## 一、概述
AbstractQueuedSynchronizer简称为AQS，是并发包中用于实现并发工具的基础类，非常明显，它是一个抽象类。

它提供了一个依赖于FIFO等待队列的框架用于实现各种阻塞锁与同步器。

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
    static final int CANCELLED =  1;// 表示当前节点封装的线程被取消
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

> await()：
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

> awaitNanos(long nanosTimeout):
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
> awaitUninterruptibly()：
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
> signal()：
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

> signalAll()：
```java
public class ConditionObject implements Condition, java.io.Serializable {
    public final void signalAll() {
        // 1-校验当前线程时候独享式持有共享锁，如果不持有则抛出异常
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        // 2-如果队列不为空，则执行节点唤醒操作
        if (first != null)
            doSignalAll(first);
    }
    private void doSignalAll(Node first) {
        lastWaiter = firstWaiter = null;// 要唤醒所有线程节点，那么等待队列就是被清空，那么就将这两个指针置为null
        // 3-针对等待队列中的节点一个一个进行唤醒操作
        do {
            Node next = first.nextWaiter;// 保存二节点
            first.nextWaiter = null;
            transferForSignal(first);// 将首节点转移到同步队列
            first = next;// 重置首节点，将二节点作为新的首节点
        } while (first != null);
    }
}
```
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
很明显上面的addWaiter方法中出现了添加新节点到同步队列的逻辑，而在之后的enq方法中再次出现，主要目的就是为了能在执行enq方法之前可以先进行一次尝试，看能否一次执行成功，若成功，则皆大欢喜，不必走下面的逻辑，若不成功，再走enq方法，来通过无限循环的方式强制执行成功。所以前面的逻辑可以看成是一次简单的enq操作。

acquireQueued方法源码：
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;// 中断标记
            for (;;) {
                final Node p = node.predecessor();// 获取前置节点
                // 如果前节点是头节点，并且当前线程获取同步状态成功，则将
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
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





参考：
- [【死磕Java并发】-----J.U.C之AQS：同步状态的获取与释放](https://www.jianshu.com/p/3b5ea60c5ad9)