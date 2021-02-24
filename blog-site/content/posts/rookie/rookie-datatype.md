---
title: "面向菜鸟编程-数据类型篇"
date: 2021-01-20
draft: false
tags: ["中文", "Java", "面向菜鸟编程"]
slug: "rookie-datatype"
---
- [基本类型](#基本类型)
    - [数值类型](#数值类型)
        - [整型](#整型)
        - [浮点型](#浮点型)
    - [字符类型](#字符类型)
    - [布尔类型](#布尔类型)
- [类型转换](#类型转换)
    - [自动类型转换](#自动类型转换)
    - [隐式类型转换](#隐式类型转换)
- [包装类型](#包装类型)
    - [拆箱与装箱](#拆箱与装箱)
        - [自动拆装箱](#自动拆装箱)
        - [自动拆装箱带来的问题](#自动拆装箱带来的问题)
    - [缓存池](#缓存池)
- [引用类型](#引用类型)
    - [与基本类型区别](#与基本类型区别)
    - [注意事项](#注意事项)
- [常量](#常量)
    - [定义](#定义)
    - [常量池](#常量池)
        - [Class常量池](#class常量池)
            - [Class文件](#class文件)
            - [存放内容](#存放内容)
        - [运行时常量池](#运行时常量池)
            - [存放位置及存放来源](#存放位置及存放来源)
        - [String常量池](#string常量池)
            - [intern()](#intern)
            - [经典案例](#经典案例)
            - [优点](#优点)
- [String 类](#string-类)
    - [概览](#概览)
    - [不可变性优点](#不可变性优点)
    - [使用](#使用)
        - [创建字符串](#创建字符串)
        - [连接字符串](#连接字符串)
            - ["+"连接字符串](#连接字符串)
            - [concat方法](#concat方法)
            - [StringBuffer和StringBuilder](#stringbuffer和stringbuilder)
            - [比较和使用](#比较和使用)
        - [常用方法](#常用方法)
            - [substring](#substring)
            - [replace](#replace)
            - [replaceAll](#replaceall)
            - [valueOf](#valueof)
    - [长度限制](#长度限制)
    - [编码问题](#编码问题)
        - [Unicode](#unicode)
        - [乱码原因](#乱码原因)
        - [相关方法](#相关方法)
            - [getBytes(String charsetName)](#getbytesstring-charsetname)
            - [new String(byte bytes[], String charsetName)](#new-stringbyte-bytes-string-charsetname)

## 基本类型
Java语言提供了八种基本类型。六种数值类型（四个整数型，两个浮点型），一种字符类型，还有一种布尔型.  **俗称4类8种**


这里只介绍称4类8种.实际上，JAVA中还存在另外一种基本类型 void，它也有对应的包装类`java.lang.Void`，不过我们无法直接对它们进行操作.

> 一个Byte(字节)等于8个bit(位),bit是最小的单位,1B(字节)=8bit(位). <br>
一般情况下，一个汉字是两个字节，英文与数字是一个字节

### 数值类型

#### 整型
- byte/8
    - byte 数据类型是8位、有符号的，以二进制补码表示的整数;
    - 最小值是 `-128（-2^7)`
    - 最大值是 `127（2^7-1）`
    - 默认值是 0
    - byte 类型用在大型数组中节约空间，主要代替整数，因为byte变量占用的空间只有int类型的四分之一
    - 例子：`byte a = 100，byte b = -50`


- short/16
    - short 数据类型是16位、有符号的以二进制补码表示的整数
    - 最小值是 `-32768（-2^15）`
    - 最大值是 `32767（2^15 - 1）`
    - Short 数据类型也可以像byte那样节省空间.一个short变量是int型变量所占空间的二分之一
    - 默认值是 0
    - 例子：`short s = 1000，short r = -20000`


- int/32
    - int 数据类型是32位、有符号的以二进制补码表示的整数
    - 最小值是 `-2,147,483,648（-2^31）`
    - 最大值是 `2,147,483,647（2^31 - 1）`
    - 一般地整型变量默认为 int 类型
    - 默认值是 0 
    - 例子：`int a = 100000, int b = -200000`


- long/64
    - long 数据类型是 64 位、有符号的以二进制补码表示的整数
    - 最小值是 `-9,223,372,036,854,775,808（-2^63）`
    - 最大值是 `9,223,372,036,854,775,807（2^63 -1）`
    - 这种类型主要使用在需要比较大整数的系统上
    - 默认值是 `0L`
    - 例子： `long a = 100000L，Long b = -200000L`

#### 浮点型
- float/32
    - float 数据类型是单精度32位.符合IEEE 754标准的浮点数
    - float 在储存大型浮点数组的时候可节省内存空间
    - 最小值`1.4E-45`,最大值`3.4028235E38`
    - 默认值是` 0.0f`
    - 浮点数不能用来表示精确的值，如货币
    - 例子：`float f = 234.5f`


- double/64
    - double 数据类型是双精度、64位、符合IEEE754标准的浮点数
    - 浮点数的默认类型为double类型
    - double类型同样不能表示精确的值，如货币
    - 最小值`4.9E-324`,最大值`1.7976931348623157E308`
    - 默认值是 `0.0d`
    - 例子：`double d = 123.4`

`Float`和`Double`的最小值和最大值都是以科学记数法的形式输出的，结尾的"E+数字"表示E之前的数字要乘以10的多少次方.比如`3.14E3`就是`3.14 × 10^3 =3140`，`3.14E-3` 就是 `3.14 x 10^-3 =0.00314`.

### 字符类型
- char/16
    - char类型是一个单一的 16 位 Unicode 字符
    - 最小值是 `\u0000`（即为 0）
    - 最大值是 `\uffff`（即为65、535）
    - 默认值是 `\u0000`
    - char 数据类型可以储存任何字符
    - 例子：`char letter = 'A'`

### 布尔类型
- boolean/~
    - boolean数据类型表示一位的信息
    - 只有两个取值：true 和 false
    - 这种类型只作为一种标志来记录 true/false 情况
    - 默认值是 `false`
    - 例子：`boolean one = true`


boolean 只有两个值：`true、false`，可以使用 1 bit 来存储，但是具体大小没有明确规定。**JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。**


## 类型转换

**整型、实型（常量）、字符型数据可以混合运算。运算中，不同类型的数据先转化为同一类型，然后进行运算。**

转换从低级到高级。

```java
byte,short,char,int,long,float,double
```

数据类型转换必须满足如下规则：

- 不能对boolean类型进行类型转换。

- 不能把对象类型转换成不相关类的对象。

- 在把容量大的类型转换为容量小的类型时必须使用强制类型转换。

- 浮点数到整数的转换是通过舍弃小数得到，而不是四舍五入

  ```java
  (int)23.7 == 23;        
  (int)-45.89f == -45
  ```

- 转换过程中可能导致溢出或损失精度,在运算时要避免该问题.例如：

  ```java
  // 因为 byte 类型是 8 位，最大值为127，所以当 int 强制转换为 byte 类型时，值 128 时候就会导致溢出
  int i =128;   
  byte b = (byte)i;
  ```

### 自动类型转换

自动类型转换必须满足**转换前的数据类型的位数要低于转换后的数据类型**. 即可以 `long l = 100;`而不可以`int l = 100L;`

```java
public class Test{
        public static void main(String[] args){
            char c1='a';//定义一个char类型
            int i1 = c1;//char自动类型转换为int
            System.out.println("char自动类型转换为int后的值等于"+i1);
            char c2 = 'A';//定义一个char类型
            int i2 = c2+1;//char 类型和 int 类型计算
            System.out.println("char类型和int计算后的值等于"+i2);
        }
}
// 结果:
// char自动类型转换为int后的值等于97
// char类型和int计算后的值等于66
```



### 隐式类型转换

**Java 不能隐式执行向下转型，因为这会使得精度降低。**

`1.1 `字面量属于 `double` 类型，不能直接将 `1.1 `直接赋值给 `float `变量，因为这是向下转型。

```java
float f = 1.1;
```

`1.1f` 字面量才是 float 类型。

```java
float f = 1.1f;
```

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。

```java
short s1 = 1;
s1 = s1 + 1;
```

但是使用 += 或者 ++ 运算符可以执行隐式类型转换。

```java
s1 += 1;
s1++;
```

上面的语句相当于将 ``s1 + 1`` 的计算结果进行了向下转型：

```java
//强制类型转换
s1 = (short) (s1 + 1);
```

简而言之,不能够直接的将精度高的类型 直接的赋值给精度低的类型.如: ``float f = 1.1; short s = 1;`` 如果想要赋值可用Java的隐式类型转换 如:``float f+=1.1; s += 1;``

**由大到小需要强制转换,由小到大不需要强转.  顺序:`byte , short , char , int ,long,float,double`**

```java
byte b=1; int a = b;//由小到大
int c = 1;
byte d = (byte) c;//由大到小
```


## 包装类型


| 基本数据类型 | 包装类    |
| ------------ | --------- |
| byte         | Byte      |
| boolean      | Boolean   |
| short        | Short     |
| char         | Character |
| int          | Integer   |
| long         | Long      |
| float        | Float     |
| double       | Double    |

在这八个类名中，除了 `Integer` 和 `Character` 类以后，其它六个类的类名和基本数据类型一致，只是类名的第一个字母大写即可.

```
graph TD
Object-->Number

Number-->Byte
Number-->Short
Number-->Integer
Number-->Long
Number-->Float
Number-->Double

Object-->Boolean
Object-->Character

```

`Integer、Long、Byte、Double、Float、Short`都是抽象类`Number`的子类.

因为 Java 是一种面向对象语言，很多地方都需要使用对象而不是基本数据类型。比如，在集合类中，我们是无法将` int 、double` 等类型放进去的。因为集合的容器要求元素是 `Object` 类型.所以才有了对应基本类型分包装类型.

### 拆箱与装箱
基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

以`Integer int` 为例

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

#### 自动拆装箱

自动装箱: 就是将基本数据类型自动转换成对应的包装类.

自动拆箱：就是将包装类自动转换成对应的基本数据类型.
```java
    Integer i = 10;  //自动装箱
    int b = i;     //自动拆箱
```
反编译得
```java
    public static  void main(String[]args){
        Integer integer=Integer.valueOf(1);
        int i=integer.intValue();
    }
```
`int`的自动装箱都是通过` Integer.valueOf()` 方法来实现的，Integer 的自动拆箱都是通过 `integer.intValue` 来实现的

> 自动装箱都是通过包装类的 `valueOf()` 方法来实现的.自动拆箱都是通过包装类对象的 `xxxValue() `来实现的。

#### 自动拆装箱带来的问题
- **比较**
包装对象的数值比较,不能简单的使用`==`,虽然 -128 到 127 之间的数字可以,但是这个范围之外还是需要使用 `equals`方法进行比较.


- **NPE**
因为有自动拆箱的机制，如果初始的包装类型对象为`null`，那么在自动拆箱的时候的就会报`NullPointerException`，在使用时需要格外注意.


- **内存浪费**
如果一个 for 循环中有大量拆装箱操作，会浪费很多资源


### 缓存池

案例: **``new Integer(123) ``与 ``Integer.valueOf(123)`` 的区别在于：**

- ``new Integer(123)`` 每次都会新建一个对象；
- ``Integer.valueOf(123)`` 会使用缓存池中的对象，多次调用会取得同一个对象的引用

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

``valueOf()`` 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

**在 Java 8 中，Integer 缓存池的大小默认为 -128~127**

```java
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

**编译器会在自动装箱过程调用 ``valueOf()`` 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。**

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

`Integer,int `在 `-127~128`之前是不会创建新的对象的,即

```java
 Integer a = new Integer(12);
 int b = 12;
 System.out.println(a==b);//true
```

`Integer`和`int`自动装箱拆箱会通过`valueOf()`方法实现,当这个数在-127~128之间直接从缓存里边取,不会重新new对象

**基本类型对应的缓冲池如下：**

- ``boolean values true and false``
- ``all byte values``
- ``short values between -128 and 127``
- ``int values between -128 and 127``
- ``char in the range \u0000 to \u007F``

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。==两种浮点数类型的包装类Float,Double并没有实现常量池技术。==

> 在 JDK 1.8 所有的数值类缓冲池中，Integer 的缓冲池`` IntegerCache`` 很特殊，这个缓冲池的下界是 - 128，上界默认是 127,但是这个上界是可调的:<br>在启动JVM的时候，通过``-XX:AutoBoxCacheMax=<size>``来指定这个缓冲池的大小,该选项在 JVM 初始化的时候会设定一个名为 ``java.lang.IntegerCache.high`` 系统属性,然后 ``IntegerCache``初始化的时候就会读取该系统属性来决定上界。 



## 引用类型

- 类
- 数组类型
- 接口类型
- 注解类型
- 枚举类型

引用类型指向一个对象(类似于C的指针),指向对象的变量是引用变量

所有引用类型的默认值都是`null`


### 与基本类型区别

1. 基本数据类型在被创建时,在栈中直接划分一块内存,将数值直接存入栈中,引用数据类型再被创建时,先在堆中开辟内存创建存放值,然后引用到栈中的是在堆中的地址值


2. 传递参数的时候不同.基本数据类型是值传递,引用数据类型是引用传递.

### 注意事项
- `String`类是引用类型,不是基本类型
- 默认`double`类型，如果`float`加后缀F(不区分大小写).如果`long`类型加后缀L(不区分大小写)
- 浮点数可能是个近似值,不是精确值.如果想要表示精确用`BigDecimal`类型


## 常量

案例: Java中常量,变量和字面量
```java
int a = 10; //a为变量，10为字面量
final int b = 10;  //b为常量，10为字面量
static str = "Hello World";  //str为变量,HelloWorld为字面量
```
- 常量：程序运行中，固定不变的量.
- 变量：值可以修改的数据类型。
- 字面量：字面量就是数据，是由字母、数字等构成的字符串或者数值，如30（整型）、3.15（浮点型）、“我是字符串”（字符串型）、‘中’（字符型）、true（布尔型）、false（布尔型）等,只能作为右值出现如`int a = 10`;这里的a是左值,10为右值.


字面量

> 在计算机科学中，字面量（literal）是用于表达源代码中一个固定值的表示法（notation）。几乎所有计算机编程语言都具有对基本值的字面量表示，诸如：整数、浮点数以及字符串；而有很多也对布尔类型和字符类型的值也支持字面量表示；还有一些甚至对枚举类型的元素以及像数组、记录和对象等复合类型的值也支持字面量表示法。

Java常量

> JAVA常量就是在程序运行过程中一直不会改变的量的量称为常量.常量在整个程序中只能被赋值一次.

常量简单理解为`final`修饰的变量


### 定义
在 Java 中使用`final`关键字来修饰常量，声明方式和变量类似

要声明一个常量，第一需要制定数据类型，第二需要通过`final`关键字进行限定
格式：`final 数据类型 常量名称[=值]`

**常量在程序运行时是不能被修改的(final作用).所以在定义常量时就需要对该常量进行初始化.为了与变量区别，常量取名一般都用大写字符**
```java
final double PI = 3.1415927;
```
`final` 关键字表示最终的，它可以修改很多元素，修饰变量就变成了常量.之后会详细说明`final`关键字
```java
public class HelloWorld {
    // 静态常量
    public static final double PI = 3.14;
    // 声明成员常量
    final int y = 10;
    public static void main(String[] args) {
        // 声明局部常量
        final double x = 3.3;
    }
}
```

### 常量池

在Java体系中，共用三种常量池.分别是字符串常量池、Class常量池(静态常量池)和运行时常量池.

**我们常说的常量池，就是指方法区中的运行时常量池。**


#### Class常量池


##### Class文件

> class文件全名称为Java class文件，主要在平台无关性和网络移动性方面使Java更适合网络。它在平台无关性方面的任务是：为Java程序提供独立于底层主机平台的二进制形式的服务。(会产生字节码)

> 有了字节码，无论是哪种平台（如Windows、Linux等），只要安装了虚拟机，都可以直接运行字节码。目前Java虚拟机已经可以支持很多除Java语言以外的语言了，如Groovy、JRuby、Jython、Scala等。之所以可以支持，就是因为这些语言也可以被编译成字节码。而虚拟机并不关心字节码是有哪种语言编译而来的。


将下面代码通过`javac`命令编译
```java
public class HelloWorld {
    public static void main(String[] args) {
        String s = "123";
    }
}
```
生成`.class`文件.`vi`命令查看

```class
Êþº¾^@^@^@4^@^Q
^@^D^@^M^H^@^N^G^@^O^G^@^P^A^@^F<init>^A^@^C()V^A^@^DCode^A^@^OLineNumberTable^A^@^Dmain^A^@^V([Ljava/lang/String;)V^A^@
SourceFile^A^@^OHelloWorld.java^L^@^E^@^F^A^@^C123^A^@7com/example/springboot/example/security/util/HelloWorld^A^@^Pjava/lang/Object^@!^@^C^@^D^@^@^@^@^@^B^@^A^@^E^@^F^@^A^@^G^@^@^@^]^@^A^@^A^@^@^@^E*·^@^A±^@^@^@^A^@^H^@^@^@^F^@^A^@^@^@^C^@        ^@      ^@
^@^A^@^G^@^@^@ ^@^A^@^B^@^@^@^D^R^BL±^@^@^@^A^@^H^@^@^@
^@^B^@^@^@^E^@^C^@^F^@^A^@^K^@^@^@^B^@^L
```
> 如何使用16进制打开class文件：使用 vim xxx.class ，然后在交互模式下，输入:%!xxd 即可。

```class 
00000000: cafe babe 0000 0034 0011 0a00 0400 0d08  .......4........
00000010: 000e 0700 0f07 0010 0100 063c 696e 6974  ...........<init
00000020: 3e01 0003 2829 5601 0004 436f 6465 0100  >...()V...Code..
00000030: 0f4c 696e 654e 756d 6265 7254 6162 6c65  .LineNumberTable
00000040: 0100 046d 6169 6e01 0016 285b 4c6a 6176  ...main...([Ljav
00000050: 612f 6c61 6e67 2f53 7472 696e 673b 2956  a/lang/String;)V
00000060: 0100 0a53 6f75 7263 6546 696c 6501 000f  ...SourceFile...
00000070: 4865 6c6c 6f57 6f72 6c64 2e6a 6176 610c  HelloWorld.java.
00000080: 0005 0006 0100 0331 3233 0100 3763 6f6d  .......123..7com
00000090: 2f65 7861 6d70 6c65 2f73 7072 696e 6762  /example/springb
000000a0: 6f6f 742f 6578 616d 706c 652f 7365 6375  oot/example/secu
000000b0: 7269 7479 2f75 7469 6c2f 4865 6c6c 6f57  rity/util/HelloW
000000c0: 6f72 6c64 0100 106a 6176 612f 6c61 6e67  orld...java/lang

```
> HelloWorld.class文件中的前八个字母是cafe babe，这就是Class文件的魔数（Java中的”魔数”）

```class
cafe babe   0000      0034        0011      0a00 0400 0d08
   魔数    此版本号   主版本号    常量池计数器      常量池计数区
```

> 我们需要知道的是,在Class文件的4个字节的魔数后面的分别是4个字节的Class文件的版本号（第5、6个字节是次版本号，第7、8个字节是主版本号，我生成的Class文件的版本号是52，这时Java 8对应的版本。也就是说，这个版本的字节码，在JDK 1.8以下的版本中无法运行）在版本号后面的，就是Class常量池入口了.


##### 存放内容

**常量池中主要存放两大类常量：字面量和符号引用。**
字面量前面已经记录过了,来记录下符号引用.

**符号引用**

符号引用是编译原理中的概念，是相对于直接引用来说的。主要包括了以下三类常量： **类和接口的全限定名  字段的名称和描述符 方法的名称和描述符.**

> 符号引用 ：符号引用以一组符号来描述所引用的目标。符号引用可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可，符号引用和虚拟机的布局无关。


在编译的时候每个java类都会被编译成一个class文件，但在编译的时候虚拟机并不知道所引用类的地址，所以就用符号引用来代替，而在这个==解析阶段==就是为了把这个符号引用转化成为真正的地址的阶段。

> 解析阶段:<br>
Java类从加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括，加载 ,验证 , 准备 , 解析 , 初始化 , 卸载 ，总共七个阶段。其中验证 ,准备 , 解析 统称为连接。<br>
而在解析阶段会有一步将常量池当中二进制数据当中的符号引用转化为直接引用的过程。<br>

 在Java编译阶段,由`.java`文件会生成`.class`文件.Class文件中除了包含类的版本、字段、方法、接口等描述信息外，还有一项信息就是常量池，用于存放编译器生成的各种字面量和符号引用。
 
 由于不同的Class文件中包含的常量的个数是不固定的，所以在Class文件的常量池入口处会设置两个字节的常量池容量计数器，记录了常量池中常量的个数。

 
 Class常量池可以理解为是Class文件中的资源仓库。
 
 class常量池是Class文件中的资源仓库，其中保存了各种常量。而这些常量都是开发者定义出来，需要在程序的运行期使用的。
 > Java代码在进行Javac编译的时候,在Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些字段、方法的符号引用不经过运行期转换的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。
 
 Class是用来保存常量的一个媒介场所，并且是一个中间场所。在JVM真的运行时，需要把常量池中的常量加载到内存中.
 


#### 运行时常量池

> 运行时常量池是每一个类或接口的常量池的运行时表示形式.<br>
<br>
> 它包括了若干种不同的常量：从编译期可知的数值字面量到必须运行期解析后才能获得的方法或字段引用。运行时常量池扮演了类似传统语言中符号表的角色，不过它存储数据范围比通常意义上的符号表要更为广泛。<br><br>
每一个运行时常量池都分配在 Java 虚拟机的方法区之中，在类和接口被加载到虚拟机后，对应的运行时常量池就被创建出来。

**简单说来就是JVM虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存在方法区中，我们常说的常量池，就是指方法区中的运行时常量池。**

##### 存放位置及存放来源

在不同版本的JDK中，运行时常量池所处的位置也不一样.以`HotSpot`为例:<br/>
JDK1.7之前方法区位于堆内存中的永久代.由于一些原因在JDK1.8之后彻底祛除了永久代,用元空间代替.

存放在JVM内存模型中的==方法区==

> **根据JVM规范,JVM内存共分为虚拟机栈，堆，方法区，程序计数器，本地方法栈五个部分.**
方法区:<br>
- 被所有方法线程共享的一块内存区域.
- 用于存储已经被虚拟机加载的类信息，常量，静态变量等.
- 这个区域的内存回收目标主要针对常量池的回收和堆类型的卸载.


**运行时常量池是方法区的一部分**`.CLass`文件中除了有类的版本、字段、方法、接口等描述信息外,还有一项信息是常量池，用于存放**编译期**生成的**各种字面量和符号引用**,这部分内容将在类加载后进入方法区的运行时常量池中存放.

**运行时常量池内容包含了Class常量池中的常量和字符串常量池中的内容.**



#### String常量池

在JVM中，为了减少相同的字符串的重复创建，为了达到节省内存的目的。会单独开辟一块内存，用于保存字符串常量，这个内存区域被叫做字符串常量池.

字符串常量池保存着所有字符串字面量,这些字面量在编译时期就确定.

##### intern()

==`intern`方法作用:== 
**当一个字符串调用 ``intern()`` 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 ``equals()``方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用.**

下面示例中，s1 和 s2 采用`` new String()`` 的方式新建了两个不同字符串，而 s3 和 s4 是通过 ``s1.intern()`` 方法取得一个字符串引用。``intern()`` 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
```

如果是采用"bbb"这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。(方法区)而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致`OOM`(``OutOfMemoryError ``)错误。


**上面的示例说明了可以使用 String的``intern()``方法在运行过程中将字符串添加到字符串常量池中。**


##### 经典案例
String常量池常考的一个问题就是`new String("abc")`会创建几个对象?

答案: 两个字符串对象(前提是 String常量池中还没有 "abc" 字符串对象).

- "abc" 属于字符串字面量，因此编译时期会在 String 常量池 中创建一个字符串对象，指向这个 "abc" 字符串字面量;
- 而使用 new 的方式会在堆中创建一个字符串对象。

```java
// 创建一个测试类，其 main 方法中使用这种方式来创建字符串对象。
public class MainTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
```

使用 ``javap -verbose`` 进行反编译，得到以下内容：

```java
// ...
Constant pool:
// ...
   #2 = Class              #18            // java/lang/String
   #3 = String             #19            // abc
// ...
  #18 = Utf8               java/lang/String
  #19 = Utf8               abc
// ...

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
// ...
```

在 ``Constant Pool`` 中，#19 存储这字符串字面量 ``"abc"``，#3 是 String Pool 的字符串对象，它指向 ``#19`` 这个字符串字面量。在 ``main`` 方法中，``0:`` 行使用 ``new #2`` 在堆中创建一个字符串对象，并且使用 ``ldc #3`` 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

##### 优点
常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享.
避免了相同内容的字符串的创建，节省了内存，省去了创建相同字符串的时间，同时提升了性能.



## String 类


### 概览
String是Java中一个比较基础的类.广泛应用 在 Java 编程中，在Java中字符串属于对象，Java 提供了 String 类来创建和操作字符串.

String被声明为`final`,因此它不可被继承.(当然Integer等包装类也不能被继承)

**在 Java 8 中，String 内部使用 char 数组存储数据。**

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

**在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。**

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

**value数组被声明为`final`，这意味着value数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证`String`不可变.**

String是Java中一个不可变的类，所以他一旦一个string对象在内存(堆)中被创建出来，他就无法被修改。**特别要注意的是，String类的所有方法都没有改变字符串本身的值，都是返回了一个新的对象。==字符串不可变的根本原因应是处于安全性考虑。==**

### 不可变性优点

**1. 可以缓存 hash 值**

因为`String`的`hash`值经常被使用,像`Set、Map`结构中的 key 值也需要用到`HashCode`来保证唯一性和一致性，因此不可变的 `HashCode` 才是安全可靠的

**2. String常量池的需要**

字符串常量池的基础就是字符串的不可变性，如果字符串是可变的，那想一想，常量池就没必要存在了。
如果一个 `String` 对象已经被创建过了，那么就会从`String常量池中取得引用。只有`String`是不可变的，才可能使用 String常量池。

**3. 安全性**

`String`经常作为参数,`String`不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果`String`是可变的，那么在网络连接过程中，`String`被改变，改变`String`对象的那一方以为现在连接的是其它主机,而实际情况却不一定是.

实际项目中会用到,比如数据库连接串、账号、密码等字符串，只有不可变的连接串、用户名和密码才能保证安全性.

**4. 线程安全**

字符串在 Java 中的使用频率可谓高之又高，那在高并发的情况下不可变性也使得对字符串的读写操作不用考虑多线程竞争的情况.

String不可变性天生具备线程安全,可以在多个线程中安全地使用.


### 使用

#### 创建字符串

```java
// String 直接创建
String str1 = "123";
// String 对象创建
String str2 = new String("123");
// 引用创建
String str3 = str1;
```
格式化创建字符串
```java
String str = String.format("浮点型变量: " +
                   "%f, 整型变量: " +
                   " %d, 字符串变量: " +
                   " %s", 1.0f, 1, "1");
System.out.println(str);
````
**String 直接创建的字符串存储在String常量池中，而 new 创建的字符串对象在堆上.**

#### 连接字符串
**因为String类是不可变的.所以所谓字符串拼接，都是重新生成了一个新的字符串.**

以下是字符串的几种拼接的方式
```java
// 原字符串
String str1 = "123";

//1. concat 方法连接字符串
String str2 = "123".concat("456");
System.out.println(str2);

//2. 用 "+" 连接字符串
// 在编译时会编译成 String str4 = "123456";
String str3 = "123" + "456";
// 会调用StringBuilder.append方法
String str4 = str3 + "789";
System.out.println(str3);
System.out.println(str4);

//3. 用JDK内置处理字符串类 Stringbuilder, StringBuffer
StringBuilder builder = new StringBuilder();
StringBuffer buffer = new StringBuffer();
String str5 = builder.append(str).append("456").toString();
String str6 = buffer.append(str).append("456").toString();
System.out.println(str5);
System.out.println(str6);

//4. Java8新增String.join方法连接字符串
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
String str7 = String.join("-", list);
String str8 = String.join("~", new String[]{"1","2","3"}r);
System.out.println(str7);
System.out.println(str8);

//5. 用第三方处理字符串工具类. 例如: apache.commons.
String str9 = StringUtils.join(new String[]{str, "456", "789"});

// Java8提供的String.join方法和`apache.commons`方法相似.
// 主要作用是将数组或集合以某拼接符拼接到一起形成新的字符串

```
##### "+"连接字符串
拼接字符串最简单的方式就是直接使用符号+来拼接. + 是Java提供的一个语法糖.
> 语法糖：语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·兰丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。语法糖让程序更加简洁，有更高的可读性。
```java
    public static void main(String[] args) {
        String str = "12" + "3";
    }
```
两个都为编译期常量,编译器会进行常量折叠变为`String str = "123";`


```java
    public static void main(String[] args) {
        String str1 = "123";
        String str2 = "456";
        String str3 = str1 + str2;
        System.out.println(str3);
    }
```
打开文件所在位置,用`javap -verbose`命令进行反编译.
```byte
// ...
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: ldc           #2                  // String 123
         2: astore_1
         3: ldc           #3                  // String 456
         5: astore_2
         6: new           #4                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        13: aload_1
        14: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        17: aload_2
        18: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        21: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        24: astore_3
        25: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
        28: aload_3
        29: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        32: return
      // ...

```
**在字符串拼接过程中,是将String转成了`StringBuilder`后，使用其`append`方法进行处理的.**

##### concat方法

源码
```java
    public String concat(String str) {
        // 获取字符串长度 
        int otherLen = str.length();
        // 判空
        if (otherLen == 0) {
            return this;
        }
        // 获取已有字符串长度
        int len = value.length;
        // 创建新的字符数组.
        // 长度为:已有字符串+待拼接字符长度.
        // 将两个字符串的值复制到新的字符数组
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        // 创建新的字符串对象
        return new String(buf, true);
    }
```
##### StringBuffer和StringBuilder
在`Stringbuilder`和`StringBuffer`共同的父类中
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

    // ...
}
```
用于存贮的`value`并没有用`final`修饰,说明是可以修改的.还有一个属性字段`count`,用来保存数组中已经使用的字符个数.

**append方法**

`StringBuilder.append`方法
```java
    @Override
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```
`StringBuffer.append`方法
```java
    @Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```
由此可以看出`StringBuilder`和`StringBuffer`原理是相似的,最大的区别就是`StringBuffer`是线程安全的.原因是用了`synchronized`修饰.

`append`方法原理的在父类中.需要注意的是,如果`append`方法`append(null)`会直接拼接字符串"null"
```java
    public AbstractStringBuilder append(String str) {
        // 判空
        if (str == null)
            /**
             * final char[] value = this.value;
             * value[c++] = 'n';
             * value[c++] = 'u';
             * value[c++] = 'l';
             * value[c++] = 'l';
             */
           // 如果是'null' 则返回 字符串 "null"
            return appendNull();
        // 获取字符长度
        int len = str.length();
        /**  if (count + len - value.length > 0) {
                // 拷贝字符到内部的字符数组中.如果字符数组长度不够，会进行扩展
                value = Arrays.copyOf(value,newCapacity(count+len));
             }
        **/
        ensureCapacityInternal(count + len);
        // 复制数组
        // System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
        str.getChars(0, len, value, count);
        // 拼接字符长度
        count += len;
        return this;
    }
```



和 String 类不同的是，`StringBuffer` 和 `StringBuilder` 类的对象能够被多次的修改，并且不产生新的未使用对象。

如果你需要一个可修改的字符串，应该使用`StringBuffer` 或者 `StringBuilder`。否则会有大量时间浪费在垃圾回收上，因为每次试图修改都有新的`String`对象被创建出来.

##### 比较和使用

时间比较(短-->长):

`StringBuilder`<`StringBuffer`<`concat`<`+`<`StringUtils.join`

使用注意:

- 如果不是在**循环体中**进行字符串拼接的话,直接使用+就好了,如果在循环体内使用"+"拼接字符串对象会在每一次循环都会创建`StringBuilder`对象,导致程序效率降低.


- 如果在并发场景中进行字符串拼接的话,要使用`StringBuffer`来代替`StringBuilder`.

#### 常用方法

本文是基于Java8整理的.
如果查看其他方法请参照[Java8API官方文档](https://docs.oracle.com/javase/8/docs/api/index.html) `java.lang.String`

##### substring
对字符串进行截取.返回一个新的字符串,它是此字符串的一个子字符串.

源码
```java
    public String substring(int beginIndex) {
        // 判空
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        // 需要截取的长度不能超过源字符的长度
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        // 如果传入的长度不等于被截字符串的长度 则创建新的字符串
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }
    
    public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```
使用
```java
    public static void main(String[] args) {
        String str = "123456";
        String substring = str.substring(2);
        // 3456
        System.out.println(substring);
        // 索引从1开始截取字符串
        String substring2 = str.substring(2,4);
        // 34
        System.out.println(substring2);
    }
```

##### replace
替换字符串.返回一个新的字符串,它是通过用 `newChar` 替换此字符串中出现的所有 `oldChar` 得到的.

源码
```java
 public String replace(char oldChar, char newChar) {
        // 校验: 需要进行替换新旧字符不能相同
        if (oldChar != newChar) {
            // 获取源字符串中的长度
            int len = value.length;
            int i = -1;
            // 获取源字符串
            char[] val = value; /* avoid getfield opcode */
            // 判断源字符串是否存在 需要替换的旧字符
            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            if (i < len) {
                // 创建新的字符数组 用于替换后保存新字符串 
                char buf[] = new char[len];
                for (int j = 0; j < i; j++) {
                    // 将旧字符存入到新字符数组
                    buf[j] = val[j];
                }
                while (i < len) {
                    char c = val[i];
                    // 替换字符
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                // 创建新字符串对象
                return new String(buf, true);
            }
        }
        return this;
    }
```
使用
```java
    public static void main(String[] args) {
        String str = "123456";
        // 223456
        System.out.println(str.replace('1', '2'));
    }
```

##### replaceAll
使用给定的 `replacement` 替换此字符串所有匹配给定的正则表达式的子字符串.

源码
```java
    public String replaceAll(String regex, String replacement) {
        return Pattern.compile(regex).matcher(this).replaceAll(replacement);
    }
    
```
```java
    // complile 解析正则表达式 获得 Pattern对象
    public static Pattern compile(String regex) {
        return new Pattern(regex, 0);
    }
    
    //matcher 获取匹配器对象
    public Matcher matcher(CharSequence input) {
        if (!compiled) {
            synchronized(this) {
                if (!compiled)
                    compile();
            }
        }
        Matcher m = new Matcher(this, input);
        return m;
    }
    
    // replaceAll 进行字符串替换
    public String replaceAll(String replacement) {
    // 对当前Matcher类进行重置，即对其中记录匹配结果的开始和结束位置索引，以及分组信息重置
        reset();
        // 执行第一次搜索
        boolean result = find();
         // 第一次搜索匹配成功
        if (result) {
            // 用于记录最终的替换结果字符串
            StringBuffer sb = new StringBuffer();
            do {
            // 重点：用于将从上一次匹配子字符串的下一个索引位置开始，到当前匹配的子字符串的结束索引位置的所有字符 append到字符串sb中
                appendReplacement(sb, replacement);
                result = find();
            } while (result);
            // 将从最后一次匹配子字符串的下一个索引位置，到字符串的结尾的所有字符append到字符串sb中
            appendTail(sb);
            return sb.toString();
        }
        return text.toString();
    }
```
重点看一下`replaceAll`中调用的`appendReplacement`方法
```java
    public Matcher appendReplacement(StringBuffer sb, String replacement) {
            
            // ...
            
            // 用于跟踪 replacement 字符串的索引
            int cursor = 0;
            
            // 对当前匹配到子字符串替换后的结果字符串
            StringBuffer result = new StringBuffer();
            
            // 遍历 replacement字符串
            while (cursor < replacement.length()) {
                
                char nextChar = replacement.charAt(cursor);
                
                if (nextChar == '\\') {
                    // 重点1：当字符为'\'时，跳过，并获取其后面的字符，追加到result
                    cursor++;
                    nextChar = replacement.charAt(cursor);
                    result.append(nextChar);
                    cursor++;
                } else if (nextChar == '$') {
                    
                    // 重点2：当字符为$时，跳过，并获取其后面的数值，并以此如果$后面第一个不为数字则抛异常，
                    // Skip past $
                    cursor++;
                    
                    // The first number is always a group
                    int refNum = (int)replacement.charAt(cursor) - '0';
                    
                    // 此处代码用于计算$符号后的数值，数值结果赋予 refNum
                    // ...
                    
                    // group(refNum) 用于获取正则表达式第refNum个分组表示的字符串，不详说了
                    if (group(refNum) != null)
                         // 追加到result
                        result.append(group(refNum));
                } else {
                    
                    // 当前字符不为\ 或 $ 则直接追加到result
                    result.append(nextChar);
                    cursor++;
                }
            }
            
            // 将从上一次匹配的子字符串的结尾索引，到当前匹配的第一个字符串索引的字符串追加到sb
            // lastAppendPosition参数为上一次执行appendReplacement方法最后追加的字符在原始字符串中的索引位置。
            // first 参数为当前待替换的子字符串的首个字符在原始字符串中的索引位置
            sb.append(getSubSequence(lastAppendPosition, first));
            
            // 将当前配置子字符串替换后的结果字符串追加到sb
            sb.append(result.toString());
            
            // 更新lastAppendPosition，供下一个匹配执行appendReplacement方法使用
            lastAppendPosition = last;
            
            /* sb中追加了当前匹配的子字符串与前一次匹配子字符串中间的字符，以及当前匹配子字符串被替换后的字符串
             */
            return this;
        }
```
`replaceAll`中第二个参数`replacement`中，\ 有转义的作用, $ 用于获取分组匹配的当前子字符串 因为引入了 $ 符的分组功能,所以为了解决能输出 $ 字符，故引入 \ 转义功能.

使用
```java
    public static void main(String[] args) {
        String str = "111111";
        // 222222
        System.out.println(str.replaceAll("1", "2"));
    }
```

##### valueOf
该方法作用是将对象转成String类型.

源码
```java
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();
    }
```
使用
```java
    public static void main(String[] args) {
        Integer integer = 11111;
        String str = String.valueOf(integer);
        // 11111
        System.out.println(str);
    }
```

### 长度限制
翻阅String源码在String源码中发现有定义字符串长度的构造函数
```java
    // count 就是 字符串定义长度 
    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```
通过源码可以看到`int`的最大长度就是`String`的支持的最大长度.
```java
    public static void main(String[] args) {
        // 2,147,483,648 = 2^31 - 1 
        System.out.println(Integer.MAX_VALUE);
    }
```
**注意`new String(char value[], int offset, int count)`是运行时`String`支持的最大长度.**


在`String`编译声明期间,用`javac`编译 长度为`2^31 -1`的字符串.
```java
    public static void main(String[] args) {
        // 长度: 2^31 -1
        String str = "1111 ... ";
        System.out.println(str);
    }
```
```
java: 常量字符串过长
```

在`Gen`类中相关报错信息源码
```java
private void checkStringConstant(DiagnosticPosition var1, Object var2) {
    if (this.nerrs == 0 && var2 != null && var2 instanceof String && ((String)var2).length() >= 65535) {
        this.log.error(var1, "limit.string", new Object[0]);
        ++this.nerrs;
    }
}
```
可以看到程序中如果`String`长度大于等于65535会导致编译失败

在编译期的时候,字面量要进字符串常量池.所以要遵守[《Java®虚拟机规范》(Java8)中对String常量池的描述.](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.3)

`CONSTANT_String_info` 用于表示 `java.lang.String` 类型的常量对象结构体

`CONSTANT_String_info`格式如下:
```java 
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```

> **tag**<br>
结构`CONSTANT_String_info`的标签项的值为`CONSTANT_String(8)`

> **string_index**<br>
`string_index` 项的值必须是对常量池的有效索引，常量池在该索引处的项必须是 `CONSTANT_Utf8_info` 结构，表示一组 `Unicode` 码点序列，这组 Unicode 码点序列最终会被初始化为一个 下 `Unicode` 对象

`CONSTANT_Utf8_info`是一个`CONSTANT_Utf8`类型的常量池数据项，它存储的是一个常量字符串。常量池中的所有字面量几乎都是通过`CONSTANT_Utf8_info`描述的。`CONSTANT_Utf8_info`的定义如下：
```java
ONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```
`length`则指明了 `bytes`数组的长度，其类型为`u2`

通过查阅《JVM规范》发现`u2`表示两个字节的无符号数，那么1个字节有8位，2个字节就有16位。16位无符号数可表示的最大值位`2^16 - 1 = 65535`。也就是说，`Class`文件中常量池的格式规定了，其字符串常量的长度不能超过65535。


**关于编译器字符串最大长度65534的问题**
>如果一个方法的Java虚拟机代码长度正好是65535字节，并且以一个1字节长的指令结束，那么该指令不能被异常处理程序保护。编译器作者可以通过将任何方法、实例初始化方法或静态初始化器(任何代码数组的大小)生成的Java虚拟机代码的最大大小限制为65534字节来解决这个问题

**简单来说**
- 在程序运行时String最大长度为int最大长度为`2^31 -1`
- 在程序编译期根据`String`JVM常量池规范String字符串在声明时最大为 65535,但是为了修复`Java`的遗留问题改为65534

在程序开发中,需要注意如果你用`String`变量接收Base64图片或音频视频需要注意不要超过在程序运行时字符串的最大阈值.

### 编码问题

因为全世界有很多编程人员，有很多语言，不同的国家使用不同的语言，如果说没有一套统一的编码规则，这么多语言混在一起，很容易出现乱码现象，本着既方便又节约内存的理念大家基本都是用`utf-8`码来编写程序.

#### Unicode
> Unicode（中文：万国码、国际码、统一码、单一码）是计算机科学领域里的一项业界标准。它对世界上大部分的文字系统进行了整理、编码，使得计算机可以用更为简单的方式来呈现和处理文字.

> Unicode伴随着通用字符集的标准而发展，同时也以书本的形式对外发表。Unicode至今仍在不断增修，每个新版本都加入更多新的字符。目前最新的版本为2018年6月5日公布的11.0.0，已经收录超过13万个字符（第十万个字符在2005年获采纳）。Unicode涵盖的数据除了视觉上的字形、编码方法、标准的字符编码外，还包含了字符特性，如大小写字母。

`Unicode`是一种编码规范，是为解决全球字符通用编码而设计的，而`UTF-8 UTF-16`等是这种规范的一种实现.`Unicode`是字符集而`UTF-8`是编码规则。

Java内部采用`Unicode`编码规范，也就是支持多语言的，具体采用的`UTF-16`编码方式.

**不管程序过程中用到了`GBK`还是`ISO8859-1`等格式，在存储与传递的过程中实际传递的都是`Unicode`编码的数据，要想接收到的值不出现乱码，就要保证传过去的时候用的是X编码，接收的时候也用X编码来转换接收**

#### 乱码原因
**编码时格式和解码时格式不一致.**

`string`在文件里面底层保存形式是二进制,底层用`byte[]`数组存储(Java9. Java8是用`char`数组储存).`byte[]`数组里面的内容可以按照不同的编码格式存放.在读取字符串的时候,也可以按照不同的解码格式存放.这样就造成了**乱码**.

简单理解为

在编码(字符串到字节)的时候是用一种编码;而在解码(从字节到字符串)的时候用另一种编码;所以导致乱码问题.**所以想要避免乱码问题最简单的办法就是从始至终,都用同一种字符格式**

#### 相关方法

`String`类有两种比较常用的操作编码方式
```java
    // 注意处理异常
    public static void main(String[] args) throws UnsupportedEncodingException {
        // 本地使用的是 utf-8 的编码
        String str = "你好";
        byte[] bytes = str.getBytes("utf-8");
        // 你好
        System.out.println(new String(bytes));
        String string = new String(str.getBytes(), "utf-8");
        // 你好
        System.out.println(string);
    }
```
##### getBytes(String charsetName)
该方法会根据指定的`decode`编码返回某字符串在该编码下的`byte`数组表示

==**源码分析之后会补上 ...**==

源码
```java
    public byte[] getBytes(String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null) throw new NullPointerException();
        return StringCoding.encode(charsetName, value, 0, value.length);
    }
```
`StringCoding.encode`方法
```java
// len: 当前字符串长度
static byte[] encode(String charsetName, char[] ca, int off, int len)
        throws UnsupportedEncodingException
    {
        StringEncoder se = deref(encoder);
        // 如果为空 默认ISO-8859-1
        String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
        // 
        if ((se == null) || !(csn.equals(se.requestedCharsetName())
                              || csn.equals(se.charsetName()))) {
            se = null;
            try {
                // 根据编码获取 Charset对象
                Charset cs = lookupCharset(csn);
                if (cs != null)
                    se = new StringEncoder(cs, csn);
            } catch (IllegalCharsetNameException x) {}
            if (se == null)
                throw new UnsupportedEncodingException (csn);
            set(encoder, se);
        }
        return se.encode(ca, off, len);
    }
```


##### new String(byte bytes[], String charsetName)
该方法为字节数组构造

`char[]`数组是以`unicode`码来存储的,`String`和`char`为内存形式.`byte`是网络传输或存储的序列化形式.可以通过`charset`来解码指定的`byte`数组，将其解码成`unicode`的`char[]`数组，构造`String`.

==**源码分析之后会补上 ...**==

源码
```java
    public String(byte bytes[], String charsetName)
            throws UnsupportedEncodingException {
        this(bytes, 0, bytes.length, charsetName);
    }
```
```java
    public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(charsetName, bytes, offset, length);
    }
```
```java
    static char[] decode(String charsetName, byte[] ba, int off, int len)
        throws UnsupportedEncodingException
    {
        StringDecoder sd = deref(decoder);
        String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
        if ((sd == null) || !(csn.equals(sd.requestedCharsetName())
                              || csn.equals(sd.charsetName()))) {
            sd = null;
            try {
                Charset cs = lookupCharset(csn);
                if (cs != null)
                    sd = new StringDecoder(cs, csn);
            } catch (IllegalCharsetNameException x) {}
            if (sd == null)
                throw new UnsupportedEncodingException(csn);
            set(decoder, sd);
        }
        return sd.decode(ba, off, len);
    }
```