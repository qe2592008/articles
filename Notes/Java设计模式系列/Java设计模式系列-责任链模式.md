# Java设计模式系列-责任链模式
## 一、概述
职责链模式（称责任链模式）将请求的处理对象像一条长链一般组合起来，形成一条对象链。请求并不知道具体执行请求的对象是哪一个，这样就实现了请求与处理对象之间的解耦。
  
生活中这种情况其实很常见，公司部门之中，政府部门之中都有体现，在公司部门中，当你提交一份请求文件给你的直接上级时，你的直接上级可以处理这个文件，若他觉得自己不够资格，会将文件传递为他的直接上级，这样文件请求在这条链中传递，直到被某位感觉自己足够资格处理文件的人给处理掉为止，在政府部门也是如此。
  
职责链模式需要一个总接口，用来定义处理对象的公共部分（一般使用抽象类来定义），公共部分包括：一个后继处理器，设置和获取后继处理器的方法，具体的请求处理方法（这个方法需要在每个具体处理对象中实现），这里定义为抽象方法。
## 二、示例
我们就以公司部门为例来进行实例：
  
领导抽象类：Lingdao
```java
public abstract class Lingdao {
    private Lingdao nextLingdao;
    public Lingdao getNextLingdao() {
        return nextLingdao;
    }
    public void setNextLingdao(Lingdao nextLingdao) {
        this.nextLingdao = nextLingdao;
    }
    abstract void chuli(Files file);
}
```
领导实现类：Zongjingli、Fujingli、Bumenjingli
```java
/**
 * 总经理
 */
public class Zongjingli extends Lingdao {
    private final String name = "总经理";
    private final int level = 0;//最大
    @Override
    public void chuli(Files file) {
        if(this.level > file.getLevel()){
            System.out.println(name + "未处理文件《" + file.getFileName() + "》");
            getNextLingdao().chuli(file);
        }else{
            System.out.println(name + "处理了文件《" + file.getFileName() + "》");
        }
    }
}

/**
 * 副经理
 */
public class Fujingli extends Lingdao {
    private final String name = "副经理";
    private final int level = 1;
    @Override
    public void chuli(Files file) {
        if(this.level > file.getLevel()){
            System.out.println(name + "未处理文件《" + file.getFileName() + "》");
            getNextLingdao().chuli(file);
        }else{
            System.out.println(name + "处理了文件《" + file.getFileName() + "》");
        }
    }
}

/**
 * 部门经理
 */
public class Bumenjingli extends Lingdao{
    private final String name = "部门经理";
    private final int level = 2;
    @Override
    public void chuli(Files file) {
        if(this.level > file.getLevel()){
            System.out.println(name + "未处理文件《" + file.getFileName() + "》");
            getNextLingdao().chuli(file);
        }else{
            System.out.println(name + "处理了文件《" + file.getFileName() + "》");
        }
    }
}
```
文件类：Files
```java
public class Files {
    private String fileName;
    private int level;
    public String getFileName() {
        return fileName;
    }
    public void setFileName(String fileName) {
        this.fileName = fileName;
    }
    public int getLevel() {
        return level;
    }
    public void setLevel(int level) {
        this.level = level;
    }
}
```
测试类：Clienter
```java
public class Clienter {
    public static void main(String[] args) {
        //定义职责链
        Lingdao zongjingli = new Zongjingli();
        Lingdao fujingli = new Fujingli();
        Lingdao bumenjingli = new Bumenjingli();
        bumenjingli.setNextLingdao(fujingli);
        fujingli.setNextLingdao(zongjingli);
        //定义两份文件
        Files f1 = new Files();
        f1.setFileName("正确对待旱鸭子");
        f1.setLevel(1);
        Files f2 = new Files();
        f2.setFileName("年会在那里举行");
        f2.setLevel(0);
        //提交文件
        bumenjingli.chuli(f1);
        bumenjingli.chuli(f2);
    }
}
```
执行结果：
```text
部门经理未处理文件《正确对待旱鸭子》
副经理处理了文件《正确对待旱鸭子》
部门经理未处理文件《年会在那里举行》
副经理未处理文件《年会在那里举行》
总经理处理了文件《年会在那里举行》
```
## 三、模式解析
实例清晰易懂，职责链的模式重在这个链的组成，如何组成链呢？
1. 第一步需要在处理者对象类中定义后继处理者对象，将这部分代码抽象到抽象类中实现，降低了代码重复性。
2. 第二步就是在处理者对象类中的处理方法中如果当前处理者对象无法处理，就将其传递到后继对象去处理。
3. 第三步就是在测试类中将这些处理者类的实例串联成链。
  
其实这个模式有个缺陷，那就是虽然可以实现请求额度传递，但是也不保证在这里链中一定存在能处理请求的处理这类，一旦不存在，那么这个请求将一直存在与链中，如果将链设置成环形，那么这个请求将会永远在环形链中传递不休；还有一点就是由于请求的传递，请求无法立即精确的找到处理者，处理效率会降低。