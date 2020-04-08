---
title: JAVA设计模式：原型模式与拷贝
author: vczyh
date: 2020-2-4 22:06:15
updated: 2020-2-4 22:06:34
tags: 
  - JAVA
  - 设计模式
  - 深拷贝
  - 浅拷贝
---

### 深拷贝与浅拷贝

**浅拷贝：**对象A进行赋值操作得到对象B，这就是浅拷贝，修改对象A的属性会影响到B的属性

```java
// 引用类型 sb1调用自身方法会影响到sb2，赋值操作就是对地址的复制，指向同一个实例
StringBuilder sb1 = new StringBuilder("hello");
StringBuilder sb2 = sb1;
sb1.append(" world");
System.out.println(sb1.toString());  // hello world
System.out.println(sb2.toString());  // hello world
```

**深拷贝：**深拷贝就是希望对象A和对象B的操作互不影响。

<!--more-->

#### 如何实现深拷贝

```java
// 对User的对象进行深拷贝
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    private String name;
    private Address address;

}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Address {
    String province;
    String city;
}
```

##### 方法一：使用new

```java
// 被复制的对象
Address address = new Address("hebei", "zhangjiakou");
User user = new User("zhang", address);
// 使用 new 深拷贝
Address addressCopy = new Address(address.getProvince(), address.getCity());
User userCopy = new User(user.getName(), addressCopy);
```

当嵌套的对象越来越多，这种方法显得繁琐而且易出错

##### 方法二：使用clone()

既然是复制，那么可以把User实例所在的内存区域拷贝一份，然后用新引用指向新区域，事实上Java也提供了这样的操作，即 `Object.clone()`

进行拷贝的类需要实现`Cloneable`接口，这是个标记接口，没有任何方法，实现这个接口的类表示调用`clone()`合法。不实现`Cloneable`调用`clone()`会抛出`CloneNotSupportedException`

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User implements Cloneable{

    private String name;
    private Address address;

    @Override
    protected User clone() throws CloneNotSupportedException {
        return (User) super.clone();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Address {
    String province;
    String city;
}
```

```java
// 被复制的对象
Address address = new Address("hebei", "zhangjiakou");
User user = new User("zhang", address);
// 使用 clone() 深拷贝
User userCopy = user.clone();
// 检查
user.getAddress().setCity("handan");
System.out.println(userCopy.getAddress().getCity()); // handan
```

这依然是浅拷贝，因为user实例内存区域的address对象依然是个地址，所以需要对address进行拷贝。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User implements Cloneable {

    private String name;
    private Address address;
	
    // change
    @Override
    protected User clone() throws CloneNotSupportedException {
        User user = (User) super.clone();
        Address address = user.getAddress().clone();
        user.setAddress(address);
        return user;
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Address implements Cloneable {
    String province;
    String city;
	
    // change
    @Override
    protected Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }
}
```

```java
// 被复制的对象
Address address = new Address("hebei", "zhangjiakou");
User user = new User("zhang", address);
// 使用 clone() 深拷贝
User userCopy = user.clone();
// 检查
user.getAddress().setCity("handan");
System.out.println(userCopy.getAddress().getCity()); // zhangjiakou
```

这样在调用上比new优雅许多，但在`clone()`里面也需要注意嵌套调用，那么有没有更方便的方法呢。

##### 方法三：序列化

首先是JAVA自带的序列化功能

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User implements Serializable {

    private String name;
    private Address address;
	// change
    public User deepClone() throws IOException, ClassNotFoundException {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(this);
        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);
        return (User) ois.readObject();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class Address implements Serializable {
    String province;
    String city;
}
```

```java
// 被复制的对象
Address address = new Address("hebei", "zhangjiakou");
User user = new User("zhang", address);
// 使用 Serialize 深拷贝 change
User userCopy = user.deepClone();
// 检查
user.getAddress().setCity("handan");
System.out.println(userCopy.getAddress().getCity()); // zhangjiakou
```

使用JSON序列化也可以

```java
// 被复制的对象
Address address = new Address("hebei", "zhangjiakou");
User user = new User("zhang", address);
// 使用 JSON序列化 深拷贝
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(user);
User userCopy = mapper.readValue(json, User.class);
// 检查
user.getAddress().setCity("handan");
System.out.println(userCopy.getAddress().getCity()); // zhangjiakou
```

### 原型模式

简单来说，原型模式就是通过一个方法获得一个实例的深拷贝，这里的深拷贝是通过`clone()`，具体代码就是上面的代码，原型模式很简单，主要是理解浅拷贝和深拷贝。

### 原型模式在Spring中的应用

// todo

