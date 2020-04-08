---
title: JAVA设计模式：单例模式
date: 2020-2-5 19:41:56
updated: 2020-2-5 19:42:11
tags:
  - JAVA
  - 设计模式
  - 线程安全
---

要求一个类只能有一个实例，第一个想到的是把构造方法私有化，内部产生对象，由此引出下面几种单例模式的实现方式。

<!--more-->

### 懒汉式：线程安全

```java
public class Singleton {

    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {

    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

变种

```java
public class Singleton {

    public static final Singleton INSTANCE;

    static {
        INSTANCE = new Singleton();
    }

    private Singleton() {

    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### 饿汉式：线程不安全

```java
public class Singleton {

    public static Singleton instance = null;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton(); // 1
        }
        return instance;
    }
}
```

当有多个线程同时访问`getInstance()`可能会导致1执行多次

通过同步锁改进，但是在多线程环境中效率不高，因为即使instance创建后，多线程依然会相互阻塞。

```java
public class Singleton {

    public static Singleton instance = null;

    private Singleton() {

    }

    public synchronized static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

通过双重校验锁提高效率，同步锁颗粒尽可能小，一旦instance创建完成后，多线程不会造成阻塞。

```java
public class Singleton {

    public static Singleton instance = null;

    private Singleton() {

    }

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

### 静态内部类：线程安全

```java
public class Singleton {

    private final static class SingletonHolder {
        public static final Singleton INSTANCE = new Singleton();
    }

    private Singleton() {

    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

### 枚举：线程安全

```java
public enum  Singleton {

    SINGLETON;

    public static Singleton getInstance() {
        return SINGLETON;
    }
}
```

### 单例模式在Spring中的应用

// todo