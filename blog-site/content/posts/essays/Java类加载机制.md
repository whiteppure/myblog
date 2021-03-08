---
title: "Java类加载机制"
date: 2021-02-05
draft: false
tags: ["Java","Jvm"]
slug: "jvm-classloader"
---

## 类加载过程
Java 的类加载过程可以分为5个阶段：加载、验证、准备、解析和初始化。其中验证,准备,解析统称为连接。这5个阶段一般是顺序发生的，但在动态绑定的情况下，解析阶段发生在初始化阶段之后。

需要类加载器寻找类的字节码文件，并构造出类在JVM内部表示的对象组件。在Java中，[类加载器](#类加载器)把一个类装入JVM中，要经过以下步骤：

### 加载
> 加载阶段是类加载过程的第一个阶段。在这个阶段，JVM 的主要目的是将字节码从各个位置（网络、磁盘等）转化为二进制字节流加载到内存中，接着会为这个类在 JVM 的方法区创建一个对应的 Class 对象，这个 Class 对象就是这个类各种数据的访问入口。

类的加载指的是将类的`.class`文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个`java.lang.Class`对象，用来封装类在方法区内的数据结构.简单来说，加载指的是把class字节码文件从各个来源通过类加载器装载入内存中。

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了`.class`文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（`LinkageError`错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

 JVM不是一开始就把所有的类都加载进内存中，而是只有第一次遇到某个需要运行的类时才会加载，且只加载一次。

JVM在类加载阶段作用
- 通过类的全限定名获取该类的二进制字节流
- 将字节流所代表的存储结构转化为方法区的运行时的数据结构
- 在内存中生成一个该类的`java.lang.Class`对象作为方法区的这个类的各种数据访问入口.

### 验证
JVM会在该阶段对二进制字节流进行校验，只有符合JVM字节码规范的才能被 JVM 正确执行。

大致都会完成以下四个阶段的验证
- 文件格式验证
- 元数据验证,是否符合java语言的规范
- 字节码验证,确保程序语义合法,符合逻辑
- 符号引用验证,确保下一步解析能正常执行

### 准备
为静态变量在方法取分配内存,并设置默认初始值。将在方法区中进行分配.

举个例子:
```
public String var1 = "var1";
public static String var2 = "var2";
public static final String var3 = "var3";
```
变量`var1`不会被分配内存,但是`var2`会被分配.`var2`会被分配初始值为`null`而不是'var2'.

这里所设置的初始值通常情况下是数据类型默认的零值（如`0、0L、null、false`等），而不是被在Java代码中被显式地赋予的值。

实例变量会在对象实例化时随着对象一块分配在Java堆中。

这里不包含`final`修饰的`static`,因为`final`在编译的时候就已经分配了.也就是说`var3`被分配的值为'var3'

### 解析
虚拟机将常量池中的**符号引用**替换为**直接引用**.

*符号引用*

符号引用是编译原理中的概念，是相对于直接引用来说的。主要包括了以下三类常量： 类和接口的全限定名 字段的名称和描述符 方法的名称和描述符.

> 符号引用 ：符号引用以一组符号来描述所引用的目标。符号引用可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可，符号引用和虚拟机的布局无关。

在编译的时候每个java类都会被编译成一个class文件，但在编译的时候虚拟机并不知道所引用类的地址，所以就用符号引用来代替，而在这个解析阶段就是为了把这个符号引用转化成为真正的地址的阶段。

*直接引用*

直接引用和虚拟机的布局是相关的，不同的虚拟机对于相同的符号引用所翻译出来的直接引用一般是不同的。如果有了直接引用，那么直接引用的目标一定被加载到了内存中。

直接引用通过对符号引用进行解析，找到引用的实际内存地址.

### 初始化
在准备阶段，静态变量已经被赋过默认初始值，而在初始化阶段，静态变量将被赋值为代码期望赋的值.

举个例子
```
public static String var2 = "var2";
```
在准备阶段变量`var2`的值为`null`,在初始化阶段值为'var2'.

在Java中对类变量进行初始值设定有两种方式：
- 声明类变量时指定初始值
- 使用静态代码块为类变量指定初始值

#### 初始化步骤
- 如果这个类还没有被加载和连接，则程序先加载并连接该类
- 如果该类的直接父类还没有被初始化，则先初始化其直接父类
- 如果类中有初始化语句，则系统依次执行这些初始化语句

#### 何时初始化
- 创建类的实例，也就是`new`一个对象需要初始化
- 读取或者设置静态字段的时候需要初始化(但被`final`修饰的字段,在编译时就被放入静态常量池的字段除外.)
- 调用类的静态方法
- 使用反射`Class.forName("");`对类反射调用的时候,该类需要初始化
- 初始化一个类的时候,有父类,先初始化父类
    - 接口除外,父接口在调用的时候才会被初始化;
    - 子类引用父类的静态字段,只会引发父类的初始化;
- 被标明为启动类的类(即包含`main()`方法),需要初始化    

#### 初始化顺序
存在继承关系的代码中,初始化顺序
```
父类静态域 --> 子类静态域 --> 父类成员初始化 -->父类构造块 --->父类构造方法 -->子类成员初始化 -->子类构造块 -->子类构造方法
```
`静态域是指静态代码块和静态变量`

一些初始化规则:
> - 类从顶至底的顺序初始化，所以声明在顶部的字段的早于底部的字段初始化
>- 超类早于子类和衍生类的初始化
>- 如果类的初始化是由于访问静态域而触发，那么只有声明静态域的类才被初始化，而不会触发超类的初始化或者子类的
>- 初始化即使静态域被子类或子接口或者它的实现类所引用。
>- 接口初始化不会导致父接口的初始化。
>- 静态域的初始化是在类的静态初始化期间，非静态域的初始化时在类的实例创建期间。这意味这静态域初始化在非静态域之前。
>- 非静态域通过构造器初始化，子类在做任何初始化之前构造器会隐含地调用父类的构造器，他保证了非静态或实例变量（父类）初始化早于子类

## 类加载器
JVM设计者把**类加载阶段中的"通过'类全名'来获取定义此类的二进制字节流"** 这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。实现这个动作的代码模块称为“类加载器”。

类加载器。一般包括**启动类加载器，扩展类加载器，应用类加载器，以及用户的自定义类加载器.**

### 双亲委派模型
从虚拟机的角度来说，只存在两种不同的类加载器：一种是启动类加载器(`Bootstrap ClassLoader`)，该类加载器使用C++语言实现(这里仅限于`Hotspot`，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的)，属于虚拟机自身的一部分。另外一种就是所有其它的类加载器，这些类加载器是由Java语言实现，独立于JVM外部，并且全部继承自抽象类`java.lang.ClassLoader。`

站在Java开发人员的角度来看，类加载器可以大致划分为以下三类:

- 启动类加载器：Bootstrap ClassLoader，跟上面相同。它负责加载存放在`JDK\jre\lib`(JDK代表JDK的安装目录，下同)下，或被`-Xbootclasspath`参数指定的路径中的，并且能被虚拟机识别的类库（如`rt.jar`，所有的`java.*`开头的类均被`Bootstrap ClassLoader`加载）。启动类加载器是无法被Java程序直接引用的。


- 扩展类加载器：Extension ClassLoader，该加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载`JDK\jre\lib\ext`目录中，或者由`java.ext.dirs`系统变量指定的路径中的所有类库（如`javax.*`开头的类），开发者可以直接使用扩展类加载器。


- 应用程序类加载器：Application ClassLoader，该类加载器由`sun.misc.Launcher$AppClassLoader`来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，**一般情况下这个就是程序中默认的类加载器。**

![类加载器](/myblog/posts/images/essays/类加载器.png)

 这种层次关系称为**类加载器的双亲委派模型。** 我们把每一层上面的类加载器叫做当前层类加载器的父加载器，当然，它们之间的父子关系并不是通过继承关系来实现的，而是使用组合关系来复用父加载器中的代码。该模型在JDK1.2期间被引入并广泛应用于之后几乎所有的Java程序中，但它并不是一个强制性的约束模型，而是Java设计者们推荐给开发者的一种类的加载器实现方式。

双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

使用这种模型来组织类加载器之间的关系的好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如`java.lang.Object`类，无论哪个类加载器去加载该类，最终都是由启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。否则的话，如果不使用该模型的话，如果用户自定义一个`java.lang.Object`类且存放在`classpath`中，那么系统中将会出现多个Object类，应用程序也会变得很混乱。如果我们自定义一个`rt.jar`中已有类的同名Java类，会发现JVM可以正常编译，但该类永远无法被加载运行。

在`rt.jar`包中的`java.lang.ClassLoader`类中，我们可以查看类加载实现过程`loadClass`方法的代码，具体源码如下：
```
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            // 首先检查该name指定的class是否有被加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                   // 如果parent不为null，则调用parent的loadClass进行加载
                        c = parent.loadClass(name, false);
                    } else {
                    // parent为null，则调用BootstrapClassLoader进行加载  
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   // 如果从非空父类加载器中找不到类，则抛出ClassNotFoundException
                }

                if (c == null) {
                // 如果仍然无法加载成功，则调用自身的findClass进行加载 
                    long t1 = System.nanoTime();
                    c = findClass(name);

                   // 这是定义类加载器；记录统计数据
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

根据代码以及代码中的注释可以很清楚地了解整个过程：先检查是否已经被加载过，如果没有则调用父加载器的`loadClass()`方法，如果父加载器为空则默认使用启动类加载器作为父加载器。如果父类加载器加载失败，则先抛出`ClassNotFoundException`,然后再调用自己的`findClass()`方法进行加载。

### 自定义类加载器

> PS 为什么会有自定义类加载器？
> - 一方面是由于java代码很容易被反编译，如果需要对自己的代码加密的话，可以对编译后的代码进行加密，然后再通过实现自己的自定义类加载器进行解密，最后再加载。
> - 另一方面也有可能从非标准的来源加载代码，比如从网络来源，那就需要自己实现一个类加载器，从指定源进行加载。

若要实现自定义类加载器，只需要继承`java.lang.ClassLoader`类.按需重写相关方法即可.
通过对上边`Classloader`类中`loadClass()`源码分析,可以看出
- 如果不想打破双亲委派模型，那么只需要重写`findClass`方法
- 如果想打破双亲委派模型，那么就重写整个`loadClass`方法

> **在JDK1.2之前，类加载尚未引入双亲委派模式，因此实现自定义类加载器时常常重写`loadClass`方法，提供双亲委派逻辑，从JDK1.2之后，双亲委派模式已经被引入到类加载体系中，自定义类加载器时不需要在自己写双亲委派的逻辑，因此不鼓励重写`loadClass`方法，而推荐重写`findClass`方法。**
>
>**在Java中，任意一个类都需要由加载它的类加载器和这个类本身一同确定其在java虚拟机中的唯一性，即比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提之下才有意义，否则，即使这两个类来源于同一个Class类文件，只要加载它的类加载器不相同，那么这两个类必定不相等(这里的相等包括代表类的Class对象的`equals()`方法、`isAssignableFrom()`方法、`isInstance()`方法和`instanceof`关键字的结果)。**

**重写`findClass`方法**

准备一个class文件，编译后放到C盘根目录下
```
public class People {
	private String name;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}

```
自定义类加载器,继承`ClassLoader`重写`findClass`方法（其中`defineClass`方法可以把二进制流字节组成的文件转换为一个`java.lang.Class`）
```
public class MyClassLoader extends ClassLoader
{
    public MyClassLoader(){}
    
    public MyClassLoader(ClassLoader parent)
    {
        super(parent);
    }
    
    protected Class<?> findClass(String name) throws ClassNotFoundException
    {
    	File file = new File("C:/People.class");
        try{
            byte[] bytes = getClassBytes(file);
            //defineClass方法可以把二进制流字节组成的文件转换为一个java.lang.Class
            Class<?> c = this.defineClass(name, bytes, 0, bytes.length);
            return c;
        } 
        catch (Exception e)
        {
            e.printStackTrace();
        }
        
        return super.findClass(name);
    }
    
    private byte[] getClassBytes(File file) throws Exception
    {
        // 这里要读入.class的字节，因此要使用字节流
        FileInputStream fis = new FileInputStream(file);
        FileChannel fc = fis.getChannel();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        WritableByteChannel wbc = Channels.newChannel(baos);
        ByteBuffer by = ByteBuffer.allocate(1024);
        
        while (true){
            int i = fc.read(by);
            if (i == 0 || i == -1)
            break;
            by.flip();
            wbc.write(by);
            by.clear();
        }
        fis.close();
        return baos.toByteArray();
    }
}

```

