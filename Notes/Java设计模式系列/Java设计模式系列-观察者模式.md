# Java设计模式系列-观察者模式
## 一、概述
观察者模式，又可以称之为发布-订阅模式，观察者，顾名思义，就是一个监听者，类似监听器的存在，一旦被观察/监听的目标发生的情况，就会被监听者发现，这么想来目标发生情况到观察者知道情况，其实是由目标将情况发送到观察者的。
  
观察者模式多用于实现订阅功能的场景，例如微博的订阅，当我们订阅了某个人的微博账号，当这个人发布了新的消息，就会通知我们。
  
现在我们举一个类似的情况，并使用代码来实现，为大家提供一个比较明显的认识。
## 二、示例
警察在找到嫌犯的时候，为了找到幕后主使，一般都会蹲点监察，这里我有三名便衣警察来蹲点监察2名嫌犯，三名便衣分别是：张昊天、石破天、赵日天，两名嫌犯是：大熊与黑狗，详见代码：
  
观察者接口：Observer
```java
public interface Observer {
    void update(String message,String name);
}
```
定义三名便衣观察者：Bianyi1、Bianyi2、Bianyi3
```java
/**
 * 便衣警察张昊天
 */
public class Bianyi1 implements Observer {
    //定义姓名
    private String bname = "张昊天";
    @Override
    public void update(String message,String name) {
        System.out.println(bname+":"+name+"那里有新情况："+ message);
    }
}

/**
 * 便衣警察石破天
 */
public class Bianyi2 implements Observer {
    //定义姓名
    private String bname = "石破天";
    @Override
    public void update(String message,String name) {
        System.out.println(bname+":"+name+"那里有新情况："+ message);
    }
}

/**
 * 便衣警察赵日天
 */
public class Bianyi3 implements Observer {
    //定义姓名
    private String bname = "赵日天";
    @Override
    public void update(String message,String name) {
        System.out.println(bname+":"+name+"那里有新情况："+ message);
    }
}
```
目标接口：Huairen
```java
public interface Huairen {
    //添加便衣观察者
    void addObserver(Observer observer);
    //移除便衣观察者
    void removeObserver(Observer observer);
    //通知观察者
    void notice(String message);
}
```
定义两个嫌疑犯：XianFan1、XianFan2
```java
import java.util.*;
/**
 * 嫌犯大熊
 */
public class XianFan1 implements Huairen {
    //别称
    private String name = "大熊";
    //定义观察者集合
    private List<Observer> observerList = new ArrayList<Observer>();
    //增加观察者
    @Override
    public void addObserver(Observer observer) {
        if(!observerList.contains(observer)){
            observerList.add(observer);
        }
    }
    //移除观察者
    @Override
    public void removeObserver(Observer observer) {
        if(observerList.contains(observer)){
            observerList.remove(observer);
        }
    }
    //通知观察者
    @Override
    public void notice(String message) {
        for(Observer observer:observerList){
            observer.update(message,name);
        }
    }
}

import java.util.*;
/**
 * 嫌犯黑狗
 */
public class XianFan2 implements Huairen {
    //别称
    private String name = "黑狗";
    //定义观察者集合
    private List<Observer> observerList = new ArrayList<Observer>();
    //增加观察者
    @Override
    public void addObserver(Observer observer) {
        if(!observerList.contains(observer)){
            observerList.add(observer);
        }
    }
    //移除观察者
    @Override
    public void removeObserver(Observer observer) {
        if(observerList.contains(observer)){
            observerList.remove(observer);
        }
    }
    //通知观察者
    @Override
    public void notice(String message) {
        for(Observer observer:observerList){
            observer.update(message,name);
        }
    }
}
```
测试类：Clienter
```java
public class Clienter {
    public static void main(String[] args) {
        //定义两个嫌犯
        Huairen xf1 = new XianFan1();
        Huairen xf2 = new XianFan2();
        //定义三个观察便衣警察
        Observer o1 = new Bianyi1();
        Observer o2 = new Bianyi2();
        Observer o3 = new Bianyi3();
        //为嫌犯增加观察便衣
        xf1.addObserver(o1);
        xf1.addObserver(o2);
        xf2.addObserver(o1);
        xf2.addObserver(o3);
        //定义嫌犯1的情况
        String message1 = "又卖了一批货";
        String message2 = "老大要下来视察了";
        xf1.notice(message1);
        xf2.notice(message2);
    }
}
```
测试结果：
```text
张昊天:大熊那里有新情况：又卖了一批货
石破天:大熊那里有新情况：又卖了一批货
张昊天:黑狗那里有新情况：老大要下来视察了
包拯:黑狗那里有新情况：老大要下来视察了
```
## 三、模式解析
通过上面的实例可以很明显的看出，观察者模式的大概模型，关键是什么呢？
  
关键点：  
- **针对观察者与被观察者分别定义接口，有利于分别进行扩展。**
- **重点就在被观察者的实现中：**
    - **定义观察者集合，并定义针对集合的添加、删除操作，用于增加、删除订阅者（观察者）**
    - **定义通知方法，用于将新情况通知给观察者用户（订阅者用户）**
- **观察者中需要有个接收被观察者通知的方法。**  

如此而已！
  
观察者模式定义的是一对多的依赖关系，一个被观察者可以拥有多个观察者，并且通过接口对观察者与被观察者进行逻辑解耦，降低二者的直接耦合。
  
如此这般，想了一番之后，突然发现这种模式与桥接模式有点类似的感觉。
  
桥接模式也是拥有双方，同样是使用接口（抽象类）的方式进行解耦，使双方能够无限扩展而互不影响，其实二者还是有者明显的区别：  
1. 主要就是使用场景不同，桥接模式主要用于实现抽象与实现的解耦，主要目的也正是如此，为了双方的自由扩展而进行解耦，这是一种多对多的场景。观察者模式侧重于另一方面的解耦，侧重于监听方面，侧重于一对多的情况，侧重于一方发生情况，多方能获得这个情况的场景。
2. 另一方面就是编码方面的不同，在观察者模式中存在许多独有的内容，如观察者集合的操作，通知的发送与接收，而在桥接模式中只是简单的接口引用。