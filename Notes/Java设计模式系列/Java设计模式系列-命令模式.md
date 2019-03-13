# Java设计模式系列-命令模式（有问题）
## 一、概述
命令模式是行为型模式中的一种，涉及到三方，包括命令方、命令发起方、命令接收方。

命令模式中发起方和接收方是固定的（类），命令却可以自有扩展，也就是说命令是可以有多种的，那么命令必然需要抽象。
## 二、实例
看下面的实例：

命令接收方：Receiver
```java
/**
 * 命令接收方
 */
public class Receiver {
    public Receiver(String name){
        this.name = name;
    }
    private String name;
    public void toDo(String message){
        System.out.println(name+"执行命令："+message);
    }
}
```
命令抽象层：Command
```java
/**
 * 命令抽象层
 */
public abstract class Command {
    private Receiver receiver;
    protected String message;
    public Command(Receiver receiver){
        this.receiver = receiver;
    }
    public void execute(){
        receiver.toDo(message);
    }
}
```
具体的命令：MyCommand
```java
/**
 * 具体的命令实现
 */
public class MyCommand extends Command {
    public MyCommand(Receiver receiver, String message) {
        super(receiver);
        this.message = message;
    }
}
```
命令发起方：Invoker
```java
/**
 * 发令方
 */
public class Invoker {
    private Command command;
    public Invoker(Command command){
        this.command = command;
    }
    public void order(){
        command.execute();
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
        Receiver receiver = new Receiver("董存瑞");
        Command command = new MyCommand(receiver, "去把那个碉堡给炸了");
        Invoker invoker = new Invoker(command);
        invoker.order();
    }
}
```
执行结果：
```text
董存瑞执行命令：去把那个碉堡给炸了
```
## 三、解析
从上面的实例中我们能看出来三方是通过组合的方式来发起调用的，Invoker中持有Command，Command中持有Receiver，然后用持有的实例调用目标方法来完成命令的传递。

这里命令抽象化之后，可以自有扩展，定义各种类型的命令，主要是将命令的发送方与接收方进行了解耦，实现请求和执行分开。

## 四、使用场景
