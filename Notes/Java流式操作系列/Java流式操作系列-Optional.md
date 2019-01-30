# Java流式操作系列-Optional
## 一、概述
Optional的引入是为了解决null的问题，那么到底是解决null的什么问题呢？

我们知道当我们针对null调用方法的之后，就会抛出空指针异常，Optional就是为了解决这个问题而来的。

Optional通过封装目标对象的方式来表示，当我们使用的时候，Optional是必然存在的，因为如果结果为null，会返回一个固定的EMPTY实例，这样就不会存在null引用的问题了。

那该如何来使用Optional呢？

我们不能滥用Optional，只有在逻辑上（业务上）可能为null时才使用Optional来封装目标对象，如果逻辑上（业务上）肯定不可能为null时，就不要使用Optional封装对象，这样当阅读代码的人看到有Optional则表示目标对象是可以为null的。

这样之后，如果程序中又出现了null指针异常，那么只能是你的代码逻辑有误，而不可能是非逻辑原因了，也就是不该为null的时候出现了null，这肯定是你逻辑搞错了。
## 二、Optional
### 2.1 创建Optional实例
#### 2.1.1 创建一个空的Optional
```java
public class OptionalTest {
    public static void main(String[] args) {
        Optional<String> o = Optional.empty();// 创建一个空Optional
    }
}
```
#### 2.1.2 创建一个有值的Optional
```java
public class OptionalTest {
    public static void main(String[] args) {
        Optional<String> op = Optional.of("123");// 创建一个目标对象必须有值的Optional
    }
}
```
这种创建方式，如果of的参数为null，则直接抛出异常。
#### 2.1.3 创建一个可为空的Optional
```java
public class OptionalTest {
    public static void main(String[] args) {
        Optional<String> opt = Optional.ofNullable(null);// 创建一个目标对象可为null的Optional
    }
}
```
### 2.2 中间操作-处理Optional
#### 2.2.1 filter
这个filter是一个校验器，由于Optional中也只有一个元素，称其为过滤有点太大了，如果Optional中的元素满足给定的校验条件，则将封装元素的Optional返回，否则返回空的Optional。
```java
public class OptionalTest {
    public static void main(String[] args) {
        filterTest();
    }
    public static void filterTest(){
        Optional<String> os = Optional.of("123456").filter(e -> e.length()>7);
        System.out.println(os.isPresent());
    }
}
```
执行结果：
```text
false
```
因为字符串“123456”长度小于7，所以返回一个空的Optional。
#### 2.2.2 map
map是映射之意，就是针对Optional封装的元素进行操作，然后返回一个封装着操作结果的新的Optional。
```java
public class OptionalTest {
    public static void main(String[] args) {
        mapTest();
    }
    public static void mapTest(){
        Optional<Integer> oi = Optional.of("abcdefg").map(e -> e.length());
        System.out.println(oi.get());
    }
}
```
执行结果：
```text
7
```
#### 2.2.3 flatMap
这个方法是扁平化映射，所谓扁平就是处理嵌套的Optional。

比如有一个Person类，其中有一个Optional<String>类型的字段name，我们在别处获取到了一个Optional<Person>类型的实例，现在想要获取到这个Person的的姓名，这就是嵌套的Optional。
```java
class Person{
    Optional<String> name;
    public Person(String name){
        this.name = Optional.of(name);
    }

    public Optional<String> getName(){
        return name;
    }
}
```
如果我们使用map获取到的是如下的：
```java
public class OptionalTest {
    public static void main(String[] args) {
        flatMapTest();
    }
    public static void flatMapTest(){
        Optional<Person> op = Optional.of(new Person("huahua"));
        Optional<Optional<String>> oos = op.map(Person::getName);
        String name = oos.get().orElseGet(()->"noName");
        System.out.println(name);
    }
}
```
而我们使用flatMap的话就可以是这样的：
```java
public class OptionalTest {
    public static void main(String[] args) {
        flatMapTest();
    }
    public static void flatMapTest(){
        Optional<Person> op = Optional.of(new Person("huahua"));
        Optional<String> os = op.flatMap(Person::getName);
        String name = os.orElseGet(()->"noName");
        System.out.println(name);
    }
}
```
嵌套的Optional要获取到目标对象，必须通过多次去壳才行，这也是使用map映射的原理，但是使用flatMap，扁平化功能，一次去壳操作就完成了。
### 2.3 从Optional中获取目标值
有四种方式：
- public T get()：直接获取Optional中的值，如果是空的要抛出NoSuchElementException异常
- public T orElse(T other)：如果Optional不为空，则将值返回，否则返回指定的other(默认值)
- public T orElseGet(Supplier<? extends T> other)：如果Optional不为空，则返回值，否则将通过指定方式other生成的值返回
- public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X：如果Optional不为空，返回值，否则抛出由指定的exceptionSupplier生成的异常
```java
public class OptionalTest {
    public static void main(String[] args) {
        getTest();
        orElseTest();
    }
    public static void getTest(){
        Optional<String> os = Optional.of("123456");
        System.out.println(os.get());
    }
    public static void orElseTest(){
        Optional<String> os = Optional.empty();
        System.out.println(os.orElse("default"));
        System.out.println(os.orElseGet(()->"default"));
        System.out.println(os.orElseThrow(RuntimeException::new));
    }
}
```
执行结果：
```text
123456
default
default
Exception in thread "main" java.lang.RuntimeException
	at java.util.Optional.orElseThrow(Optional.java:290)
	at com.dh.stream.OptionalTest.orElseTest(OptionalTest.java:29)
	at com.dh.stream.OptionalTest.main(OptionalTest.java:17)
```
## 三、使用方式
Optional使用时不能直接get，这时会走进老路，get的时候可能为null，就会抛出异常，这时候你就会想要在get之前进行isPresent判断，这个不使用Optional又有什么区别的。

我们使用Optional就是为了简化null判断，所以拒绝使用get方法，Optional提供了正确使用的方法是：orElse、orElseGet、orElseThrow三个方法。

使用orElse方法，我们可以从非空的Optional中获取到值，如果是空的Optional，可以返回orElse方法参数指定的默认值。

使用orElseGet方法，我们可以在空的Optional的情况下主动构建一个默认返回结果。

orElseThrow方法，在空Optional的情况下会抛出一个指定的异常。
## 四、总结
说了这么多，最终是为了在开发中使用Optional，正如开始时说的，我们要明确Optional规避的是那种null，不能在所有的地方都使用它。

当项目的业务规则下某个对象可能为null（就是业务允许的null），这种情况下，出现null是正常现象，这种情况需要规避，我们使用Optional封装目标对象，保证不会存在null调用抛出空指针。

但是如果在业务规则下某个对象不可能为null（就是业务不允许为null），这种情况下，出现null就是程序出错了，并不是正常现象，这种时候我们不能用Optional去封装目标对象来规避问题，而是要直接使用，一旦出错就可以及时的排查问题，不至于被Optional将问题给隐藏掉。
## 五、扩展
Java在提供Optional的基础上还额外提供了三个专用的Optional:
- OptionalInt
- OptionalLong
- OptionalDouble

这三个类是专业性扩展，OptionalInt内部只能封装Integer类型的值和null，OptionalLong内部只能封装Long类型的值和null，OptionalDouble内部只能封装Double类型的值和null。
