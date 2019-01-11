# Java基础系列-equals方法和hashCode方法
## 概述
&#160;&#160;&#160;&#160;&#160;&#160;&#160;equals方法和hashCode方法都是有Object类定义的。
```java
public class Object {
    public native int hashCode();
    public boolean equals(Object obj) {
        return (this == obj);
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;任何的类都是Object类的子类，所有它们默认都拥有这两个方法。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;equals方法用于定义两个对象的比较方式，而hashCode方法是native方法，主要用户计算对象的hash值。
## equals
&#160;&#160;&#160;&#160;&#160;&#160;&#160;equals方法主要用于定义两个对象的比较方式，默认的比较方式是比较内存地址，相对于基本类型来说就是值，而相对于引用类型来说就是堆中具体对象的地址。那么就只有值相同的基本类型，和同一个对象的两个引用才能相等。但是在我们实际业务系统中，两个对象的相等一般指的是两个对象的内容相同（逻辑相同），而不是说它两个是同一个对象，这种情况使用默认的equals就无法实现相等（因为两个不同对象地址值一定不同），这时候我们就需要对equals方法进行重写，定义新的比较方式。
### 准则
- 自省性：对于非null的x，存在：x.equals(x)返回true
- 对称性：对于非null的x和y，存在：x.equals(y)==y.equals(x)
- 传递性：对于非null的x、y、z，存在：当x.equals(y)返回true，y.equals(z)返回true，则x.equals(z)一定为true
- 一致性：对于非null的x和y，多次调用x.equals(y)所得的结果是不变的
- 非空性：对于非null的x，存在x.equals(null)返回false
### 重写
&#160;&#160;&#160;&#160;&#160;&#160;&#160;其实Java中已经为我们展示了如何重equals方法了，最经典的就是String的equals方法：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    public boolean equals(Object anObject) {
        // 首先判断两个对象是不是同一个，地址相同否
        if (this == anObject) {
            return true;
        }
        // 判断给定的对象是否是String类型，这里instanceof关键字是重写equals方法时经常使用的一个关键字
        // instanseof用于判断右边的类型是否是当前对象的类型或者超类型，超接口类型等
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            // 校验两个字符串的长度相同否
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                // 循环校验两个字符串中的每个字符是否相同
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;注意，使用instanceof在针对存在子类的情况下，可能会出现违反对称性和传递性的情况，为了避免这种情况，可以通给getClass的方式比较类型。  
&#160;&#160;&#160;&#160;&#160;&#160;&#160;自定义重写：
```java
public class EqualsTest {
    private int id;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    @Override
    public boolean equals(Object obj) {
        // 满足非空性
        if(obj == null){
            return false;
        }
        // 满足自省性
        if(this == obj){
            return true;
        }
        // 满足对称性、传递性、一致性
        if(this.getClass() == obj.getClass()
                && this.getClass().getClassLoader() == obj.getClass().getClassLoader()
                && this.id == ((EqualsTest)obj).getId()){
            return true;
        }
        return false;
    }
}
```
&#160;&#160;&#160;&#160;&#160;&#160;&#160;注意：这里如果是有不同的类加载器加载的同一类的实例也是无法相等的。
## hashCode
&#160;&#160;&#160;&#160;&#160;&#160;&#160;hashCode一般用于计算对象的hash值，它在类重写equals的时候一起重写，重写它的目的是为了保证equals相同的两个对象的hashCode结果一致，为什么要保证这一点呢，那就归结到java中的那几个基于Hash实现的集合上了，比如HashMap、HashSet等，这些集合需要用到对象的hash值来参与计算定位。
### 实现方式
- 链地址法（理解）
- 开放寻址法（了解）
    - 线性探查法
    - 二次探查法
    - 双重散列法
- 再HASH法（知道）
- 建立公共溢出区法（知道）

&#160;&#160;&#160;&#160;&#160;&#160;&#160;hashCode的实现方式并不是随手而来的，需要考虑各种情况，选择合适的方式来实现，举个例子，在Java的HashMap集合中，采用的就是链地址法来处理hash冲突。
