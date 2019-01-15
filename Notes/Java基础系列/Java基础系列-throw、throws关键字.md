# Java基础系列-throw、throws关键字
## 一、概述
throw和throws就是异常相关的关键字，在java中异常机制是一个非常重要的机制，我们需要重点掌握。

既然说到了异常，简单描述下异常机制很有必要，这也对后文的讲述提供前提。
## 二、Java异常机制
在Java中表示异常的接口是Exception，与其同一层次的还有一个Error接口，用于描述不可挽回的系统级错误，它们两个都继承自Throwable接口，这个接口是所有异常和错误的超接口。

在Java中只有Throwable的实例才能在虚拟机或者java代码中被抛出，一切想要抛出的异常都必须作为Throwable的子类。

异常又分为两种，一种是受检异常，另一种是未受检异常。

受检异常是Java内部定义的一系列异常类，它们都实现了Exception接口，这些异常必须被手动捕捉或者手动抛出，否则无法通过编译，处于强制处理异常，属于编译期异常。

未受检异常则是一些可以不进行捕捉的异常，这些异常一般是由运行时逻辑引发，这些异常可以不捕捉，也可以进行捕捉或抛出，如果未进行捕捉、抛出处理，那么一旦运行时引发了这些异常，那么会被JVM直接处理，它包括运行时异常和Error。

运行时异常是典型的未受检异常，Java内部为我们提供了一部分这类型异常，其中最常见的空指针异常就属此类。

其实正确的做法是尽可能的对异常进行捕捉处理。
## 三、throw作用
异常的抛出需要使用throw关键字，抛出的是一个异常对象，一般我们采用下面的方式进行抛出:
```java
public class ExceptionTest {
    public void throwTest (){
        throw new NullPointerException();
    }
}
```
再结合try...catch语句，或者if语句进行组合使用，将可能的异常抛出。

有时候我们使用try...catch语句的catch块中加入throw语句，可以将一个受检异常转化为一个未受检异常。
## 四、throws作用
当我们没有条件在当前的方法中进行某个异常的处理时，可以在方法声明处抛出，在这里抛出异常将被调用者捕捉处理，或者再次抛出，多个异常可用逗号隔开。
```java
public class ExceptionTest {
    public void throwTest () throws NumberFormatException,NoSuchElementException {
        throw new NullPointerException();
    }
}
```
----------------------------------------
以下内容来自（http://blog.csdn.net/luoweifu/article/details/10721543） 
## 五、异常习惯
1. 在写程序时，对可能会出现异常的部分通常要用try{...}catch{...}去捕捉它并对它进行处理；
2. 用try{...}catch{...}捕捉了异常之后一定要对在catch{...}中对其进行处理，那怕是最简单的一句输出语句，或栈输入e.printStackTrace();
3. 如果是捕捉IO输入输出流中的异常，一定要在try{...}catch{...}后加finally{...}把输入输出流关闭；
4. 如果在函数体内用throw抛出了某种异常，最好要在函数名中加throws抛异常声明，然后交给调用它的上层函数进行处理。

参考：
- [深入理解java异常处理机制](http://www.importnew.com/14688.html)
- [再探java基础——throw与throws](http://blog.csdn.net/luoweifu/article/details/10721543)