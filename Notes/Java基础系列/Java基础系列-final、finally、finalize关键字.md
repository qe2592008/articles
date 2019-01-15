# Java基础系列-final、finally关键字
## 一、概述
final是Java关键字中最常见之一，表示“最终的，不可更改”之意，在Java中也正是这个意思。

有final修饰的内容，就会变得与众不同，它们会变成终极存在，其内容成为固定的存在。

finally关键字不同于final关键字，这是一个需要与异常体系结构配合使用的关键字，旨在定义必须要进行操作，一般用于在发生异常的时候进行一些收尾操作，比如释放资源等。

另外还有个finalized，它是一个方法，它需要与垃圾收集体系配合使用。主要在对象被垃圾收集之前进行一些操作，这些操作只会被执行一次，即使一个对象多次被标记为下次进行垃圾收集，也只有第一次会执行。
## 二、final作用
### 2.1 final修饰变量
变量被final修饰就会变成为常量，常量被保存在方法区中。

变量一旦被final修饰，必须手动进行初始化，未进行初始化的final常量是无法通过编译的。

如果只有final修饰的变量的初始化可以采用：
- 定义时赋值
- 代码块赋值
- 构造器赋值

如果被static和final同时修饰的变量的初始化可以采用：
- 定义时赋值
- 静态代码块赋值

一旦final变量被static修饰，那么它就脱离了对象的组织（代码块、构造器都是对象的组织），升级为类的组织，所以需要在类级别的静态代码块中进行初始化。
```java
public class FinalTest {
    final int i = 1;
    int j = 2;
    static int m = 3;
    static final int n = 4;
}
```
或
```java
public class FinalTest {
    final int i;
    int j;
    static int m;
    static final int n;
    {
        i = 1;
    }
    static {
        n = 3;
    }
}
```
如果将上面的代码改成：
```java
public class FinalTest {
    final int i;//2-
    int j;
    static int m;
    static final int n;//5-
}
```
上面代码第2行和第5行会报错，原因就是未进行初始化。

那么我们总结下final和static的现象，用于区分二者：
- **static修饰将内容脱离对象成为类成员。**
- **final修饰将内容改造成必须被手动初始化的成员，一旦赋值，不再改变。**

**注意**：同时被final和static修饰的变量成为静态常量，类常量，这种类常量在编译阶段会进行常量传播优化，将该类常量的值直接保存到调用类的常量池中，那么在程序执行到调用位置时，实际上与定义该类常量的类已经毫无关系，我（调用方）可以直接在我的常量池中获取到编译阶段优化过来的值，不再需要通过常量定义类的类型去调用其中定义的那个类常量了。

二者可以同时存在，各起各的作用。
### 2.2 final修饰方法
被final修饰的方法，可以被子类继承，但是不能被子类重写，也就是说这个方法在此以后其内部的实现就是固定不变的了，不能被改变。
### 2.3 final修饰类
被final修饰的类，被称之为最终类，其不再拥有子类，不可再进行扩展，最常见的final类就是String类。

String类被final修饰之后，其每个对象都是不变的，一旦定义就不再发生改变。
### 2.4 final修饰局部变量
final修饰的局部变量，该变量就不再是保存在栈空间中，而是保存在方法区中，不会随方法结束而失效，放大了局部变量的生命周期。

最常使用的地方就是局部内部类在访问方法的局部变量的情况下，这些局部变量就需要使用final修饰，因为当局部内部类访问局部变量时，会放大局部变量的作用域，局部变量一般在方法结束时就失效了，但是却有可能任然被内部类的对象持有使用。将该局部变量定义为final之后，它不再保存于栈空间，而是保存在方法区中，自然不会因为方法的结束而丢失。
```java
public class FinalTest{
    public void outMethod(){
       final int s = 1;// 3-
        class innerClass{
            public void innerMethod(){
                System.out.println(s);
            }
        }
    }
}
```
如果去掉第3行的final，第5行就会报错。
### 2.5 final修饰方法参数
如果方法的参数被final修饰，那么这个参数的值在从方法调用时赋值开始就不能再改变，不能被重新赋值（不能改成他值）。
```java
public class FinalTest{
    public void outMethod(final int s){
       s = 1;
    }
}
```
如上代码，方法参数s为final的，那么若去掉第2行的代码，为s重新赋值，则会报错。
## 三、finally作用
finally只有一种用法，那就是在try...catch..语句末尾使用。语法如下：
```java
public class FinallyTest{
    public void test(){
        try{
          //someExecute
        }catch(Exception e){
          //someExecute
        }finally{
          //someExecute
        }
    }
}
```
finally块中的语句是一定会被执行的，无论是否会发生异常，重点：这里的异常指的是在try块中的部分，如果实在try块之前发生了异常，还没来得及执行try块语句，那么finally块中的内容也不会被执行，所以finally针对的是try块中的内容而设的。

如果在try块或catch块中存在return语句，那么，finally块中的内容必然会在return之前执行。

finally经常用于发生异常的情况下关闭打开的资源，比如io流，网络资源等。
## 四、finalized作用
finalized是Object类的protected方法。

当垃圾回收器发现一个对象不存在任何引用的时候，就会触发该方法的调用，调用由垃圾回收器发起。

子类重写该方法一般用于处理系统资源或者一些清理工作。

该方法并不被确保一定会调用，但是可以保证的是，一旦被调用，调用的线程并不会持有任何的同步锁，而且如果执行发生了异常，则忽略异常，同时停止执行。
```java
class FinalizedTest{
    @Override
    public void finalize(){
        // do something
    }
}
```
简述finalized执行流程：

当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象“复活”。（摘自参考文章）

参考：
- [java finalize方法总结、GC执行finalize的过程](https://www.cnblogs.com/Smina/p/7189427.html)