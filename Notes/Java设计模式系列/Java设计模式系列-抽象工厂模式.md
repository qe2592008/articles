# Java设计模式系列-抽象工厂模式
## 一、概述
抽象工厂模式是对工厂方法模式的再升级，但是二者面对的场景稍显差别。

工厂方法模式面对的目标一般都是单类的，就比如[《Java设计模式系列-工厂方法模式》]()中所举的例子，目标就是桌子这一类商品。

如果是这样的呢：生产的是桌椅组合，目标的一套商品，每一套商品中的每类商品的种类的不同的，不同的组合形成不同的套装。

这种情况下，就需要使用抽象工厂模式
## 二、实例
我们还以家具为例：

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
public class WoodenDesk implements Desk {
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
椅子接口：Chair
```java
/**
 * 椅子接口
 */
public interface Chair {
    String getType();
}
```
木质椅子：WoodenChair
```java
/**
 * 木质椅
 */
public class WoodenChair implements Chair {
    private String type = "木质椅";
    @Override
    public String getType() {
        return type;
    }
}
```
塑料椅：PlasticChair
```java
/**
 * 塑料椅
 */
public class PlasticChair implements Chair {
    private String type = "塑料椅";
    @Override
    public String getType() {
        return type;
    }
}
```
家具工厂接口：FurnitureFactory
```java
/**
 * 家具工厂
 */
public interface FurnitureFactory {
    Desk createDesk();
    Chair createChair();
}
```
木质家具工厂：WoodenFurnitureFactory
```java
/**
 * 木质家具工厂
 */
public class WoodenFurnitureFactory implements FurnitureFactory {
    @Override
    public Desk createDesk() {
        return new WoodenDesk();
    }

    @Override
    public Chair createChair() {
        return new WoodenChair();
    }
}
```
塑料家具工厂：PlasticFurnitureFactory
```java
/**
 * 塑料家具工厂
 */
public class PlasticFurnitureFactory implements FurnitureFactory {
    @Override
    public Desk createDesk() {
        return new PlasticDesk();
    }

    @Override
    public Chair createChair() {
        return new PlasticChair();
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
        FurnitureFactory factory = new PlasticFurnitureFactory();
        Desk desk = factory.createDesk();
        Chair chair = factory.createChair();
        System.out.println(desk.getType());
        System.out.println(chair.getType());
    }
}
```
执行结果：
```text
塑料桌
塑料椅
```
## 三、解析
通过上面的例子，对比[《Java设计模式系列-工厂方法模式》]()中工厂方法模式的例子，可以看出二者场景的不同之处，抽象工厂模式面对的是一个组合体，如果将这一点排除的话，其他方面看起来，二者还是相似的。

这里在目标没添加一种组合时，就需要新建一个工厂实现来对应，这一点满足开闭原则，不会修改已有类。

但是有一种情况，会导致修改原有类，那就是当目标需要在家具中新增一种家具类型的时候，比如例子中，家具组合中只包含桌子和椅子，如果再添加一种书柜，那么所有的工厂包括工厂接口都面临修改。
