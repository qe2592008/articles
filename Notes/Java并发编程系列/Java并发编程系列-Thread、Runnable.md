# Java并发编程系列-Thread、Runnable
## 一、概述
Runnable是一个函数式接口，表示一个任务，其中只有一个run方法，用于子类实现具体的线程任务。

Runnable的run方法定义的线程任务并不是被直接调用来启动的，而是需要用Thread的start来间接调用启动。

Thread是一个继承自Runnable的类，代表线程。一个Thread实例就是一个线程。

每个线程都有一个优先级，高优先级的线程将被优先执行，被CPU分配较多的时间片来执行。

在一个线程中创建一个线程，新建的线程默认继承父级线程的优先级和守护线程。
## 二、Runnable
首先来看Runnable接口。
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
简单明了，可以使用Lambda表达式来定义线程任务。
```java
public class RunnableTest {
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("1");
        }
    });
    Thread t2 = new Thread(()-> System.out.println("2"));
}
```
## 三、Thread
Thread类是Runnable的实现类。
```java
public class Thread implements Runnable {}
```
### 3.1 内部类解析
#### 3.1.1 State
```java
public enum State {
    // 创建好但未启动的线程状态
    NEW,
    // 可运行状态，启动线程之后进入该状态
    // 有两个子状态：
    // READY，准备好状态，
    // RUNNING，运行状态，
    RUNNABLE,
    // 阻塞状态，表示线程等待获取一个监视器锁
    BLOCKED,
    // 等待状态，可由以下方法导致:
    // Object.wait();
    // thread.join();
    // LockSupport.park()
    WAITING,
    // 限时等待状态，可由以下方法导致:
    // sleep(long);
    // Object.wait(long);
    // .join(long);
    // LockSupport.parkNanos(long);
    // .parkUntil(long);
    TIMED_WAITING,
    // 线程执行完毕
    TERMINATED;
}
```
State定义了线程的六种状态：
- NEW
- RUNNABLE
- BLOCKED，只与获取锁有关，等待获取锁而陷入阻塞
- WAITING
- TIMED_WAITING
- TERMINATED
#### 3.2 字段解析
```java
public class Thread implements Runnable {
    // 线程的名称
    private volatile String name;
    // 线程优先级
    private int            priority;
    // 
    private Thread         threadQ;
    // 
    private long           eetop;

    // 
    private boolean     single_step;

    // 是否守护线程，默认false
    private boolean     daemon = false;

    // JVM状态
    private boolean     stillborn = false;

    /* What will be run. */
    private Runnable target;

    /* The group of this thread */
    private ThreadGroup group;

    /* The context ClassLoader for this thread */
    private ClassLoader contextClassLoader;

    /* The inherited AccessControlContext of this thread */
    private AccessControlContext inheritedAccessControlContext;

    /* For autonumbering anonymous threads. */
    private static int threadInitNumber;
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    /*
     * The requested stack size for this thread, or 0 if the creator did
     * not specify a stack size.  It is up to the VM to do whatever it
     * likes with this number; some VMs will ignore it.
     */
    private long stackSize;

    /*
     * JVM-private state that persists after native thread termination.
     */
    private long nativeParkEventPointer;

    /*
     * Thread ID
     */
    private long tid;

    /* For generating thread ID */
    private static long threadSeqNumber;

    /* Java thread status for tools,
     * initialized to indicate thread 'not yet started'
     */

    private volatile int threadStatus = 0;
    /**
     * The argument supplied to the current call to
     * java.util.concurrent.locks.LockSupport.park.
     * Set by (private) java.util.concurrent.locks.LockSupport.setBlocker
     * Accessed using java.util.concurrent.locks.LockSupport.getBlocker
     */
    volatile Object parkBlocker;

    /* The object in which this thread is blocked in an interruptible I/O
     * operation, if any.  The blocker's interrupt method should be invoked
     * after setting this thread's interrupt status.
     */
    private volatile Interruptible blocker;
    private final Object blockerLock = new Object();
    /**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;

   /**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;    
}
```