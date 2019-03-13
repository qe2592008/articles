# Java设计模式系列-工厂方法模式
## 一、概述
工厂，就是生产产品的地方。

在Java设计模式中使用工厂的概念，那就是生成对象的地方了。

本来直接就能创建的对象为何要增加一个工厂类呢？

这就需要了解工厂方法要解决的是什么问题了，如果只有一个类，我们直接new一个对象完事，这是最简单的；但是如果有多个类呢，而且这些类还需要针对不同的情况来创建不同的对象，这时候就需要工厂了，我们可以在工厂中根据条件来创建具体的对象。

这样一来就将调用方和具体的目标类进行了解耦，调用方根本就不知道需要创建那个对象，它只是提出了条件，然后工厂就可以根据给定的条件来决定创建哪一个对象。
## 二、简单工厂方法模式
要说工厂方法模式，不得不先了解下简单工程方法模式，这个模式并不是23种设计模式中的内容。

所谓简单工厂方法模式，就是为目标类创建一个工厂，当有多个目标实现的时候，在这个工厂内部进行逻辑判断来根据条件创建不同的目标实例。

下面看个例子，我就以桌子为例来写：

桌子接口：Desk
```java
/**
 * 桌子接口
 */
public interface Desk {
    String getType();
}
```
木质桌子：WoodenDesk
```java
/**
 * 木质桌子
 */
public class WoodenDesk implements Desk{
    private String type = "木质桌";
    @Override
    public String getType() {
        return type;
    }
}
```
塑料桌子：PlasticDesk
```java
/**
 * 塑料桌
 */
public class PlasticDesk implements Desk {
    private String type = "塑料桌";
    @Override
    public String getType() {
        return type;
    }
}
```
类型枚举：Type
```java
/**
 * 类型
 */
public enum Type {
    PLASTIC,WOODEN;
}
```
桌子工厂：DeskFactory
```java
/**
 * 桌子工厂
 */
public class DeskFactory {
    public static Desk createDesk(Type type) {
        switch (type) {
            case WOODEN:
                return new WoodenDesk();
            case PLASTIC:
                return new PlasticDesk();
            default:
                return null;
        }
    }
}
```
测试类：Clienter
```java
/**
 * 测试类
 */
public class Clineter {
    public static void main(String[] args) {
        Desk desk = DeskFactory.createDesk(Type.PLASTIC);
        System.out.println(desk.getType());
    }
}
```
执行结果
```text
塑料桌
```
> 这就是简单工厂方法，只有一个工厂类来面向多个目标实现。当目标实现增多时，我们不得不去修改工厂类的方法，使其兼容新的实现类型，这明显违背了开闭原则，所以出现了工厂方法模式。
## 三、工厂方法模式
工厂方法模式是对简单工厂模式的抽象升级，将工厂这个概念抽象出来成为接口，然后针对每种目标实现类创建一个工厂实现，一对一来实现，当新增了目标实现，只要同时新增一个工厂实现即可。

下面看看实例：

桌子接口：Desk
```java
/**
 * 桌子接口
 */
public interface Desk {
    String getType();
}
```
木质桌子：WoodenDesk
```java
/**
 * 木质桌子
 */
public class WoodenDesk implements Desk{
    private String type = "木质桌";
    @Override
    public String getType() {
        return type;
    }
}
```
塑料桌子：PlasticDesk
```java
/**
 * 塑料桌
 */
public class PlasticDesk implements Desk {
    private String type = "塑料桌";
    @Override
    public String getType() {
        return type;
    }
}
```
桌子工厂接口：DeskFactory
```java
/**
 * 桌子工厂接口
 */
public interface DeskFactory {
    Desk createDesk();
}
```
木质桌子工厂：WoodenDeskFactory
```java
/**
 * 木质桌子工厂
 */
public class WoodenDeskFactory implements DeskFactory{
    @Override
    public Desk createDesk(){
        return new WoodenDesk();
    }
}
```
塑料桌子工厂：
```java
/**
 * 塑料桌子工厂
 */
public class PlasticDeskFactory implements DeskFactory {
    @Override
    public Desk createDesk() {
        return new PlasticDesk();
    }
}
```
测试类：Clienter
```java
/**
 * 测试类
 */
public class Clienter {
    public static void main(String[] args) {
        DeskFactory factory = new WoodenDeskFactory();
        Desk desk = factory.createDesk();
        System.out.println(desk.getType());
    }
}
```
执行结果：
```text
木质桌
```
## 四、解析
从上面的实例中可以很容易看出来，工厂方法模式的重点就在这个工厂接口了。

目标可以无限扩展，工厂类也要随之扩展，一对一存在，满足了开闭原则，但如果目标实现较多，工厂实现类也会增多，不简洁。

就我实际情况下看到的，在MyBatis中使用的比较多：事务模块和数据源模块都使用了工厂方法模式。