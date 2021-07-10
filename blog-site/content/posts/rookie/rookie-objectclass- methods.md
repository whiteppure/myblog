---
title: "Object类方法"
date: 2021-07-10
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-objectclass- methods"
---

## 概览
Object 类位于 `java.lang` 包中，编译时会自动导入，我们创建一个类时，如果没有明确继承一个父类，那么它就会自动继承`Object`，成为`Object`的子类。

`Object`类可以显示继承，也可以隐式继承,效果都是一样的.
```
class A extends Object{
    // to do
}

class A {
    // to do
}
```
Java `Object`类是所有类的父类，也就是说 Java 的所有类都继承了`Object`，子类可以使用`Object`的所有方法。

|方法名称 |方法作用
---|---
[equals](#equals)|比较两个对象是否相同
[hashCode](#hashCode)|获取对象的`hash`值
[toString](#toString)|返回对象的字符串表示形式
[clone](#clone)|创建并返回一个对象的拷贝
[finalize](#finalize)|当垃圾收集确定不再有对对象的引用时，由垃圾收集器在对象上调用
[getClass](#getClass)|获取对象所在的类
[notify](#notify)|唤醒在该对象上等待的某个线程
[notifyAll](#notifyAll)|唤醒在该对象上等待的所有线程
[wait](#wait)|让当前线程进入等待(阻塞)状态。直到其他线程调用此对象的`notify()`方法或`notifyAll()`方法。

## equals
`Object`类中的`equals()`方法作用是比较两个对象，是判断两个对象引用指向的是同一个对象，即比较两个对象的内存地址是否相等。

`Object`类中的`equals()`源码如下
```
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

### 等价关系
在Java规范中，`equals()`方法的使用存在如下特性：
- 自反性：`x.equals(x); // true`
- 对称性：`x.equals(y) == y.equals(x); // true`
- 传递性： `if (x.equals(y) && y.equals(z))  =>  x.equals(z); // true;`
- 一致性：`x.equals(y) == x.equals(y); // true` 多次调用`equals()`方法结果不变
- 与`null`的比较：`x.equals(null); // false;` 对任何不是`null`的对象x调用 `x.equals(null)` 结果都为`false`

### 与双等号
- 对于基本类型，`==` 判断两个值是否相等，基本类型没有 `equals() `方法。
- 对于引用类型，`==` 判断两个变量是否引用同一个对象，而 `equals()`判断引用的对象是否等价。

```
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

`equals()`作用是判断两个对象是否相等,但一般有两种情况:
1. 类没有覆盖`equals`方法,则相当于通过 `==`来比较这两个对象的地址;
2. 类覆盖`equals`方法,一般我们通过`equals()`来比较两个对象的内容是否相等,相等则返回true；

`equals()`在不重写的情况下与 `==` 作用一样都是比较的内存中的地址.但是`equals()`可以重写。

### 重写equals方法
重写`equals`方法一般思路：
- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将`Object`对象进行转型；
- 判断每个关键域是否相等;

```
public class EqualExample {

    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

## hashCode
在Java中`hashcode`方法是`Object`类的`native`方法，返回值为int类型，根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为hash值(散列值)。

> `hashCode`通用约定:
> - 若`x.equals(y)`返回true ，则`x.hashCode()==y.hashCode()`，其逆命题不一定成立。
> - 尽量使 hashCode 方法返回的散列码总体上呈均匀分布，可以提高哈希表的性能。
> - 程序运行时，若对象的`equals`方法中使用的字段没有改变，则在程序结束前，多次调用`hashCode`方法都应返回相同的散列码；程序结束后再执行时则没有此要求。

`hashCode`方法源码：
```
public native int hashCode();
```
根据这个方法的声明可知，该方法返回一个`int`类型的数值，并且是本地方法，因此在`Object`类中并没有给出具体的实现。

### 使用场景
对于包含容器类型的程序设计语言来说，基本上都会涉及到`hashCode`。在Java中也一样，`hashCode`方法的主要作用是为了配合基于散列的集合一起正常运行，这样的散列集合包括`HashSet、HashMap`以及`HashTable`。

在集合中已经存在上万条数据或者更多的数据场景下向集合中插入对象时，如何判别在集合中是否已经存在该对象了？

如果采用`equals`方法去逐一比较，效率必然是一个问题。此时`hashCode`方法的优点就体现出来了。因为两个不同的对象可能会有相同的`hashCode`值，所有不能通过`hashCode`值来判断两个对象是否相等，但是可以直接根据`hashcode`值判断两个对象不等，如果两个对象的`hashCode`值不等，则必定是两个不同的对象。
当集合要添加新的对象时，先调用这个对象的`hashCode`方法，得到对应的`hashcode`值，如果存放的`hash`值中没有该`hashcode`值，它就可以直接存进去，不用再进行任何比较了；如果存在该`hashcode`值，就调用它的`equals`方法与新元素进行比较，相同的话就不存了，不相同就去存。

> 需要额外注意的是:
> `设计hashCode()`时最重要的因素就是，无论何时，对同一个对象调用`hashCode()`都应该产生同样的值。
>
>如果在将一个对象用`put()`添加进`HashMap`时产生一个`hashCdoe`值，而用`get()`取出时却产生了另一个`hashCode`值，那么就无法获取该对象了。
>所以如果你的`hashCode`方法依赖于对象中易变的数据，就要当心了，因为此数据发生变化时，`hashCode()`方法就会生成一个不同的散列码，从而获取不到该对象。
>
> **所以在重写`hashCode`方法和`equals`方法的时候，如果对象中的数据易变，则最好在`equals`方法和`hashCode`方法中不要依赖于该字段。**


如下代码
```
public class MainTest {
    public static void main(String[] args) {
        Person p1 = new Person("lucy", 22);
        // 85134311
        System.out.println(p1.hashCode());
        HashMap<Person, Integer> hashMap = new HashMap<>();
        hashMap.put(p1, 1);
        p1.setAge(13);
        // null
        System.out.println(hashMap.get(p1));
    }
}

class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int hashCode() {
        return name.hashCode() * 37 + age;
    }

    @Override
    public boolean equals(Object obj) {
        return this.name.equals(((Person) obj).name) && this.age == ((Person) obj).age;
    }
}

```

### hashCode与equals
`hashCode()`返回散列值，而`equals()`是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

`equals()`地址比较是通过对象的哈希值来比较的。`hash`值是由`hashCode`方法产生的，`hashCode`属于`Object`类的本地方法,默认使用`==`比较两个对象,如果`equals()`相等`,hashcode`一定相等,如果`hashcode`相等,`equals`不一定相等。

所以在覆盖 `equals()` 方法时应当总是覆盖` hashCode() `方法，保证等价的两个对象散列值也相等。

下面的代码中，新建了两个等价的对象，并将它们添加到`HashSet`中。我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是**因为`EqualExample`没有实现`hashCode()`方法，因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。**

```
public class MainTest {
    public static void main(String[] args) {
        EqualExample e1 = new EqualExample(1, 1, 1);
        EqualExample e2 = new EqualExample(1, 1, 1);
        // true
        System.out.println(e1.equals(e2));
        HashSet<EqualExample> set = new HashSet<>();
        set.add(e1);
        set.add(e2);
        // 2
        System.out.println(set.size());
    }
}

```
所以**在覆盖 `equals()`方法时应当总是覆盖`hashCode()`方法，保证等价的两个对象散列值也相等。**

### 重写hashCode方法
重写`hashCode`方法规则：
- 把某个非零的常数值，保存在一个名为`result`的int类型的常量中
- 字段值哈希码的计算
    - 如果是boolean类型，`true`为1，`false`则为0
    - 如果是`byte、char、short`和`int`类型，需要强制转为int的值
    - 如果是long类型，计算`(int)(f^(f>>32))`
    - 如果是float类型，计算`Float.floatToIntBits(f)`
    - 如果是double类型，计算`Double.doubleToLongBits(f)`，再按照long的方法进行计算
    - 如果是引用类型，则调用其`hashCode`方法（假设其`hashCode`满足你的需求）
- 代入公式`result = result * 31 + c`，返回`result`

《Effective Java》的作者推荐使用基于17和31的散列码的算法:
```
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```
Java 7新增的`Objects`类提供了计算`hashCode`的通用方法，可以很简洁实现`hashCode`方法：
```
@Override
public int hashCode() {
    return Objects.hash(name,age);
}
```

## toString
`toString`方法是`Object`类里定义的，返回只类型是`String`默认返回类名和它的引用地址:`ToStringExample@4554617c`这种形式，其中@后面的数值为散列码的无符号十六进制表示。

`Object`类`toString`源代码如下：
```
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

### 重写toString方法
当我们打印一个对象的引用时，实际是默认调用这个对象的`toString()`方法,当打印的对象所在类没有重写`Object`中的`toString()`方法时，默认调用的是`Object`类中`toString()`方法.返回此对象所在的类及对应的堆空间对象实体的首地址值。
```
public class MainTest {
    public static void main(String[] args) {
        // object.ToStringDemo@511d50c0
        System.out.println(new ToStringDemo());
    }
}
class ToStringDemo {
    private String name;
}
```
当我们打印对象所在类重写了`toString()`，调用的就是已经重写了的`toString()`方法，一般重写是将类对象的属性信息返回。
```
public class MainTest {
    public static void main(String[] args) {
        ToStringDemo toStringDemo = new ToStringDemo();
        toStringDemo.setName("lucy");
        // ToStringDemo{name='lucy'}
        System.out.println(toStringDemo);
    }
}
class ToStringDemo {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "ToStringDemo{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

### 使用
在进行String类与其他类型的连接操作时，自动调用`toString()`方法：
```
public class MainTest {
    public static void main(String[] args) {
        Date time = new Date();
        System.out.println("time = " + time);//相当于下一行代码
        System.out.println("time = " + time.toString());
    }
}
```
在实际应用中，可以根据需要在用户自定义类型中重写`toString()`方法:
```
public class MainTest {
    public static void main(String[] args) {
        // null 没有toString()方法 *会报错*
        Object var0 = null;
        System.out.println(var0.toString());

        // 布尔型数据true和false返回对应的'true'和'false'
        Boolean var1 = false;
        Boolean var2 = true;
        System.out.println(var1.toString());
        System.out.println(var2.toString());

        // 字符串类型原值返回
        String var3 = "string";
        System.out.println(var3.toString());

        // 正浮点数及NaN、Infinity加引号返回
        Double var4 = 1.23d;
        System.out.println(var4.toString());
        Double nan = Double.NaN;
        System.out.println(nan.toString());
        Double negativeInfinity = Double.NEGATIVE_INFINITY;
        Double positiveInfinity = Double.POSITIVE_INFINITY;
        System.out.println(negativeInfinity.toString());
        System.out.println(positiveInfinity.toString());

        // 负浮点数或加'+'号的正浮点数直接跟上.toString()，相当于先运行toString()方法，再添加正负号，转换为数字
        Double var5 = -1.23d;
        Double var6 = +1.23d;
        System.out.println(var5.toString());
        System.out.println(var6.toString());
    }
}
```
基本数据类型转换为`String`类型就是调用了对应包装类的`toString()`方法：
```
int i = 10;
System.out.println("i=" + i);
```

## clone

### Cloneable接口
`clone()`是 `Object` 的 `protected` 方法，它不是被 `public`修饰；一个类不显式的去重写`clone()`，其它类就不能直接去调用该类实例的 `clone()`方法：

```
public class CloneExample {
    private int a;
    private int b;
}
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```

重写`clone()`方法得到以下实现：

```
public class CloneExample {
    private int a;
    private int b;

    @Override
    public CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
```

```
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

以上抛出了 ``java.lang.CloneNotSupportedException: CloneExample``，这是因为 ``CloneExample ``没有实现 ``Cloneable`` 接口。

应该注意的是，`clone()` 方法并不是 ``Cloneable`` 接口的方法，而是 `Object` 的一个 `protected` 方法。``Cloneable`` 接口只是规定，如果一个类没有实现 ``Cloneable ``接口又调用了 `clone()` 方法，就会抛出 ``CloneNotSupportedException``。

```
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

### 浅拷贝
拷贝的对象和原始对象的引用类型,为同一个对象。

```
public class ShallowCloneExample implements Cloneable {

    private int[] arr;

    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}

```

```
ShallowCloneExample e1 = new ShallowCloneExample();
ShallowCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 222
```

### 深拷贝
拷贝的对象和原始对象的引用类型，为不同对象。

```
public class DeepCloneExample implements Cloneable {

    private int[] arr;

    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}

```

```
DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

### clone的替代

使用 ``clone()`` 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。《Effective Java》 书上讲到，最好不要去使用 ``clone()``，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}

```

```
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

## finalize
`finalize()`方法是Java提供的对象终止机制，允许开发人员提供对象被销毁之前的自定义处理逻辑。
当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的`finalize()`方法。

`finalize()` 方法允许在子类中被重写，用于在对象被回收时进行资源释放。
通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。

文档注释大意：当GC确定不再有对对象的引用时，由垃圾收集器在对象上调用。子类重写`finalize`方法来释放系统资源或执行其他清理。
```
   /**
     * Called by the garbage collector on an object when garbage collection
     * determines that there are no more references to the object.
     * A subclass overrides the {@code finalize} method to dispose of
     * system resources or to perform other cleanup.
     */
    protected void finalize() throws Throwable { }
```

简而言之，`finalize`方法是与Java中的垃圾回收器有关系。即：当一个对象变成一个垃圾对象的时候，如果此对象的内存被回收，那么就会调用该类中定义的`finalize`方法。

当一个对象可被回收时，就需要执行该对象的 `finalize()` 方法，那么就有可能在该方法中让对象重新被引用，从而实现自救。自救只能进行一次，如果回收的对象之前调用了 `finalize()` 方法自救，后面回收时不会再调用该方法。

**永远不要主动调用某个对象的`finalize`方法应该交给垃圾回收机制调用的原因：**

- 在调用`finalize`方法时时可能会导致对象复活；
- `finalize`方法的执行时间是没有保障的，它完全由GC线程决定，极端情况下，若不发生GC，则`finalize`方法将没有执行机会;因为优先级比较低，即使主动调用该方法，也不会因此就直接进行回收；
- 一个糟糕的`finalize`方法会严重影响GC的性能;

**由于`finalize`方法的存在,虚拟机中的对象一般可能处于三种状态：**

如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。
但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。一个无法触及的对象有可能在某一个条件下“复活”自己，如果这样，那么对它的回收就是不合理的，为此，虚拟机中定义了的对象可能的三种状态：
- 可触及的：从根节点开始，可以到达这个对象；对象存活被使用；
- 可复活的：对象的所有引用都被释放，但是对象有可能在`finalize`中复活；对象被复活，对象在`finalize`方法中被重新使用；
- 不可触及的：对象的`finalize`方法被调用，并且没有复活，那么就会进入不可触及状态；对象死亡，对象没有被使用；

只有在对象不可触及时才可以被回收。不可触及的对象不可能被复活，因为`finalize()`只会被调用一次。

**`finalize`对象终止机制判定一个对象能否被回收过程：**

判定一个对象是否可回收，至少要经历两次标记过程：
- 如果对象没有没有引用链，则进行第一次标记
- 进行筛选，判断此对象是否有必要执行`finalize`方法
    1. 如果对象没有重写`finalize`方法，或者`finalize`方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，对象被判定为不可触及的。
    2. 如果对象重写了`finalize`方法，且还未执行过，那么会被插入到`F-Queue`队列中，由一个虚拟机自动创建的、低优先级的`Finalizer`线程触发其`finalize`方法执行。
    3. `finalize`方法是对象逃脱死亡的最后机会，稍后GC会对`F-Queue`队列中的对象进行第二次标记。如果对象在`finalize`方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，该对象会被移出“即将回收”集合。
    之后，对象会再次出现没有引用存在的情况。在这个情况下，`finalize`方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的`finalize`方法只会被调用一次。
    
代码演示对象能否被回收：
```
public class MainTest {

    public static MainTest var;

    /**
     * 此方法只能被调用一次
     * 可对该方法进行注释，来测试finalize方法是否能复活对象
     */
    @Override
    protected void finalize() throws Throwable {
        System.out.println("调用当前类重写的finalize()方法");
        // 复活对象 让当前带回收对象重新与引用链中的对象建立联系
        var = this;
    }

    public static void main(String[] args) throws InterruptedException {
        var = new MainTest();
        var = null;
        System.gc();
        System.out.println("-----------------第一次gc操作------------");
        // 因为Finalizer线程的优先级比较低，暂停2秒，以等待它
        Thread.sleep(2000);
        if (var == null) {
            System.out.println("对象已经死了");
            // 如果第一次对象就死亡了 就终止
            return;
        } else {
            System.out.println("对象还活着");
        }

        System.out.println("-----------------第二次gc操作------------");
        var = null;
        System.gc();
        // 下面代码和上面代码是一样的，但是 对象却自救失败了
        Thread.sleep(2000);
        if (var == null) {
            System.out.println("对象已经死了");
        } else {
            System.out.println("对象还活着");
        }
    }

}
```
## getClass

### getClass与instanceof
