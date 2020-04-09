---
date: 2020-2-18 14:29:08
update: 2020-2-18 14:29:17
tags: 
  - JAVA
  - JVM学习笔记
  - JVM

---



### 无关性

- 平台无关：在源代码和机器指令添加了一层字节码指令
- 语言无关：只要编译后的文件符合class文件规范均可以在jvm上运行

### Class类文件结构

- Class文件是以字节为单位的二进制流，各个数据项目中间没有添加任何分隔符，紧凑的排列在文件中
- 当遇到需要占用八个字节以上空间的数据项时，则会按照高位在前（数据高位存储在地址低位）的方式分割成若干个字节进行存储
- Class只有俩种数据类型：
  - 无符号数：u1、u2、u4、u8分别代表1个字节、2个字节、4个字节、8个字节的无符号数
  - 表：由多个无符号数或者其他表作为数据项构成的复合数据类型

#### 魔数与Class文件的版本

- 魔数：唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件，比文件后缀名更具有标识性。Class文件的魔数取得很有浪漫气息，值为0xCAFEBABE（咖啡宝贝），魔数占4个字节。
- 次版本号：第5和第6个字节
  - 标识“技术预览版”功能特性的支持，一般不使用，全部固定为零
- 主版本号：第7和第8个字节
  - JDK向下兼容以前版本的Class文件，但不能运行以后版本的Class文件，例如JDK8的版本号为52.0，JDK13的版本号为57，JDK13可以运行JDK8编译的Class文件，但是JDK8不能运行JDK13编译的Class文件

#### 常量池

- 是表数据类型，被喻为Class文件里的资源仓库

- 常量池主要存放俩大类常量：

  - 字面量：文本字符串、被声明为final的常量值等
  - 符号引用：字段名称和描述符、方法名称和描述符、类和接口的全限定名、方法句柄和方法类型等

- 截止到JDK13，常量表中分别有17种不同类型的常量

  | 类型                             | 标志 | 描述                           |
  | -------------------------------- | ---- | ------------------------------ |
  | CONSTANT_Utf8_info               | 1    | UTF-8                          |
  | CONSTANT_Integer_info            | 3    | 整型字面量                     |
  | CONSTANT_Float_info              | 4    | 浮点型字面量                   |
  | CONSTANT_Long_info               | 5    | 长整型字面量                   |
  | CONSTANT_Double_info             | 6    | 双精度浮点型字面量             |
  | CONSTANT_Class_info              | 7    | 类或接口的符号引用             |
  | CONSTANT_String_info             | 8    | 字符串类型字面量               |
  | CONSTANT_Fieldref_info           | 9    | 字段的符号引用                 |
  | CONSTANT_Methodref_info          | 10   | 类中方法的符号引用             |
  | CONSTANT_InterfaceMethodref_info | 11   | 接口中方法的符号引用           |
  | CONSTANT_NameAndType_info        | 12   | 字段或方法的部分符号引用       |
  | CONSTANT_MethodHandle_info       | 15   | 表示方法句柄                   |
  | CONSTANT_MethodType_info         | 16   | 表示方法类型                   |
  | CONSTANT_Dynamic_info            | 17   | 表示一个动态计算量             |
  | CONSTANT_InvokeDynamic_info      | 18   | 表示一个动态方法调用点         |
  | CONSTANT_Module_info             | 19   | 表示一个模块                   |
  | CONSTANT_Package_info            | 20   | 表示一个模块中开放或者导出的包 |


#### 访问标志

在常量池结束后，紧接着的2个字节代表访问标志（access_flags），这个标志用于识别一些类或者接口层次的访问信息，包括：

- 这个Class是类还是接口
- 是否定义为public类型
- 是否为abstract类型
- 如果是类的话，是否被声明为final等

具体的标志位以及含义：

access_flags中一共有16个标志位可以使用，当前只定义了9个

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final，只有类可设置                              |
| ACC_SUPER      | 0x0020 | 是否允许使用invokespecial字节码指令的新语义，<br />invokespecial指令的语义在JDK1.0.2发生过改变，<br />为了区别这条指令使用哪种语义，JDK1.0.2之后<br />编译出来的类的标志都必须为真 |
| ACC_INTERFACE  | 0x0200 | 表示这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或抽象类来说，<br />此标志值为真，其他类型值为假 |
| ACC_SYNTHETIC  | 0x1000 | 标识这个类并非由用户代码产生的                               |
| ACC_ANNOTATION | 0x2000 | 标识这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标识这是一个枚举                                             |
| ACC_MODULE     | 0x8000 | 标识这是一个模块                                             |

#### 类索引、父类索引与接口索引集合

类索引（this_class）和父类索引（super_class）都是一个u2数据类型，而接口索引集合（interfaces）是一组u2类型的数据的集合，Class文件由这三项数据来确定该类型的继承关系。

- 除了Java.lang.Object外，所有类的父类索引都不为0
- 接口顺序从左到右排列在接口索引集合中
- 类索引和父类索引用俩个u2类型的索引值表示，各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过CONSTANT_Class_info类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的全限定名字符串
- 对于接口索引集合，入口的第一项u2数据类型的数据为几口计数器（interface_count），表示索引表的容量

#### 字段表集合

字段表（field_info）用于描述接口或者类中声明的变量。字段包括类级别变量以及实例级别变量，但不包括在方法内部声明的局部变量。

字段表结构：

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

字段修饰符放在access_flags项目中，与类的access_flags非常类似，都是一个u2的数据类型，其中可以设置的标志位和含义如下：

| 标志名称      | 标志值 | 含义                     |
| ------------- | ------ | ------------------------ |
| ACC_PUBLIC    | 0x0001 | 字段是否public           |
| ACC_PRIVATE   | 0x0002 | 字段是否private          |
| ACC_PROTECTED | 0x0004 | 字段是否protected        |
| ACC_STATIC    | 0x0008 | 字段是否static           |
| ACC_FINAL     | 0x0010 | 字段是否final            |
| ACC_VOLATILE  | 0x0040 | 字段是否volatile         |
| ACC_TRANSIENT | 0x0080 | 字段是否transient        |
| ACC_SYNTHETIC | 0x1000 | 字段是否由编译器自动产生 |
| ACC_ENUM      | 0x4000 | 是否是enum               |

name_index和descriptor_index都是对常量池项的引用，分别代表字段的简单名称、字段和方法的描述符

- 全限定名：com/vczyh/TestClass
- 简单名称：没有类型和参数修饰的方法或者字段的名称
- 字段描述符：例如 D 表示double类型
- 方法描述符：例如`(I)Ljava/lang/String` 表示 `String toStr(int a)`

attribute_info和属性表有关，  descriptor_index后面会跟着一个属性表集合，用于存储一些额外信息，字段表可以在属性表中附加描述零至多项的额外信息

#### 方法表集合

方法表（method_info）结构和字段表结构一样，但方法访问标志有所不同：

| 标志名称         | 标志值 | 含义                                 |
| ---------------- | ------ | ------------------------------------ |
| ACC_PUBLIC       | 0x0001 | 方法是否为public                     |
| ACC_PRIVATE      | 0x0002 | 方法是否为private                    |
| ACC_PROTECTED    | 0x0004 | 方法是否为protected                  |
| ACC_STATIC       | 0x0008 | 方法是否为static                     |
| ACC_FINAL        | 0x0010 | 方法是否为final                      |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否为synchronized               |
| ACC_BRIDGE       | 0x0040 | **方法是不是由编译器产生的桥接方法** |
| ACC_VARARGS      | 0x0080 | 方法是否接受不定参数                 |
| ACC_NATIVE       | 0x0100 | 方法是否为native                     |
| ACC_ABSTRACT     | 0x0400 | 方法是否为abstract                   |
| ACC_STRICT       | 0x0800 | **方法是否为strictfp**               |
| ACC_SYNTHETIC    | 0x1000 | 方法是否由编译器自动产生             |

方法中的代码存放在方法属性表集合中一个名为“Code”的属性里面，属性表作为Class文件格式中最具扩展性的一种数据项目

- 父类方法在子类中没有被重写，方法表集合中就不会出现来自父类的方法信息，但可能会出现由编译器自动添加的方法，最常见的便是**类构造器”`<clinit>`“**方法和**实例构造器”`<init>`“**方法

#### 属性表集合

Class文件、字段表、方法表都可以携带自己的属性表（attribute_info）集合，以描述某些场景专有的信息

虚拟机规范预定义的属性：

| 属性名称      | 使用位置 | 含义                       |
| ------------- | -------- | -------------------------- |
| Code          | 方法表   | Java代码编译成的字节码指令 |
| ConstantValue | 字段表   | 由final关键字定义的常量值  |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |
|               |          |                            |

### 字节码指令简介  // TODO

#### 字节码与数据类型

#### 加载和存储指令

#### 运算指令

#### 类型转换指令

#### 对象创建与访问指令

#### 控制转移指令

#### 方法调用和返回指令

#### 异常处理指令

#### 同步指令

如果方法设置了synchronized标志，执行线程就要求先成功持有管程然后才执行方法，最后完成方法或非正常完成时释放管程。虚拟机的指令集中有monitorenter和monitorexit俩条指令来支持synchronized关键字的语义。