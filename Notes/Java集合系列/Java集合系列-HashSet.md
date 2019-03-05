# Java集合系列-HashSet
## 一、概述
HashSet是基于哈希实现的set集合，其实它底层是一个value固定的HashMap。
HashMap是无序存储的，所以HashSet也一样是无序的，而且HashSet允许null值，但只能拥有一个null值，即不允许存储相同的元素。
## 二、常量变量
```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    //...
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    //...
}
```
上面的map即为HashSet底层的HashMap，针对HashSet的操作，全部转交给这个map来完成。
上面的PRESENT即为底层HashMap中键值对的值的固定值。应为在HashSet中只关注键。
## 三、构造器
```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable{
    //...
    public HashSet() {
        map = new HashMap<>();
    }
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }    
    //...
}
```
很明显，所有的HashSet的构造器最终都在创建底层的HashMap。
最后一个构造器创建了一个LinkedHashMap实例，其实它也是一个HashMap，因为它继承自HashMap，是对HashMap的一个功能扩展集合，它支持多种顺序的遍历（插入顺序和访问顺序）。
## 四、操作
```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable{
    //...
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
    public int size() {
        return map.size();
    }
    public boolean isEmpty() {
        return map.isEmpty();
    }
    public boolean contains(Object o) {
        return map.containsKey(o);
    }
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
    public void clear() {
        map.clear();
    }
    //...
}
```
上面的所有基础操作，全部已开HashMap的对应方法来完成。
## 五、序列化操作
### 5.1 序列化
HashSet实例的序列化执行时，并不会序列化map属性，因为其被transient关键字所修饰。参照源码：
```java
// 在执行序列化操作的时候会执行这个writeObject方法
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    //...
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        // 用于将对象中的非static和非transient的字段值写入流中
        s.defaultWriteObject();
        // Write out HashMap capacity and load factor
        // 将底层HashMap的当前容量和加载因子写入流中
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());
        // Write out size
        // 将底层HashMap的当前元素数量size写入流中
        s.writeInt(map.size());
        // Write out all elements in the proper order.
        // 最后将所有的元素写入流中
        for (E e : map.keySet())
            s.writeObject(e);
    }
    //...
}
```
### 5.2 反序列化
```java
// 在执行反序列化操作的时候会执行这个readObject方法
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    //...
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        // 读取流中对应的当前类的非static和非transient的值
        s.defaultReadObject();
        // Read capacity and verify non-negative.
        // 读取流中的容量值
        int capacity = s.readInt();
        if (capacity < 0) {
            throw new InvalidObjectException("Illegal capacity: " +
                                             capacity);
        }
        // Read load factor and verify positive and non NaN.
        // 读取流中的加载因子值
        float loadFactor = s.readFloat();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        }
        // Read size and verify non-negative.
        // 读取流中的元素数量值
        int size = s.readInt();
        if (size < 0) {
            throw new InvalidObjectException("Illegal size: " +
                                             size);
        }
        // Set the capacity according to the size and load factor ensuring that
        // the HashMap is at least 25% full but clamping to maximum capacity.
        capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
                HashMap.MAXIMUM_CAPACITY);
        // Constructing the backing map will lazily create an array when the first element is
        // added, so check it before construction. Call HashMap.tableSizeFor to compute the
        // actual allocation size. Check Map.Entry[].class since it's the nearest public type to
        // what is actually created.
        SharedSecrets.getJavaOISAccess()
                     .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));
        // Create backing HashMap
        // 创建底层HashMap实例
        map = (((HashSet<?>)this) instanceof LinkedHashSet ?
               new LinkedHashMap<E,Object>(capacity, loadFactor) :
               new HashMap<E,Object>(capacity, loadFactor));
        // Read in all elements in the proper order.
        // 读取流中保存的元素，并将其逐个添加到新创建的HashMap实例中
        for (int i=0; i<size; i++) {
            @SuppressWarnings("unchecked")
                E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
    //...
}
```
## 六、总结
HashSet就是依靠HashMap实现的。