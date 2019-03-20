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

简单来看，那就是将ThreadLocal提供了一种将

## 二、使用

## 三、原理

## 四、问题

## 五、总结