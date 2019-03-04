# Java设计模式系列-备忘录模式

## 一、概述
备忘录模式的目的是将目标实例中的值临时保存起来，保存的目的正是为了之后的恢复之用。

备忘录需要一个对象来临时保存目标实例中的值，要保证值的类型的对应。
## 二、实例
看下面的实例：

目标类：Target
```java
/**
 * 目标类
 */
public class Target {
    private String name;
    private Date date;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }
    public Target(String name, Date date){
        this.date = date;
        this.name = name;
    }
    public Ttemporary store(){
        return new Ttemporary(name, date);
    }
    public void back(Ttemporary ttemporary){
        this.name = ttemporary.getName();
        this.date = ttemporary.getDate();
    }
}
```
临时存储类：Ttemporary
```java
public class Ttemporary {
    private String name;
    private Date date;
    public Ttemporary(String name, Date date){
        this.date = date;
        this.name = name;
    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }
}
```
备忘录类：Storge
```java
/**
 * 备忘录类
 */
public class Storge {
    private Ttemporary ttemporary;
    public Storge(Ttemporary ttemporary){
        this.ttemporary = ttemporary;
    }

    public Ttemporary getTtemporary() {
        return ttemporary;
    }

    public void setTtemporary(Ttemporary ttemporary) {
        this.ttemporary = ttemporary;
    }
}
```
测试类：Clienter
```java
/**
 * 测试类
 */
public class Clienter {
    public static void main(String[] args) throws InterruptedException {
        Target target = new Target("huahua", new Date());
        System.out.println("保存之前的数据：name="+target.getName() + ",date="+ target.getDate());
        Storge storge = new Storge(target.store());
        Thread.sleep(1000L);
        target.setName("caocao");
        target.setDate(new Date());
        System.out.println("修改之后的数据：name="+target.getName() + ",date="+ target.getDate());
        target.back(storge.getTtemporary());
        System.out.println("恢复之后的数据：name="+target.getName() + ",date="+ target.getDate());
    }
}
```
执行结果为：
```text
保存之前的数据：name=huahua,date=Sun Mar 03 16:38:36 CST 2019
修改之后的数据：name=caocao,date=Sun Mar 03 16:38:37 CST 2019
恢复之后的数据：name=huahua,date=Sun Mar 03 16:38:36 CST 2019
```
## 三、解析

## 四、应用场景

## 五、总结