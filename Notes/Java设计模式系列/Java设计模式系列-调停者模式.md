# Java设计模式系列-调停者模式
## 一、概述
调停者模式。
  
我们想象一下这样的场景：一个系统内部通过许多的类互相之间相互调用来完成一系列的功能，这个系统内部的每个类都会存在至少一次的调用与被调用，多者数不胜数，这种情况下，一旦某个类发生问题，进行修改，无疑会影响到所有调用它的类，甚至它调用的类，可见这种情况下，类与类之间的耦合性极高（体现为太多的复杂的直接引用）。
  
这正是调停者模式的主场，调停者犹如第三方中介一般，将所有的类与类之间的引用都导向调停者类，所有类的请求，一致发向调停者，由调停者再发向目标类，这样原本复杂的网状的类关系，变成了简单的星型类关系，调停者类位于核心，所有其他类位于外围，指向调停者。如此这般，类与类之间的直接调用耦合被解除（通过统一的第三方来发起调用），某个类发生问题，发生修改，也只会影响调停者，而不会直接影响到简介发起调用的那些类。
## 二、示例
下面举个生活中的实例：一个公司部门，有一个经理来充当调停者，其下的员工充当互相作用的类，这是一个很形象的实例。如果所有职员之间的互动都由职工之间直接进行，一旦某个员工不在，那么必须由此员工操作的事情便无法互动起来，或者某个员工被更换，员工之间不熟悉，也无法进行互动，这样，经理这个调停者的作用就来了，发起需求的员工将需求告诉经理，经理再找其他员工操作这个需求，明显的调停者模式。
  
下面看看示例代码：
  
调停者接口：Mediator
```java
/**
 * 调停者接口
 */
public interface Mediator {
    void change(String message,ZhiYuan zhiyuan,String nname);
}
```
职工抽象类：ZhiYuan
```java
/**
 * 职员接口
 */
public abstract class ZhiYuan {
    String name;
    private Mediator mediator;
    public ZhiYuan(Mediator mediator,String name){
        this.mediator = mediator;
        this.name = name;
    }
    //被调停者调用的方法
    public void called(String message,String nname){
        System.out.println(name + "接收到来自"+ nname + "的需求：" + message);
    }
    //调用调停者
    public void call(String message,ZhiYuan zhiyuan,String nname){
        System.out.println(nname + "发起需求："+ message);
        mediator.change(message,zhiyuan,nname);
    }
}
```
具体的调停者：Jingli
```java
/**
 * 调停者：经理
 */
public class Jingli implements Mediator {
    @Override
    public void change(String message,ZhiYuan zhiyuan,String nname) {
        System.out.println("经理收到" + nname + "的需求：" + message);
        System.out.println("经理将" + nname + "的需求发送给目标职员");
        zhiyuan.called(message,nname);
    }
}
```
具体的职员：ZhiyuanA、ZhiyuanB、ZhiyuanC
```java
/**
 * 职员A
 */
public class ZhiyuanA extends ZhiYuan {
    public ZhiyuanA(Mediator mediator, String name) {
        super(mediator, name);
    }
}

/**
 * 职员B
 */
public class ZhiyuanB extends ZhiYuan {
    public ZhiyuanB(Mediator mediator, String name) {
        super(mediator, name);
    }
}

/**
 * 职员C
 */
public class ZhiyuanC extends ZhiYuan {
    public ZhiyuanC(Mediator mediator, String name) {
        super(mediator, name);
    }
}
```
测试类：Clienter
```java
public class Clienter {
    public static void main(String[] args) {
        //分配职员与经理
        Mediator jingli = new Jingli();
        ZhiYuan zhiyuanA = new ZhiyuanA(jingli,"职员A");
        ZhiYuan zhiyuanB = new ZhiyuanB(jingli,"职员B");
        ZhiYuan zhiyuanC = new ZhiyuanC(jingli,"职员C");
        //职员A的需求
        String messageA = "这些资料需要B职员操作";
        zhiyuanA.call(messageA,zhiyuanB,zhiyuanA.name);
        //职员C的请求
        String messageC = "这些资料需要B职员签名";
        zhiyuanC.call(messageC, zhiyuanB,zhiyuanC.name);
    }
}
```
执行结果：
```text
职员A发起需求：这些资料需要B职员操作
经理收到职员A的需求：这些资料需要B职员操作
经理将职员A的需求发送给目标职员
职员B接收到来自职员A的需求：这些资料需要B职员操作
职员C发起需求：这些资料需要B职员签名
经理收到职员C的需求：这些资料需要B职员签名
经理将职员C的需求发送给目标职员
职员B接收到来自职员C的需求：这些资料需要B职员签名
```
如上所列，职工A和职工C都需要请求职工B，但是假如他们不认识职工B，那么就将工作需求提交给经理，经理再将工作需求发送给职工B。
## 三、模式解析
使用调停者模式貌似要比原本的结构消耗时间，但是却将需求的发起者与执行者之间的强耦合进行了降低，极大的优化了系统内部的维护工作。
  
调停者模式降低的是系统内部的耦合性，而外观模式降低的是系统之间的耦合性。
  
调停者模式更加细化，针对的是系统内部类与类之间的强耦合的解除，外观模式则较为统筹，针对的是整个系统对外的耦合性解除，二者都都有屏蔽复杂性的作用。
