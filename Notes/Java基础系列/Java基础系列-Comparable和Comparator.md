# Java基础系列-Comparable和Comparator
## 概述
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Java中的排序是由Comparable和Comparator这两个接口来提供的。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Comparable表示可被排序的，实现该接口的类的对象自动拥有排序功能。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Comparator则表示一个比较器，实现了该接口的的类的对象是一个针对目标类的对象定义的比较器，一般情况，这个比较器将作为一个参数进行传递。
## Comparable
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Comparable的中文意思就是可被排序的，代表本身支持排序功能。只要我们的类实现了这个接口，那么这个类的对象就会自动拥有了可被排序的能力。而且这个排序被称为类的自然顺序。这个类的对象的列表可以被Collections.sort和Arrays.sort来执行排序。同时这个类的实例具备作为sorted map的key和sorted set的元素的资格。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;假如a和b都是实现了Comparable接口的类C的实例，那么只有当a.compareTo(b)的结果与a.equals(b)的结果一致时，才称类C的自然顺序与equals一致。强烈建议将类的自然顺序和equals的结果保持一致，因为如果不一致的话，由该类对象为键的sorted map和由该类对象为元素的sorted set的行为将会变得很怪异。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;例如对于一个实现了Comparable接口的元素的有序集合sorted set而言，如果a.equals(b)结果为false，并且a.compareTo(b)==0，则第二个元素的添加操作将会失败，因为在sorted set看来，二者在排序上是一致的，它不报保存重复的元素。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;事实上，Java中的类基本都是自然顺序与equals一致的，除了BigDecimal，因为BigDecimal中的自然顺序和值相同但精度不同的元素（例如4和4.00）的equals均一致。
### 源码解析
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;从源码中可以看到，该接口只有一个抽象方法compareTo，这个方法主要就是为了定义我们的类所要排序的方式。compareTo方法用于比较当前元素a与指定元素b，结果为int值，如果a > b，int>0；如果a=b，int=0；如果a<b，int<0。
## Comparator
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Comparator中文译为比较器，它可以作为一个参数传递到Collections.sort和Arrays.sort方法来指定某个类对象的排序方式。同时它也能为sorted set和sorted map指定排序方式。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;同Comparable类似，指定比较器的时候一般也要保证比较的结果与equals结果一致，不一致的话，对应的sorted set和sorted map的行为同样会变得怪异。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;推荐实现的比较器类同时实现java.io.Serializable接口，以拥有序列化能力，因为它可能会被用作序列化的数据结构（TreeSet、TreeMap）的排序方法。
### 源码解析
```java
@FunctionalInterface
public interface Comparator<T> {
    // 唯一的抽象方法，用于定义比较方式（即排序方式）
    // o1>o2，返回1；o1=o2，返回0；o1<o2，返回-1
    int compare(T o1, T o2);
    boolean equals(Object obj);
    // 1.8新增的默认方法：用于反序排列
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    // 1.8新增的默认方法：用于构建一个次级比较器，当前比较器比较结果为0，则使用次级比较器比较
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }
    // 1.8新增默认方法：指定次级比较器的
    // keyExtractor表示键提取器，定义提取方式
    // keyComparator表示键比较器，定义比较方式
    default <U> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        return thenComparing(comparing(keyExtractor, keyComparator));
    }
    // 1.8新增默认方法：用于执行键的比较，采用的是由键对象内置的比较方式
    default <U extends Comparable<? super U>> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        return thenComparing(comparing(keyExtractor));
    }
    // 1.8新增默认方法：用于比较执行int类型的键的比较
    default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
        return thenComparing(comparingInt(keyExtractor));
    }
    // 1.8新增默认方法：用于比较执行long类型的键的比较
    default Comparator<T> thenComparingLong(ToLongFunction<? super T> keyExtractor) {
        return thenComparing(comparingLong(keyExtractor));
    }
    // 1.8新增默认方法：用于比较执行double类型的键的比较
    default Comparator<T> thenComparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        return thenComparing(comparingDouble(keyExtractor));
    }
    // 1.8新增静态方法：用于得到一个相反的排序的比较器，这里针对的是内置的排序方式（即继承Comparable）
    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }
    // 1.8新增静态方法：用于得到一个实现了Comparable接口的类的比较方式的比较器
    // 简言之就是将Comparable定义的比较方式使用Comparator实现
    @SuppressWarnings("unchecked")
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }
    // 1.8新增静态方法：得到一个null亲和的比较器，null小于非null，两个null相等，如果全不是null,
    // 则使用指定的比较器比较，若未指定比较器，则非null全部相等返回0
    public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(true, comparator);
    }
    // 1.8新增静态方法：得到一个null亲和的比较器，null大于非null，两个null相等，如果全不是null,
    // 则使用指定的比较器比较，若未指定比较器，则非null全部相等返回0
    public static <T> Comparator<T> nullsLast(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(false, comparator);
    }
    // 1.8新增静态方法：使用指定的键比较器用于执行键的比较
    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }
    // 1.8新增静态方法：执行键比较，采用内置比较方式，key的类必须实现Comparable
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
    // 1.8新增静态方法：用于int类型键的比较
    public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }
    // 1.8新增静态方法：用于long类型键的比较
    public static <T> Comparator<T> comparingLong(ToLongFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Long.compare(keyExtractor.applyAsLong(c1), keyExtractor.applyAsLong(c2));
    }
    // 1.8新增静态方法：用于double类型键的比较
    public static<T> Comparator<T> comparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Double.compare(keyExtractor.applyAsDouble(c1), keyExtractor.applyAsDouble(c2));
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;老版本的Comparator中只要两个方法，就是前两个方法，后面的所有默认方法均为1.8新增的方法，采用的是1.8新增的功能：接口可添加默认方法。即便拥有如此多方法，该接口还是函数式接口，compare用于定义比较方式。
## 二者比较
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Comparable可以看做是内部比较器，Comparator可以看做是外部比较器。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;一个类，可以通过实现Comparable接口来自带有序性，也可以通过额外指定Comparator来附加有序性。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;二者的作用其实是一致的，所以不要混用。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;我们看个例子吧：  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;首先定义个模型：User
```java
public class User implements Serializable, Comparable<User> {
    private static final long serialVersionUID = 1L;
    private int age;
    private String name;
    public User (){}
    public User (int age, String name){
        this.age = age;
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public int compareTo(User o) {
        return this.age - o.age;
    }
    @Override
    public String toString() {
        return "[user={age=" + age + ",name=" + name + "}]";
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;在定义一个Comparator实现类MyComparator
```java
public class MyComparator implements Comparator<User> {
    @Override
    public int compare(User o1, User o2) {
        return o1.getName().charAt(0)-o2.getName().charAt(0);
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;最后是测试类：Main
```java
public class Main {
    public static void main(String[] args) {
        User u1 = new User(12, "xiaohua");
        User u2 = new User(10, "abc");
        User u3 = new User(15,"ccc");
        User[] users = {u1,u2,u3};
        System.out.print("数组排序前：");
        printArray(users);
        System.out.println();
        Arrays.sort(users);
        System.out.print("数组排序1后：");
        printArray(users);
        System.out.println();
        Arrays.sort(users, new MyComparator());
        System.out.print("数组排序2后：");
        printArray(users);
        System.out.println();
        Arrays.sort(users, Comparator.reverseOrder());// 针对内置的排序进行倒置
        System.out.print("数组排序3后：");
        printArray(users);
    }
    public static void printArray (User[] users) {
        for (User user:users) {
            System.out.print(user.toString());
        }
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;运行结果为：
```text
数组排序前：[user={age=12,name=xiaohua}][user={age=10,name=abc}][user={age=15,name=ccc}]
数组排序1后：[user={age=10,name=abc}][user={age=12,name=xiaohua}][user={age=15,name=ccc}]
数组排序2后：[user={age=10,name=abc}][user={age=15,name=ccc}][user={age=12,name=xiaohua}]
数组排序3后：[user={age=15,name=ccc}][user={age=12,name=xiaohua}][user={age=10,name=abc}]
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;通过上面的例子我们有一个结论，那就是两种方式定义排序的优先级，明显Comparator比较器要优先于内部排序Comparable。
## 总结
- Comparable为可排序的，实现该接口的类的对象自动拥有可排序功能。
- Comparator为比较器，实现该接口可以定义一个针对某个类的排序方式。
- Comparator与Comparable同时存在的情况下，前者优先级高。

参考：
- [Java 中 Comparable 和 Comparator 比较](https://www.cnblogs.com/skywang12345/p/3324788.html)