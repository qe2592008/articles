# Java基础系列-原子变量类

## 概述

​	原子变量类指的是在`java.util.concurrent.atomic`包下面的以Atomic开头的类。

​	原子变量类是依靠volatile和CAS一起来实现操作的线程安全性的，其中volatile保证变量的可见性和有序性，CAS保证变量操作的原子性。

​	有关volatile和CAS的内容可以查看[Java基础系列-volatile关键字](https://www.cnblogs.com/V1haoge/p/7833881.html)和[ava基础系列-CAS操作]()。

## 