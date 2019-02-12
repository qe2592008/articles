# Java集合系列-ArrayList
## 一、概述
ArrayList底层使用的是数组。是List的可变数组实现，这里的可变是针对List而言，而不是底层数组。

数组有自身的特点，不变性，一旦数组被初始化，那么其长度就固定了，不可被改变。这就导致了ArrayList中的一个重要特性：扩容。
## 二、源码解析
### 2.1 声明
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{/*...*/}
```
可以看到ArrayList类实现了四个接口：
- List<E>：支持List接口中提供的方法
- RandomAccess：支持快速随机访问
有关RandomAccess可见：[Java集合系列-RandomAccess](Java集合系列-RandomAccess)
- Cloneable：支持对象克隆功能
有关Cloneable可见：[Java基础系列-浅拷贝和深拷贝](../Java基础系列/Java基础系列-浅拷贝和深拷贝)
- Serializable：支持序列化功能
有关Serializable可见：[Java基础系列-序列化与反序列化](../Java基础系列/Java基础系列-序列化与反序列化)

还继承自AbstractList<E>抽象类，这个抽象类是List<E>的抽象实现，实现了一些List中的公共方法。
### 2.2 字段解析
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 默认的初始容量
    private static final int DEFAULT_CAPACITY = 10;
    // 共享使用的空实例，这个空实例是没有容量的空实例
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 共享使用的空实例，这个空实例可被扩容到初始容量（10）
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // ArrayList中保存元素的缓冲数组，DEFAULTCAPACITY_EMPTY_ELEMENTDATA标识的空数组在第一个添加元素时会被扩容到10个大小。
    transient Object[] elementData; // non-private to simplify nested class access
    // ArrayList集合中包含的元素数量
    private int size;
    // 集合的容量最大值为Integer的最大值-8，这里为什么减去8呢？主要是因为一些虚拟机会在数组中保存一些头信息，这些信息是区别于使用者添加的元素之外的存在，如果最大为Integer的最大，当头信息添加之后，再添加元素就有可能会造成内存溢出。
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```
字段中有几个需要注意的点：
1. EMPTY_ELEMENTDATA和DEFAULTCAPACITY_EMPTY_ELEMENTDATA的区别：前者表示的是一个空数组，后者表示的也是一个空数组，但是不同在于后者是可以扩容的，当往进添加首个元素的时候就会触发扩容机制，容量会扩容到10个长度。
2. elementData字段是保存元素的缓冲数组，被transient修饰表示它不会被序列化，这意味着集合对象保存的元素不会被自动序列化，所以后面添加了writeObject和readObject方法，用来序列化和反序列化数组中的元素。
### 2.3 构造器解析
ArrayList有三个构造器：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 构建一个初始容量是10的ArrayList
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    // 根据给定的初始容量initialCapacity构建一个ArrayList
    // 如果initialCapacity>0则直接直接创建容量为initialCapacity的ArrayList
    // 如果initialCapacity=0则直接使用EMPTY_ELEMENTDATA空集合
    // 如果initialCapacity<0，则出错。
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    // 将给定的集合转换为ArrayList
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }    
}
```
我们最常使用的其实是第一种，但是在我们实际编程时，如果可以预估到集合的最大容量，那么可以使用第二种方式，这样可以减少扩容的时间和内存消耗，一次性到位。
### 2.4 添加
#### 2.4.1 添加指定元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
}
```
ensureCapacityInternal方法主要用于校验当前List的容量是否已经达到极限，如果达到极限需要进行扩容。具体参照2.11中扩容解析。

剩下的就是添加新元素的逻辑，简单至极，直接将新元素到添加到底层数组elementData的下一下标位size++即可。
#### 2.4.2 添加指定元素到指定位置
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
}
```
首先校验给定的添加位置index，index必须小于size的值，并大于等于0。

然后同样校验容量是否达到极限，达到极限需要扩容。

之后执行一个本地方法，System.arraycopy方法用于将指定位置及其后面的所有元素整个通过复制迁移到从index+1开始的位置，即整体后移一位，将index位空出来用于保存新元素。

最后将新元素添加到空出的index位置。
#### 2.4.3 将指定集合中的元素添加到List末尾
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
}
```
首先将给定的集合转换为数组Object[]。

然后以目标List的size+给定集合转化的数组的容量为总容量进行容量校验，若容量不足，执行扩容操作。

再然后通过本地方法执行数组复制操作将给定集合转换数组的元素复制到目标List的底层数组的尾部。

最后不要忘记将size增加。
#### 2.4.4 将指定集合中的元素添加到List指定位置
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
}
```
首先校验下标index，index必须小于size，大于等于0。

然后将给定集合转换为数组Object[]，再执行容量校验，扩容操作。

通过本地方法数组复制操作将给定位置开始的所有元素整体后移一定的距离，具体的距离为给定集合转换后数组的容量大小，这样就能空出容量大小的空位来存放给定的集合元素。

最后再次通过本地数组复制方法将给定的集合转换的数组元素整体复制到上一步空出来的位置上。
### 2.5 修改
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
}
```
修改指定位置的元素为新元素，首先需要校验给定index的值，index必须大于等于0，小于size，然后将新元素保存到index位置，并将旧元素返回。
### 2.6 获取
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
}
```
获取指定下标位置的元素值，首先需要校验给定的下标index，index必须大于等于0，小于size。
### 2.7 定位
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
}
```
indexOf是通过正序遍历的方式搜索给定的元素的下标，lastIndexOf是通过逆序遍历的方式搜索给定元素的下标，这两个方法找到的下标都是正序或者逆序该元素首次出现的位置下标。如果o为null，那么将会搜索第一个null值元素的下标。
### 2.8 移除
#### 2.8.1 移除指定下标的元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
}
```
首先校验给定的下标值index,index必须小于size，这里的校验和添加元素的下标校验有点不同：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
}
```
前者是此处的index校验，后者是添加元素的index校验。

那为何后者比前者要多一个index<0的校验呢，那是因为在add(int,E)方法中，校验完成后紧接着就是调用本地方法进行数组复制操作，如果index小于0，那么出错位置在C代码中，无法在Java代码中得以体现，所以提前进行校验，保证调用本地C代码之前参数的准确性。前者校验完成之后，紧接着的是Java代码获取指定下标的元素，如果下标小于0，也会出错但是JVM会抛出异常，不会无声无息，所以没有必要校验是否小于0。

index校验完成后，通过本地方法数组复制将index+1及其之后的元素整体复制到index位置。

最后将原来的最后一个位置元素置空。
#### 2.8.2 移除指定元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }    
}
```
首先通过循环操作找到首个指定的元素，然后将针对找到的元素执行删除操作。

删除操作还是依靠本地的数组复制操作完成的。
#### 2.8.3 清空元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
}
```
至于清空元素，就是通过循环将List中的每个元素都删除，将整个List置空。
#### 2.8.4 移除当前List中所有(不)包含在给定集合中的元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 删除当前List中所有包含在给定集合中的元素
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
    // 删除当前List中所有不包含在给定集合中的元素
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
}
```

### 2.9 遍历
ArrayList的遍历方式有很多：
#### 2.9.1 ListIterator
ListIterator是继承自Iterator的，在其基础上添加了反向遍历的功能方法。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 截取从执行下标开始的元素组成迭代器实例，进行遍历
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }
    // 将集合中所有元素组成迭代器实例，进行遍历
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
}
```
源码中的ListItr是ListIterator的实现类。
#### 2.9.2 Iterator
Iterator拥有正向遍历的功能。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public Iterator<E> iterator() {
        return new Itr();
    }
}
```
源码中的Itr就是Iterator的实现类。
#### 2.9.3 Spliterator
Spliiterator是分割迭代器，详情参见[Java集合系列-Spliterator](Java集合系列-Spliterator)
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public Spliterator<E> spliterator() {
        return new ArrayListSpliterator<>(this, 0, -1, 0);
    }
}
```
#### 2.9.4 forEach
forEach方式是java 1.8中新增的方式，接受一个行为作为参数，即接收一个方法引用或者Lambda表达式。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // action代表接受的行为，是一个函数式接口类型Consumer，表示消费之意，消费就是将资源处理掉，所以有一个入参，无返回值。
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);// 执行函数式接口行为
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
}
```
实例：
```java
public class ArrayListTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.addAll(Arrays.asList("123","444444","2123"));
        ListIterator<String> listIterator = list.listIterator();// 第一种
        listIterator.forEachRemaining(System.out::println);
        System.out.println("-------------");
        ListIterator<String> listIterator1 = list.listIterator(1);// 第二种
        listIterator1.forEachRemaining(System.out::println);
        System.out.println("-------------");
        Iterator<String> iterator = list.iterator();// 第三种
        iterator.forEachRemaining(System.out::println);
        System.out.println("-------------");
        Spliterator<String> spliterator = list.spliterator();// 第四种
        spliterator.forEachRemaining(System.out::println);
        System.out.println("-------------");
        list.forEach(System.out::println);// 第五种
    }
}
```
### 2.10 校验
#### 2.10.1 是否为空
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean isEmpty() {
        return size == 0;
    }
}
```
size表示的就是List中包含的元素的个数。
#### 2.10.2 是否包含某元素
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
}
```
该校验通过indexOf()方法来实现，如果能找到元素的下标，则存在，否则不存在。
### 2.11 底层数组扩容
在add和addAll方法中多次出现的ensureCapacityInternal方法就是通向扩容逻辑的通道。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 确保底层数组的容量足够保存当前的元素或元素集，如果容量不足即进行扩容。
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    // 处理首次添加元素时的容量扩容操作，被指定为DEFAULTCAPACITY_EMPTY_ELEMENTDATA的空数组在首次添加元素时需要自动扩容到默认容量10
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    // 校验是否需要扩容，只有当给定容量值比当前数组的长度要大时，才需要扩容，
    // 因为一般情况下给定容量即为新添加元素后的容量，当前容量达不到这个值是没有位置保存当前元素的，所以才需要扩容。
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
}
```
ensureCapacityInternal方法的目的是确保给定的参数指定的容量值。

真正的扩容逻辑位于grow方法中：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);// 扩容为原容量的1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果最后决定扩容的容量比允许的最大数组容量值要大，那么则进行超限处理
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    // 处理超限问题
    // 如果给定的minCapacity为负数（首位为1）则抛出异常错误OutOfMemoryError
    // 如果给定容量大于数组最大容量，则取整数的最大值为容量，否则使用数组的最大容量作为扩容容量
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
}
```
首先要根据规则计算一个新容量newCapacity，然后将这个新容量值与给定需要的容量值minCapacity进行比较，如果新容量值大于给定容量值，则用新容量值进行扩容，否则使用给定容量值进行扩容。然后进行超限校验和处理。

最后使用确定好的容量newCapacity来作为新的底层数组容量来进行扩容操作：创建一个新的数组，并迁移元素。
### 2.12 排序
Java中排序可以通过两种方式实现：
- 实现Comparable接口
- 使用Comparator比较器

具体参见[Java基础系列-Comparable和Comparator](../Java基础系列/Java基础系列-Comparable和Comparator.md)

这里很明显ArrayList的继承体系中并无Comparable接口，那么只能通过Comparator来实现，这就涉及到了ArrayList中的sort方法：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
}
```
使用这种方式来排序需要传递一个Comparator比较器作为参数，最简单的方式就是匿名内部类方式，在Java 1.8之后直接使用Lambda来实现。
```java
public class ComparatorTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.addAll(Arrays.asList("123","45612","7839"));
        list.sort((o1, o2) -> o1.length()-o2.length());
        list.forEach(System.out::println);
    }
}
```
执行结果为：
```text
123
7839
45612
```
### 2.13 克隆
因为ArrayList实现了Cloneable接口，重写了clone方法，便拥有了对象克隆的功能。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
}
```
这是一个浅拷贝的实现。
### 2.14 序列化/反序列化
由于ArrayList中使用transient修饰了elementData，它代表的是底层的元素数组，序列化的主要内容就是它，或者说是它里面的内容，而它又无法被序列化，因此我们只能通过自定义writeObject方法来手动序列化，定义readObject方法来手动反序列化。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }    
}
```
至此，ArrayList的大部分内容都介绍完毕了。
## 三、总结
最后做一下总结，知识点归纳：
- ArrayList底层采用数组实现，拥有快速随机访问能力，但是非线程安全的集合。
- ArrayList默认容量为10，扩容规则为当要保存的新元素所需的容量不足时触发，基本规则为扩容1.5倍。
- 如果在遍历的时候发生结构性变化，会触发ConcurrentModificationException异常。
- 结构性变化包括：添加新元素，删除元素。
- ArrayList支持序列化功能，支持克隆（浅拷贝）功能，排序功能等。