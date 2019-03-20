# Java集合系列-LinkedHashMap

## 一、概述
HashMap是无序的，那么是否存在有序的Map呢？

当然是存在的，而且常见的就有两个：TreeMap和LinkedHashMap。

TreeMap是基于红黑树实现了Map。它的排序方式是自然顺序（Conparable提供），或者是指定的排序规则（Comparator提供）。

LinkedHashMap是基于链表和HashMap实现的有序Map。他可以有两种排序方式，由参数accessOrder指定，true代表访问顺序，false代表插入顺序。

访问顺序就是按照Map中元素被访问（获取get）的顺序排序。

插入顺序就是按照元素插入（存储put）Map的顺序来排序。

因为HashMap属于非线程安全的集合，所以LinkedHashMap同样是非线程安全的集合。
## 二、结构解析
### 2.1 继承体系
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V> {}
```
这里重点要说的就是HashMap，LinkedHashMap继承自HashMap，那么它先天就拥有HashMap的所有功能。

或者说LinkedHashMap是在HashMap的基础上扩展出来的。
### 2.2 重点内部类
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V> {
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
}
```
这个Entry内部类用于封装LinkedHashMap中的节点，他拥有before和after两个引用，分别指向当前节点的前节点与后节点，形成双向链表结构。
### 2.3 重点字段
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V> {
    transient LinkedHashMap.Entry<K,V> head;
    transient LinkedHashMap.Entry<K,V> tail;
    final boolean accessOrder;
}
```