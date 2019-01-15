# Java设计模式系列-模板模式
## 一、概述
模板模式，顾名思义，就是通过模板拓印的方式。
  
定义模板，就是定义框架、结构、原型。定义一个我们共同遵守的约定。
  
定义了模板，我们的剩余工作就是对其进行充实、丰润，完善它的不足之处。
  
定义模板采用抽象类来定义，公共的结构化逻辑需要在抽象类中完成，只将非公共的部分逻辑抽象成抽象方法，留待子类充实实现。
  
所以上文所述不足之处就是这些抽象方法。
  
总的来说，模板模式就是通过抽象类来定义一个逻辑模板，逻辑框架、逻辑原型，然后将无法决定的部分抽象成抽象类交由子类来实现，一般这些抽象类的调用逻辑还是在抽象类中完成的。这么看来，模板就是定义一个框架，比如盖房子，我们定义一个模板：房子要封闭，有门，有窗等等，但是要什么样的门，什么样的窗，这些并不在模板中描述，这个交给子类来完善，比如门使用防盗门，窗使用北向的窗等等。
## 二、示例
我们不凡就以建房为例来见识一下模板模式如何：
  
模板抽象类：HouseTemplate
```java
public abstract class HouseTemplate {

    protected HouseTemplate(String name){
        this.name = name;
    }

    protected String name;

    protected abstract void buildDoor();

    protected abstract void buildWindow();

    protected abstract void buildWall();

    protected abstract void buildBase();

    //公共逻辑
    public final void buildHouse(){

        buildBase();
        buildWall();
        buildDoor();
        buildWindow();

    }

}
```
子类1：HouseOne
```java
public class HouseOne extends HouseTemplate {

    HouseOne(String name){
        super(name);
    }

    @Override
    protected void buildDoor() {
        System.out.println(name +"的门要采用防盗门");
    }

    @Override
    protected void buildWindow() {
        System.out.println(name + "的窗户要面向北方");
    }

    @Override
    protected void buildWall() {
        System.out.println(name + "的墙使用大理石建造");
    }

    @Override
    protected void buildBase() {
        System.out.println(name + "的地基使用钢铁地基");
    }
    
}
```
子类2：HouseTwo
```java
public class HouseTwo extends HouseTemplate {

    HouseTwo(String name){
        super(name);
    }

    @Override
    protected void buildDoor() {
        System.out.println(name + "的门采用木门");
    }

    @Override
    protected void buildWindow() {
        System.out.println(name + "的窗户要向南");
    }

    @Override
    protected void buildWall() {
        System.out.println(name + "的墙使用玻璃制造");
    }

    @Override
    protected void buildBase() {
        System.out.println(name + "的地基使用花岗岩");
    }

}
```
测试类：Clienter
```java
public class Clienter {

    public static void main(String[] args){
        HouseTemplate houseOne = new HouseOne("房子1");
        HouseTemplate houseTwo = new HouseTwo("房子2");
        houseOne.buildHouse();
        houseTwo.buildHouse();
    }

}
```
执行结果：
```text
房子1的地基使用钢铁地基
房子1的墙使用大理石建造
房子1的门要采用防盗门
房子1的窗户要面向北方
房子2的地基使用花岗岩
房子2的墙使用玻璃制造
房子2的门采用木门
房子2的窗户要向南
```
通过以上例子，我们认识了模板模式中的基本方法和模板方法，其中HouseTemplate中的buildHouse方法就是基本方法，其余四个均为模板方法。其中基本方法一般会用final修饰，保证其不会被子类修改，而模板方法则使用protected修饰，表明其需要在子类中实现。
  
其实，模板模式中还有一个钩子方法的概念，有人称，具有钩子方法的模板模式才算完整，也许吧。
  
钩子方法时干啥的呢？钩子就是给子类一个授权，允许子类通过重写钩子方法来颠覆基本逻辑的执行，这有时候是非常有用的。就比如在盖房子的时候，有一个需要子类来决定是否建造厕所间的需求时，可以这么实现：
  
模板抽象类：HouseTemplate
```java
public abstract class HouseTemplate {

    protected HouseTemplate(String name){
        this.name = name;
    }

    protected String name;

    protected abstract void buildDoor();

    protected abstract void buildWindow();

    protected abstract void buildWall();

    protected abstract void buildBase();

    protected abstract void buildToilet();

    //钩子方法
    protected boolean isBuildToilet(){
        return true;
    }

    //公共逻辑
    public final void buildHouse(){

        buildBase();
        buildWall();
        buildDoor();
        buildWindow();
        if(isBuildToilet()){
            buildToilet();
        }
    }

}
```
子类1：HouseOne
```java
public class HouseOne extends HouseTemplate {

    HouseOne(String name){
        super(name);
    }

    HouseOne(String name, boolean isBuildToilet){
        this(name);
        this.isBuildToilet = isBuildToilet;
    }

    public boolean isBuildToilet;

    @Override
    protected void buildDoor() {
        System.out.println(name +"的门要采用防盗门");
    }

    @Override
    protected void buildWindow() {
        System.out.println(name + "的窗户要面向北方");
    }

    @Override
    protected void buildWall() {
        System.out.println(name + "的墙使用大理石建造");
    }

    @Override
    protected void buildBase() {
        System.out.println(name + "的地基使用钢铁地基");
    }

    @Override
    protected void buildToilet() {
        System.out.println(name + "的厕所建在东南角");
    }

    @Override
    protected boolean isBuildToilet(){
        return isBuildToilet;
    }

}
```
子类2：HouseTwo
```java
public class HouseTwo extends HouseTemplate {

    HouseTwo(String name){
        super(name);
    }

    @Override
    protected void buildDoor() {
        System.out.println(name + "的门采用木门");
    }

    @Override
    protected void buildWindow() {
        System.out.println(name + "的窗户要向南");
    }

    @Override
    protected void buildWall() {
        System.out.println(name + "的墙使用玻璃制造");
    }

    @Override
    protected void buildBase() {
        System.out.println(name + "的地基使用花岗岩");
    }

    @Override
    protected void buildToilet() {
        System.out.println(name + "的厕所建在西北角");
    }

}
```
测试类：Clienter
```java
public class Clienter {

    public static void main(String[] args){
        HouseTemplate houseOne = new HouseOne("房子1", false);
        HouseTemplate houseTwo = new HouseTwo("房子2");
        houseOne.buildHouse();
        houseTwo.buildHouse();
    }

}
```
执行结果：
```text
房子1的地基使用钢铁地基
房子1的墙使用大理石建造
房子1的门要采用防盗门
房子1的窗户要面向北方
房子2的地基使用花岗岩
房子2的墙使用玻璃制造
房子2的门采用木门
房子2的窗户要向南
房子2的厕所建在西北角
```
通过直接结果我们可以清晰的看到，我们通过重写钩子方法自定义了房子1不需要建造厕所（fasle）。
  
钩子方法的作用也就一目了然啦。
## 三、模式解析
模板模式的关键点：
- **使用抽象类定义模板类，并在其中定义所有的基本方法、模板方法，钩子方法，不限数量，以实现功能逻辑为主。其中基本方法使用final修饰，其中要调用基本方法和钩子方法，基本方法和钩子方法可以使用protected修饰，表明可被子类修改。**
- **定义实现抽象类的子类，重写其中的模板方法，甚至钩子方法，完善具体的逻辑。**
## 四、使用场景
使用场景：
1. 在多个子类中拥有相同的方法，而且逻辑相同时，可以将这些方法抽出来放到一个模板抽象类中。
2. 程序主框架相同，细节不同的情况下，也可以使用模板方法。