---
title: JAVA设计模式：代理模式
date: 2018-8-17 12:19:25
tags: 
  - JAVA
  - 设计模式
---

代理模式三种角色：

- 接口：规定具体方法
- 委托类：实现接口，完成具体的业务逻辑
- 代理类：实现接口，在方法里面调用委托类的方法，自己不实现核心业务，在调用委托类的方法前后可以执行其他操作，也就是增强

<!--more-->

**代理重要的特征就是：** **委托类和代理类实现了相同的接口**，由此对于调用者来说委托类和代理类没有什么不同，而且代理类的定制性更强，这也是和适配器模式的重要区别。

### **静态代理**

静态代理是在编译时期就将接口，实现类，代理类全部编码，也就是在运行之前所有的class文件已经存在。

**接口：**

``` java
public interface Service {

    int increaseOne(int num);

    String toUpperCase(String str);
}
```
**接口实现类（委托类）：**

``` java
public class ServiceImpl implements Service {

    @Override
    public int increaseOne(int num) {
        return ++num;
    }

    @Override
    public String toUpperCase(String str) {
        return str.toUpperCase();
    }
}
```

**代理类：**

``` java
public class ServiceProxy implements Service{

    private Service service;

    public ServiceProxy(Service service) {
        this.service = service;
    }

    @Override
    public int increaseOne(int num) {
        System.out.println("将要执行increaseOne");
        int i = service.increaseOne(num);
        System.out.println("increaseOne执行完毕");
        return i;
    }

    @Override
    public String toUpperCase(String str) {
        return service.toUpperCase(str);
    }
}
```

**测试类：**

``` java
public class MainTest {

    public static void main(String[] args) {

        Service service = new ServiceImpl();
        ServiceProxy serviceProxy = new ServiceProxy(service);
        /**
         * 在执行真正 increaseOne方法之前或者之后可以执行一系列操作
         * 这里用sout模拟
         */
        System.out.println(serviceProxy.increaseOne(5));

        /**
         * 调用真正的 toUpperCase方法
         */
        System.out.println(serviceProxy.toUpperCase("hello world"));
    }
}
```

**输出：**

``` java
将要执行increaseOne
increaseOne执行完毕
6
HELLO WORLD
```

### **JDK动态代理**

JDK动态代理简化了静态代理的繁琐操作，不需要人工硬编码代理类实现接口。

**实现`InvocationHandler接口`：**

``` java
public class DynamicProxyHandler implements InvocationHandler {

    private Object target;

    public DynamicProxyHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result;
        if ("increaseOne".equals(method.getName())) {
            System.out.println("将要执行increaseOne");
            result = method.invoke(target, args);
            System.out.println("increaseOne执行完毕");
        } else {
            result = method.invoke(target, args);
        }

        return result;
    }

```

**测试类：**

``` java
public class MainTest {

    public static void main(String[] args) {

        Service service = new ServiceImpl();

        DynamicProxyHandler dynamicProxyHandler = new DynamicProxyHandler(service);

        Service serviceProxy = (Service) Proxy.newProxyInstance(
                Thread.currentThread().getContextClassLoader(),
                service.getClass().getInterfaces(), dynamicProxyHandler);

        System.out.println(serviceProxy.increaseOne(5));
        System.out.println(serviceProxy.toUpperCase("hello world"));
    }
}

```

**输出：**

``` java
将要执行increaseOne
increaseOne执行完毕
6
HELLO WORLD
```

### 比较分析

**静态代理的实现方式：**

代理类实现接口，实例化代理类，代理类的成员变量引用委托类对象，代理类调用方法，方法内部委托类对象调用方法。

**JDK动态代理的实现方式：**

与静态代理的过程是一样的，但是代理类编码是由JVM完成，我们只需要关注方法的实现细节。

- **实现`InvocationHandler`接口：**

  先看看`InvocationHandler`接口：

  ``` java
  public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
  }
  ```

  只有`invoke`方法，三个参数：

  | 类型     | 参数   | 说明                           |
  | -------- | ------ | ------------------------------ |
  | Object   | proxy  | 自动生成的代理对象，后面详细说 |
  | Method   | method | 调用的方法                     |
  | Object[] | args   | 调用的方法的参数               |

  实现：`method.invoke(target, args)`使用反射让委托对象调用方法，`target`就是委托对象

  ``` java
  public class DynamicProxyHandler implements InvocationHandler {
  
    private Object target;
    
    public DynamicProxyHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result;
        result = method.invoke(target, args);
        return result;
    }
  }
  ```

- **生成代理对象：**通过`Proxy`类的静态方法`newProxyInstance`  

  ```java
  public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
  ```

  `newProxyInstance`需要三个参数：

  | 类型              | 参数       | 说明                                          |
  | ----------------- | ---------- | --------------------------------------------- |
  | ClassLoader       | loadery    | 类加载器，使用类的或者当前线程的都可以        |
  | Class<?>          | interfaces | 代理类代理的接口，相当于示例中的`Service`接口 |
  | InvocationHandler | h          | 回调对象，代理方法的具体实现细节在这里完成    |

  

- **具体过程：**

  - 调用`Proxy.newProxyInstance(ClassLoader, Class<?>[] , InvocationHandler)`

    ``` java
     public static Object newProxyInstance(ClassLoader loader,
                                              Class<?>[] interfaces,
                                              InvocationHandler h)
            throws IllegalArgumentException
        {
            Objects.requireNonNull(h);
    
            final Class<?>[] intfs = interfaces.clone();
            final SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
            }
    
            /*
             * Look up or generate the designated proxy class.
             */
            Class<?> cl = getProxyClass0(loader, intfs);
    
            /*
             * Invoke its constructor with the designated invocation handler.
             */
            try {
                if (sm != null) {
                    checkNewProxyPermission(Reflection.getCallerClass(), cl);
                }
    
                final Constructor<?> cons = cl.getConstructor(constructorParams);
                final InvocationHandler ih = h;
                if (!Modifier.isPublic(cl.getModifiers())) {
                    AccessController.doPrivileged(new PrivilegedAction<Void>() {
                        public Void run() {
                            cons.setAccessible(true);
                            return null;
                        }
                    });
                }
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
    ```

    重要的有俩步：

    -  `Class<?> cl = getProxyClass0(loader, intfs)`根据类加载器和接口参数会生成代理类的字节码，
    -  `return cons.newInstance(new Object[]{h})`返回生成代理类的实例

    来看看生成的代理类（在内存中生成.class文件，需要先输出保存，再反编译）：

    ``` java
    public final class $ServiceProxy extends Proxy implements Service {
        private static Method m1;
        private static Method m2;
        private static Method m3;
        private static Method m4;
        private static Method m0;
    
        public $ServiceProxy(InvocationHandler var1) throws  {
            super(var1);
        }
    
        public final boolean equals(Object var1) throws  {
            try {
                return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }
    
        public final String toString() throws  {
            try {
                return (String)super.h.invoke(this, m2, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }
    
        public final String toUpperCase(String var1) throws  {
            try {
                return (String)super.h.invoke(this, m3, new Object[]{var1});
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }
    
        public final int increaseOne(int var1) throws  {
            try {
                return (Integer)super.h.invoke(this, m4, new Object[]{var1});
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }
    
        public final int hashCode() throws  {
            try {
                return (Integer)super.h.invoke(this, m0, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }
    
        static {
            try {
                m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
                m2 = Class.forName("java.lang.Object").getMethod("toString");
                m3 = Class.forName("com.Service").getMethod("toUpperCase", Class.forName("java.lang.String"));
                m4 = Class.forName("com.Service").getMethod("increaseOne", Integer.TYPE);
                m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            } catch (NoSuchMethodException var2) {
                throw new NoSuchMethodError(var2.getMessage());
            } catch (ClassNotFoundException var3) {
                throw new NoClassDefFoundError(var3.getMessage());
            }
        }
    }
    
    ```

    生成的代理类实现l了`Service`接口，这和静态代理的代码类似，但是这里还继承了`Proxy` 类，构造函数接受`InvocationHandler`实现类对象，然后调用`Proxy`的构造函数，把对象传给`Proxy`的`InvocationHandler`类型成员变量，示例中获取代理对象后有个强制转换类型，之前一直有疑问为什么不会报错，直到反编译代理类后看到实现了`Service`接口才明白。

  - 代理类对象调用接口方法：

    继续看代理类，例如调用`increaseOne`方法

    ``` java
        public final int increaseOne(int var1) throws  {
            try {
                return (Integer)super.h.invoke(this, m4, new Object[]{var1});
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }
    ```

    其实最终调用的就是我们传入的`InvocationHandler`对象的`invoke`方法，`this`和`new Object[]{var1}`

    可以理解，但是`m4`什么时候生成的呢

    代理类有个`static`代码块：

    ``` java
        static {
            try {
                m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
                m2 = Class.forName("java.lang.Object").getMethod("toString");
                m3 = Class.forName("com.Service").getMethod("toUpperCase", Class.forName("java.lang.String"));
                m4 = Class.forName("com.Service").getMethod("increaseOne", Integer.TYPE);
                m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            } catch (NoSuchMethodException var2) {
                throw new NoSuchMethodError(var2.getMessage());
            } catch (ClassNotFoundException var3) {
                throw new NoClassDefFoundError(var3.getMessage());
            }
        }
    ```

    可以看到`m3`和`m4`是`Service`接口方法的引用，其他的三个方法引用是`Object`类的`equals`，`toString`以及`hashCode`方法，而且这三个方法里面也调用了`invoke`方法，所以不能在`invoke`方法里面输出`proxy`对象，相当于循环调用方法。

### CGLIB动态代理

CGLIB代理的是类，不同于JDK只能代理接口

**委托类：**

```  java
public class DelegateClass {

    public int increaseOne(int num) {
        return ++num;
    }

    public String toUpperCase(String str) {
        return str.toUpperCase();
    }
}

```

**代理类：**

  ``` java
public class CGLIBProxy implements MethodInterceptor {

    private Object target;

    public CGLIBProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        Object result;
        if ("increaseOne".equals(method.getName())) {
            System.out.println("将要执行increaseOne");
            result = method.invoke(target, args);
            System.out.println("increaseOne执行完毕");
        } else {
            result = method.invoke(target, args);
        }
        return result;
    }
}
  ```

**测试类：**

``` java
public class MainTest {

    public static void main(String[] args) {

        DelegateClass delegateClass = new DelegateClass();
        MethodInterceptor methodInterceptor = new CGLIBProxy(delegateClass);

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(DelegateClass.class);
        enhancer.setCallback(methodInterceptor);
        DelegateClass cglibProxy = (DelegateClass) enhancer.create();

        System.out.println(cglibProxy.increaseOne(5));
        System.out.println(cglibProxy.toUpperCase("hello world"));
    }
}
```

**分析：**

- **自动生成的代理类：**

  ``` java
  public class DelegateClass$$EnhancerByCGLIB$$50d8bc59 extends DelegateClass implements Factory {
      private boolean CGLIB$BOUND;
      public static Object CGLIB$FACTORY_DATA;
      private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
      private static final Callback[] CGLIB$STATIC_CALLBACKS;
      private MethodInterceptor CGLIB$CALLBACK_0;
      private static Object CGLIB$CALLBACK_FILTER;
      private static final Method CGLIB$toUpperCase$0$Method;
      private static final MethodProxy CGLIB$toUpperCase$0$Proxy;
      private static final Object[] CGLIB$emptyArgs;
      private static final Method CGLIB$increaseOne$1$Method;
      private static final MethodProxy CGLIB$increaseOne$1$Proxy;
      private static final Method CGLIB$equals$2$Method;
      private static final MethodProxy CGLIB$equals$2$Proxy;
      private static final Method CGLIB$toString$3$Method;
      private static final MethodProxy CGLIB$toString$3$Proxy;
      private static final Method CGLIB$hashCode$4$Method;
      private static final MethodProxy CGLIB$hashCode$4$Proxy;
      private static final Method CGLIB$clone$5$Method;
      private static final MethodProxy CGLIB$clone$5$Proxy;
  
      static void CGLIB$STATICHOOK1() {
          CGLIB$THREAD_CALLBACKS = new ThreadLocal();
          CGLIB$emptyArgs = new Object[0];
          Class var0 = Class.forName("com.cglibproxy.DelegateClass$$EnhancerByCGLIB$$50d8bc59");
          Class var1;
          Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
          CGLIB$equals$2$Method = var10000[0];
          CGLIB$equals$2$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$2");
          CGLIB$toString$3$Method = var10000[1];
          CGLIB$toString$3$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$3");
          CGLIB$hashCode$4$Method = var10000[2];
          CGLIB$hashCode$4$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$4");
          CGLIB$clone$5$Method = var10000[3];
          CGLIB$clone$5$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$5");
          var10000 = ReflectUtils.findMethods(new String[]{"toUpperCase", "(Ljava/lang/String;)Ljava/lang/String;", "increaseOne", "(I)I"}, (var1 = Class.forName("com.cglibproxy.DelegateClass")).getDeclaredMethods());
          CGLIB$toUpperCase$0$Method = var10000[0];
          CGLIB$toUpperCase$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)Ljava/lang/String;", "toUpperCase", "CGLIB$toUpperCase$0");
          CGLIB$increaseOne$1$Method = var10000[1];
          CGLIB$increaseOne$1$Proxy = MethodProxy.create(var1, var0, "(I)I", "increaseOne", "CGLIB$increaseOne$1");
      }
  
      final String CGLIB$toUpperCase$0(String var1) {
          return super.toUpperCase(var1);
      }
  
      public final String toUpperCase(String var1) {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          return var10000 != null ? (String)var10000.intercept(this, CGLIB$toUpperCase$0$Method, new Object[]{var1}, CGLIB$toUpperCase$0$Proxy) : super.toUpperCase(var1);
      }
  
      final int CGLIB$increaseOne$1(int var1) {
          return super.increaseOne(var1);
      }
  
      public final int increaseOne(int var1) {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          if (var10000 != null) {
              Object var2 = var10000.intercept(this, CGLIB$increaseOne$1$Method, new Object[]{new Integer(var1)}, CGLIB$increaseOne$1$Proxy);
              return var2 == null ? 0 : ((Number)var2).intValue();
          } else {
              return super.increaseOne(var1);
          }
      }
  
      final boolean CGLIB$equals$2(Object var1) {
          return super.equals(var1);
      }
  
      public final boolean equals(Object var1) {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          if (var10000 != null) {
              Object var2 = var10000.intercept(this, CGLIB$equals$2$Method, new Object[]{var1}, CGLIB$equals$2$Proxy);
              return var2 == null ? false : (Boolean)var2;
          } else {
              return super.equals(var1);
          }
      }
  
      final String CGLIB$toString$3() {
          return super.toString();
      }
  
      public final String toString() {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          return var10000 != null ? (String)var10000.intercept(this, CGLIB$toString$3$Method, CGLIB$emptyArgs, CGLIB$toString$3$Proxy) : super.toString();
      }
  
      final int CGLIB$hashCode$4() {
          return super.hashCode();
      }
  
      public final int hashCode() {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          if (var10000 != null) {
              Object var1 = var10000.intercept(this, CGLIB$hashCode$4$Method, CGLIB$emptyArgs, CGLIB$hashCode$4$Proxy);
              return var1 == null ? 0 : ((Number)var1).intValue();
          } else {
              return super.hashCode();
          }
      }
  
      final Object CGLIB$clone$5() throws CloneNotSupportedException {
          return super.clone();
      }
  
      protected final Object clone() throws CloneNotSupportedException {
          MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
          if (this.CGLIB$CALLBACK_0 == null) {
              CGLIB$BIND_CALLBACKS(this);
              var10000 = this.CGLIB$CALLBACK_0;
          }
  
          return var10000 != null ? var10000.intercept(this, CGLIB$clone$5$Method, CGLIB$emptyArgs, CGLIB$clone$5$Proxy) : super.clone();
      }
  
      public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
          String var10000 = var0.toString();
          switch(var10000.hashCode()) {
          case -1961280283:
              if (var10000.equals("increaseOne(I)I")) {
                  return CGLIB$increaseOne$1$Proxy;
              }
              break;
          case -508378822:
              if (var10000.equals("clone()Ljava/lang/Object;")) {
                  return CGLIB$clone$5$Proxy;
              }
              break;
          case 893465080:
              if (var10000.equals("toUpperCase(Ljava/lang/String;)Ljava/lang/String;")) {
                  return CGLIB$toUpperCase$0$Proxy;
              }
              break;
          case 1826985398:
              if (var10000.equals("equals(Ljava/lang/Object;)Z")) {
                  return CGLIB$equals$2$Proxy;
              }
              break;
          case 1913648695:
              if (var10000.equals("toString()Ljava/lang/String;")) {
                  return CGLIB$toString$3$Proxy;
              }
              break;
          case 1984935277:
              if (var10000.equals("hashCode()I")) {
                  return CGLIB$hashCode$4$Proxy;
              }
          }
  
          return null;
      }
  
      public DelegateClass$$EnhancerByCGLIB$$50d8bc59() {
          CGLIB$BIND_CALLBACKS(this);
      }
  
      public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {
          CGLIB$THREAD_CALLBACKS.set(var0);
      }
  
      public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {
          CGLIB$STATIC_CALLBACKS = var0;
      }
  
      private static final void CGLIB$BIND_CALLBACKS(Object var0) {
          DelegateClass$$EnhancerByCGLIB$$50d8bc59 var1 = (DelegateClass$$EnhancerByCGLIB$$50d8bc59)var0;
          if (!var1.CGLIB$BOUND) {
              var1.CGLIB$BOUND = true;
              Object var10000 = CGLIB$THREAD_CALLBACKS.get();
              if (var10000 == null) {
                  var10000 = CGLIB$STATIC_CALLBACKS;
                  if (CGLIB$STATIC_CALLBACKS == null) {
                      return;
                  }
              }
  
              var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];
          }
  
      }
  
      public Object newInstance(Callback[] var1) {
          CGLIB$SET_THREAD_CALLBACKS(var1);
          DelegateClass$$EnhancerByCGLIB$$50d8bc59 var10000 = new DelegateClass$$EnhancerByCGLIB$$50d8bc59();
          CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
          return var10000;
      }
  
      public Object newInstance(Callback var1) {
          CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});
          DelegateClass$$EnhancerByCGLIB$$50d8bc59 var10000 = new DelegateClass$$EnhancerByCGLIB$$50d8bc59();
          CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
          return var10000;
      }
  
      public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {
          CGLIB$SET_THREAD_CALLBACKS(var3);
          DelegateClass$$EnhancerByCGLIB$$50d8bc59 var10000 = new DelegateClass$$EnhancerByCGLIB$$50d8bc59;
          switch(var1.length) {
          case 0:
              var10000.<init>();
              CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
              return var10000;
          default:
              throw new IllegalArgumentException("Constructor not found");
          }
      }
  
      public Callback getCallback(int var1) {
          CGLIB$BIND_CALLBACKS(this);
          MethodInterceptor var10000;
          switch(var1) {
          case 0:
              var10000 = this.CGLIB$CALLBACK_0;
              break;
          default:
              var10000 = null;
          }
  
          return var10000;
      }
  
      public void setCallback(int var1, Callback var2) {
          switch(var1) {
          case 0:
              this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;
          default:
          }
      }
  
      public Callback[] getCallbacks() {
          CGLIB$BIND_CALLBACKS(this);
          return new Callback[]{this.CGLIB$CALLBACK_0};
      }
  
      public void setCallbacks(Callback[] var1) {
          this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];
      }
  
      static {
          CGLIB$STATICHOOK1();
      }
  }
  ```

  类继承了委托类`DelegateClass`，**测试类**代码中一行代码

  ``` java
  DelegateClass cglibProxy = (DelegateClass) enhancer.create();
  ```

  `cglibProxy`引用的就是`DelegateClass$$EnhancerByCGLIB$$50d8bc59`

  的实例，属于子类强制转父类，除了代理`increaseOne`和`toUpperCase`方法外，还有`toString`，`equal`，`hashCode`，`clone`方法也被代理，代理类的方法会判断是否有回调对象，也就是`MethodInterceptor`的实例，有的话调用回调对象的`intercept`方法，否则使用`super.increaseOne(var1)`调用委托类的方法，没有增强处理。
