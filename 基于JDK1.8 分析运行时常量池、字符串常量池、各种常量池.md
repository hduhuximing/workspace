# 基于JDK1.8 分析运行时常量池、字符串常量池、各种常量池

[TOC]



## Java中的常量池分为三种类型：

- 类文件中常量池（The Constant Pool）
- 运行时常量池（The Run-Time Constant Pool）
- String常量池

### 类文件中常量池 ---- 存在于Class文件中
所处区域：堆

诞生时间：编译时

内容概要：符号引用和字面量

class常量池是在编译的时候每个class都有的，在编译阶段，存放的是常量的符号引用。

常量池中存放的是符号信息，java虚拟机在执行指令的时候会依赖这些信息。常量池中的所有项都具有如下通用格式：

```java
cp_info {
 u1 tag;     //表示cp_info的单字节标记位
 u1 info[];  //两个或更多的字节表示这个常量的信息，信息格式由tag的值确定
}
```

Constant Type	Value
CONSTANT_Class	7
CONSTANT_Fieldref	9
CONSTANT_Methodref	10
CONSTANT_InterfaceMethodref	11
CONSTANT_String	8
CONSTANT_Integer	3
CONSTANT_Float	4
CONSTANT_Long	5
CONSTANT_Double	6
CONSTANT_NameAndType	12
CONSTANT_Utf8	1
CONSTANT_MethodHandle	15
CONSTANT_MethodType	16
CONSTANT_InvokeDynamic	18
举几个典型的例子来说明常量池中数据是如何存储的：

CONSTANT_Class结构 -- 表示类或者接口，他的格式如下：

```java
CONSTANT_Class_info {
 u1 tag;       //这个值为 CONSTANT_Class (7)
 u2 name_index;//注意这是一个index，他表示一个索引，引用的是CONSTANT_UTF8_info
}
```


注意观察 这个CONSTANT_Class_info类型的常量内部结构是由一个tag（CONSTANT_Class（7））和一个name_index组成，name_index中注意这个index，他表示一个索引的，什么的索引呢？CONSTANT_Utf8_info结构的索引，这个结构用来表示一个有效的类或者接口的二进制名称的内部形式。class文件结构中出现的类或者接口名称都是通过全限定形式来表示的，也被称作二进制名称【题外话：全限定类名含义就类似 java.lang包中定义的Object类的完全限定名称为java.lang.Object】。

那我们接着看CONSTANT_Utf8_info结构，他用于表示字符常量的值，他的结构如下所示：

```java
CONSTANT_Utf8_info {
 u1 tag;
 u2 length;
 u1 bytes[length];
}
```

我们注意到第一个tag肯定表示为：CONSTANT_Utf8（1）；后面的length指明了bytes[]数组的长度；最后一个bytes[]数组引用了上一个length作为其长度。字符常量采用改进过的UTF-8编码表示。

### 运行时常量池 ---- 存在于内存的元空间中
诞生时间：JVM运行时

内容概要：class文件元信息描述，编译后的代码数据，引用类型数据，类文件常量池。

所谓的运行时常量池其实就是将编译后的类信息放入运行时的一个区域中，用来动态获取类信息。

运行时常量池是在类加载完成之后，将每个class常量池中的符号引用值转存到运行时常量池中，也就是说，每个class都有一个运行时常量池，类在解析之后，将符号引用替换成直接引用，与全局常量池中的引用值保持一致。

### 字符串常量池 ---- 存在于堆中
从上述结果可以看出，JDK 1.6下，会出现“PermGen Space”的内存溢出，而在 JDK 1.7和 JDK 1.8 中，会出现堆内存溢出，并且 JDK 1.8中 PermSize 和 MaxPermGen 已经无效。因此，可以大致验证 JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代的结论。

字符串池里的内容是在类加载完成，经过验证，准备阶段之后在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到string pool中（记住：string pool中存的是引用值而不是具体的实例对象，具体的实例对象是在堆中开辟的一块空间存放的）。 在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个哈希表，里面存的是驻留字符串(也就是我们常说的用双引号括起来的)的引用（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个StringTable引用之后就等同被赋予了”驻留字符串”的身份。这个StringTable在每个HotSpot VM的实例只有一份，被所有的类共享。

