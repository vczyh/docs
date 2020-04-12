---
title: 设计模式：简单工厂模式-工厂模式-抽象工厂模式
date: 2020-2-6 22:32:02
updated: 2020-2-6 22:32:14
tags:
  - Java
  - 设计模式
---

## 抽象工厂

::: details 展示
![](http://p.vczyh.com/blog/20200207150055.png)
:::

::: tip 产品族
一个产品族可以理解为同一个厂家的系列产品
:::

::: tip 产品等级
一个产品等级可以理解为不同厂家的相同产品
:::

-----

现在情况是这样的，进货商要购进一批手机和电脑， 为了方便测试购进设备的功能，于是进货商提供了描述手机基本功能的抽象类和描述电脑基本功能的抽象类。

```java
public abstract class Phone {
    public abstract void start();  // 手机开机测试
}
public abstract class Computer {
    public abstract void start();  // 电脑开机测试
}
```
---

进货商又嫌购货渠道太麻烦，于是又提供了一个抽象类用于生产设备，华为和小米只要实现具体细节，进货商只需要调用方法即可，这个抽象类就是抽象工厂。

```java
public abstract class DeviceFactory {
    public abstract Phone createPhone();
    public abstract Computer createComputer();
}
```
---

小米为了给进货商供货，于是有了产品具体实现和工厂具体实现。

::: details 产品具体实现
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
:::

::: details 工厂具体实现
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
:::

---

华为的产品具体实现和工厂具体实现。

::: details 产品具体实现
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
:::

::: details 工厂具体实现
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
:::

---

进货商需要购买小米设备，只需：

```java
DeviceFactory factory = new MIDeviceFactory();
Phone phone = factory.createPhone();
phone.start(); // 测试是否可以开机
Computer computer = factory.createComputer();
computer.start();
```
---

小米设备够了，切换成购买华为设备，切换的编码成本非常小。

```java
DeviceFactory factory = new HUAWEIDeviceFactory();
Phone phone = factory.createPhone();
phone.start(); // 测试是否可以开机
Computer computer = factory.createComputer();
computer.start();
```

---

### 抽象工厂模式的四种角色

| 角色            | 描述                               |
| --------------- | ---------------------------------- |
| **AbstractFactory** | **抽象工厂**，声明了创建各个产品的方法。 |
| **ConcreteFactory** | **具体工厂**，实现抽象工厂中的方法。 |
| **AbstractProduct** | **抽象产品**，声明各个产品的业务方法。   |
| **ConcreteProduct** | **具体产品**，实现抽象产品中的方法。 |

-----

### 优点

1. 创建细节对客户端完全隐藏，这使得更换一个具体工厂变得容易，不同的环境决定软件的不同行为，因此抽象工厂得到了广泛应用。
2. 增加新的产品族很方便，只需增加具体产品和具体工厂，符合**开闭原则**。
  ::: details 开闭原则
  对扩展开放，对修改关闭，即增加新功能不会修改原来的代码，不影响原来的测试和运行。
  :::

-----

### 缺点

1. 增加新的产品等级，需要扩展现有的抽象工厂，这会涉及到全部具体工厂的修改，违背了**开闭原则**。

-----

### 开闭原则的倾斜性
1. 如果增加产品族，符合**开闭原则**。
2. 如果增加产品等级，不符合**开闭原则**。

-----

### 何时使用

1. 不关心产品创建细节，将产品创建和使用解耦。
2. 系统中有多个产品族，但每次只使用一个，如果同时购进小米和华为产品将使设计模式失去优势。
3. 根据环境不同软件有不同的行为。
4. 系统稳定后，不会添加新的产品等级或者删除已有的产品等级。

## 工厂模式

当抽象工厂只含有一个产品等级，也就是抽象工厂只声明一个方法时，抽象工厂模式退化为工厂模式， 这时候每个具体工厂只能生产一种产品。

## 简单工厂模式

当没有抽象工厂，而且具体工厂只有一个静态方法，这个方法根据不同的参数创建不同的产品时，抽象工厂模式退化为简单工厂模式。

简单工厂模式不属于23种设计模式。


