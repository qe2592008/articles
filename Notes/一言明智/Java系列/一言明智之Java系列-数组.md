数组保存在堆中，由JVM创建，Java中没有具体的数组类型，只是用[]来辅助定义。

数组的长度是固定的，一旦定义，不再变化。

数组的定义方式：
```java
public class Main{
    public static void main(String[] args){
        int[] ints = {1,2,3,4};
        int[] ints1 = new int[10];
    }
}
```
