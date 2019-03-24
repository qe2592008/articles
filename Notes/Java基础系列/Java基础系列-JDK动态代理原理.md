# Java基础系列-JDK动态代理原理
## 一、概述

## 二、原理
使用代理涉及到哪些方面呢？

我们不妨以静态代理这种简单的形式来罗列一下涉及点：
- 目标接口
- 目标实现类（委托类）
- 代理类
- 目标方法增强
- 目标方法调用
- 目标实现类对象
- 代理类对象
- 发起调用
> 其中，目标实现类和代理类均实现了目标接口，目标方法增强指的是在代理类中实现，增强的同时还需要发起针对目标方法的调用
> 同时代理类中持有目标实现类对象

那么我们将这些内容映射到JDK动态代理中会是如何呢？
- 目标接口
- 目标实现类（委托类）
- InvocationHandler实现类
- 代理类
- 代理类实例
- 发起调用
> 很明显动态代理将目标方法增强和调用、目标实现类对象整合在一起，并进行了高层抽象，以适应任意目标接口和目标类，称为方法调用处理器，所以顾名思义，它的作用就是定义方法调用的（指的是代理调用目标方法）
> 
> 还有一点不同就是代理实例的创建方式变化，采用反射方式创建，反而不是硬编码
## 三、实例

## 四、源码
InvocationHandler
```java
/**
 * 该类用于定义一个调用处理器，这个处理器将'方法调用'这个行为抽象成为一个接口，
 * 在实现JDK动态代理的时候是不可或缺的核心部件。
 * 它的唯一的方法invoke中描述了'方法调用'所涉及的几个方面：
 *      1. proxy：代理实例
 *      2. method：所调用的方法
 *      3. args：调用方法的参数
 * 在实现这个接口的时候，需要实现invoke方法，主要就是要通过反射调用目标类的方法
 * method.invoke(target, args)，target表示代理类所持有的目标类的实例。
 * 整个InvocationHandler将作为创建代理实例的参数之一，如下面：
 */
public interface InvocationHandler {
    /**
     * 处理代理实例上的方法调用并返回结果。
     * 当在与其关联的代理实例上调用方法时，将在调用处理程序上调用此方法。
     * 
     * @param proxy 方法被调用的代理实例，即生成的代理类的实例
     * @param method 指所要调用的目标接口中的目标方法的Method实例
     * @param args 所调用目标方法的参数列表，如果为原始类型需要进行装箱
     * @return java.lang.Object
     * @throws    
     */
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```
Proxy.newProxyInstance
```java
/**
 * 类描述
 * 
 */
public class Proxy implements java.io.Serializable {
    /**
     * 创建代理实例的静态方法，需要提供三个参数：
     * 
     * 
     * @param loader 类加载器
     * @param interfaces 基于的接口
     * @param h InvocationHandler实例
     * @return java.lang.Object
     * @throws    
     */
    @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);// InvocationHandler的null校验

        final Class<?>[] intfs = interfaces.clone();
        // 如果设置的安全管理器，则进行一些安全权限校验
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }
        // 创建代理类
        Class<?> cl = getProxyClass0(loader, intfs);
        // 使用给定的InvocationHandler实例来调用代理类的构造器生成代理实例
        try {
            // 首先还是先做安全权限校验
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            // 首先根据给定的构造器参数列表找到目标构造器
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            // 下面针对不可达构造器设置为可达
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            // 使用找到的构造器创建代理实例并返回
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        // 接口数量校验
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        // If the proxy class defined by the given loader implementing
        // the given interfaces exists, this will simply return the cached copy;
        // otherwise, it will create the proxy class via the ProxyClassFactory
        // 如果有给定加载器加载的代理类实现了给定接口的话，此处直接返回代理类缓存中的副本，
        // 否则只能通过代理类工厂（ProxyClassFactory）创建一个代理类
        return proxyClassCache.get(loader, interfaces);
    }
}
```
ProxyClassFactory
```java
public class Proxy implements java.io.Serializable {
    /**
     * A factory function that generates, defines and returns the proxy class given
     * the ClassLoader and array of interfaces.
     */
    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // 生成的代理类的名称前缀
        private static final String proxyClassNamePrefix = "$Proxy";
        // 用于生成下一个代理类的唯一数值（编号）
        private static final AtomicLong nextUniqueNumber = new AtomicLong();
        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            // 循环校验所有接口
            for (Class<?> intf : interfaces) {
                // 校验接口类是否是由给定的类加载器加载解析的
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                // 校验给定类是否代表一个接口
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                // 校验当前接口是否重复
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            String proxyPkg = null;     // 定义代理类所在包
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
            // 校验所有给定接口的包是否一致
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();// 获取修饰符类型
                if (!Modifier.isPublic(flags)) {
                    // 针对非public的接口
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));// 截取包名
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }
            // 如果没有非public代理接口，那么使用默认的包名：com.sun.proxy package
            if (proxyPkg == null) {
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }
            // 构造代理类的名称:包名+前缀+num
            long num = nextUniqueNumber.getAndIncrement();// 原子增长
            String proxyName = proxyPkg + proxyClassNamePrefix + num;
            // 生成指定的代理类文件
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                // 加载解析生成的代理类字节码文件，并返回生成的代理类的Class实例
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
                /*
                 * A ClassFormatError here means that (barring bugs in the
                 * proxy class generation code) there was some other
                 * invalid aspect of the arguments supplied to the proxy
                 * class creation (such as virtual machine limitations
                 * exceeded).
                 */
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
    // 加载解析生成的代理类字节码文件
    private static native Class<?> defineClass0(ClassLoader loader, String name,
                                                byte[] b, int off, int len);
}
```
ProxyGenerator.generateProxyClass
```java
public class ProxyGenerator {
    /**
     * 用于生成代理类Class文件
     * 
     * @param var0
     * @param var1
     * @param var2
     * @return byte[]
     * @throws    
     */
    public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
        ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);// 构造代理生成器
        final byte[] var4 = var3.generateClassFile();// 生成代理类的Class字节码文件（在内存中）
        // 如果设置了需要保存生成的文件，那么下面将内存中的文件写入磁盘，否则不持久化
        if (saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    try {
                        int var1 = var0.lastIndexOf(46);
                        Path var2;
                        if (var1 > 0) {
                            Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                            Files.createDirectories(var3);
                            var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                        } else {
                            var2 = Paths.get(var0 + ".class");
                        }

                        Files.write(var2, var4, new OpenOption[0]);
                        return null;
                    } catch (IOException var4x) {
                        throw new InternalError("I/O exception saving generated file: " + var4x);
                    }
                }
            });
        }

        return var4;
    }
    // 生成代理类字节码文件
    private byte[] generateClassFile() {
        // 将Object中定义的hashCode方法、equals方法、toString方法添加到字节码文件中
        this.addProxyMethod(hashCodeMethod, Object.class);
        this.addProxyMethod(equalsMethod, Object.class);
        this.addProxyMethod(toStringMethod, Object.class);
        Class[] var1 = this.interfaces;// 接口列表
        int var2 = var1.length;// 接口数量

        int var3;
        Class var4;
        for(var3 = 0; var3 < var2; ++var3) {// 遍历接口
            var4 = var1[var3];
            Method[] var5 = var4.getMethods();// 获取每个接口中定义的方法列表
            int var6 = var5.length;// 每个接口中方法数量

            for(int var7 = 0; var7 < var6; ++var7) {// 遍历接口中的方法
                Method var8 = var5[var7];
                this.addProxyMethod(var8, var4);// 将接口中的方法一一添加到字节码文件中
            }
        }
        // 校验相同签名的方法的返回类型是否一致
        Iterator var11 = this.proxyMethods.values().iterator();
        List var12;
        while(var11.hasNext()) {
            var12 = (List)var11.next();
            checkReturnTypes(var12);
        }
        // 下面的逻辑为生成字节码文件的逻辑
        Iterator var15;
        try {
            this.methods.add(this.generateConstructor());// 生成构造器
            var11 = this.proxyMethods.values().iterator();

            while(var11.hasNext()) {
                var12 = (List)var11.next();
                var15 = var12.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                    // 将代理类字段声明为Method，并且字段修饰符为 private static.
                    this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                    this.methods.add(var16.generateMethod());// 生成代理类的方法
                }
            }
            // 为代理类生成静态代码块对某些字段进行初始化
            this.methods.add(this.generateStaticInitializer());
        } catch (IOException var10) {
            throw new InternalError("unexpected I/O Exception", var10);
        }

        if (this.methods.size() > 65535) {// 代理类中的方法数量超过65535就抛异常
            throw new IllegalArgumentException("method limit exceeded");
        } else if (this.fields.size() > 65535) {// 代理类中字段数量超过65535也抛异常
            throw new IllegalArgumentException("field limit exceeded");
        } else {
            // 下面是对文件进行处理的过程
            this.cp.getClass(dotToSlash(this.className));
            this.cp.getClass("java/lang/reflect/Proxy");
            var1 = this.interfaces;
            var2 = var1.length;

            for(var3 = 0; var3 < var2; ++var3) {
                var4 = var1[var3];
                this.cp.getClass(dotToSlash(var4.getName()));
            }

            this.cp.setReadOnly();
            ByteArrayOutputStream var13 = new ByteArrayOutputStream();
            DataOutputStream var14 = new DataOutputStream(var13);

            try {
                var14.writeInt(-889275714);
                var14.writeShort(0);
                var14.writeShort(49);
                this.cp.write(var14);
                var14.writeShort(this.accessFlags);
                var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
                var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
                var14.writeShort(this.interfaces.length);
                Class[] var17 = this.interfaces;
                int var18 = var17.length;

                for(int var19 = 0; var19 < var18; ++var19) {
                    Class var22 = var17[var19];
                    var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
                }

                var14.writeShort(this.fields.size());
                var15 = this.fields.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                    var20.write(var14);
                }

                var14.writeShort(this.methods.size());
                var15 = this.methods.iterator();

                while(var15.hasNext()) {
                    ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                    var21.write(var14);
                }

                var14.writeShort(0);
                return var13.toByteArray();
            } catch (IOException var9) {
                throw new InternalError("unexpected I/O Exception", var9);
            }
        }
    }
}
```
## 五、总结

参考：
- [深入理解JDK动态代理机制](https://www.jianshu.com/p/471c80a7e831)