# Java基础系列-AutoCloseable
## 概述
我们在看java源码的时候，经常会在io类的继承体系中看到Closeable接口，如下所示：
```java
package java.io;

import java.io.IOException;

public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```
它表示这个资源是可关闭的，它又继承自AutoCloseable接口，这个接口是表示可自动关闭的意思，如下：
```java
package java.lang;

public interface AutoCloseable {
    void close() throws Exception;
}
```
## 简介
话说AutoCloseable是何时来的呢？有什么作用呢？

其实它是在Java 1.7版本添加进来的，大概我们还记得1.7版本新增的try-with-resource功能，主要是一种资源简化写法。

使用这种写法的时候不再需要手动进行关闭与释放资源，在发生异常或者执行完毕之后会自动执行对应的close方法来关闭与释放资源。
## try-with-resource用法
try-with-resoure使用的时候很简单，只需要将定义资源的部分代码写到try后面的圆括号中即可，这样原本写在finally里面的资源释放代码就可以去除了。
```java

```