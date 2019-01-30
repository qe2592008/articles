# Java流式操作系列-Stream
## 一、概述
Stream操作简称流操作，这里的流与IO流毫无关系，这里的流指的是流式操作，就是流水线操作。

Stream流操作主要包包括三大模块：创建流操作、中间流操作、终结流操作。

其中创建流主要是创建Stream对象。每个Stream对象只能使用一次终结操作。

中间流操作指的是各种中间流操作方法，比如去重、过滤、排序等

终结流操作指的结果操作，终结操作的目的是产生最终结果。
## 二、创建流
### 2.1 基于数组创建流
```java
public class StreamTest {
    public static void createStream() {
        // 通过数组生成流
        int[] ints = {1,2,3,4,5,6};
        IntStream s1 = Arrays.stream(ints);
        Stream s2 = Stream.of("111","222","333");
        String[] ss = {"123","321","456","654"};
        Stream<String> s3 = Arrays.stream(ss);
    }
}
```
### 3.2 通过构建器生成流
```java
public class StreamTest {
    public static void createStream() {
        // 通过构建器生成流
        Stream<Object> s4 = Stream.builder().add("123").add("321").add("444").add("@21").build();
    }
}
```
### 3.3 基于集合生成流
```java
public class StreamTest {
    public static void createStream() {
        // 通过集合生成流
        List<String> lists = Arrays.asList("123","321","1212","32321");
        Stream<String> s5 = lists.stream();
        Stream<String> s6 = lists.parallelStream();// 并行流
    }
}
```
### 3.4 创建空流
```java
public class StreamTest {
    public static void createStream() {
        // 创建空流
        Stream<String> s7  = Stream.empty();
    }
}
```
### 3.5 基于函数创建无限流
```java
public class StreamTest {
    public static void createStream() {
        // 创建无限流
        Stream.generate(()->"number"+new Random().nextInt()).limit(100).forEach(System.out::println);
        Stream.iterate(0,n -> n+2).limit(10).forEach(System.out::println);
    }
}
```
## 三、流中间操作
这里的流中间操作指的是该操作的返回值仍然是流。

| 序号 | 操作   | 方法                                                       | 说明                                                         | 备注   |
| ---- | ------ | ---------------------------------------------------------- | ------------------------------------------------------------ | ------ |
| 1    | filter | Stream<T> filter(Predicate<? super T> predicate)           | 返回当前流中满足参数predicate过滤条件的元素组成的新流        | 过滤器 |
| 2    | map    | <R> Stream<R> map(Function<? super T, ? extends R> mapper) | 返回通过给定mapper作用于当前流的每个元素之后的结果组成的新流 | 函数   |
| 3    | mapToInt| IntStream mapToInt(ToIntFunction<? super T> mapper)| 返回通过给定mapper作用于当前流的每个元素之后的结果组成的新的Int流| 函数|
|4|mapToLong|LongStream mapToLong(ToLongFunction<? super T> mapper)|返回通过给定mapper作用于当前流的每个元素之后的结果组成的新的Long流|函数|
|5|mapToDouble|DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)|返回通过给定mapper作用于当前流的每个元素之后的结果组成的新的Double流|函数|
|6|flatMap|<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)|根据给定的mapper作用于当前流的每个元素，将结果组成新的流来返回|扁平函数|
|7|flatMapToInt|IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper)|根据给定的mapper作用于当前流的每个元素，将结果组成新的Int流来返回|扁平函数|
|8|flatMapToLong|LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper)|根据给定的mapper作用于当前流的每个元素，将结果组成新的Long流来返回|扁平函数|
|9|flatMapToDouble|DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper)|根据给定的mapper作用于当前流的每个元素，将结果组成新的Double流来返回|扁平函数|
|10|distinct|Stream<T> distinct()|返回去掉当前流中重复元素之后的新流|去重|
|11|sorted|Stream<T> sorted()|返回当前流中元素排序之后的新流，需要元素类型实现Comparable|排序|
|12|sorted|Stream<T> sorted(Comparator<? super T> comparator)|返回当前流中元素排序之后的新流，需要传递一个Comparator|排序|
|13|peek|Stream<T> peek(Consumer<? super T> action)|针对流中的每个元素执行操作action|查阅|
|14|limit|Stream<T> limit(long maxSize)|返回指定的数量的元素组成的新流|限制|
|15|skip|Stream<T> skip(long n)|返回第n个之后的元素组成的新流|跳过|
> 扁平函数，就是将当前流的每个元素通过执行给定的mapper操作，从而扩充，释放每个元素内的子元素，从而形成一个由所有子元素组成的新流，比如当前流是包含N个字符串的流，使用这个方法，可以获取到包含字符串中字符组成的流。
### 3.1 filter
filter方法是过滤器方法，针对的是流中所有元素，满足条件的元素将会被保留以组成新的流。
```java
public class StreamTest {
    public static void filterTest(List<String> list){
        list.stream()
                .filter(e -> e.length() > 4 && e.length()<7)// 过滤掉长度小于等于4,大于等于7的元素
                .peek(System.out::println)// 查阅中间流结果
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        filterTest(list);
    }
}
```
执行结果为：
```text
asdaa
3e3e3e
```
filter方法的参数是Predicate类型，这个函数式接口用于获取一个参数返回一个boolean值，整个参数作为过滤条件。
### 3.2 map
map方法可以理解为函数，需要针对流中的每个元素执行，然后将执行的结果组成新的流返回。
```java
public class StreamTest {
    public static void mapTest(List<String> list){
        list.stream()
                .map(e -> "@" + e)// 为每个元素执行操作：添加前缀
                .peek(System.out::println)// 查阅中间流结果
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        mapTest(list);
    }
}
```
执行结果为：
```text
@123
@456
@789
@1101
@asdaa
@3e3e3e
@2321eew
@212121121
```
map方法的参数类型为Function，该函数式接口用于接受一个参数，返回一个结果。

mapToInt、mapToLong、mapToDouble方法是map方法的扩展，其参数分别为ToIntFunction、ToLongFunction、ToDoubleFunction，分别接口一个参数，返回指定类型的值，分别为int、long、double，那么定义方法的时候就要注意返回值的类型了，必须一致，最后组成的新流就是一个int或long或double元素流（IntStream、LongStream、DoubleStream）。

mapToInt的简单使用（其他类似）：
```java
public class StreamTest {
    public static void mapToIntTest(List<String> list){
        list.stream()
                .mapToInt(e -> e.length())// 以元素的长度为新流
                .peek(System.out::println)// 查询中间结果
                .toArray();
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        mapToIntTest(list);
    }
}
```
执行结果为：
```text
3
3
3
4
5
6
7
9
```
### 3.3 flatMap
flatMap和map还是有点关系的，都是针对流中的每一个元素进行操作，将结果组成新流，不过flatMap含有一层扩展之意，就是当流中元素包含子元素的时候，通过该方法，获取到元素的子元素，并将子元素组成新流返回。
```java
public class StreamTest {
    public static void flatMap(List<String> list){
        list.stream()
                .filter(e -> e.length()>5 && e.length()<7)
                .peek(System.out::println)
                .map(e -> e.split(""))// 将每个字符串元素分解为字符数组
                .flatMap(Arrays::stream)//将每个字符数组并转化为流
                .peek(System.out::println)
                .collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        flatMap(list);
    }
}
```
执行结果为：
```text
3e3e3e
3
e
3
e
3
e
```
flatMapToInt、flatMapToLong、flatMapToDouble类似于之前的mapToInt之类。
### 3.4 distinct
distinct方法用于去重，很简单。
```java
public class StreamTest {
    public static void distinctTest(){
        int[] int1 = {1,2,3,4};
        int[] int2 = {5,3,7,1};
        List<int[]> ints = Arrays.asList(int1,int2);
        ints.stream()
                .flatMapToInt(Arrays::stream)
                .distinct()
                .peek(System.out::println)
                .toArray();
    }
    public static void main(String[] args) {
        distinctTest();
    }
}
```
执行结果为：
```text
1
2
3
4
5
7
```
结果中显而易见，重复的1和3被去除了。
### 3.5 sorted
sorted表示对流中的元素进行排序，需要使用Conparable和Comparator。
```java
public class StreamTest {
    public static void sortedTest(List<String> list){
        System.out.println("----自然顺序:");
        list.stream().sorted().peek(System.out::println).collect(Collectors.toList());
        System.out.println("----指定排序:");
        list.stream().sorted((a,b) -> a.length()-b.length()).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        sortedTest(list);
    }
}
```
执行结果为：
```text
----自然顺序:
1101
123
212121121
2321eew
3e3e3e
456
789
asdaa
----指定排序:
123
456
789
1101
asdaa
3e3e3e
2321eew
212121121
```
当调用无参的sorted方法时，采用自然排序法排序，当使用指定比较器的方式时，可以自由指定排序规则。
### 3.6 limit
limit可用于从首个元素开始截取N个元素，组成新流返回。
```java
public class StreamTest {
    public static void limitTest(List<String> list){
        list.stream().limit(2).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        limitTest(list);
    }
}
```
执行结果为：
```text
123
456
```
### 3.7 skip
skip表示放弃N个元素，将剩余元素组成新流返回。
```java
public class StreamTest {
    public static void skipTest(List<String> list){
        list.stream().skip(2).peek(System.out::println).collect(Collectors.toList());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","asdaa","3e3e3e","2321eew","212121121");
        skipTest(list);
    }
}
```
执行结果为：
```text
789
1101
asdaa
3e3e3e
2321eew
212121121
```
放弃了前2个元素，将剩余元素组成了新流。
## 四、流终结操作
| 序号 | 操作   | 方法                                                       | 说明                                                         | 备注   |
| ---- | ------ | ---------------------------------------------------------- | ------------------------------------------------------------ | ------ |
|1|forEach|void forEach(Consumer<? super T> action)|对流中的每个元素执行指定的操作action|遍历|
|2|forEachOrdered|void forEachOrdered(Consumer<? super T> action)|如果有序，则按序遍历流中元素，针对每个元素执行指定操作|按序遍历|
|3|toArray|Object[] toArray()|返回一个包含流中所有元素的数组|数组化|
|4|toArray|<A> A[] toArray(IntFunction<A[]> generator)|返回一个包含流中所有元素的参数指定类型的数组|数组化|
|5|reduce|T reduce(T identity, BinaryOperator<T> accumulator)|以给定初始值为基础归纳流中元素，返回一个值|归纳|
|6|reduce|Optional<T> reduce(BinaryOperator<T> accumulator)|直接归纳流中的元素，返回一个封装有结果的Optional|归纳|
|7|reduce|\<U\> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator\<U\> combiner)|以给定的初始值为基础，（并行）归纳流中元素，最后将各个线程的结果再统一归纳，返回一个值|归纳|
|8|collect|<R, A> R collect(Collector<? super T, A, R> collector)|根据给定的收集器收集元素|归纳|
|9|collect|<R> R collect(Supplier<R> supplier,BiConsumer<R, ? super T> accumulator,BiConsumer<R, R> combiner)|根据给定的各个参数归纳元素|归纳|
|10|max|Optional<T> max(Comparator<? super T> comparator)|根据给定的比较器，返回流中最大元素的Optional表示|最大值|
|11|min|Optional<T> min(Comparator<? super T> comparator)|根据给定的比较器，返回流中最小元素的Optional表示|最小值|
|12|count|long count()|返回流中元素的个数|计数|
|13|anyMatch|boolean anyMatch(Predicate<? super T> predicate)|校验流中是否有满足给定条件的元素|校验|
|14|allMatch|boolean allMatch(Predicate<? super T> predicate)|校验流中的元素是否全部满足给定条件|校验|
|15|noneMatch|boolean noneMatch(Predicate<? super T> predicate)|校验流中的元素是否全不满足给点条件|校验|
|16|findFirst|Optional<T> findFirst()|返回首个元素的Optional表示，如果为空流，返回空的Optional|返回首个元素|
|17|findAny|Optional<T> findAny()|如果流中有元素，则返回第一个元素的Optional表示，否则返回一个空的Optional|校验是否为空流|
### 4.1 forEach和forEachOrdered
forEach就是遍历操作，针对流中的每个元素做最后的操作。
```java
public class StreamTest {
    public static void forEachTest(List<String> list){
        list.stream().parallel().forEach(System.out::println);
    }
    public static void forEachOrderedTest(List<String> list){
        list.stream().parallel().forEachOrdered(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        forEachTest(list);
        System.out.println("----------");
        forEachOrderedTest(list);
    }
}
```
执行结果为：
```text
asdaa
212121121
789
1101
2321eew
3e3e3e
456
123
----------
123
456
789
1101
212121121
asdaa
3e3e3e
2321eew
```
二者都是遍历操作，从结果是可以看出来，如果是单线程（也就是不加parallel方法的情况）那么二者结果是一致的，但是如果采用并行遍历，那么就有区别了，forEach并行遍历不保证顺序（顺序随机）,forEachOrdered却是保证顺序来进行遍历的。
### 4.2 toArray
```java
public class StreamTest {
    public static void toArrayTest(List<String> list){
        Object[] objs = list.stream().filter(e -> e.length()>6).toArray();
        String[] ss = list.stream().filter(e -> e.length()>6).toArray(String[]::new);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        toArrayTest(list);
    }
}
```
toArray有两个方法，一个是无参方法，一个有参方法。

无参方法返回的只能是Object[]数组类型，而有参方法，可以指定结果数组类型，此乃二者区别。

使用有参方法可以直接完成类型转换，一次到位。
### 4.4 reduce
reduce方法有三个重载的方法，
```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    Optional<T> reduce(BinaryOperator<T> accumulator);// 编号1
    T reduce(T identity, BinaryOperator<T> accumulator);// 编号2
    <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);// 编号3
}
```
这三个方法的作用其实是一样的，就是归纳总结的意思。

首先看编号1方法，只有一个参数accumulator，这是一个累加器，方法的作用就是将这个累加器作用到流中的每一个元素，他需要两个输入参数，有一个输出参数，意思是对两个元素执行某些操作，返回一个结果，然后将这个结果与下一个元素作为参数再输入该方法，执行操作后再返回一个新结果，以此类推，直到最后一个元素执行完毕，返回的就是最终结果，因为流中的元素我们是不确定的，那么我们就无法确定reduce的结果，因为如果流为空，那么将会返回null，所以使用Optional作为返回值，妥善处理null值。

再看编号2方法，在编号1方法的基础上加了一个identity，且不再使用Optional，为什么呢，因为新加的identity其实是个初始值，后续的操作都在这个值基础上执行，那么也就是说，，如果流中没有元素的话，还有初始值作为结果返回，不会存在null的情况，也就不用Optional了。

再看编号3方法，在编号2方法的基础上又加了一个参数combiner，其实这个方法是用于处理并行流的归纳操作，最后的参数combiner用于归纳各个并行的结果，用于得出最终结果。

那么如果不使用并行流，一般使用编号2方法就足够了。

示例：
```java
public class StreamTest {
    public static void reduceTest(){
        List<Integer> ints = Arrays.asList(1,2,3,4,5,6,7,8,9);
        Optional<Integer> optional = ints.stream().reduce(Integer::sum);
        System.out.println(optional.get());
        System.out.println("-------------");
        Integer max = ints.stream().reduce(Integer.MIN_VALUE, Integer::max);
        System.out.println(max);
        System.out.println("-------------");
        Integer min = ints.parallelStream().reduce(Integer.MAX_VALUE, Integer::min, Integer::min);
        System.out.println(min);
    }
    public static void main(String[] args) {
        reduceTest();
    }
}
```
执行结果为：
```text
45
-------------
9
-------------
1
```
### 4.5 collect
collect操作是Stream中最强大的方法了，几乎可以得到任何你想要的结果，collect方法有两个重载方法：
```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);// 编号1
    <R, A> R collect(Collector<? super T, A, R> collector);// 编号2
}
```
collect是收集的意思，这里的作用就是收集归纳，将流中的数据映射为各种结果。

首先看看编号1方法，有三个参数：supplier用于生成一个R类型的结果容器来盛放结果，accumulator累加器用于定义盛放的方式，其中T为一个元素，R为结果容器，第三个参数combiner的作用是将并行操作的各个结果整合起来。
```java
public class StreamTest {
    public static void collectTest1(List<String> list){
        ArrayList<String> arrayList = list.stream().skip(4).collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
        arrayList.forEach(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        collectTest1(list);
    }
}
```
执行结果：
```text
212121121
asdaa
3e3e3e
2321eew
```
例子中，第一个：ArrayList::new表示创建一个新的ArrayList集合，第二个 ArrayList::add表示将元素一个一个添加到之前的集合中，第三个ArrayList::addAll表示将多个线程的ArrayList集合一个一个的整体添加到第一个集合中，最终整合出一个最终结果并返回。

然后我们重点来看看编号2方法。

它只需要一个Collector类型的参数，这个Collector可以称呼为收集器，我们可以随意组装一个收集器来进行元素归纳。

Collector是定义来承载一个收集器，但是JDK提供了一个Collectors工具类，在这个工具类里面预实现了N多的Collector供我们直接使用，之前的Collectors.toList()就是其用法之一。具体见下文。
```java
public class StreamTest {
    public static void collectTest2(List<String> list){
        Set<String> set = list.stream().skip(4).collect(Collectors.toSet());
        set.forEach(System.out::println);
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        collectTest2(list);
    }
}
```
执行结果为：
```text
212121121
2321eew
3e3e3e
asdaa
```
### 4.6 max\min
通过给定的比较器，得出流中最大\最小的元素，为避免null返回，这里使用Optional来封装返回值。
```java
public class StreamTest {
    public static void maxMinTest(List<String> list){
        System.out.println("长度最大：" + list.stream().max((a,b)-> a.length()-b.length()));
        System.out.println("长度最小：" + list.stream().min((a,b)-> a.length()-b.length()));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        maxMinTest(list);
    }
}
```
执行结果为：
```text
长度最大：Optional[212121121]
长度最小：Optional[123]
```
### 4.7 count
count是无参方法，用于计数，返回流中元素个数。
```java
public class StreamTest {
    public static void countTest(List<String> list){
        System.out.println("元素个数为：" + list.stream().count());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        countTest(list);
    }
}
```
执行结果为：
```text
元素个数为：8
```
### 4.8 anyMatch
该方法需要一个Predicate参数，用于校验流中的元素，只要有一个满足规则，则返回true，全不满足，返回false。
```java
public class StreamTest {
    public static void anyMatchTest(List<String> list){
        System.out.println(list.stream().anyMatch(e -> e.length()>10));
        System.out.println(list.stream().anyMatch(e -> e.length()>8));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        anyMatchTest(list);
    }
}
```
执行结果为：
```text
false
true
```
### 4.9 allMatch
该方法同样需要一个Predicate参数，用于校验流中的所有元素，只有全部满足规则才能返回true，只要有一个不满足则返回false。
```java
public class StreamTest {
    public static void allMatchTest(List<String> list){
        System.out.println(list.stream().allMatch(e -> e.length()>1));
        System.out.println(list.stream().allMatch(e -> e.length()>3));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        allMatchTest(list);
    }
}
```
执行结果为：
```text
true
false
```
### 4.10 noneMatch
该方法同样需要一个Predicate参数，用于校验流中的所有元素,只有所有元素都不满足规则的情况下返回true，否则返回false。
```java
public class StreamTest {
    public static void noneMatchTest(List<String> list){
        System.out.println(list.stream().noneMatch(e -> e.length()>10));
        System.out.println(list.stream().noneMatch(e -> e.length()>8));
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        noneMatchTest(list);
    }
}
```
执行结果为：
```text
true
false
```
### 4.11 findFirst
该方法无参数，主要用于获取流中的第一个元素，如果流无序，那么可能返回任意一个。
```java
public class StreamTest {
    public static void findFirstTest(List<String> list){
        System.out.println(list.stream().parallel().findFirst().get());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        findFirstTest(list);
    }
}
```
执行结果为：
```text
123
```
### 4.12 findAny
该方法无参数，主要用于获取流中的任一元素。
```java
public class StreamTest {
    public static void findAnyTest(List<String> list){
        System.out.println(list.stream().parallel().findAny().get());
    }
    public static void main(String[] args) {
        List<String> list = Arrays.asList("123","456","789","1101","212121121","asdaa","3e3e3e","2321eew");
        findAnyTest(list);
    }
}
```
执行结果为：
```text
asdaa
```
## 五、总结
流式操作代码描述性强，易理解，而且功能强大，可以简化很多集合操作。在我们需要对集合数据进行处理的时候，不妨试试使用流式操作来实现。

参考：
- [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)