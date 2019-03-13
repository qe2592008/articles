# JVM基础系列-ClassLoader
## 一、概述
ClassLoader是类加载器的抽象层类，主要定义了类加载的方法与流程。

子类可以通过重写一些protected方法来进行一些自定义。

我们都知道双亲委派模型，其实现逻辑就位于ClassLoader的loadClass方法中，稍后一起查看。
## 二、