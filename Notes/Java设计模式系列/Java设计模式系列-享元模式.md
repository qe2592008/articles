# Java设计模式系列-享元模式
## 一、概述
享元模式：“享”就是分享之意，指一物被众人共享，而这也正是该模式的终旨所在。
  
享元模式有点类似于单例模式，都是只生成一个对象来被共享使用。这里有个问题，那就是对共享对象的修改，为了避免出现这种情况，我们将这些对象的公共部分，或者说是不变化的部分抽取出来形成一个对象。这个对象就可以避免到修改的问题。
  
享元的目的是为了减少不会要额内存消耗，将多个对同一对象的访问集中起来，不必为每个访问者创建一个单独的对象，以此来降低内存的消耗。
## 二、示例
下面我们来看一个简单的例子：

建筑接口：JianZhu
```java
public interface Jianzhu {
    void use();
}
```
体育馆实现类：TiYuGuan
```java
public class TiYuGuan implements Jianzhu {
    private String name;
    private String shape;
    private String yundong;
    public TiYuGuan(String yundong){
        this.setYundong(yundong);
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getShape() {
        return shape;
    }
    public void setShape(String shape) {
        this.shape = shape;
    }
    public String getYundong() {
        return yundong;
    }
    public void setYundong(String yundong) {
        this.yundong = yundong;
    }
    @Override
    public void use() {
        System.out.println("该体育馆被使用来召开奥运会" + "  运动为："+ yundong+"  形状为："+shape+ "  名称为："+name);
    }
}
```
建筑工厂类：JianZhuFactory
```java
import java.util.*;

public class JianZhuFactory {
    private static final Map<String,TiYuGuan> tygs = new HashMap<String,TiYuGuan>();
    public static TiYuGuan getTyg(String yundong){
        TiYuGuan tyg = tygs.get(yundong);
        if(tyg == null){
            tyg = new TiYuGuan(yundong);
            tygs.put(yundong,tyg);
        }
        return tyg;
    }
    public static int getSize(){
        return tygs.size();
    }
}
```
测试类：Clienter
```java
public class Clienter {
    public static void main(String[] args) {
        String yundong ="足球";
        for(int i = 1;i <= 5;i++){
            TiYuGuan tyg = JianZhuFactory.getTyg(yundong);
            tyg.setName("中国体育馆");
            tyg.setShape("圆形");
            tyg.use();
            System.out.println("对象池中对象数量为："+JianZhuFactory.getSize());
        }
    }
}
```
执行结果：
```text
该体育馆被使用来召开奥运会  运动为：足球  形状为：圆形  名称为：中国体育馆
对象池中对象数量为：1
该体育馆被使用来召开奥运会  运动为：足球  形状为：圆形  名称为：中国体育馆
对象池中对象数量为：1
该体育馆被使用来召开奥运会  运动为：足球  形状为：圆形  名称为：中国体育馆
对象池中对象数量为：1
该体育馆被使用来召开奥运会  运动为：足球  形状为：圆形  名称为：中国体育馆
对象池中对象数量为：1
该体育馆被使用来召开奥运会  运动为：足球  形状为：圆形  名称为：中国体育馆
对象池中对象数量为：1
```
## 三、模式解析
如上示例中，使用工厂模式进行配合，创建对象池，测试类中的循环，你可以想象成为要举行5场比赛，每场比赛的场地就是体育馆
  
通过执行结果可以看出，在这个对象池（HashMap）中，一直都只有一个对象存在，第一次使用的时候创建对象，之后的每次调用都用的是那个对象，不会再重新创建。
  
其实在Java中就存在这种类型的实例：String。
  
Java中将String类定义为final（不可改变的），JVM中字符串一般保存在字符串常量池中，这个字符串常量池在jdk 6.0以前是位于常量池中，位于永久代，而在JDK 7.0中，JVM将其从永久代拿出来放置于堆中。
  
我们使用如下代码定义的两个字符串指向的其实是同一个字符串常量池中的字符串值。
```text
String s1 = "abc";
String s2 = "abc";
```
如果我们以s1==s2进行比较的话所得结果为：true，因为s1和s2保存的是字符串常量池中的同一个字符串地址。这就类似于我们今天所讲述的享元模式，字符串一旦定义之后就可以被共享使用，因为他们是不可改变的，同时被多处调用也不会存在任何隐患。

## 四、使用场景
当我们项目中创建很多对象，而且这些对象存在许多相同模块，这时，我们可以将这些相同的模块提取出来采用享元模式生成单一对象，再使用这个对象与之前的诸多对象进行配合使用，这样无疑会节省很多空间。