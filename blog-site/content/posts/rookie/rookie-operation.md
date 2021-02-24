---
title: "面向菜鸟编程-运算篇"
date: 2021-01-30
draft: false
tags: ["中文", "Java", "面向菜鸟编程"]
slug: "rookie-operation"
---
- [运算符与表达式](#运算符与表达式)
    - [运算符](#运算符)
        - [种类](#种类)
            - [算术运算符](#算术运算符)
                - [取余](#取余)
                - [自增和自减](#自增和自减)
            - [赋值运算符](#赋值运算符)
            - [关系运算符](#关系运算符)
            - [逻辑运算符](#逻辑运算符)
                - [短路运算符](#短路运算符)
            - [位运算符](#位运算符)
            - [其他运算符](#其他运算符)
                - [三目运算符](#三目运算符)
                - [instanceof](#instanceof)
        - [运算顺序](#运算顺序)
    - [表达式](#表达式)
        - [Java正则表达式](#java正则表达式)
            - [常用正则](#常用正则)
- [控制流语句](#控制流语句)
    - [if-else语句](#if-else语句)
    - [switch语句](#switch语句)
    - [for语句](#for语句)
    - [do-while和while语句](#do-while和while语句)
- [参数传递](#参数传递)
    - [实参与形参](#实参与形参)
    - [值传递与引用传递](#值传递与引用传递)
    - [Java传递对象参数](#java传递对象参数)
- [Java运算](#java运算)
    - [精度丢失](#精度丢失)
    - [BigDecimal](#bigdecimal)
        - [使用](#使用)
            - [基本运算](#基本运算)
            - [初始化](#初始化)
                - [valueOf](#valueOf)
                - [double类型构造器](#double类型构造器)
            - [比较](#比较)
        - [保证精度原因及注意事项](#保证精度原因及注意事项)
    - [Math](#math)
        - [常用方法](#常用方法)
            - [max](#max)
            - [min](#min)
            - [abs](#abs)
            - [ceil和floor](#ceil和floor)
            - [random](#random)
            - [rint](#rint)
            - [round](#round)

## 运算符与表达式

### 运算符
运算符指明对操作数的运算方式。组成表达式的Java操作符有很多种。运算符按照其要求的操作数数目来分，可以有**单目运算符、双目运算符和三目运算符**，它们分别对应于1个、2个、3个操作数。


#### 种类
运算符按其功能来分:**有算术运算符、赋值运算符、关系运算符、逻辑运算符、位运算符和其他运算符**.

##### 算术运算符

运算符 | 名称 | 用法描述 | 备注
---|---|---|---
+ | 加法 | 相加运算符两侧的值
- | 减法 | 左操作数减去右操作数
* | 乘法 | 相乘操作符两侧的值
/ | 除法 | 左操作数除以右操作数 | 右操作数不能为0
% | 取余(模) | (左操作数除以右操作数)的余数
++ | 自增 | 操作数的值增加1
-- | 自减 | 操作数的值减少1

基本的`+、-、*、/`就不写了.记录一下剩下的算数运算符

###### 取余
基本用法
```java
    public static void main(String[] args) {
        System.out.println(0%5); // 0
        System.out.println(1%5); // 1
        System.out.println(2%5); // 2
        System.out.println(3%5); // 3
        System.out.println(4%5); // 4
        System.out.println(5%5); // 0
        System.out.println("------------");
        System.out.println(6%5); // 1
        System.out.println(7%5); // 2
        System.out.println(8%5); // 3
        System.out.println(9%5); // 4
        System.out.println(9%1); // 0
    }
```
如果被除数小于除数,那取模的结果就是被除数.如果被除数等于除数,结果是0,如果除数是1,结果是0.


**取余应用**

当使用随机数生成器产生的结果时,取余运算（%）可将结果限制在上限为操作数最大值减1的范围.

例如:n是随机数,那么`n%10`就是`0~9`中的一个数.无论n是多大的数,`n%10`只能是`0~9`之间的一个数,其中10就是操作数.可以利用这一特性,可以在数据库分库,分表 等.

###### 自增和自减
基本用法
```java
    public static void main(String[] args) {
        int i = 1;
        System.out.println(i++); // 1
        System.out.println(i--); // 2
    }
```
**需要特别留意的是`++`和`--`运算符可以前置、后置，都是合法的语句，如`a++`和`++a`都是合法的，上面这两种写法其最终的结果都是是变量a的值加1了，但是它们之间也是有区别的，其区别是：表达是`++a`会先将a的值自增1，然后在使用变量a。而表达式a++是先使用了a的值，然后再让a的值自增1。也就是说在一些表达式，使用`a++`和使用`++a`得到的结果时不一样的。**
```java
    public static void main(String[] args) {
        int i = 1;
        System.out.println(i++);// 1
        System.out.println(++i);// 3
    }
```
自增(`++`)和自减(`--`)两个运算符只能作用于变量,而不能作用于表达式.
```java
    public static void main(String[] args) {
        int j = 0, i = 1;
        // 编译报错
        System.out.println((j+i)++);
    }
```

##### 赋值运算符

运算符 | 名称 | 用法描述
---|---|---
= | 赋值 | 将右操作数的值赋给左侧操作数
+= | 加等于 | 把左操作数和右操作数相加赋值给左操作数
-= | 减等于 | 把左操作数和右操作数相减赋值给左操作数
*= | 乘等于 | 把左操作数和右操作数相乘赋值给左操作数
/= | 除等于 | 把左操作数和右操作数相除赋值给左操作数
%= | 模等于 | 把左操作数和右操作数取模后赋值给左操作数
<<= |左位移等于 | 把左操作数和右操作数进行左移运算后赋值给左操作数
>>= | 右位移等于 | 把左操作数和右操作数进行右移运算后赋值给左操作数
&= | 按位与等于 | 把左操作数和右操作数进行按位与运算后赋值给左操作数
&#124;= | 按位或等于 | 把左操作数和右操作数进行按位或运算后赋值给左操作数
^= | 异或等于 | 把左操作数和右操作数进行按位异或运算后赋值给左操作数


##### 关系运算符

关系运算符也称为比较运算符.
用于测试两个操作数之间的关系.使用关系运算符表达式的最终结果为`boolean`型，也就是其结果只有两个`true`和`false`.

运算符| 名称| 用法描述
---|---|---
== | 双等号| 检查两个操作数的值是否相等,如果相等则条件为真.
!= | 不等号 | 检查两个操作数的值是否相等,如果值不相等则条件为真.
> | 大于 | 检查左操作数的值是否大于右操作数的值,如果是那么条件为真.
< | 小于 | 检查左操作数的值是否小于右操作数的值,如果是那么条件为真.
>= | 大于等于 | 检查左操作数的值是否大于或等于右操作数的值,如果是那么条件为真.
<= | 小于等于 | 检查左操作数的值是否小于或等于右操作数的值,如果是那么条件为真.


##### 逻辑运算符

运算符 | 名称 | 用法描述
---|---|---
&& | 逻辑与 | 当且仅当两个操作数都为真,条件才为真.
&#124;&#124; | 逻辑或 | 如果两个操作数任何一个为真,条件为真.
! | 逻辑非 | 用来反转操作数的逻辑状态.如果条件为true,通过逻辑非将得到false.

###### 短路运算符
`&&`运算符,运算顺序是从左到右计算,运算规则是如果两个操作数都是真,则返回`true`,否则返回`false`.但是当判定到第一个操作数为`false`时,其结果必定为`false`,这时候就不再会判定第二个操作数了.
```java
    public static void main(String[] args) {
        int i = 1, j = 2;
        boolean flag = (i++ == 2) && (++j == 3);
        // flag的值: false,i的值:2,j的值:2
        System.out.printf("flag的值: %s,i的值:%s,j的值:%s", flag, i, j);
    }
```



##### 位运算符

位运算符在追求代码效率和编写底层应用时,使用的比较多;在企业Java开发一般用到的较少.

因为位运算符是以`bit`运算单位的.所以想要要弄明白位运算符,就要先弄明白2进制的表示方法.

位运算符只能对整数型(`int,long,short,byte`)和字符型数据(`char`)进行操作.

运算符 | 名称 | 用法描述
---|---|--
& | 按位与 | 如果相对应位都是1,则结果为1,否则为0
&#124; | 按位或 | 如果相对应位都是 0,则结果为 0,否则为 1
^ | 按位异或 | 如果相对应位值相同,则结果为0,否则为1
~ | 按位取反 | 翻转操作数的每一位,即0变成1,1变成0.
<< | 左移 | 左操作数按位左移右操作数指定的位数.
>> | 右移 | 左操作数按位右移右操作数指定的位数.
>>> | 无符号右移 | 左操作数的值按右操作数指定的位数右移,移动得到的空位以零填充.

基本用法
```java
    public static void main(String[] args) {
        int i = 1, j = 2;
        System.out.println(i&j);//0
        System.out.println(i|j);//3
        System.out.println(i^j);//3
        System.out.println(~(j+i));//-4
        System.out.println(i<<j);//4
        System.out.println(i>>j);//0
        System.out.println(i>>>j);//0
    }
```
解析:

以下示例用十进制的1和2进行运算.1用二进制表示为`0000 0001`,2用二进制表示为`0000 0010`
[10进制2进制互相转换](https://www.google.com/search?sxsrf=ALeKk0010piT4DlhyvDp3DJUzmKw1n2QLA%3A1612180398529&ei=rusXYLncH9X_wAP5n5GwDQ&q=10%E8%BF%9B%E5%88%B62%E8%BF%9B%E5%88%B6%E4%BA%92%E7%9B%B8%E8%BD%AC%E6%8D%A2&oq=10%E8%BF%9B%E5%88%B62%E8%BF%9B%E5%88%B6%E4%BA%92%E7%9B%B8%E8%BD%AC%E6%8D%A2&gs_lcp=CgZwc3ktYWIQA1CuT1iuT2CEVmgBcAB4AIABvQGIAb0BkgEDMC4xmAEAoAECoAEBqgEHZ3dzLXdpesABAQ&sclient=psy-ab&ved=0ahUKEwi5697_z8juAhXVP3AKHflPBNYQ4dUDCA0&uact=5)怎么转就不讲了.不懂的小伙伴可自行查看链接.

&:如果相对应位都是1,则结果为1,否则为0
```txt
0000 0001
0000 0010
——————————
0000 0000
```
|:如果相对应位都是 0,则结果为 0,否则为 1
```txt
0000 0001
0000 0010
——————————
0000 0011
```
^:如果相对应位值相同,则结果为0,否则为1
```txt
0000 0001
0000 0010
——————————
0000 0011
```
~:翻转操作数的每一位,即0变成1,1变成0.
> 十进制负数转换为二进制,就是将其相反数（正数）的补码的每一位变反（1变0，0变1）最后将变完了的数值加1,就完成了负数的补码运算.这样就变成了二进制.二进制转十进制负数相反.

```txt
0000 0011
——————————
1111 1100
```
`<<`:左操作数按位左移右操作数指定的位数.

1左移两位
```txt
0000 0001
——————————
0000 0100
```
`>>`:左操作数按位右移右操作数指定的位数.

正数右移高位补0，负数右移高位补1

1右移两位
```txt
0000 0001
——————————
0000 0000
```
负一右移两位
```txt
1111 1111
——————————
1111 1111
```

`>>>`:左操作数的值按右操作数指定的位数右移,移动得到的空位以零填充.

无符号右移,无论是正数还是负数,高位通通补0

1无符号右移两位
```txt
0000 0001
——————————
0000 0000
```
负一无符号右移两位
```txt
1111 1111
——————————
0011 1111
```

##### 其他运算符

###### 三目运算符
该运算符有3个操作数,其功能是对表达式条件进行判断,根据判断结果是`true`或者`false`两种情况赋予赋予不同的返回值.
该运算符的主要是决定哪个值应该赋值给变量.

在某些情况下,可以发现三目运算符和`if...else`作用是相同的.
```java
    public static void main(String[] args) {
        int var1 = 1;
        int var2 = 0;
        var1 = var2 > 1 ?  1: 2;
        
        // 等价于if...else以下代码
        if (var2 > 1) {
            var1 = 1;
        }else {
            var1 = 2;
        }
    }
```

三目运算符的原理到底是什么,与`if...else`到底哪有啥不同?要探究这个原因,需要看JVM是对三目运算符怎么处理的.

用`javap -verbose`命令,分别把`if...else`和三目运算符相关代码进行反编译

三目运算符
```java
    public static void main(String[] args) {
        int var1 = 1;
        int var2 = 0;
        // 如果var2大于1 则var1等于1 否则等于2
        var1 = var2 > 1 ?  1: 2;
    }
```
```class
  // ...
{
 // ...
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: iconst_1 // 向栈里添加int类型 1
         1: istore_1 // 将int 1 存储到局部变量
         2: iconst_0 // 向栈里添加int类型 0
         3: istore_2 // 将int 2 存储到局部变量
         4: iload_2  // 从局部变量加载int 2
         5: iconst_1
         6: if_icmple     13  // 将1,3步骤栈中的变量弹出 进行比较
         9: iconst_1 // 比较成功 存储该值到栈中
        10: goto          14 // 改变地址
        13: iconst_2
        14: istore_1
        15: return
      LineNumberTable:
        line 6: 0
        line 7: 2
        line 8: 4
        line 9: 15
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  args   [Ljava/lang/String;
            2      14     1  var1   I
            4      12     2  var2   I
      StackMapTable: number_of_entries = 2
        frame_type = 253 /* append */
          offset_delta = 13
          locals = [ int, int ]
        frame_type = 64 /* same_locals_1_stack_item */
          stack = [ int ]
}
// ...
```
`if...else`
```java
    public static void main(String[] args) {
        int var1 = 1;
        int var2 = 0;
        if (var2 > 1){
            var1 = 1;
        } else {
            var1 = 2;
        }
    }
```
```class
  // ...
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: iconst_1
         1: istore_1
         2: iconst_0
         3: istore_2
         4: iload_2
         5: iconst_1
         6: if_icmple     14
         9: iconst_1
        10: istore_1
        11: goto          16
        14: iconst_2
        15: istore_1
        16: return
      LineNumberTable:
        line 6: 0
        line 7: 2
        line 8: 4
        line 9: 9
        line 11: 14
        line 13: 16
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      17     0  args   [Ljava/lang/String;
            2      15     1  var1   I
            4      13     2  var2   I
      StackMapTable: number_of_entries = 2
        frame_type = 253 /* append */
          offset_delta = 14
          locals = [ int, int ]
        frame_type = 1 /* same */
}
// ...

```

通过查阅[《Java虚拟机指令集》](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html),可以看到JVM对上边的代码进行了什么处理.

**通过比较两次反编译后的代码可以得出一个结论:三目运算符,省去了一步赋值操作** 所以实际开发中三目运算符的运算效率略高于`if..else`.
但是对于实际开发来说三目运算符的可读性相对不如`if...else`代码，所以在常见的编码中，三目运算符更倾向于简单的`if...else`语句的替代.

###### instanceof
`interfaceof`是一个双目运算符,该关键字的作用是判断左边的对象是不是右边类的实例,并返回一个`boolean`值

基本用法

```java
    public static void main(String[] args) {
        String str1 = "123";
        String str = (str1 instanceof Object) ? "123" : "456";
    }
```


#### 运算顺序

Java 语言中大部分运算符也是从左向右结合的,只有单目运算符、赋值运算符和三目运算符例外,是从右向左运算的.

Java 语言中运算符的优先级共分为 14 级,其中 1 级最高,14 级最低.在同一个表达式中运算符优先级高的先执行.

> 算术运算符>比较运算符>赋值运算符>逻辑运算符>三元运算符<br><br>
可使用以下口诀记住：
单目乘除为关系,逻辑三目后赋值.

**运算符优先级表**

| 优先级 | 运算符                                           |  结合性  |
| :----: | ------------------------------------------------ | :------: |
|   1    | ()、[]                                     | 从左向右 |
|   2    | !、+、-、~、++、--                               | 从右向左 |
|   3    | *、/、%                                          | 从左向右 |
|   4    | +、-                                             | 从左向右 |
|   5    | «、»、>>>                                       |  从左向  |
|   6    | <、<=、>、>=、instanceof                         | 从左向右 |
|   7    | ==、!=                                           | 从左向右 |
|   8    | &                                                | 从左向右 |
|   10   | \|                                               | 从左向右 |
|   11   | &&                                               | 从左向右 |
|   12   | \|\|                                             | 从左向右 |
|   13   | ?:                                               | 从右向左 |
|   14   | =、+=、-=、*=、/=、&=、\|=、^=、~=、<<=、>>=、>>>= | 从右向左 |


==建议==

- 不要把一个表达式写得过于复杂,如果一个表达式过于复杂，则会影响代码可读性,建议把它分成几步来完成.
- 不要过多地依赖运算符的优先级来控制表达式的执行顺序,这样可读性太差,尽量使用小括号来控制表达式的执行顺序.



### 表达式
表达式是由运算符和运算对象组成的,单独的一个运算对象(常量/变量)也可以叫做表达式.

变量的赋值与计算都离不开表达式,表达式的运算依赖于变量、常量和运算符.

在Java中表达式通常是以分号结尾的一段代码.
```java
float f1 = 1.1f;
float f2 = 1.2f;
float f3 = f1 + f2;
```
#### Java正则表达式
> 正则表达式，又称规则表达式。（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表达式通常被用来检索、替换那些符合某个模式(规则)的文本。在众多语言中都可以支持正则表达式，如`Perl、PHP、Java、Python、Ruby`等.

正则表达式并不仅限于某一种语言,但是在每种语言中有细微的差别.

例如：
> 在其他语言中，`\\` 表示：我想要在正则表达式中插入一个普通的（字面上的）反斜杠.<br>
在 Java 中，`\\` 表示：我要插入一个正则表达式的反斜线，所以其后的字符具有特殊的意义。<br>
所以，在其他的语言中,一个反斜杠`\`就足以具有转义的作用，而在 Java 中正则表达式中则需要有两个反斜杠才能被解析为其他语言中的转义作用。也可以简单的理解在 Java 的正则表达式中，两个 `\\` 代表其他语言中的一个 \，这也就是为什么表示一位数字的正则表达式是 `\\d`，而表示一个普通的反斜杠是 `\\\\`。

正则表达式是由普通字符（如英文字母）以及[特殊字符（也称为元字符）](https://www.runoob.com/regexp/regexp-metachar.html)组成的文字模式.

例如：
```java
String str = "abc^123/?[1,2]";
```

在Java语言中主要是使用正则表达式处理字符串.Java从jdk1.4开始提供了一个包`java.util.regex`来处理正则表达式.

在Java开发中主要使用`java.util.regex`包中`Pattern`类,`Matcher`类来处理字符串.
详情参照文档[《Java™平台
标准Ed. 8》](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html),[《菜鸟教程》Java正则表达式](https://www.runoob.com/java/java-regular-expressions.html) ,[《JavaSchool》Java正则表达式](http://www.51gjie.com/java/761.html) 这些文档里面有详细的教程,所以这里不作过多介绍了.

使用正则表达式示例
```java
    public static void main(String[] args) {
        // 表示匹配以字母a为开头的单词
        String regx = "\\ba\\w*\\b";
        // 将给定的正则表达式编译到具有给定标志的模式中
        Pattern pattern = Pattern.compile(regx);
        // 创建匹配给定输入与此模式的匹配器
        Matcher matcher = pattern.matcher("abcdab cccabcd aaacd");
        int index = 0;
        // 循环 查找与上边正则匹配的字符序列
        while (matcher.find()) {
            // 返回 由以前匹配操作 所匹配的 输入子序列
            String res = matcher.group();
            System.out.println(index + ":" + res);
            index++;
        }
    }
```
```
0:abcdab
1:aaacd
```
**总的来说正则表达式是对字符串操作的一种逻辑公式,用事先定义好的一些特定字符、及这些特定字符的组合,组成一个"规则字符串",这个"规则字符串"用来表达对字符串的一种过滤逻辑.**

**正则表达式的灵活性、逻辑性和功能性非常的强.可以迅速地用极简单的方式达到字符串的复杂控制.但是对于刚接触的人来说,比较晦涩难懂.**

##### 常用正则

描述 | 正则表达式
---|---
是否为数字 | `^[0-9]*$`
是否为n位数数字 | `^\d{n}$`
是否为m-n位的数字 | `^\d{m,n}$`
是否输入至少n位的数字 | `^/d{n,}$"`
是否为整数 | `^-?/d+$`
是否为负整数 | `^-[0-9]*[1-9][0-9]*$`
是否为正整数 | `^[0-9]*[1-9][0-9]*$`
是否为汉字 | `^[\u4e00-\u9fa5]{0,}$`
是否为邮箱 | `^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$`
是否为域名 | `[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(/.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+/.?`
是否为URL | `^http://([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$`
是否为手机号码 | `^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$`
是否为固话号码 | `^($$\d{3,4}-)|\d{3.4}-)?\d{7,8}$`
是否为身份证号码 | `^([0-9]){7,18}(x|X)?$ 或^\d{15}|\d{18}$或 ^\d{8,18}|[0-9x]{8,18}|[0-9X]{8,18}?$`
是否以字母开头,长度在6~18之间,只能包含字母、数字和下划线 | `^[a-zA-Z]\w{5,17}$`
是否必须包含大小写字母和数字的组合,不能使用特殊字符,长度在8-10之间 | `^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,10}$`
是否为腾讯QQ号码 | `[1-9][0-9]{4,}` (QQ号从10000开始)
是否为中国邮政编码 | `[1-9]\d{5}(?!\d)`
是否为ip地址 | `((25[0-5]|2[0-4]\d|((1\d{2})|([1-9]?\d)))\.){3}\2`



## 控制流语句

### if-else语句
`if`语句是最基本的控制语句,它只有在`If(exception)`为`true`的时候才会执行特定的代码.
```java
    public static void main(String[] args) {
        boolean var1 = true;
        if (var1){
            System.out.println("hello world ...");
        }
    }
```
`if`语句后面可以跟`else`语句.当`If(exception)`为`false`时,`else`语句体将被执行.
```java
    public static void main(String[] args) {
        boolean var1 = false;
        if (var1){
            System.out.println("if ...");
        }else{
            System.out.println("else ...");
        }
    }
```
`if`语句后面可以跟 `else if…else` 语句
```java
public class HelloWorld {
    public static void main(String[] args) {
        String var = "123";
        if ("123".equals(var)){
            System.out.println("123 ...");
        }else if("234".equals(var)){
            System.out.println("234 ...");
        }else if ("345".equals(var)){
            System.out.println("345 ...");
        }else{
            System.out.println("...");
        }
    }

}
```

### switch语句
`switch`语句是在项目开发中是比较常用的.可以和`if...else`起到相同的作用,但是用`switch`代码可读性更高.

`switch case`语句判断一个变量与一系列值中某个值是否相等,每个值称为一个分支.

`switch`语句可以包含一个`default`分支. `default` 在没有`case`语句的值和变量值相等的时候执行.`default` 分支不需要`break`语句。
```java
    public static void main(String[] args) {
        String s = "123";
        switch (s) {
            case "123":
                System.out.println("123");
                break;
            case "456":
                System.out.println("456");
                break;
             default :
                System.out.println("over ...");
        }
    }   
```
`switch`语句中的变量类型可以是： `byte,short,int,char` 或者 `enum`从 Java 7 开始,可以在`switch`条件判断语句中使用`String`对象.

**switch 不支持 long**，是因为 `switch` 的设计初衷是对那些只有少数的几个值进行等值判断，如果值过于复杂，那么还是用 `if` 比较合适。

```java
long x = 111;
 switch (x) { 
     case 111:
         System.out.println(111);
         break;
     case 222:
         System.out.println(222);
         break;
}
```

当变量的值与`case`语句的值相等时,那么`case`语句之后的语句开始执行，直到`break`语句出现才会跳出`switch`语句.如果没有`break`语句出现,程序会继续执行下一条`case`语句,直到出现`break`语句.

```java
public static void main(String[] args) {
        int i = 100;
        switch (i){
            case 100:
                System.out.println("first case ...");
            case 200:
                System.out.println("second case ...");
                break;
        }
    }
```

### for语句
for循环执行的次数是在执行前就确定的.

for语句提供了一种紧凑的方法来遍历一系列值。程序员经常将其称为“for循环”，因为它反复循环直到满足特定条件。for语句的一般形式可以表示为:
```java
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(i);
        }
    }
```
for循环的三个表达式是可选的;可以这样创建一个无限循环:

```java
/ /无限循环
for(;;){

    // to do ...
}
```

`foreach`语句是java5的新特征之一，在遍历数组、集合方面,为开发人员提供了极大的方便.

**`foreach`循环的效率大概是普通for循环效率的一半** ,但在项目开发中如果只是少量的循环,可以忽略`foreach`带来的影响.
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};
        ArrayList<String> list = new ArrayList<>(Arrays.asList(arr));
        for (String s : list) {
            System.out.println(s);
        }
        for (String s : arr) {
            System.out.println(s);
        }
    }
```
**分支语句`break`,`continue`和`return`**

`continue`语句用来结束当前循环,并进入下一次循环,即仅仅这一次循环结束了,不是所有循环结束了,后边的循环依旧进行.
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};
        for (String s : arr) {
            // 如果s等于1,则结束本次循环 执行下次循环
            if ("1".equals(s)) {
                continue;
            }
            System.out.println("...");
        }
    }
```
`break`语句作用是跳出循环.`break`主要用在循环语句或者`switch`语句中
```java
public class HelloWorld {
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};
        for (String s : arr) {
            // s等于2,结束for循环
            if ("2".equals(s)) {
                break;
            }
            System.out.println("...");
        }
    }

}
```
如果存在多层循环,要注意`break`只能跳出一层循环.
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};
        for (int i = 0; i < 10; i++) {
            for (String s : arr) {
                if ("2".equals(s)) {
                    break;
                }
                System.out.println("...");
            }
            System.out.println("外层 for ...");
        }
    }
```
如果存在多层循环,可以用以下方式,当然也可以用`break`跳两次循环
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};
        
        test:
        for (int i = 0; i < 10; i++) {
            for (String s : arr) {
                if ("2".equals(s)) {
                    break test;
                }
                System.out.println("...");
            }
            System.out.println("外层 for ...");
        }
    }
```
`continue`也可以用这种方式
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};

        test:
        for (int i = 0; i < 10; i++) {
            for (String s : arr) {
                if ("1".equals(s)) {
                    continue test;
                }
            }
            System.out.println("外层 for ...");
        }
    }
```

`return`语句表示从当前方法退出,控制流返回到调用方法的地方。


上面`continue`的示例,用`return`也能达到一样的效果
```java
    public static void main(String[] args) {
        String[] arr = {"1", "2", "3"};

        for (int i = 0; i < 10; i++) {
            for (String s : arr) {
                if ("1".equals(s)) {
                    return;
                }
            }
            System.out.println("外层 for ...");
        }
    }
```

`return`语句有两种形式:一种返回值，另一种不返回值。要返回一个值，只需将值(或计算该值的表达式)放在`return`关键字之后. 

像这样
```java
return ++count;
```


### do-while和while语句

`while`语句对表达式进行计算，表达式必须返回一个布尔值。如果表达式的计算结果为`true`,`while`语句将执行`while`块中的语句。`while`语句继续测试表达式并执行其块，直到表达式的计算结果为`false`.

```java
    public static void main(String[] args) {
        int i = 0;
       while(++i >= 1){
           i--;
        }
    }
```
死循环
```java
    public static void main(String[] args) {
        while (true) {
        }
    }
```

`do…while`循环和`while`循环相似，不同的是`do…while`循环至少会执行一次.
```java
    public static void main(String[] args) {
        int i = 0;
        do{
            i++;
            System.out.println(" ... ");
        }while (--i == 1);
    }
```

**注意:布尔表达式在循环体的后面，所以语句块在检测布尔表达式之前已经执行了.如果布尔表达式的值为`true`，则语句块一直执行，直到布尔表达式的值为`false`**


## 参数传递

### 实参与形参
> 形式参数：是在定义函数名和函数体的时候使用的参数,目的是用来接收调用该函数时传入的参数。

> 实际参数：在调用有参函数时，主调函数和被调函数之间有数据传递关系。在主调函数中调用一个函数时，函数名后面括号中的参数称为“实际参数”。

举个例子:
```java
public class HelloWorld {
    public static void main(String[] args) {
        // 实参 
        test("world");
    }
    
    // 形参
    public static void test(String param){
        System.out.println("hello " + param);
    }
}
```

### 值传递与引用传递

> 值传递: 是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

> 引用传递: 是指在调用函数时将实际参数的地址直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

 ... | 值传递 | 引用传递
---|---|---
区别 | 传参时会创建副本 | 传参数不创建副本
描述 | 方法中无法修改原始对象 | 方法中可以修改原始对象


### Java传递对象参数

很多人理解Java中,传递的参数如果是普通类型，那就是值传递，如果是对象，那就是引用传递.这个理解是错误的.

举个例子:

**如果在方法中改变对象的字段值会改变原对象该字段值，因为改变的是同一个地址指向的内容。**

```
graph LR

A(main方法)-->对象1(main方法中的对象)
对象1==>对象值
B(func方法)-->对象2(func中的对象)
对象2==>|修改|对象值
```

```java
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());          // B
    }

    private static void func(Dog dog) {
        dog.setName("B");//dog.name="B";
    }
}
```
然后有人就说Java传递对象类型参数是引用传递.这一点在官方[《Java™教程》](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html)中有相关的描述.

> Passing Reference Data Type Arguments.
Reference data type parameters, such as objects, are also passed into methods by value. This means that when the method returns, the passed-in reference still references the same object as before. However, the values of the object's fields can be changed in the method, if they have the proper access level.<br>

**传递引用数据类型参数。
引用数据类型参数(如对象)也按值传递给方法。这意味着，当方法返回时，传入的引用仍然引用与之前相同的对象。但是，如果对象的字段具有适当的访问级别，则可以在方法中更改它们的值。**

> 例如，考虑任意类中的一个移动Circle对象的方法:

```java
public void moveCircle(Circle circle, int deltaX, int deltaY) {
    // code to move origin of circle to x+deltaX, y+deltaY
    circle.setX(circle.getX() + deltaX);
    circle.setY(circle.getY() + deltaY);
        
    // code to assign a new reference to circle
    circle = new Circle(0, 0);
}
```
> 使用以下参数调用该方法:

```java
moveCircle(myCircle, 23, 56)
```

> 在这个方法中，circle最初指的是`myCircle`。该方法将`circle`引用的对象(即`myCircle`)的x坐标和y坐标分别更改了23和56。当方法返回时，这些更改将持续存在。然后`circle`被赋予一个新的`circle`对象x = y = 0的引用。但是，这种重新分配不具有持久性，因为引用是按值传入的，不能更改。在方法中，circle指向的对象已经改变，但是，当方法返回时，`myCircle`仍然引用与方法调用之前相同的`circle`对象。

值传递和引用传递最大的区别是传递的过程中有没有复制出一个副本来，如果是传递副本，那就是值传递，否则就是引用传递。
在Java中，其实是通过值传递实现的参数传递，只不过对于Java对象的传递，传递的内容是对象的引用。
所以说 **==Java的参数是以值传递的形式传入方法中,而不是引用传递.==**

**Java对象的传递，是通过复制的方式把引用关系传递了，如果我们没有改引用关系，而是找到引用的地址，把里面的内容改了，是会对调用方有影响的，因为大家指向的是同一个共享对象。**

以下代码中`Dog`类中的`dog`是一个指针，存储的是对象的地址.
```java
public class Dog {

    String name;

    Dog(String name) {
        this.name = name;
    }

    String getName() {
        return this.name;
    }

    void setName(String name) {
        this.name = name;
    }

    String getObjectAddress() {
        return super.toString();
    }
}
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        // Dog@4554617c
        System.out.println(dog.getObjectAddress()); 
        func(dog);
        // Dog@4554617c
        System.out.println(dog.getObjectAddress()); 
        // A
        System.out.println(dog.getName());          
    }

    private static void func(Dog dog) {
        // Dog@4554617c
        System.out.println(dog.getObjectAddress()); 
        //重新new生成新的对象
        dog = new Dog("B");
        // Dog@74a14482
        System.out.println(dog.getObjectAddress()); 
        // B
        System.out.println(dog.getName());          
    }
}
```


## Java运算

### 精度丢失

在Java中,使用浮点类型进行计算会造成精度丢失

例如:
```java
    public static void main(String[] args) {
        // 0.20000005
        System.out.println(1.2f - 1);
        // 0.19999999999999996
        System.out.println(1.2d - 1);
    }
```

那么为什么会浮点类型会存在精度精度丢失问题呢?

因为[Java的浮点类型在计算机中是用二进制来存储](https://www.zhihu.com/question/46432979/answer/221485161)的,也就是小数在转二进制的时候出现了精度丢失.

PS: 小数如何转二进制
> 将该数字乘以2，取出整数部分作为二进制表示的第1位；然后再将小数部分乘以2，将得到的整数部分作为二进制表示的第2位；以此类推，直到小数部分为0.<br>
特殊情况： 小数部分出现循环，无法停止，则用有限的二进制位无法准确表示一个小数，这也是在编程语言中表示小数会出现误差的原因.

例如:  0.1 转二进制
```
    0.1 转2进制
    
    0.1 x 2 = 0.2  取整数位 0
    0.2 x 2 = 0.4  取整数位 0
    0.4 x 2 = 0.8  取整数位 0
    0.8 x 2 = 1.6  取整数位 1
    0.6 x 2 = 1.2  取整数位 1
    0.2 x 2 = 0.4  取整数位 0
    0.4 x 2 = 0.8  取整数位 0
    0.8 x 2 = 1.6  取整数位 1
    0.6 x 2 = 1.2  取整数位 1
    
    ........无限循环
```
因为计算机中存储一个浮点类型所用的位数是有限的,所以遇到无限循环的小数,只能选择在某个精度进行保存.

**由于计算机中保存的小数其实是十进制的小数的近似值，并不是准确值，所以，千万不要在代码中使用浮点数来表示金额等重要的指标。**


### BigDecimal

`BigDecimal`由任意精度的整数非标度值和32位的整数标度 `scale`组成。如果为零或正数，则标度是小数点后的位数。如果为负数，则将该数的非标度值乘以 10 的负`scale` 次幂。因此,`BigDecimal`表示的数值是`unscaledValue × 10-scale`.

> Java在`java.math`包中提供的API类`BigDecimal`，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数。**在实际应用中，需要对更大或者更小的数进行运算和处理。`float`和`double`只能用来做科学计算或者是工程计算，在商业计算中要用`java.math.BigDecimal`。** `BigDecimal`所创建的是对象，我们不能使用传统的`+、-、*、/`等算术运算符直接对其对象进行数学运算，而必须调用其相对应的方法。方法中的参数也必须是`BigDecimal`的对象。构造器是类的特殊方法，专门用来创建对象，特别是带有参数的对象.

Java中提供了大数字(超过16位有效位)的操作类,即`java.math.BinInteger`类和`java.math.BigDecimal`类,用于高精度计算.

其中`BigInteger`类是针对大整数的处理类,而`BigDecimal`类则是针对大小数的处理类.`BigDecimal`类的实现用到了 `BigDecimal`BigInteger类,不同的是`BigDecimal`加入了小数的概念.

之所以要用`BigDecimal`,是因为十进制的小数在转化成二进制浮点数时会精度丢失.

#### 使用

##### 基本运算

`BigDecimal`类创建的是对象,不能使用传统的`+、-、*、/`等算术运算符直接对其进行数学运算,而必须调用其对应的方法.方法的参数也必须是`BigDecimal`类型的对象.

```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("2");
        BigDecimal num2 = new BigDecimal("1");
        BigDecimal num3;

        num3 = num1.add(num2);
        System.out.printf("num1 + num2 = %s\n",num3);

        num3 = num1.subtract(num2);
        System.out.printf("num1 - num2 = %s\n",num3);

        num3 = num1.multiply(num2);
        System.out.printf("num1 * num2 = %s\n",num3);

        num3 = num1.divide(num2);
        System.out.printf("num1 / num2 = %s\n",num3);
        
        // 绝对值
        System.out.printf("|num1 / num2| = %s\n",num3.abs());
    }
```

`BigDecimal`基本用法如上所示,重点记录一下除法.

在使用除法的时候如果两个数字,除不尽.而又没有设置精确小数位和舍入模式,就会报错.
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1");
        BigDecimal num2 = new BigDecimal("3");
        BigDecimal num3;

        num3 = num1.divide(num2);
        System.out.printf("num1 / num2 = %s\n",num3);
    }
```
```
Exception in thread "main" java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
```
为了防止报错,我们可以这样写
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1");
        BigDecimal num2 = new BigDecimal("3");
        BigDecimal num3;

        num3 = num1.divide(num2,3, BigDecimal.ROUND_UP);
        // 打印结果: num1 / num2 = 0.334
        System.out.printf("num1 / num2 = %s\n",num3);
    }
```

`BigDecimal`类`divide`方法
```java
// divisor: 除数; scale: 精确小数位; roundingMode: 舍入模式
public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)
```
舍入模式,如果不指定默认采用四舍五入方式.

```
/**
     * Rounding mode to round away from zero.  Always increments the
     * digit prior to a nonzero discarded fraction.  Note that this rounding
     * mode never decreases the magnitude of the calculated value.
     * 向远离0的方向舍入 如2.35 --> 2.4
     */
    public final static int ROUND_UP =           0;

    /**
     * Rounding mode to round towards zero.  Never increments the digit
     * prior to a discarded fraction (i.e., truncates).  Note that this
     * rounding mode never increases the magnitude of the calculated value.
     * 向零方向舍入 直接删除多余小数位 如2.35 --> 2.3
     */
    public final static int ROUND_DOWN =         1;

    /**
     * Rounding mode to round towards positive infinity.  If the
     * {@code BigDecimal} is positive, behaves as for
     * {@code ROUND_UP}; if negative, behaves as for
     * {@code ROUND_DOWN}.  Note that this rounding mode never
     * decreases the calculated value.
     * 向正无穷方向舍入
     */
    public final static int ROUND_CEILING =      2;

    /**
     * Rounding mode to round towards negative infinity.  If the
     * {@code BigDecimal} is positive, behave as for
     * {@code ROUND_DOWN}; if negative, behave as for
     * {@code ROUND_UP}.  Note that this rounding mode never
     * increases the calculated value.
     * 向负无穷方向舍入
     */
    public final static int ROUND_FLOOR =        3;

    /**
     * Rounding mode to round towards {@literal "nearest neighbor"}
     * unless both neighbors are equidistant, in which case round up.
     * Behaves as for {@code ROUND_UP} if the discarded fraction is
     * &ge; 0.5; otherwise, behaves as for {@code ROUND_DOWN}.  Note
     * that this is the rounding mode that most of us were taught in
     * grade school.
     * 向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向上舍入, 1.55保留一位小数结果为1.6
     */
    public final static int ROUND_HALF_UP =      4;

    /**
     * Rounding mode to round towards {@literal "nearest neighbor"}
     * unless both neighbors are equidistant, in which case round
     * down.  Behaves as for {@code ROUND_UP} if the discarded
     * fraction is {@literal >} 0.5; otherwise, behaves as for
     * {@code ROUND_DOWN}.
     * 向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，向下舍入, 例如1.55 保留一位小数结果为1.5 
     */
    public final static int ROUND_HALF_DOWN =    5;

    /**
     * Rounding mode to round towards the {@literal "nearest neighbor"}
     * unless both neighbors are equidistant, in which case, round
     * towards the even neighbor.  Behaves as for
     * {@code ROUND_HALF_UP} if the digit to the left of the
     * discarded fraction is odd; behaves as for
     * {@code ROUND_HALF_DOWN} if it's even.  Note that this is the
     * rounding mode that minimizes cumulative error when applied
     * repeatedly over a sequence of calculations.
     * 向（距离）最近的一边舍入，除非两边（的距离）是相等,如果是这样，如果保留位数是奇数，使用ROUND_HALF_UP ，如果是偶数，使用ROUND_HALF_DOWN 
     */
    public final static int ROUND_HALF_EVEN =    6;

    /**
     * Rounding mode to assert that the requested operation has an exact
     * result, hence no rounding is necessary.  If this rounding mode is
     * specified on an operation that yields an inexact result, an
     * {@code ArithmeticException} is thrown.
     * 计算结果是精确的，不需要舍入模式 
     */
    public final static int ROUND_UNNECESSARY =  7;
```
所以我们在使用`BigDecimal`除法的时候,最好要指定精确小数位和舍入模式.

##### 初始化
`BigDecimal`一共有两种方法可以进行初始化赋值
 - 构造器初始化赋值
 - `valueOf`方法初始化赋值

###### valueOf
`valueOf(double val)`底层也是调用`String`构造
```java
    public static BigDecimal valueOf(double val) {
        // Reminder: a zero double returns '0.0', so we cannot fastpath
        // to use the constant ZERO.  This might be important enough to
        // justify a factory approach, a cache, or a few private
        // constants, later.
        return new BigDecimal(Double.toString(val));
    }
```
`valueOf(long val)`
```java
    public static BigDecimal valueOf(long val) {
        // 判断在不在 缓存常用的小BigDecimal值中
        if (val >= 0 && val < zeroThroughTen.length)
            return zeroThroughTen[(int)val];
        else if (val != INFLATED)
            return new BigDecimal(null, val, 0, 0);
        // 会调用BigInteger中 valueOf(long val)涉及到BigDecimal原理
        return new BigDecimal(INFLATED_BIGINT, val, 0, 0);
    }
```

###### double类型构造器

重点记录一下**double类型构造器初始化赋值**

在使用`BigDecimal`中参数为`double`类型的构造器时,发现存储结果并不准确.
```java
    public static void main(String[] args) {
        // 打印结果: 0.200000000000000011102230246251565404236316680908203125
        System.out.println(new BigDecimal(0.2));
    }
```
发现Java API 中有相关的描述
> The results of this constructor can be somewhat unpredictable.
One might assume that writing {@code new BigDecimal(0.1)} in
Java creates a {@code BigDecimal} which is exactly equal to
0.1 (an unscaled value of 1, with a scale of 1), but it is
actually equal to
0.1000000000000000055511151231257827021181583404541015625.
This is because 0.1 cannot be represented exactly as a
{@code double} (or, for that matter, as a binary fraction of
any finite length).  Thus, the value that is being passed
<i>in</i> to the constructor is not exactly equal to 0.1,
appearances notwithstanding.<br><br>
>  The {@code String} constructor, on the other hand, is perfectly predictable: writing {@code new BigDecimal("0.1")} creates a {@code BigDecimal} which is <i>exactly</i> equal to
0.1, as one would expect.  Therefore, it is generally
recommended that the {@linkplain #BigDecimal(String)
<tt>String</tt> constructor} be used in preference to this one.

大概意思是说 用`double`作为参数的构造函数，无法精确构造一个`BigDecimal`对象，需要自己指定一个上下文的环境，也就是指定精确位。

而利用String对象作为参数传入的构造函数能精确的构造出一个`BigDecimal`对象.

```java
    public static void main(String[] args) {
        // 0.2
        System.out.println(new BigDecimal("0.2"));
    }
```
所以要想获得精确的结果,要使用`BigDecimal`的字符串构造函数,不要使用`double`参数的构造函数.

PS: 另外说一下`BigDecimal`转换其他类型.`BigDecimal`类提供了`intValue,byteValue,shortValue`...将`BigDecimal`对象转换成对应的值.
```java
    public static void main(String[] args) {
        BigDecimal bigDecimal = new BigDecimal("1.2");
        System.out.println(bigDecimal.byteValue());
        System.out.println(bigDecimal.shortValue());
        System.out.println(bigDecimal.intValue());
        System.out.println(bigDecimal.floatValue());
        System.out.println(bigDecimal.doubleValue());
        System.out.println(bigDecimal.longValue());
    }
```

##### 比较

用`BigDecimal`的equals来进行比较
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1.10");
        BigDecimal num2 = new BigDecimal("1.1");
        BigDecimal num3 = new BigDecimal("1.1");
        // false
        System.out.println(num1.equals(num2));
        // true
        System.out.println(num2.equals(num3));
    }
```
我们可以看到,用`BigDecimal`的`equals`方法进行比较,我们可以看到`1.10`和`1.1`数值是相等的,而`equals`方法返回的却是`false`.

为了查清原因,看一下`BigDecimal`中重写的`equals`的源码
```java
    /**
     * Compares this {@code BigDecimal} with the specified
     * {@code Object} for equality.  Unlike {@link
     * #compareTo(BigDecimal) compareTo}, this method considers two
     * {@code BigDecimal} objects equal only if they are equal in
     * value and scale (thus 2.0 is not equal to 2.00 when compared by
     * this method).
     *
     * @param  x {@code Object} to which this {@code BigDecimal} is
     *         to be compared.
     * @return {@code true} if and only if the specified {@code Object} is a
     *         {@code BigDecimal} whose value and scale are equal to this
     *         {@code BigDecimal}'s.
     * @see    #compareTo(java.math.BigDecimal)
     * @see    #hashCode
     */
    @Override
    public boolean equals(Object x) {
        if (!(x instanceof BigDecimal))
            return false;
        BigDecimal xDec = (BigDecimal) x;
        if (x == this)
            return true;
        if (scale != xDec.scale)
            return false;
        long s = this.intCompact;
        long xs = xDec.intCompact;
        if (s != INFLATED) {
            if (xs == INFLATED)
                xs = compactValFor(xDec.intVal);
            return xs == s;
        } else if (xs != INFLATED)
            return xs == compactValFor(this.intVal);

        return this.inflated().equals(xDec.inflated());
    }
```
源码API注释中有写.
> objects equal only if they are equal in value and scale. thus 2.0 is not equal to 2.00 when compared by this method

大概意思: 对象只有在`value`和`scale`相等时才相等.因此,用本方法比较时,2.0不等于2.00.

PS: `BigDecimal`对象中`scale`字段属性
```java
    /**
     * The scale of this BigDecimal, as returned by {@link #scale}.
     *
     * @serial
     * @see #scale
     */
    private final int scale;  // Note: this may have any value, so
                              // calculations must be done in longs
```
`scale`指的是小数点后面的位数.


`BigDecimal`可以通过`setScale`来提高精度,只要新设的值比原来的大.
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1.12345");
        num1.setScale(8);
        System.out.println(num1);
    }
    
```
`setScale`方法返回一个`BigDecimal`对象,它的`scale`是指定的值，并且它的值在数字上大于等于这个`BigDecimal`对象的值。

```java
public BigDecimal setScale(int newScale);
```
如果不是,则抛出`arithmex exception`.

例如: 在上面的示例中,如果设置`setScale(4)`就会报错
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1.12345");
        num1.setScale(4);
        System.out.println(num1);
    }
```
```txt
java.lang.ArithmeticException: Rounding necessary
```

`BigDecimal`也可以通过`setScale`来降低精度.因为新设的值比原来的小,所以必须保证原来数值的该位小数点后面都是0,只有这样才可以设比原来小的精度。

例如
```java
    public static void main(String[] args) {
        BigDecimal num1 = new BigDecimal("1.1234500000");
        num1.setScale(5);
        // 1.1234500000
        System.out.println(num1);
    }
```

所以不要用`equals`方法来比较`BigDecimal`对象,因为它的equals方法会比较`scale`,如果`scale`不一样,它会返回false.

比较大小建议使用`BigDecimal`类中重写的`compareTo`方法进行比较

```java
public static void main(String[] args) {
        BigDecimal a = new BigDecimal("1.10");
        BigDecimal b = new BigDecimal("1.1");

        if(a.compareTo(b) == -1){
            System.out.println("a小于b");
        }

        if(a.compareTo(b) == 0){
            System.out.println("a等于b");
        }

        if(a.compareTo(b) == 1){
            System.out.println("a大于b");
        }

        if(a.compareTo(b) > -1){
            System.out.println("a大于等于b");
        }

        if(a.compareTo(b) < 1){
            System.out.println("a小于等于b");
        }
    }

```


#### 保证精度原因及注意事项

**为什么`BigDecimal`能够保证精度?**

**因为十进制整数在转化成二进制数时不会有精度问题,那么把十进制小数扩大N倍让它在整数的维度上进行计算,并保留相应的精度信息.所以就能保证精度了.**



**注意事项**

1.在需要精确的小数计算时再使用`BigDecimal`,`BigDecimal`的性能比`double`和`float`差,在处理庞大,复杂的运算时尤为明显.故一般精度的计算没必要使用`BigDecimal`.


2. 尽量使用参数类型为String的构造函数.如果处理`double`类型数据,可使用`BigDecimal.valueOf(double val)`


3. `BigDecimal`都是不可变的的,在进行每一次四则运算时,都会产生一个新的对象,所以在做加减乘除运算时要记得要保存操作后的值.

### Math

`java.lang.Math`,该类和Java中的运算息息相关.`Math`类被`final`修饰.构造方法是私有的.`Math`类中大部分方法都被`public static` 修饰.

#### 常用方法

```java
public static void main(String[] args) {
        // 计算平方根
        System.out.printf("4的平方根: %s\n",Math.sqrt(4));

        //计算立方根
        System.out.printf("8的立方根: %s\n",Math.cbrt(8));

        // 计算n的m次方
        System.out.printf("2的3次方: %s\n",Math.pow(2,3));

        // 计算最大值
        System.out.printf("1和2中最大值: %s\n",Math.max(1,2));

        // 计算最小值
        System.out.printf("1和2中最小值: %s\n",Math.min(1,2));

        // 求绝对值
        System.out.printf("-1的绝对值: %s\n",Math.abs(-1));

        // 向上取整
        System.out.printf("1.2向上取整: %s\n",Math.ceil(1.2));

        // 向下取整
        System.out.printf("1.2向下取整: %s\n",Math.floor(1.2));

        // [0,1)区间的随机数
        System.out.printf("[0,1)区间的随机数: %s\n",Math.random());

        // 返回与参数值最接近的 double值
        double rint = Math.rint(1.5);
        System.out.printf("rint方法: %s\n",rint);

        // 四舍五入 float时返回int值，double时返回long值
        long round = Math.round(1.5);
        int round1 = Math.round(1.5f);
        System.out.printf("round方法: 1.5四舍五入: %s\n",round);
    }
```
```
4的平方根: 2.0
8的立方根: 2.0
2的3次方: 8.0
1和2中最大值: 2
1和2中最小值: 1
-1的绝对值: 1
1.2向上取整: 2.0
1.2向下取整: 1.0
[0,1)区间的随机数: 8.293290468311953E-4
rint方法: 1.5四舍五入: 2.0
round方法: 1.5四舍五入: 2
```

其中`sqrt`方法,`cbrt`方法,`pow`方法用`native`关键字修饰,是用其他语言实现的.这里不做过多分析.

##### max
```java
        // 计算最大值
        System.out.printf("1和2中最大值: %s\n",Math.max(1,2));
```

> 返回两个`{@code int}`值中较大的一个。也就是说，结果是参数更接近`{@link Integer#MAX_VALUE}`的值。如果参数具有相同的值，则结果为相同的值。

源码
```java
    public static int max(int a, int b) {
        return (a >= b) ? a : b;
    }
```

##### min
```java
        // 计算最小值
        System.out.printf("1和2中最小值: %s\n",Math.min(1,2));
```

> 返回两个`{@code int}`值中较小的一个。也就是说，参数的结果更接近`{@link Integer#MIN_VALUE}`的值。如果参数具有相同的值，则结果为相同的值。

源码
```java
    public static int min(int a, int b) {
        return (a <= b) ? a : b;
    }
```

##### abs
```java
        // 求绝对值
        System.out.printf("-1的绝对值: %s\n",Math.abs(-1));
```

> 返回`{@code int}`值的绝对值。如果实参不是负数，则返回实参。如果参数为负数，则返回参数的否定值。

源码
```java
    public static int abs(int a) {
        return (a < 0) ? -a : a;
    }
```

##### ceil和floor
`ceil`方法的功能是向上取整。`ceil`译为“天花板”，顾名思义就是对操作数取顶，`Math.ceil(a)`就是取大于a的最小整数。需要注意的是它的返回值不是`int`类型，而是`double`类型.
```java
        // 向上取整
        System.out.printf("1.2向上取整: %s\n",Math.ceil(1.2));
```
`floor`方法的功能是向下取整。`floor`译为“地板”，顾名思义是对操作数取底。`Math.floor(a)`，就会取小于a的最大整数。它的返回值类型与`ceil`一致，也是`double`类型。
```java
        // 向下取整
        System.out.printf("1.2向下取整: %s\n",Math.floor(1.2));
```
**由于`ceil`方法和`floor`方法逻辑是一样的,区别只是传入的参数不同,所以下面主要分析一个方法.**

以`ceil`方法为例
> 返回大于或等于参数的最小(最接近负无穷){@code double}值，并且等于一个数学整数。<br>
特殊情况:<br>
- 如果参数值已经等于一个数学整数，则结果与参数相同。
- 如果参数是NaN或无穷大或正零或负零，则结果与参数相同。
- 如果实参值小于零但大于-1.0，则结果为负零。
注意`{@code Math.ceil(x)}`的值正好是`{@code -Math.floor(-x)}`的值。

源码
```java
    public static double ceil(double a) {
        // default impl. delegates to StrictMath
        return StrictMath.ceil(a);
    }
    /**
     * |
     * |
     * |
     * ↓
     */
    // StrictMath.ceil
    public static double ceil(double a) {
        return floorOrCeil(a, -0.0, 1.0, 1.0);
    }
    /**
     * |
     * |
     * |
     * ↓
     */
   /*
    *内部方法共享floor和ceil之间的逻辑。
    *
    * @param a 被floored或ceiled的值
    * @param negativeBoundary (- 1,0)中的值的结果
    * @param positiveBoundary (0,1)中的值的结果
    * @param increment 当参数是非整数时要添加的值
    */
    private static double floorOrCeil(double a,
                                      double negativeBoundary,
                                      double positiveBoundary,
                                      double sign) {
        /**
         * 返回用于{@code double}表示的无偏指数。
         *
         * 特殊情况:
         * 
         * 如果参数为NaN或infinite，则结果为{@link Double#MAX_EXPONENT} + 1。
         * 如果参数为零或低于标准，则结果为{@link Double#MIN_EXPONENT} -1。
         */
        // 将浮点数或双精度数转换为浮点表示形式.该方法从表示中返回指数部分
        int exponent = Math.getExponent(a);

        if (exponent < 0) {
            /*
             * 参数的绝对值小于1
             * floorOrceil(-0.0) => -0.0
             * floorOrceil(+0.0) => +0.0
             */
            return ((a == 0.0) ? a :
                    ( (a < 0.0) ?  negativeBoundary : positiveBoundary) );
        } else if (exponent >= 52) {
            /*
             * 无穷，NaN，或者一个很大的值.但一定是整数
             */
            return a;
        }
        // 否则，参数要么是一个已经异或运算的整数值
        // 必须四舍五入为1
        assert exponent >= 0 && exponent <= 51;

        long doppel = Double.doubleToRawLongBits(a);
        long mask   = DoubleConsts.SIGNIF_BIT_MASK >> exponent;

        if ( (mask & doppel) == 0L )
            return a; // integral value
        else {
            double result = Double.longBitsToDouble(doppel & (~mask));
            if (sign*a > 0.0)
                result = result + sign;
            return result;
        }
    } 
    
```


##### random
返回一个大于0的`double`类型数据，该值大于等于0.0且小于1.0，返回的是一个伪随机选择数，在该范围内（几乎）均匀分布.
```java
        // [0,1)区间的随机数
        System.out.printf("[0,1)区间的随机数: %s\n",Math.random());
```
API中`random`方法描述
> 返回一个大于或等于`{@code 0.0}`且小于`{@code 1.0}`的`{@code double}`值，带有正号。返回值是在该范围内(近似)均匀分布的伪随机选择的。<br>
当这个方法第一次被调用时，它会创建一个新的伪随机数生成器，与表达式`{@code new java.util.Random()}`完全一样。
此方法被正确同步以允许多个线程正确使用。但是，如果许多线程需要以非常快的速度生成伪随机数，那么每个线程拥有自己的伪随机数生成器就可以减少争用。

API中`nextDouble`方法中描述
> 返回该随机数生成器序列中在`{@code 0.0}`和`{@code 1.0}`之间均匀分布的`{@code double}`值的下一个伪随机值。
`{@code nextDouble}`的一般约定是，从`{@code 0.0d}`(包括)到`{@code 1.0d}`(排除)范围内(近似地)统一选择一个`{@code double}`值，伪随机生成并返回。

源码
```java
    public static double random() {
        return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
    }
    /**
     * |
     * |
     * |
     * ↓
     */
    // Random.nextDouble
    public double nextDouble() {
        return (((long)(next(26)) << 27) + next(27)) * DOUBLE_UNIT;
    }
```
通过源码可以看到,`Math.random`底层是调用了`Random`类中`nextDouble`方法.

也就是说以下打印语句**执行代码相同**
```java
    // [0,1)区间的随机数
    System.out.printf("[0,1)区间的随机数: %s\n",Math.random());
    System.out.printf("[0,1)区间的随机数: %s\n",new Random().nextDouble());
```

**Random**类
> 该类的一个实例用于生成**伪随机数**流。该类使用48位种子，并使用线性同余公式进行修改。(参见Donald Knuth， 计算机编程的艺术，第二卷，第3.2.1节。)

> 如果用相同的种子创建了`{@code Random}`的两个实例，并且对每个实例进行了相同的方法调用序列，那么它们将生成并返回相同的数字序列。为了保证这个属性，为类`{@code Random}`指定了特定的算法。为了保证Java代码的绝对可移植性，Java实现必须对`{@code Random}`类使用这里显示的所有算法。然而，`{@code Random}`类的子类约定了所有的方法。

`Random`类位于`java.util`包下，此类的实例用于生成伪随机数流。之所以称之为伪随机，是因为真正意义上的随机数（或者称为随机事件）在某次产生过程中是按照实验过程表现的分布概率随机产生的，其结果是不可预测，不可见的。而计算机中的随机函数是按照一定的算法模拟产生的，其结果是确定的，可见的。我们认为这样产生的数据不是真正意义上的随机数，因而称之为伪随机。

该类提供了两种构造方法.

无参构造底层调用的也是有参构造.将`System.nanoTime()`作为参数传递。即如果使用无参构造，默认的seed值为`System.nanoTime()`。
```java
    public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }
```

```java
    public Random(long seed) {
        if (getClass() == Random.class)
            this.seed = new AtomicLong(initialScramble(seed));
        else {
            // subclass might have overriden setSeed
            this.seed = new AtomicLong();
            setSeed(seed);
        }
    }
```

要注意的是用有参构造创建`Random`对象,如果随机种子相同,不管执行多少次,最后结果都是相同的.

例如
```java
    public static void main(String[] args) {
        Random random = new Random(1);
        // 第一次执行程序打印结果: 随机数: -1155869325
        // 第二次执行程序打印结果: 随机数: -1155869325
        System.out.printf("随机数: %s\n",random.nextInt());
    }
```
**随机数应用举例**

生成100个不重复的随机数,1~100的范围
```java
    public static void main(String[] args) {
        int[] nums=new int[100];
        boolean[] flag=new boolean[101];
        Random random=new Random();
        for (int i = 0; i < nums.length; i++) {
            int num=random.nextInt(100)+1;
            while (flag[num]) {
                num=random.nextInt(100)+1;
            }
            flag[num]=true;
            nums[i]=num;
        }
        Arrays.sort(nums);
        System.out.println(Arrays.toString(nums));
        System.out.println(Arrays.toString(flag));
    }
```


##### rint
> 返回与参数值最接近的`{@code double}`值，该值等于一个数学整数。如果两个数学整数`{@code double`}值相同，则结果为偶数整数值.<br><br>   
特殊情况:<br>
如果参数值已经等于一个数学整数，则结果与参数相同。<br>
如果参数是NaN或无穷大或正零或负零，则结果与参数相同

```java
        // 返回与参数值最接近的 double值
        double rint = Math.rint(1.5);
        System.out.printf("rint方法: %s\n",rint);
```
若存在两个这样的数,则返回其中的偶数值

例如
```java
    public static void main(String[] args) {
        double rint = Math.rint(100.5);
        double rint2 = Math.rint(101.5);  
        // 100.0
        System.out.printf(" %s\n",rint);
        // 102.0
        System.out.printf(" %s\n",rint2); 
    }
```


##### round
> round 表示"四舍五入"，算法为`Math.floor(x+0.5)` ，即将原来的数字加上 0.5 后再向下取整，所以 `Math.round(11.5)` 的结果为 12，`Math.round(-11.5)` 的结果为 -11。
```java
        // 四舍五入 float时返回int值，double时返回long值
        long round = Math.round(1.5);
        int round1 = Math.round(1.5f);
        System.out.printf("round方法: 1.5四舍五入: %s\n",round);
```

> 返回与实参最接近的`{@code int}`，并四舍五入到正无穷。
特殊情况:
- 如果参数是NaN，结果是0。
如果参数是负无穷或任何值小于或等于{`@code Integer.MIN_VALUE}`，结果等于`{@code Integer.MIN_VALUE}`的值。
- 如果参数是正无穷或任何大于或等于`{@code Integer.MAX_VALUE}`，结果等于`{@code Integer.MAX_VALUE}`的值.

> **PS: NAN
NaN表示非数值，例如：0.0/0结果为NAN，负数的平方根结果也为NAN.**

```java
   public static void main(String[] args) {
        // 四舍五入 float时返回int值，double时返回long值
        int round1 = Math.round(0);
        long round2 = Math.round(Double.NaN);
        int round3 = Math.round(2147483648123L);
        int round4 = Math.round(-2147483647123L);
        System.out.printf("0四舍五入: %s\n", round1);
        System.out.printf("Double.NaN=[%s],四舍五入: %s\n", Double.NaN, round2);
        System.out.printf("Integer.MAX_VALUE=[%s] + 1 四舍五入: %s\n", Integer.MAX_VALUE, round3);
        System.out.printf("Integer.MIN_VALUE=[%s] - 1 四舍五入: %s\n", Integer.MIN_VALUE, round4);
    }
```
```
0四舍五入: 0
Double.NaN=[NaN],四舍五入: 0
Integer.MAX_VALUE=[2147483647] + 1 四舍五入: 2147483647
Integer.MIN_VALUE=[-2147483648] - 1 四舍五入: -2147483648
```

