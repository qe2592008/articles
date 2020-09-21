# Java并发编程系列-ThreadLocal
## 一、概述
ThreadLocal表述的是一种线程本地变量的概念。

我们可以看看Thread的源码，里面就有一个ThreadLocal变量：
```java
public class Thread implements Runnable {
    //...
    ThreadLocal.ThreadLocalMap threadLocals = null;
    //...
}
```
它的操作需要靠ThreadLocal中的get和set方法来实现。

线程中

## 二、使用
1. ThreadLocal线程变量是属于线程的，存活于线程的执行周期中，那么，我们可以使用它来保存一些需要在整个线程运行过程中使用的内容，也就是所谓线程级的内容，例如，一些记录线程执行阶段的数据，在线程执行各个阶段（多为业务阶段），存取相关的数据。
2. 一个线程中可以同时使用多个的ThreadLocal变量。
3. 一般来说在一些不便于通过传参的方式来传递变量的地方，可以酌情选择本地变量的方式来传递参数，只是一定要注意变量内容的释放，避免
## 三、原理

## 四、问题

## 五、总结