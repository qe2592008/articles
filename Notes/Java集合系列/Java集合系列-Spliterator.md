# Java集合系列-Spliterator
## 一、概述
Spliterator是分割迭代器，是JDK1.8的新功能。

几乎每个集合都实现了它，它是实现并行遍历的基础。

使用这个迭代器，可以实现多线程遍历集合的功能，其中每个线程遍历集合中的一段，因此没有线程安全问题。
## 二、Spliterator
源码解析：
```java
public interface Spliterator<T> {
    // 针对单个元素执行某个行为
    // 接收一个行为（Lambda表达式或方法引用）
    boolean tryAdvance(Consumer<? super T> action);
    // 针对所有元素执行某个行为，通过tryAdvance实现
    // 接收一个行为（Lambda表达式或方法引用）
    default void forEachRemaining(Consumer<? super T> action) {
        do { } while (tryAdvance(action));
    }
    // 分割集合，返回一个新的Spliterator迭代器，
    // 一旦元素被返回的迭代器携带，则从当前迭代器消失
    // 一般分割是截取一半
    Spliterator<T> trySplit();
    // 获取剩余遍历元素的数量的估计值
    long estimateSize();
    // 当前迭代器拥有SIZED特征值时，返回剩余元素个数，否则返回-1
    default long getExactSizeIfKnown() {
        return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
    }
    // 返回当前迭代器的特征值
    int characteristics();
    // 校验当前迭代器是否拥有指定的特征值
    default boolean hasCharacteristics(int characteristics) {
        return (characteristics() & characteristics) == characteristics;
    }
    // 返回比较器
    default Comparator<? super T> getComparator() {
        throw new IllegalStateException();
    }
}
```

