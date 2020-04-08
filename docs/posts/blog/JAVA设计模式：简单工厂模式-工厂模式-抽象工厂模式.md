---
title: JAVA设计模式：简单工厂模式-工厂模式-抽象工厂模式
date: 2020-2-6 22:32:02
updated: 2020-2-6 22:32:14
tags:
  - JAVA
  - 设计模式
---

### 概述

一直不清楚这三个工厂模式的区别，于是看了几篇博客，想着把看到的和自己的理解梳理一下，于是有了这篇博客。我认为学习设计模式尽量结合平时的编码习惯以及一些优秀框架的源码学习，前者让我们关注设计模式如何改进编码风格，例如写web项目一般分service，serviceImpl，然后实例的引用是service，而不是serviceImpl，这是利用了oop的多态，后者很考验框架设计人员能力，spring自从设计以来都是在基础上不断扩展，这正是由于spring运用了大量设计模式，在刚开始功能简单和模块少的时候就写了大量看似冗余的代码，但这些代码为spring提供了极大扩展性和易读性。

<!--more-->

### 抽象工厂

![](http://p.vczyh.com/blog/20200207150055.png)

一个产品族可以理解为同一个厂家的系列产品，一个产品等级可以理解为不同厂家的相同产品。

现在情况是这样的，进货商要购进一批手机和电脑， 为了方便测试购进设备的功能，于是进货商提供了描述手机基本功能的抽象类和描述电脑基本功能的抽象类

```java
public abstract class Phone {
    public abstract void start();  // 手机开机测试
}
public abstract class Computer {
    public abstract void start();  // 电脑开机测试
}

```

进货商又嫌购货渠道太麻烦，于是又提供了一个抽象类用于生产设备，华为和小米只要实现具体细节，进货商只需要调用方法即可，这个抽象类就是抽象工厂

```java
public abstract class DeviceFactory {
    public abstract Phone createPhone();
    public abstract Computer createComputer();
}
```



小米为了给进货商供货，于是有了产品具体实现和工厂具体实现

```java
public class MIPhone extends Phone {
    @Override
    public void start() {
        System.out.println("成功开机");
    }
}
public class MIComputer extends Computer {
    @Override
    public void start() {
        System.out.println("成功开机");
    }
}

```

```java
public class MIDeviceFactory extends DeviceFactory {
    @Override
    public Phone createPhone() {
        return new MIPhone();
    }

    @Override
    public Computer createComputer() {
        return new MIComputer();
    }
}
```

华为同样

```java
public class HUAWEIPhone extends Phone {
    @Override
    public void start() {
        System.out.println("成功开机");
    }
}
public class HUAWEIComputer extends Computer {
    @Override
    public void start() {
        System.out.println("成功开机");
    }
}
```

```java
public class HUAWEIDeviceFactory extends DeviceFactory {
    @Override
    public Phone createPhone() {
        return new HUAWEIPhone();
    }

    @Override
    public Computer createComputer() {
        return new HUAWEIComputer();
    }
}
```

进货商需要购买小米设备，只需

```java
DeviceFactory factory = new MIDeviceFactory();
Phone phone = factory.createPhone();
phone.start(); // 测试是否可以开机
Computer computer = factory.createComputer();
computer.start();
```

小米设备够了，切换成购买华为设备，切换的编码成本非常小

```java
DeviceFactory factory = new HUAWEIDeviceFactory();
Phone phone = factory.createPhone();
phone.start(); // 测试是否可以开机
Computer computer = factory.createComputer();
computer.start();
```

抽象工厂模式的四种角色：

- AbstractFactory：抽象工厂，声明了创建各个产品的方法
- ConcreteFactory：具体工厂，实现抽象工厂中声明的方法
- AbstractProduct：抽象产品，为各个产品声明业务方法
- ConcreteProduct：具体产品，实现抽象产品中声明的方法

#### 优点

- 创建细节对客户端完全隐藏，这使得更换一个具体工厂变得容易，不同的环境决定软件的不同行为，因此抽象工厂得到了广泛应用

- 增加新的产品族很方便，只需增加具体产品和具体工厂，符合”开闭原则“

  > 开闭原则：对扩展开放，对修改关闭，增加新功能新模块不会修改原来代码，不影响原来的测试和运行。

#### 缺点

- 增加新的产品等级，需要扩展现有的抽象工厂，这会涉及到全部具体工厂的修改，违背了“开闭原则”
- 开闭原则的倾斜性
  - 增加产品族：符合”开闭原则“
  - 增加产品等级：不符合”开闭原则“

#### 何时使用

- 不关心产品创建细节，将产品创建和使用解耦
- 系统中有多个产品族，但每次只使用一个，示例中如果同时购进小米产品和华为产品将使设计模式失去优势
- 根据环境不同软件有不同的行为
- 系统稳定后，不会添加新的产品等级或者删除已有的产品等级

### 工厂模式

当抽象工厂只含有一个产品等级，也就是抽象工厂只声明一个方法时，抽象工厂模式退化为工厂模式， 这时候每个具体工厂只能生产一种产品。

### 简单工厂模式

没有抽象工厂，具体工厂只有一个创建产品实例的静态方法，这个方法根据不同的参数创建不同的产品。简单工厂模式不属于23种设计模式，因为它太简单了。

### 总结

虽然有这么多工厂，但是使用最广泛，最好用的还是抽象工厂模式。

