# Java集合系列-RandomAccess
Random是随机的意思，Access是访问的意思，合起来就是随机访问的意思。

RandomAccess接口是一个标记接口，用以标记实现的List集合具备快速随机访问的能力。

那么什么是随机访问的能力呢？其实很简单，随机访问就是随机的访问List中的任何一个元素。

所有的List实现都支持随机访问的，只是基于基本结构的不同，实现的速度不同罢了，这里的快速随机访问，那么就不是所有List集合都支持了。
- rrayList基于数组实现，天然带下标，可以实现常量级的随机访问，复杂度为O(1)
- LinkedList基于链表实现，随机访问需要依靠遍历实现，复杂度为O(n)

二者的差距显而易见，所以ArrayList具备快速随机访问功能，而LinkedList没有。我们也能看到：
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{/*...*/}
```
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{/*...*/}
```
ArrayList实现了RandomAccess接口，而LinkedList没有。

那么到底这个接口有什么用呢？

当一个List拥有快速访问功能时，其遍历方法采用for循环最快速。而没有快速访问功能的List，遍历的时候采用Iterator迭代器最快速。

当我们不明确获取到的是Arraylist，还是LinkedList的时候，我们可以通过RandomAccess来判断其是否支持快速随机访问，若支持则采用for循环遍历，否则采用迭代器遍历，如下方式：
```java
public class RandomAccessTest {
    private List<String> list = null;
    public RandomAccessTest(List<String> list){
        this.list = list;
    }
    public void loop(){
        if(list instanceof RandomAccess) {
            // for循环
            System.out.println("采用for循环遍历");
            for (int i = 0;i< list.size();i++) {
                System.out.println(list.get(i));
            }
        } else {
            // 迭代器
            System.out.println("采用迭代器遍历");
            Iterator it = list.iterator();
            while(it.hasNext()){
                System.out.println(it.next());
            }
        }
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1110");
        List<String> list1 = new LinkedList<>();
        list1.add("aaa");
        list1.add("bbb");
        list1.add("ccc");
        new RandomAccessTest(list).loop();
        new RandomAccessTest(list1).loop();
    }
}
```
执行结果：
```text
采用for循环遍历
123
456
789
1110
采用迭代器遍历
aaa
bbb
ccc
```
那么都有哪些类实现了这个接口呢？
- ArrayList
- Vector
- CopyOnWriteArrayList
- RandomAccessSubList
- UnmodifiableArrayList
