---
layout: post
title: '设计模式之单例模式'
tags: [code]
---

单例模式顾名思义就是每个类只有一个实例对象。一般来说，单例模式有五种实现方式：懒汉、饿汉、双重检验锁、静态内部类、枚举。一般情况下直接使用饿汉实现就好了，如果明确要求要懒加载会倾向于使用静态内部类实现，如果涉及到反序列化创建对象时会试着使用枚举的方式来实现。

## 懒汉模式实现

懒汉模式的懒体现在不调用getInstance()方法时不实例化对象。但当有多个线程并行调用 getInstance() 的时候，就会创建多个实例。也就是说在多线程下不能正常工作。

```java
/**
 * 单例模式（懒汉模式实现）线程不安全
 **/
public class Singleton {

    // 1.构造方法私有化 保证外部无法实例化该对象
    private Singleton() {
    }

    // 2.静态变量确保唯一性
    private static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

}
```

## 饿汉模式实现

饿汉模式饿体现在不管有没有调用getInstance()方法都会实例化对象。线程安全，`final` 关键字确保实例在初始化后不会再被改变 

```java
/**
 * 单例模式（饿汉模式实现）线程安全的
 **/

public class Singleton2 {

    private Singleton2() {
    }

    private static final Singleton2 instance = new Singleton2();

    public static Singleton2 getInstance() {
        return instance;
    }
}
```

## 双重检验锁实现

**加锁实现线程安全**

```java
/**
 * 线程安全，且需要对象时才实例化对象，但每次获取对象都要同步，耗时
 **/
public class Singleton {

    private Singleton() {
    }

    private static Singleton instance;

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

这种实现的缺点：
实际上只有第一次执行该方法时，才真正需要同步。而给该方法加上synchronized关键字，每次调用都会加锁，是种累赘。

**更好的一种实现--使用双重检查加锁**

使用双重检查加锁，首先检查该实例是否已经创建，如果尚未创建才进行同步操作。这样就只有一次同步操作

```java
/**
 * 单例模式（双重校验锁实现）线程安全的
 **/
public class Singleton {

    private Singleton() {
    }

    private volatile static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

## 静态内部类实现

由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候不会进行同步，没有性能缺陷。

```java
/**
 * 单例模式（静态内部类实现）线程安全的
 **/
public class Singleton3 {
    private Singleton3() {
    }

    public static Singleton3 getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton3 INSTANCE = new Singleton3();
    }
}

```

## 枚举实现

在所有的单例实现方式中，枚举是一种在代码写法上最简单的方式，之所以代码十分简洁，是因为Java给我们提供了`enum`关键字，我们便可以很方便的声明一个枚举类型，而不需要关心其初始化过程中的线程安全问题，因为枚举类在被虚拟机加载的时候会保证线程安全的被初始化。

除此之外，在序列化方面，Java中有明确规定，枚举的序列化和反序列化是有特殊定制的。这就可以避免反序列化过程中由于反射而导致的单例被破坏问题。

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() { 
        
    }  
}  
```

