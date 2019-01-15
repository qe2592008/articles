# Java设计模式系列-外观模式
## 一、概述
外观模式，一般用在子系统与访问者之间，用于对访问者屏蔽复杂的子系统调用，采用耳目一新的外观类提供的简单的调用方法，具体的实现由外观类去子系统调用。
  
外观模式任然是一种中间件类型的模式，使用外观模式之后子系统的方法调用并非完全屏蔽，只是为访问者提供了一种更佳的访问方式，如果你不嫌麻烦，任然可以直接进行子系统方法调用。
  
甚至于在子系统与子系统之间进行调用时也可以通过各自的外观类来进行调用，这样代码方便管理。
  
下面请看代码实例：更能显示这种情况
## 二、示例
子系统方法1
```java
public class SubMethod1 {
    public void method1(){
        System.out.println("子系统中类1的方法1");
    }
}
```
子系统方法2
```java
public class SubMethod2 {
    public void method2(){
        System.out.println("子系统中类2方法2");
    }
}
```
子系统方法3
```java
public class SubMethod3 {
    public void method3(){
        System.out.println("子系统类3方法3");
    }
}
```
外观类：
```java
public class Facader {
    private SubMethod1 sm1 = new SubMethod1();
    private SubMethod2 sm2 = new SubMethod2();
    private SubMethod3 sm3 = new SubMethod3();
    public void facMethod1(){
        sm1.method1();
        sm2.method2();
    }
    public void facMethod2(){
        sm2.method2();
        sm3.method3();
        sm1.method1();
    }
}
```
测试类：
```java
public class Clienter {
    public static void main(String[] args) {
        Facader face = new Facader();
        face.facMethod1();
//        face.facMethod2();
    }
}
```
其实直接调用也会得到相同的结果，但是采用外观模式能规范代码，外观类就是子系统对外的一个总接口，我们要访问子系统是，直接去子系统对应的外观类进行访问即可！
## 三、外观模式应用场景
当我们访问的子系统拥有复杂额结构，内部调用繁杂，初接触者根本无从下手时，不凡由资深者为这个子系统设计一个外观类来供访问者使用，统一访问路径（集中到外观类中），将繁杂的调用结合起来形成一个总调用写到外观类中，之后访问者不用再繁杂的方法中寻找需要的方法进行调用，直接在外观类中找对应的方法进行调用即可。
  
还有就是在系统与系统之间发生调用时，也可以为被调用子系统设计外观类，这样方便调用也，屏蔽了系统的复杂性。