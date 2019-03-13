# Java并发编程系列-Thread、Runnable
## 一、概述
Runnable是一个函数式接口，表示一个任务，其中只有一个run方法，用于子类实现具体的线程任务。

Runnable的run方法定义的线程任务并不是被直接调用来启动的，而是需要用Thread的start来间接调用启动。

Thread是一个继承自Runnable的类，代表线程。

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
Thread类是