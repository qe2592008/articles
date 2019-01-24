# Java基础系列-浅拷贝和深拷贝
## 一、概述
Java中的拷贝功能是由Object类的clone方法定义的。
```java
public class Object{
    //...
    protected native Object clone() throws CloneNotSupportedException;
    //...
}
```
这是一个可被重写的本地方法。

这个方法是与Cloneable接口相关联的，如果针对一个没有实现Cloneable接口的方法执行器clone方法，会抛出CloneNotSupportedException异常。
## 二、Cloneable
Cloneable接口就是一个标记接口，表示实现了该接口的类具备了可以克隆的功能。

一般来说，如果一个类实现了该接口，那么同时需要重写Object中的clone方法。

> 这里要谨记一点：clone方法是Object中定义的方法，而不是Cloneable中定义的，后者只是一个标记接口，其中没有任何方法定义。

## 三、clone()
当一个类实现了Cloneable之后，具备了可被克隆的基础条件，下面就需要具体完善克隆工作的细节了。

细节当然需要依靠重写clone方法来实现。

到这里就涉及到了本文的重点：浅拷贝与深拷贝。

简单说下二者的区别：

浅拷贝就是克隆出来的新对象与原对象完全一致无二，包括引用类型字段的值。这就会有一个现象，针对引用类型的字段，两个对象的引用地址一致，如此一来新旧对象之间强关联，修改其中一个对象的内容，极可能影响到另外一个的内容。

深拷贝是针对浅拷贝而言的，那就是将所有引用的对象同样做深拷贝，这样一来，新旧对象就不再强关联，当我们修改其中一个对象时，不会影响到另外一个。

### 3.1 浅拷贝
实现浅拷贝非常简单，直接调用Object中定义的clone方法即可。也即，Object中定义的clone方法实现的是浅拷贝功能。

让我们来看看下面的例子：
```java
class A {
    int i = 20;
    A(){}
    A(int i){
        this.i = i;
    }
}
class B implements Cloneable{
    private int i = 10;
    private A a = new A(11);
    public Object clone () {
        B b = null;
        try {
            b = (B)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return b;
    }
    public A getA() {
        return a;
    }
    public void setA(A a) {
        this.a = a;
    }
}
public class CloneTest {
    public static void main(String[] args) {
        B b1 = new B();
       // 执行浅拷贝
       System.out.println("执行浅拷贝：");
       B cb1 = (B)b.clone();
       System.out.println(cb1.equals(b));// false-表示clone的对象与原对象不同
       System.out.println((cb1.getA()).equals(b.getA()));// true-表示浅拷贝引用对象不变
    }
}
```
执行结果为：
```text
执行浅拷贝：
false
true
```
通过实例我们可以看出，要写实现浅拷贝需要如下几步：
- 被拷贝对象的类实现Cloneable接口
- 被拷贝对象的类重写Object的clone方法
- 执行拷贝操作，就是调用重写的clone方法来完成克隆

从实例的执行结果中可以看出新的clone对象实例cb1与原对象实例b1，之间equals不等，这里的equals比较的是两个对象的地址，与“==”效果一致。而cb1的a字段指向的A实例与b1的a字段指向的A实例却是一样的，这就是浅拷贝。

浅拷贝是浅层拷贝，只在意目标对象一个，与其关联的其他对象一概不管。

但是在实际使用中，使用浅拷贝的情况很少，多数情况下，我们需要的是一个完全崭新的对象，包括它的字段引用，那么我们就需要实现深拷贝了。
### 3.2 深拷贝
所谓深拷贝就是相对浅拷贝而言的，在浅拷贝的基础上进一步进行深层拷贝即可，那么具体实现呢？
```java
class A implements Cloneable{
    int i = 20;
    A(){}
    A(int i){
        this.i = i;
    }
    public Object clone (){
        A a = null;
        try {
            a = (A)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return a;
    }
}
class B implements Cloneable{
    private int i = 10;
    private A a = new A(11);
    public Object clone () {
        B b = null;
        try {
            b = (B)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return b;
    }
    public A getA() {
        return a;
    }
    public void setA(A a) {
        this.a = a;
    }
}
public class CloneTest {
    public static void main(String[] args) {
        B b = new B();
        // 执行浅拷贝
        System.out.println("执行浅拷贝：");
        B cb1 = (B)b.clone();
        System.out.println(cb1.equals(b));// false-表示clone的对象与原对象不同
        System.out.println((cb1.getA()).equals(b.getA()));// true-表示浅拷贝引用对象不变
        // 执行深拷贝
        System.out.println("----------------");
        System.out.println("执行深拷贝：");
        B cb2 = (B)b.clone();
        A ca = (A)b.getA().clone();
        cb2.setA(ca);
        System.out.println(cb2.equals(b));// false-表示clone的对象与原对象不同
        System.out.println((cb2.getA()).equals(b.getA()));// true-表示浅拷贝引用对象不变
    }
}
```
执行结果：
```text
执行浅拷贝：
false
true
----------------
执行深拷贝：
false
false
```
通过实例我们可以清楚看出，要想实现深拷贝需要如下几步：
- 被拷贝的目标对象的类的依赖对象的类A需要实现Cloneable接口
- 类A需要重写Object的clone方法
- 在b拷贝完成后，要继续执行b中被依赖对象a的拷贝，并将拷贝结果赋值给b的拷贝对象。

可见深拷贝其实就是一个递归拷贝的过程，将目标对象的类依赖的所有对象的类全部实现Cloneable接口并重写clone方法进行二层拷贝，然后是依赖对象的依赖对象...层层深入拷贝。

## 四、注意事项
Java中提供的类中存在一些无法被clone的类，比如StringBuffer，它既没有实现Cloneable接口，也没有重写clone方法，那么它就无法执行克隆操作，那么遇到它该怎么办呢？

这里提供一种办法，既然是新对象，那么直接创建一个新的StringBuffer对象，将原来的值赋值进来，就等于是完成了克隆，虽然这种方式要比Object中的clone方法慢许多，但不失为一种补救措施。

还有一些类型如基本类型的装箱类型、String，它们虽然也是引用类型，但是它们是final修饰的，且没有实现Cloneable接口，没有重写clone方法，要克隆也需要向如上的方式来实现。

参考：
- [java Clone使用方法详解](https://www.cnblogs.com/felixzh/p/6021886.html)