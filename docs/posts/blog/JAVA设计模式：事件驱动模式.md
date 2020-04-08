---
title: JAVA设计模式：事件驱动模式
date: 2018-8-14 11:24:10
updated: 2020-2-5 15:12:38
tags:
  - JAVA
  - 设计模式
---

### 角色

- 事件
- 事件源
- 事件监听器

<!--more-->

### 事件

事件类一般继承自`java.util.EventObject`类，封装了事件源以及跟事件有关的信息

`source`：事件源
`getSource`()：获取事件源

```java
public class EventObject implements java.io.Serializable {

    private static final long serialVersionUID = 5516075349620653480L;

    /**
     * The object on which the Event initially occurred
     */
    protected transient Object  source;

    /**
     * Constructs a prototypical Event.
     *
     * @param    source    The object on which the Event initially occurred.
     * @exception  IllegalArgumentException  if source is null.
     */
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");

        this.source = source;
    }

    /**
     * The object on which the Event initially occurred.
     *
     * @return   The object on which the Event initially occurred.
     */
    public Object getSource() {
        return source;
    }

    /**
     * Returns a String representation of this EventObject.
     *
     * @return  A a String representation of this EventObject.
     */
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```

### 事件源

每个事件包含一个事件源，事件源是事件发生的地方，由于事件源的某项属性或状态改变时，就会生成相应的事件对象，然后将事件对象通知给所有监听该事件源的监听器。

### 事件监听器

事件监听器一般需要实现`java.util.EventListener接口 `

``` java
public interface EventListener {
}
```

`EventListener` 是个空接口，监听器必须要有回调方法供事件源回调，这个回调方法可以在继承或者实现`EventListener` 接口的时候自定义。

### 实现源码

- **事件监听器：**事件监听器可以直接实现`EventListener`接口，但是一般框架会有自己的监听器接口，所以这里先继承`EventListener`接口

  ``` java
  public interface ApplicationListener extends EventListener{
  
      void onApplicationEvent(ApplicationListener event);
  }
  ```

- **事件：**事件类可以直接使用`EventObject`，这里通过继承`EventObject`类实现自己的事件类

  ``` java
  public class ApplicationEvent  extends EventObject {
      /**
       * Constructs a prototypical Event.
       *
       * @param source The object on which the Event initially occurred.
       * @throws IllegalArgumentException if source is null.
       */
      public ApplicationEvent(Object source) {
          super(source);
      }
  }
  ```

- **运行上下文环境**

  ``` java
  public class ApplicationContext {
  
      /**
       * 存放所有的监听器
       */
      Set<ApplicationListener> listeners;
  
      public ApplicationContext() {
          this.listeners = new HashSet<>();
      }
  
      /**
       * 添加监听器
       * @param listener 监听器
       */
      public void addApplicationListener(ApplicationListener listener) {
          this.listeners.add(listener);
      }
  
      /**
       * 发布事件
       * 回调所有监听器的回调方法
       * @param event 事件
       */
      public void publishEvent(ApplicationEvent event) {
          for (ApplicationListener listener : listeners) {
              listener.onApplicationEvent(event);
          }
      }
  }
  
  ```

- **测试类**

  ``` java
  public class MainTest {
  
      public static void main(String[] args) {
  
          ApplicationContext applicationContext = new ApplicationContext();
  
          /**
           * 添加监听事件源为整型的监听器
           */
          applicationContext.addApplicationListener(event -> {
              Object source = event.getSource();
              if (source instanceof Integer) {
                  int now = (int) source;
                  System.out.println("检测到事件源为整型：事件源变为" + now);
              }
          });
  
          /**
           * 添加监听事件源为字符串类型的监听器
           */
          applicationContext.addApplicationListener(event -> {
              Object source = event.getSource();
              if (source instanceof String) {
                  String now = (String) source;
                  System.out.println("检测到事件源为字符串类型：事件源变为" + now);
              }
          });
  
          /**
           * 发布事件
           */
  //        applicationContext.publishEvent(new ApplicationEvent(1001));
          applicationContext.publishEvent(new ApplicationEvent("Hello"));
  
      }
  }
  
  ```

- 输出

  ``` 
  检测到事件源为字符串类型：事件源变为Hello
  ```


### Spring事件驱动

spring的事件驱动和上面的实现方式有些不同，最大的区别是监听器的添加方式，spring一般的添加方式是实现`ApplicationListener`接口，然后把实现类注册为Bean，spring的上下文初始化的时候会自动扫描路径下的实现`ApplicationListener`接口的类，然后添加到监听器集合里面，发布事件的时候会通知所有的监听器，这一切要依赖spring的容器管理Bean的功能。