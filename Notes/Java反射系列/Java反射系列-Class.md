# Java反射系列-Class

Java反射技术可以动态的获取已加载class的各种信息，包括类型信息、方法、构造器、字段等，同时还能进行各种操作，包括创建实例，调用方法等。

我们一般开发很少用到反射，但是看过框架源码的人一定对反射深有印象。Java反射可以算是框架开发的必备利器。

一般使用反射都有这么个简单流程，那即是首先需要获取到要操作的类的Class实例，这是第一步，然后就可以根据需要从这个Class实例中获取到自己想要的内容进行操作了。
## Class
定义一个类备用
```java
public class Person{
    private String name;
    public Person(){}
    public Person(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
    public void setName(String name){
        this.name = name;
    }
}
```
### 获取Class实例
```java
public class RefectTest{
    public static void main(String[] args){
        // 1-已知类名的情况下：
        Class class1 = Person.class;
        // 2-已知类的全限定名的情况下：
        Class class2 = Class.forName("com.dh.Person");
        // 3-拥有实例的情况下：
        Person person = new Person();
        Class class3 = person.getClass();
    }
}
```
### 创建对象
```java
public class RefectTest{
    public static void main(String[] args){
        Class c = Person.class;
        // 1-拥有无参构造器的情况下：
        Person person = c.newInstance();
        // 2-需要使用指定的带参构造器的情况：
        Constructor cs = c.getConstructor(String.class);
        Person person = (Person)cs.newInstance("huahua");
    }
}
```
### 获取构造器
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-用于获取目标类中的所有public修饰的构造器：
    public Constructor<?>[] getConstructors() throws SecurityException{/*...*/}
    // 2-用于获取目标类中的所有构造器，包括私有的：
    public Constructor<?>[] getDeclaredConstructors() throws NoSuchMethodException, SecurityException{/*...*/}
    // 3-用于获取目标类中指定参数类型的public构造器：
    public Constructor<T> getConstructor(Class<?>... parameterTypes) throws SecurityException{/*...*/}
    // 4-用于获取目标类中指定参数类型的构造器，不论修饰符：
    public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException{/*...*/}
    // 5-获取当前Class是在哪个构造器中定义的内部类或匿名内部类:
    public Constructor<?> getEnclosingConstructor() throws SecurityException{/*...*/}
}
```
### 获取方法
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-用于获取目标类中的所有public方法：
    public Method[] getMethods() throws SecurityException{/*...*/}
    // 2-用于获取目标类中的所有方法，包括私有的：
    public Method[] getDeclaredMethods() throws SecurityException{/*...*/}
    // 3-用于获取目标类中指定方法名和参数类型的public方法：
    public Method getMethod(String name, Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException{/*...*/}
    // 4-用于获取目标类中指定方法名和参数类型的方法，不论修饰符：
    public Method getDeclaredMethod(String name, Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException{/*...*/}
    // 5-获取当前Class是在哪个方法中定义的内部类或匿名内部类:
    public Method getEnclosingMethod() throws SecurityException{/*...*/}
}
```
### 获取字段
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-用于获取目标类中的所有public字段：
    public Field[] getFields() throws SecurityException{/*...*/}
    // 2-用于获取目标类中的所有字段，包括私有的：
    public Field[] getDeclaredFields() throws SecurityException{/*...*/}
    // 3-用于获取目标类中指定字段名的public字段：
    public Field getField(String name) throws NoSuchFieldException, SecurityException{/*...*/}
    // 4-用于获取目标类中指定字段名的字段，不论修饰符：
    public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException{/*...*/}
}
```
### 获取注解
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-获取目标类上的指定类型的注解：
    public <A extends Annotation> A getAnnotation(Class<A> annotationClass){/*...*/}
    // 2-获取目标类上的所有注解：
    public Annotation[] getAnnotations(){/*...*/}
    // 3-获取与当前元素关联的指定类型的所有注解：
    public <A extends Annotation> A[] getAnnotationsByType(Class<A> annotationClass){/*...*/}
    // 4-获取目标类上的所有直接注解，不包含超类中继承下来的注解：
    public <A extends Annotation> A getDeclaredAnnotation(Class<A> annotationClass){/*...*/}
    // 5-获取当前存在于目标上的所有注解，不包含超类中继承下来的注解:
    public Annotation[] getDeclaredAnnotations(){/*...*/}
    // 6-获取与当前元素关联的指定类型的所有直接注解，不包含哪些超类中继承下来的注解:
    public <A extends Annotation> A[] getDeclaredAnnotationsByType(Class<A> annotationClass){/*...*/}
}
```
### 获取类和接口
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-用于获取目标类中的所有public内部类和接口：
    public Class<?>[] getClasses(){/*...*/}
    // 2-获取目标类中的所有内部类和接口：
    public Class<?>[] getDeclaredClasses() throws SecurityException{/*...*/}
    // 3-获取当前Class是在哪个类中定义的内部类，即返回所在外部类型：
    public Class<?> getDeclaringClass() throws SecurityException{/*...*/}
    // 4-获取当前Class是在哪个类中定义的内部类或匿名内部类,即返回所在外部类类型：
    public native Class<? super T> getSuperclass();
    // 5-获取当前Class的超类或超接口，如果当前Class是Object.class、接口、
    // 原始类型、void，则返回null，若未数组，则返回Object.class:
    public Annotation[] getDeclaredAnnotations(){/*...*/}
    // 6-获取当前Class的直接超类或者超接口，如果当前Class是Object.class、
    // 接口、原始类型、void，则返回null，若未数组，则返回Object.class:
    public Type getGenericSuperclass(){/*...*/}
    // 7-获取当前Class实现的所有接口，包括多层实现：
    public Class<?>[] getInterfaces(){/*...*/}
    // 8-获取当前Class直接实现的所有接口：
    public Type[] getGenericInterfaces(){/*...*/}
    // 9-获取数组的组件类型，如果当前Class不是数组，返回null
    public native Class<?> getComponentType();
}
```
### 验证isXXX方法
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-验证给定的obj实例是否是当前Class实例的对象实例:
    public native boolean isInstance(Object obj);
    // 2-验证当前Class实例所代表的类或接口与给定的Class实例cls的类或者接口相同，或是否为其超类或超接口:
    public native boolean isAssignableFrom(Class<?> cls);
    // 3-验证当前Class实例是否接口类型:
    public native boolean isInterface();
    // 4-验证当前Class实例是否数组类型:
    public native boolean isArray();
    // 5-验证当前Class实例是否原始类型（8大基本类型、void）:
    public native boolean isPrimitive();
    // 6-验证当前Class实例是否注解:
    public boolean isAnnotation(){/*...*/}
    // 7-验证当前Class实例是否一个合成的类:
    public boolean isSynthetic(){/*...*/}
    // 8-验证当前Class实例是否一个匿名类:
    public boolean isAnonymousClass(){/*...*/}
    // 9-验证当前Class实例是否一个本地类:
    public boolean isLocalClass(){/*...*/}
    // 10-验证当前Class实例是否一个成员类:
    public boolean isMemberClass(){/*...*/}
    // 11-验证当前Class实例是否一个枚举
    public boolean isEnum(){/*...*/}
    // 12-验证当前类实例上是否存在指定的注解
    public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass){/*...*/}
}
```
### 其他
```java
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type, AnnotatedElement {
    // 1-获取当前Class所在包:
    public Package getPackage(){/*...*/}
    // 2-获取当前Class的类加载器:
    public ClassLoader getClassLoader(){/*...*/}
    // 3-获取当前Class的所有参数化类型：
    public TypeVariable<Class<T>>[] getTypeParameters(){/*...*/}
    // 4-获取Java语言修饰符:
    public native int getModifiers();
    // 5-获取类的签名:
    public native Object[] getSigners();
    // 6-设置类的签名:
    public native void setSigners(Object[] signers);
    // 7-获取枚举所有值，如果不是枚举，返回null:
    public T[] getEnumConstants(){/*...*/}
    // 8-将给定的对象obj强转为当前Class类型：
    public T cast(Object obj){/*...*/}
    // 9-如果给定类型时当前类的超类，则将其转为给定类型,当前类型为子类：
    public <U> Class<? extends U> asSubclass(Class<U> clazz){/*...*/}
}
```
## Constructor
