# Java并发编程系列-CountDownLatch
## 一、概念
## 二、原理
CountDownLatch将同步器作为一个外置的锁，而这个锁自动被限定数量的线程所持有，每个执行countDown的线程都在释放这个锁，释放之后同步器的state减1，当state变成0之前，所有执行了await的线程将全部处于等待状态（挂起状态），一旦state变成0，那么所有await的线程将全部开始执行（）

CountDownLatch就是借助了AQS功能实现的一个同步器，是和Lock同一级别的，
## 三、实例
## 四、源码
### 内部类Sync
CountDownLatch底层正是依赖于concurrent包中的公共同步器AQS来实现的，这一点我们可以从CountDownLatch的内部类Sync中看出来。
```java
public class CountDownLatch {
    // 同步器，作为CountDownLatch实现的基石
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;
        // 构造器，被外部CountDownLatch的构造器调用，来初始化计数值state
        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }
        // 尝试共享式获取同步状态
        protected int tryAcquireShared(int acquires) {
            // 当计数值state=0，返回1，表示获取到同步状态，其实是表示执行countDown的线程已全部执行
            // 当计数值state!=0，返回-1，表示未获取到同步状态，其实表示执行countDown的线程尚未全部执行
            return (getState() == 0) ? 1 : -1;
        }
        // 尝试共享式释放同步状态
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))// CAS更新计数值，每次减1
                    return nextc == 0;
            }
        }
    }
}
```
### 构造器
首先来看看构造器：
```java
public class CountDownLatch {
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
}
```
这个是CountDownLatch唯一的一个构造器，主要用于我们手动创建一个CountDownLatch实例，创建的时候必须要指定初始计数器值count。

这个构造器底层调用了内部类Sync的构造器，而Sync的构造器又调用了AQS的构造器来构建底层同步器基件，而这个count值就赋给了AQS中的state。

然后我们来看看CountDownLatch提供的常用的公共方法：
### await方法
```java
public class CountDownLatch {
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
}
```
这个方法的作用是挂起当前线程，等待其他线程执行countDown来削减计数值，直到计数值变成0为止，然后当前线程
## 五、总结