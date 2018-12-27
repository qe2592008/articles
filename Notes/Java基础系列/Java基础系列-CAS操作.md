# Java基础系列-CAS操作

## 概述

​	我们都知道在Java多线程编程中要用到锁，来保证线程安全性，但是Java还给我们提供了另外的选择，那就是CAS。	

​	什么是CAS操作呢？

​	所谓CAS就是Compare And Swap，就是比较并替换的意思，它是利用操作系统的原子指令来将比较和替换整合在一个原子操作中，保证不中断。

​	如果将锁看作是Java中悲观锁的体现，那么CAS就是乐观锁的体现了。

> 引申-悲观锁
>
> ​	总是悲观的认为代码执行会出现线程安全性问题，干脆直接加锁，将所有可能不可能的线程不安全全部摒弃在外，这样确实保证了线程安全性，但是也增加了系统开销。

> 引申-乐观锁
>
> ​	总是乐观的觉得代码执行不会出现线程安全性问题，总是直接就是执行，但是一旦出现安全性问题，我也不慌，我可以选择再次尝试，或者就此中断。

​	原子操作有什么好处呢？

​	原子操作本身是原子的，是不会被中断、一次性执行结束的一段指令操作。原子操作重点就是不会被中断，这样只要它开始执行，它呈现给其他人的结果是确切的，是不存在问题的。

​	Java中CAS操作是由Unsafe类提供的。

## Unsafe

​	从类名我们就能看出这个类所要表述的意思，它是不安全的。Java官方并不推荐JDK用户直接使用这个类，因为如果没有对其有深入了解，贸然使用，那就是炸弹一个，其实我们完全可以使用通过它实现的各种安全的类来完成我们的需求，比如原子更新类AtomicXXX、ConcurrentHashMap等。

​	首先还是简单看看这个类。

​	Unsafe类内部的方法基本上都是Native的，使用这些方法可以通过调用底层的C代码来直接操作内存。Java去除了C语言中的指针，却在这里又提供了指针的功能。

### 分配内存

```java
// 分配内存指定大小var1的内存
public native long allocateMemory(long var1);
// 根据给定的内存地址var1设置重新分配指定大小var3的内存
public native long reallocateMemory(long var1, long var3);
```

### 释放内存

```java
// 用于释放allocateMemory和reallocateMemory申请的内存,释放指定内存地址var1的内存
public native void freeMemory(long var1);
```

> 注意：在使用了分配内存的操作之后，一定要记得手动执行内存释放

### 修改内存

```java
// 将指定对象var1的指定偏移量var2的内存地址的内存块的值设置为固定值var6
public native void setMemory(Object var1, long var2, long var4, byte var6);
```

### 创建对象

```java
// 创建指定的类型var1的实例，不通过构造器
public native Object allocateInstance(Class<?> var1) throws InstantiationException;
```

### CAS操作

```java
// 将指定对象var1的指定偏移量var2的内存位置的值从var4修改为var5
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
// 将指定对象var1的指定偏移量var2的内存位置的int值从var4修改为var5
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
// 将指定对象var1的指定偏移量var2的内存位置的long值从var4修改为var6
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

​	这三个方法就是用来实现CAS操作的。

> 注意：我们自己使用Unsafe的时候是不能直接用getUnsafe()方法获取到Unsafe类的实例的，因为该类被设计为仅仅可以被Bootstrap类加载器加载，我们只能通过反射的方式来获取其实例了。
>
> ```java
> @CallerSensitive
> public static Unsafe getUnsafe() {
>     Class var0 = Reflection.getCallerClass();
>     // 这里会校验当前的类加载器是否存在，因为Bootstrap类加载器由c语言实现，所以这里是null,如果我们
>     // 直接使用这个方法，那么类加载器就是AppClassLoader，这里不为null,就会抛出异常。
>     if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
>         throw new SecurityException("Unsafe");
>     } else {
>         return theUnsafe;
>     }
> }
> ```
>
> 使用反射方式获取Unsafe实例
>
> ```java
> static Unsafe unsafe = null;
> static {
>     try {
>         Class unsafeClass = Unsafe.class;
>         Constructor<Unsafe> constructor = unsafeClass.getDeclaredConstructor(null);
>         constructor.setAccessible(true);// 设置私有构造器可访问
>         unsafe = constructor.newInstance(null);
>     } catch (IllegalAccessException e) {
>         e.printStackTrace();
>     } catch (InstantiationException e) {
>         e.printStackTrace();
>     } catch (NoSuchMethodException e) {
>         e.printStackTrace();
>     } catch (InvocationTargetException e) {
>         e.printStackTrace();
>     }
> }
> ```

## 总结

​	说了这么多，总结起来就是：CAS能够保证变量操作的原子性，除此之外，无任何其他保证，但是volatile却是能保证变量操作的可见性和有序性，二者正好互补，所以一般情况下将二者结合使用，使用volatile来修饰变量用来保存当前值(旧值)，使用CAS来对变量进行修改，这样线程安全性的三大特性均得到保证。

​	其实Java并不推荐用户直接使用CAS，而是内部为我们提供了许多通过CAS实现的类来供我们使用，我们直接使用这些类就能体验到CAS的功能，这些类包括java.util.concurrent.atomic包的各种原子工具类，ConcurrentHashMap集合，FutureTask等，在JDK1.5中新增的java.util.concurrent多线程包下的类很多都是依靠CAS来实现的。

参考：

- [Java基础系列-原子变量类]()



​	