---
title: "面向对象总结"
date: 2021-02-15
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-object-oriented"
---



> 面向对象是一种编程思想，包括三大特性和六大原则，其中，三大特性指的是封装、继承和多态；六大原则指的是[单一职责原则](#单一职责原则)、[开放封闭原则](#开放封闭原则)、[迪米特原则](#迪米特法则)、[里氏替换原则](#里氏替换原则)、[依赖倒置原则](#依赖倒置原则)以及[接口隔离原则](#接口隔离原则)，其中，单一职责原则是指一个类应该是一组相关性很高的函数和数据的封装，这是为了提高程序的内聚性，而其他五个原则是通过抽象来实现的，目的是为了降低程序的耦合性以及提高可扩展性。

面向对象简称OO(object-oriented)是相对面向过程(procedure-oriented)来说的,是一种编程思想.Java就是一门面向对象的语言.

面向对象编程简称OOP(Object-oriented programming),是将事务高度抽象化的编程模式.
面向对象编程是以功能来划分问题的,将问题分解成一个一个步骤,对每个步骤进行相应的抽象,形成对应对象,通过不同对象之间的调用,组合成某个功能解决问题.

## 对比面向过程

> PS: 面向过程编程简称POP(Procedural oriented programming),面向过程是以过程为中心的编程思想.是自顶而下的编程.

举个栗子: 下五子棋
```

面向过程 {

  1.开始游戏

  2.黑子先走

  3.绘制画面

  4.判断输赢

  5.轮到白子

  6.绘制画面

  7.判断输赢

  8.返回到 黑子先走
 
}
```

```
面向对象 {

    1.创建黑棋,白棋
    
    2.创建棋盘
    
    3.创建规则
    
    4.赋予每个对象相关属性和指定行为
    
    5.各个功能之间互相调用
}
```

**面向对象是模型化的,你只需抽象出几个类,进行封装成各个功能,通过不同对象之间的调用来解决问题.而面向过程需要把问题分解为几个步骤,每个步骤用对应的函数调用即可.面向过程是具体化的,流程化的,解决一个问题,需要你一步一步的分析,一步一步的实现.**

面向对象的底层其实还是面向过程,把面向过程抽象成类,然后进行封装,方便我们我们使用,就是面向对象了.

简而言之,用面向过程的方法写出来的程序是一份蛋炒饭,而用面向对象写出来的程序是一份盖浇饭(就是在一碗白米饭上面浇上一份盖菜，你喜欢什么菜,你就浇上什么菜).
通过例子可以看出面向对象更重视不重复造轮子,即创建一次,重复使用.

面向对象
> - 优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 更加灵活、更加易于维护。
>
> - 缺点：性能比面向过程低

面向过程
>- 优点：流程化使得编程任务明确，在开发之前基本考虑了实现方式和最终结果，具体步骤清楚，便于节点分析;    效率高，面向过程强调代码的短小精悍，善于结合数据结构来开发高效率的程序。
>- 缺点：没有面向对象易维护、易复用、易扩展

抽象会使复杂的问题更加简单化,面向对象更符合人类的思维,而面向过程则是机器的思想.

## 软件设计原则
**设计原则的目的是为了让程序达到高内聚、低耦合，提高可扩展性的目的，其实现手段是面向对象的三大特性：封装、继承以及多态。**

设计原则名称 | 核心思想
---|---
[单一职责原则](#单一职责原则) | 一个类只负责一个功能领域中的相应职责
[开放封闭原则](#开放封闭原则) | 软件实体应对扩展开放，而对修改关闭
[依赖倒转原则](#依赖倒转原则) | 抽象不应该依赖于细节，细节应该依赖于抽象
[里氏替换原则](#里氏替换原则) | 所有引用基类对象的地方能够透明地使用其子类的对象
[接口隔离原则](#接口隔离原则) | 使用多个专门的接口，而不使用单一的总接口
[合成复用原则](#合成复用原则) | 尽量使用对象组合，而不是继承来达到复用的目的
[迪米特法则](#迪米特法则) | 一个软件实体应当尽可能少地与其他实体发生相互作用


### 单一职责原则
> **其核心思想为：一个类，最好只做一件事，只有一个引起它的变化。单一职责原则可降低类的复杂度,提高代码可读性,可维护性,降低变更风险.** 单一职责原则可以看做是低耦合、高内聚在面向对象原则上的引申，将职责定义为引起变化的原因，以提高内聚性来减少引起变化的原因。职责过多，可能引起它变化的原因就越多，这将导致职责依赖，相互之间就产生影响，从而大大损伤其内聚性和耦合度。通常意义下的单一职责，就是指只有一种单一功能，不要为类实现过多的功能点，以保证实体只有一个引起它变化的原因。 专注，是一个人优良的品质；同样的，单一也是一个类的优良设计。交杂不清的职责将使得代码看起来特别别扭牵一发而动全身，有失美感和必然导致丑陋的系统错误风险。

代码实现

```
public class MainTest {
    public static void main(String[] args) {
        Vehicle vehicle = new Vehicle();
        vehicle.running("汽车");
        // 飞机不是在路上行驶
        vehicle.running("飞机");
    }
}

/**
 * 在run方法中违反了单一职责原则
 * 解决方法根据不同的交通工具,分解成不同的类即可
 */
class Vehicle{
    public void running(String name) {
        System.out.println(name + "在路上行驶 ....");
    }
}

```
```
// 解决
public class MainTest {
    public static void main(String[] args) {
        Driving driving = new Driving();
        driving.running("汽车");
        Flight flight = new Flight();
        flight.running("飞机");
    }
}

class Driving {
    public void running(String name) {
        System.out.println(name + "在路上行驶 ....");
    }
}

class Flight {
    public void running(String name) {
        System.out.println(name + "在空中飞行 ....");
    }
}
```
**通常情况下,我们应当遵循单一职责原则,只要逻辑足够简单,才可以在代码里边违反单一职责原则;只要类中方法数量足够少,可以在方法级别保持单一职责原则.**

```
public class MainTest {
    public static void main(String[] args) {
        Vehicle2 vehicle2 = new Vehicle2();
        vehicle2.driving("汽车");
        vehicle2.flight("飞机");
    }
}
/*
 * 改进
 *↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
 */

class Vehicle2 {
    public void driving(String name) {
        System.out.println(name + "在路上行驶 ....");
    }
    public void flight(String name) {
        System.out.println(name + "在空中飞行 ....");
    }
}
```

### 开放封闭原则
> **软件实体应该是可扩展的，而不可修改的。也就是，对(提供方)扩展开放，对(使用方)修改封闭的。** 开放封闭原则主要体现在两个方面
>- 对扩展开放，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。
>- 对修改封闭，意味着类一旦设计完成，就可以独立完成其工作，而不要对其进行任何尝试的修改。 
>
>实现开放封闭原则的核心思想就是对抽象编程，而不对具体编程，因为抽象相对稳定。让类依赖于固定的抽象，所以修改就是封闭的；而通过面向对象的继承和多态机制，又可以实现对抽象类的继承，通过覆写其方法来改变固有行为，实现新的拓展方法，所以就是开放的。 “需求总是变化”没有不变的软件，所以就需要用封闭开放原则来封闭变化满足需求，同时还能保持软件内部的封装体系稳定，不被需求的变化影响。**编程中遵循其他原则,以及使用其他设计模式的目的就是为了遵循开闭原则.**

当软件需要变化时,尽量使用扩展的软件实体的方式行为来实现变化,而不是通过修改已有的代码来实现变化.

代码实现

```

public class MainTest {
    public static void main(String[] args) {
        Mother mother = new Mother();

        Son son = new Son();
        Daughter daughter = new Daughter();

        // 注入子类对象 如果扩展需要其他类 换成其他对象即可
        mother.setAbstractFather(son);
        mother.display();
    }
}

abstract class AbstractFather {

    protected abstract void display();

}
class Son  extends AbstractFather{
    @Override
    protected void display() {
        System.out.println("son class ...");
    }
}
class Daughter  extends AbstractFather{

    @Override
    protected void display() {
        System.out.println("daughter class ...");
    }
}

class Mother {

    private AbstractFather abstractFather;

    public void setAbstractFather(AbstractFather abstractFather) {
        this.abstractFather = abstractFather;
    }

    public void display() {
        abstractFather.display();
    }

}
```

### 依赖倒置原则
> **该原则依赖于抽象。具体而言就是高层模块不依赖于底层模块，二者都同依赖于抽象；抽象不依赖于具体，具体依赖于抽象。** 我们知道，依赖一定会存在于类与类、模块与模块之间。当两个模块之间存在紧密的耦合关系时，最好的方法就是分离接口和实现：在依赖之间定义一个抽象的接口使得高层模块调用接口，而底层模块实现接口的定义，以此来有效控制耦合关系，达到依赖于抽象的设计目标。 抽象的稳定性决定了系统的稳定性，因为抽象是不变的，依赖于抽象是面向对象设计的精髓，也是依赖倒置原则的核心。 依赖于抽象是一个通用的原则，而某些时候依赖于细节则是在所难免的，必须权衡在抽象和具体之间的取舍，方法不是一层不变的。**依赖于抽象，就是对接口编程，不要对实现编程。**

代码实现
```

public class MainTest {

    public static void main(String[] args) {
        Computer computer = new Computer();
        // 对接口编程，不要对实现编程
        // 如果没有接口 则代码很难实现扩展
        Disk disk = new CustomDisk();
        Memory memory = new CustomMemory();

        computer.setDisk(disk);
        computer.setMemory(memory);
        computer.run();
    }

}

interface Disk{
    void diskMethod();
}

interface Memory{
    void memoryMethod();
}

class CustomDisk implements Disk{

    @Override
    public void diskMethod() {
        System.out.println("i am disk ...");
    }
}

class  CustomMemory implements Memory{

    @Override
    public void memoryMethod() {
        System.out.println("i am memory ...");
    }
}

class Computer {

    private Memory memory;

    private Disk disk;

    public void setDisk(Disk disk) {
        this.disk = disk;
    }

    public void setMemory(Memory memory) {
        this.memory = memory;
    }

    public Disk getDisk() {
        return disk;
    }

    public Memory getMemory() {
        return memory;
    }
    public void run() {
        System.out.println(" computer is running ...");
        memory.memoryMethod();
        disk.diskMethod();
    }
}
```


### 接口隔离原则
> **使用多个小的专门的接口，而不要使用一个大的总接口。** 具体而言，接口隔离原则体现在：接口应该是内聚的，应该避免“胖”接口。一个类对另外一个类的依赖应该建立在最小的接口上，不要强迫依赖不用的方法，这是一种接口污染。 接口有效地将细节和抽象隔离，体现了对抽象编程的一切好处，接口隔离强调接口的单一性。而胖接口存在明显的弊端，会导致实现的类型必须完全实现接口的所有方法、属性等；而某些时候，实现类型并非需要所有的接口定义，在设计上这是“浪费”，而且在实施上这会带来潜在的问题，对胖接口的修改将导致一连串的客户端程序需要修改，有时候这是一种灾难。在这种情况下，将胖接口分解为多个特点的定制化方法，使得客户端仅仅依赖于它们的实际调用的方法，从而解除了客户端不会依赖于它们不用的方法。 
>
>分离的手段主要有以下两种：
>- 委托分离，通过增加一个新的类型来委托客户的请求，隔离客户和接口的直接依赖，但是会增加系统的开销。
>- 多重继承分离，通过接口多继承来实现客户的需求，这种方式是较好的。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        FuncImpl func = new FuncImpl();
        func.func1();
        func.func2();
        func.func3();
    }
}

interface Function1{
    void func1();
    // 如果将接口中的方法都写在一个接口就会造成实现该接口就要重写该接口所有方法。
    // 当然Java 8 接口可以有实现，降低了维护成本，解了决该问题；
    // 但是我们还是应当遵循该原则，使得接口看起来更加清晰
    // void func2();
    // void func3();
}
interface Function2 {
    void func2();
}
interface Function3 {
    void func3();
}

class FuncImpl implements Function1,Function2,Function3{

    @Override
    public void func1() {
        System.out.println("i am function1 impl");
    }

    @Override
    public void func2() {
        System.out.println("i am function2 impl");
    }

    @Override
    public void func3() {
        System.out.println("i am function3 impl");
    }
}
```


### 里氏替换原则
> **子类必须能够替换其基类。** 这一思想体现为对继承机制的约束规范，只有子类能够替换基类时，才能保证系统在运行期内识别子类，这是保证继承复用的基础。在父类和子类的具体行为中，必须严格把握继承层次中的关系和特征，将基类替换为子类，程序的行为不会发生任何变化。同时，这一约束反过来则是不成立的，子类可以替换基类，但是基类不一定能替换子类。 里氏替换原则，主要着眼于对抽象和多态建立在继承的基础上，因此只有遵循了里氏替换原则，才能保证继承复用是可靠地。
>
>实现的方法是面向接口编程：将公共部分抽象为基类接口或抽象类，通过`Extract Abstract Class`，在子类中通过覆写父类的方法实现新的方式支持同样的职责。 里氏替换原则是关于继承机制的设计原则，违反了里氏替换原则就必然导致违反开放封闭原则。 里氏替换原则能够保证系统具有良好的拓展性，同时实现基于多态的抽象机制，能够减少代码冗余，避免运行期的类型判别。

简单来说就是子类可以扩展父类的功能,但是尽量不要重写父类的功能.如果通过重写父类方法来完成新的功能,这样写起来虽然简单,但整个体系的可复用性会非常差,特别是运用多态比较频繁时,程序运行出错的概率会非常大.

代码实现
```

public class MainTest {
    public static void main(String[] args) {
        Rectangle rectangle = new Rectangle();
        rectangle.setWidth(20);
        rectangle.setHeight(10);
        resize(rectangle);
        print(rectangle);
        System.out.println("=======================");
        // 因为 Square类 重写了父类set的方法导致调用时出错
        Rectangle square = new Square();
        square.setWidth(10);
        resize(square);
        print(square);
    }
    public static void resize(Rectangle rectangle){
        while (rectangle.getWidth() >= rectangle.getHeight()){
            rectangle.setHeight(rectangle.getHeight() + 1);
        }
    }

    public static void print(Rectangle rectangle){
        System.out.println(rectangle.getWidth());
        System.out.println(rectangle.getHeight());
    }
}

class Square  extends Rectangle{
    private Integer width;
    private Integer height;

    @Override
    public void setWidth(Integer width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(Integer height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}
class Rectangle {
    private Integer width;
    private Integer height;

    public void setWidth(Integer width) {
        this.width = width;
    }

    public void setHeight(Integer height) {
        this.height = height;
    }

    public Integer getWidth() {
        return width;
    }

    public Integer getHeight() {
        return height;
    }
}
```

### 合成复用原则
> **尽量使用对象组合，而不是继承来达到复用的目的。** 在面向对象设计中，可以通过两种方法在不同的环境中复用已有的设计和实现，**即通过组合/聚合关系或通过继承，但首先应该考虑使用组合/聚合**，组合/聚合可以使系统更加灵活，降低类与类之间的耦合度，一个类的变化对其他类造成的影响相对较少；其次才考虑继承，在使用继承时，需要严格遵循里氏代换原则，有效使用继承会有助于对问题的理解，降低复杂度，而滥用继承反而会增加系统构建和维护的难度以及系统的复杂度，因此需要慎重使用继承复用。

代码实现

详见[继承与组合](#继承与组合)

### 迪米特法则
> 迪米特法则又叫最少知识原则，就是说一个对象应当对其他对象有尽可能少的了解。
其核心思想是: **降低类之间的耦合.如果类与类之间的关系越密切，耦合度越大，当一个类发生改变时，对另一个类的影响也越大.一个对象应该对其他对象有最少的了解。** 通俗地讲，一个类应该对自己需要耦合或调用的类知道得最少，你（被耦合或调用的类）的内部是如何复杂都和我没关系，那是你的事情，我就知道你提供的`public`方法，我就调用这么多，其他的一概不关心.迪米特法则其根本思想，是强调了类之间的松耦合。类之间的耦合越弱，越有利于复用，一个处在弱耦合的类被修改，不会对有关系的类造成搏击，也就是说，信息的隐藏促进了软件的复用。

迪米特法则还有个更简单的定义：只与直接的朋友通信

朋友定义：**每个对象都会与其他对象有耦合关系，只要两个对象之间有耦合关系，我们就说这两个对象之间是朋友关系。** 耦合的方式很多，依赖，关联，组合，聚合等。其中，我们称出现**成员变量，方法参数，方法返回值中的类为直接的朋友**，而出现在**局部变量中的类不是直接的朋友**。也就是说，**陌生的类最好不要以局部变量的形式出现在类的内部。**

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        //创建了一个 SchoolManager 对象
        SchoolManager schoolManager = new SchoolManager();
        // SchoolManager直接朋友: CollegeManager (方法参数) Employee(返回值)
        // CollegeEmployee以局部变量的形式出现在SchoolManager类中 所以违反了迪米特法则
        schoolManager.printAllEmployee(new CollegeManager());
    }
}


class Employee {
    private String id;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}


class CollegeEmployee {
    private String id;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

}

class CollegeManager {
    public List<CollegeEmployee> getAllEmployee() {
        List<CollegeEmployee> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            CollegeEmployee emp = new CollegeEmployee();
            emp.setId("学院员工 id= " + i);
            list.add(emp);
        }
        return list;
    }
}

class SchoolManager {

    public List<Employee> getAllEmployee() {
        List<Employee> list = new ArrayList<>();

        for (int i = 0; i < 5; i++) {
            //这里我们增加了 5 个员工到
            Employee emp = new Employee();
            emp.setId("学校总部员工 id= " + i);
            list.add(emp);
        }
        return list;
    }

    void printAllEmployee(CollegeManager sub) {
        //获取到学院员工
        List<CollegeEmployee> list1 = sub.getAllEmployee();
        System.out.println("------------学院员工------------");
        for (CollegeEmployee e : list1) {
            System.out.println(e.getId());
        }
        //获取到学校总部员工
        List<Employee> list2 = this.getAllEmployee();
        System.out.println("------------学校总部员工------------");
        for (Employee e : list2) {
            System.out.println(e.getId());
        }
    }
}
```
```

class CollegeManager {
    public List<CollegeEmployee> getAllEmployee() {
        List<CollegeEmployee> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            CollegeEmployee emp = new CollegeEmployee();
            emp.setId("学院员工 id= " + i);
            list.add(emp);
        }
        return list;
    }
    // 修改后
    public void printAllEmployee() {
        List<CollegeEmployee> list1 = this.getAllEmployee();
        System.out.println("------------学院员工------------");
        for (CollegeEmployee e : list1) {
            System.out.println(e.getId());
        }
    }
}

class SchoolManager {

    public List<Employee> getAllEmployee() {
        List<Employee> list = new ArrayList<>();

        for (int i = 0; i < 5; i++) {
            //这里我们增加了 5 个员工到
            Employee emp = new Employee();
            emp.setId("学校总部员工 id= " + i);
            list.add(emp);
        }
        return list;
    }

    void printAllEmployee(CollegeManager sub) {
        //获取到学院员工
        // 修改后
        sub.printAllEmployee();

        //获取到学校总部员工
        List<Employee> list2 = this.getAllEmployee();
        System.out.println("------------学校总部员工------------");
        for (Employee e : list2) {
            System.out.println(e.getId());
        }
    }
}
```


## 三大特性

### 封装

封装是面向对象方法的重要原则，就是把对象的属性和操作（或服务）结合为一个独立的整体，并尽可能隐藏对象的内部实现细节.简单的说,一个类就是一个封装了数据以及操作这些数据的代码的逻辑实体。在一个对象内部，**某些代码或某些数据可以是私有的**，不能被外界访问。通过这种方式，对象对内部数据提供了不同级别的保护，以防止程序中无关的部分意外的改变或错误的使用了对象的私有部分。


**封装的目的是增强安全性和简化编程,使用者不必了解具体的实现细节,而只是要通过外部接口,以特定的访问权限来使用类的成员.**

#### 优点

-  良好的封装能够减少耦合;提高了可维护性和灵活性以及可重用性;
-  类内部的结构可以自由修改;
- 可以对成员变量进行更精确的控制;
-  隐藏信息，实现细节;

#### 访问权限

Java的封装可以通过修改属性的可见性限制对属性的访问来体现.

**Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加访问修饰符，表示包级可见(default)。**


修饰符 | 当前类 | 同一包下 | 其他包的子类|  不同包的子类 | 其他包
---|---|---|---|---|---
public | Y | Y | Y | Y | Y 
protected | Y | Y | Y | Y/N | N
default | Y | Y | Y | N | N
private | Y | N | N | N | N
这四种访问权限的控制符能够控制类中成员的可见性.当然需要满足在不使用Java反射的情况下.

**注意**
- `protected`用于修饰成员,表示在继承体系中成员对于子类可见.如果不存在继承关系则不能访问`protected`修饰的实例.
- 类可见表示其它类可以用这个类创建实例对象;
- 成员可见表示其它类可以用这个类的实例对象访问到该成员;

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为**信息隐藏或封装**。因此**访问权限应当尽可能地使每个类或者成员不被外界访问**。

**如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别**。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足**里氏替换原则**。

**某个类的字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，其他类可以对其随意修改。**
例如下面的例子中，``AccessExample``拥有id公有字段，如果在某个时刻，我们想要使用`int`存储`id`字段，那么就需要修改所有类中的代码。

```
public class AccessExample {
    public String id;
    // public int id;
}
```

可以使用公有的 ``getter`` 和 ``setter`` 方法来替换公有字段，这样的话就可以控制对字段的修改行为。实现了封装

```
public class AccessExample {

    private int id;

    public String getId() {
        return id + "";
    }

    public void setId(String id) {
        this.id = Integer.valueOf(id);
    }
}
```

**但是也有例外，如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。**

```
public class AccessWithInnerClassExample {

    private class InnerClass {
        int x;
    }

    private InnerClass innerClass;

    public AccessWithInnerClassExample() {
        innerClass = new InnerClass();
    }

    public int getValue() {
        return innerClass.x;  // 直接访问
    }
}
```


### 继承
继承可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”。继承的过程，就是从一般到特殊的过程.**要实现继承，可以通过“继承”（Inheritance）和“组合”（Composition）来实现.** 继承概念的实现方式有两类：实现继承与接口继承。实现继承是指直接使用基类的属性和方法而无需额外编码的能力；接口继承是指仅使用属性和方法的名称、但是子类必须提供实现的能力.

> 继承: 如果多个类的某个部分的功能相同,那么可以抽象出一个类来,把相同的部分放到父类中,让他们继承这个类
>
> 实现：如果多个类处理的目标是一样的，但是处理的方法方式不同，那么就定义一个接口，也就是一个**标准**，让他们的实现这个接口，各自实现自己具体的处理方法来处理那个目标

**继承的根本原因是因为要复用，而实现的根本原因是需要定义一个标准.**

#### 继承与组合

继承是实现复用代码的重要手段,但是继承会破坏封装.组合也是代码复用的重要方式,可以提供良好的封装性.

##### 实现继承

```
// 继承

public class MainTest {

    public static void main(String[] args) {
        B b = new B();
        b.test();
    }
}

class A {

    protected int i;

    protected void test() {
        System.out.println("I am super class ... ");
    }

}

class B  extends A{

    // 调用父类成员 
    public void t() {
        System.out.println(i);
    }

}
```
通过以上代码,可以发现,子类可以访问父类的成员变量方法,并且通过重写可以改变父类方法实现.从而破坏了封装性.

> 在继承结构中，父类的内部细节对于子类是可见的。所以我们通常也可以说通过继承的代码复用是一种**白盒式代码复用**。（如果基类的实现发生改变，那么派生类的实现也将随之改变。这样就导致了子类行为的不可预知性；）

为了保证父类有良好的封装性,不会对子类随意更改,设计父类时应遵循以下原则:
> - 尽量隐藏父类的内部数据.尽量把所有父类的所有成员变量都用`private`修饰,不要让子类直接访问父类的成员.
> -  不要让子类随意的修改访问父类的方法.父类中那些仅为辅助其他的工具方法,应该使用private修饰,让子类无法访问该方法;如果父类中的方法需要被外部类调用,则需以public修饰,但又不希望重写父类方法可以使用final来修饰方法;但如果希望父类某个方法被重写,但又不希望其他类访问自由,可以使用protected修饰.
> - 尽量不要在父类构造器中调用将要被子类重写的方法.


继承是类与类或者接口与接口之间最常见的关系,继承是一种`is-a`关系。

##### 实现组合
组合是把旧类对象作为新类对象的成员变量组合进来,用以实现新类的功能.

```

public class MainTest {

    public static void main(String[] args) {
        B b = new B(new A());
        b.test();
    }
}

class A {

    protected int i;

    protected void test() {
        System.out.println("I am super class ... ");
    }

}

class B {

    private final A a;

    public B(A a) {
        this.a = a;
    }
    public void test() {
        // 复用 A 类提供的 test 方法
        a.test();
    }
}
```
> 组合是通过对现有的对象进行拼装（组合）产生新的、更复杂的功能。因为在对象之间，各自的内部细节是不可见的，所以我们也说这种方式的代码复用是**黑盒式代码复用**。（因为组合中一般都定义一个类型，所以在编译期根本不知道具体会调用哪个实现类的方法）

组合强调的是整体与部分、拥有的关系，即`has-a`的关系.

##### 比较
> 继承，在写代码的时候就要指名具体继承哪个类，所以，在编译期就确定了关系。（从基类继承来的实现是无法在运行期动态改变的，因此降低了应用的灵活性。）

> 组合，在写代码的时候可以采用面向接口编程。所以，类的组合关系一般在运行期确定。

组合 | 继承
---|---
优点：不破坏封装，整体类与局部类之间松耦合，彼此相对独立 | 缺点：破坏封装，子类与父类之间紧密耦合，子类依赖于父类的实现，子类缺乏独立性
优点：具有较好的可扩展性 | 缺点：支持扩展，但是往往以增加系统结构的复杂度为代价
优点：支持动态组合。在运行时，整体对象可以选择不同类型的局部对象 | 缺点：不支持动态继承。在运行时，子类无法选择不同的父类
优点：整体类可以对局部类进行包装，封装局部类的接口，提供新的接口 | 缺点：子类不能改变父类的接口
缺点：整体类不能自动获得和局部类同样的接口 | 优点：子类能自动继承父类的接口
缺点：创建整体类的对象时，需要创建所有局部类的对象 | 优点：创建子类的对象时，无须创建父类的对象

##### 使用选择
经过以上比较,可以得出结论: 组合比继承更加灵活.所以在写代码如果这个功能组合和继承都能够完成,那么应该优先选择组合.
但是继承在一些场景还是要优先于组合的.
> - 继承要慎用，其使用场合仅限于你确信使用该技术有效的情况。一个判断方法是，问一问自己是否需要从新类向基类进行向上转型。如果是必须的，则继承是必要的。反之则应该好好考虑是否需要继承。
> - 只有当子类真正是超类的子类型时，才适合用继承。换句话说，对于两个类A和B，只有当两者之间确实存在is-a关系的时候，类B才应该继承类A。

#### super
- 访问父类的构造函数：可以使用``super()``函数访问父类的构造函数，从而委托父类完成一些初始化的工作。**应该注意到，子类一定会调用父类的构造函数来完成初始化工作，一般是调用父类的默认构造函数**，**如果子类需要调用父类其它构造函数，那么就可以使用super函数。**
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 `super `关键字来引用父类的方法实现。

```
public class SuperExample {

    protected int x;
    protected int y;

    public SuperExample(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void func() {
        System.out.println("SuperExample.func()");
    }
}
public class SuperExtendExample extends SuperExample {

    private int z;

    public SuperExtendExample(int x, int y, int z) {
        super(x, y);
        this.z = z;
    }

    @Override
    public void func() {
        super.func();
        System.out.println("SuperExtendExample.func()");
    }
}
SuperExample e = new SuperExtendExample(1, 2, 3);
e.func();
SuperExample.func()
SuperExtendExample.func()
```

#### 抽象类与接口
抽象类和接口也是Java继承体系中的重要组成部分.抽象类是用来捕捉子类的通用特性的，而接口则是抽象方法的集合；抽象类不能被实例化，只能被用作子类的超类，是被用来创建继承层级里子类的模板，而接口只是一种形式，接口自身不能做任何事情。

##### 抽象类

**抽象类和抽象方法都使用`abstract`关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。**

抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类。

```
public abstract class AbstractClassExample {

    protected int x;
    private int y;

    public abstract void func1();

    public void func2() {
        System.out.println("func2");
    }
}
public class AbstractExtendClassExample extends AbstractClassExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}

// 实例化抽象类
// AbstractClassExample ac1 = new AbstractClassExample(); 
// 实例化抽象类子类
// AbstractClassExample ac2 = new AbstractExtendClassExample();
// ac2.func1();
```

##### 接口

**接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。**

**从 Java 8 开始**，**接口也可以拥有默认的方法实现**，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类,现在不用修改所有实现该接口的类.

接口的成员（字段 + 方法）默认都是`public`的，并且不允许定义为`private`或者 `protected`。

接口的字段默认都是用`static`和`final`修饰的.

```
public interface InterfaceExample {

    void func1();

    default void func2(){
        System.out.println("func2");
    }

    int x = 123;
    // int y;               // Variable 'y' might not have been initialized
    public int z = 0;       // Modifier 'public' is redundant for interface fields
    // private int k = 0;   // Modifier 'private' not allowed here
    // protected int l = 0; // Modifier 'protected' not allowed here
    // private void fun3(); // Modifier 'private' not allowed here
}
public class InterfaceImplementExample implements InterfaceExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}

// InterfaceExample ie1 = new InterfaceExample(); // 'InterfaceExample' is abstract; cannot be instantiated
InterfaceExample ie2 = new InterfaceImplementExample();
ie2.func1();//func1
System.out.println(InterfaceExample.x);//123
```

Java的接口可以多继承
```
interface Action extends Serializable,AutoCloseable {
	// to do ...
}
```


##### 比较
- 从设计层面上看，抽象类提供了一种`is-a`关系，那么就必须满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 `like-a` 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 `is-a` 关系。
- 从使用上来看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是`static`和`final`类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是`public`的，而抽象类的成员可以有多种访问权限。

##### 使用选择

**使用接口**
- 需要让不相关的类都实现一个方法，例如:不相关的类都可以实现 ``Compareable`` 接口中的 ``compareTo()`` 方法；
- 需要使用多重继承。

**使用抽象类**
- 需要在几个相关的类中共享代码。
- 需要能控制继承来的成员的访问权限，而不是都为`public`。
- 需要继承非静态和非常量字段。

**在很多情况下，接口优先于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从 Java 8 开始，接口也可以有默认的方法实现，使得修改接口的成本也变的很低。**

### 多态

多态即多种表现形态;同一个行为具有多个不同表现形式或形态的能力.

多态存在的前提
- 有类继承或者接口实现
- 子类要重写父类的方法
- 父类的引用指向子类的对象;例如：`Parent p = new Child();`

简单说来: 父类引用指向子类对象，调用方法时会调用子类的实现，而不是父类的实现，称为多态.
```
class Parent {

    void contextLoads(){
        System.out.println("i am Parent ... ");
    }
}
class Child  extends Parent {
    @Override
    void contextLoads(){
        System.out.println("i am Child ... ");
    }
}
class mainTest{
    public static void main(String[] args) {
        Parent child = new Child();
        // i am Child ... 
        child.contextLoads();
    }
}
```

#### 优缺点

**优点**
- 消除类型之间的耦合关系
- 可替换性
- 可扩充性
- 接口性
- 灵活性
- 简化性

**缺点**

不能使用子类特有的方法和属性.在编写代码期间使用多态调用方法或属性时，编译工具首先会检查父类中是否有该方法和属性，如果没有，则会编译报错。
```
class Parent {
    void contextLoads(){
        System.out.println("i am Parent ... ");
    }
}
class Child  extends Parent {

    String  c = "child";

    @Override
    void contextLoads(){
        System.out.println("i am Child ... ");
    }
    void test() {
        System.out.println("i am test method ...");
    }
}
class mainTest{
    public static void main(String[] args) {
        Parent child = new Child();
        // 编译报错: 无法解析 'Parent' 中的方法 'test'
        child.test();
        // 编译报错: 不能解决符号 'c'
        child.c;
    }
}
```


#### 重写与重载

##### 重写

**重写(Override)存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。**

> 为了满足里式替换原则，重写有以下三个限制：
>
> - 子类方法的访问权限必须大于等于父类方法；
> - 子类方法的返回类型必须是父类方法返回类型或为其子类型。
> - 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。

使用`@Override`注解，可以让编译器帮忙检查是否满足上面的三个限制条件。


下面的示例中，``SubClass ``为 ``SuperClass`` 的子类，``SubClass`` 重写了 ``SuperClass`` 的 ``func()`` 方法。其中：

- **子类方法访问权限为`public`，大于父类的`protected`。**
- **子类的返回类型为``ArrayList``，是父类返回类型`List`的子类。**
- **子类抛出的异常类型为 ``Exception``，是父类抛出异常 ``Throwable`` 的子类。**
- **子类重写方法使用``@Override``注解，从而让编译器自动检查是否满足限制条件。**

```
class SuperClass {
    protected List<Integer> func() throws Throwable {
        return new ArrayList<>();
    }
}

class SubClass extends SuperClass {
    @Override
    public ArrayList<Integer> func() throws Exception {
        return new ArrayList<>();
    }
}
```



在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有查找到再到父类中查看，看是否有继承来的方法。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：

- ``this.func(this)``
- ``super.func(this)``
- ``this.func(super)``
- ``super.func(super)``

```
/*继承关系
    A
    |
    B
    |
    C
    |
    D
 */


class A {

    public void show(A obj) {
        System.out.println("A.show(A)");
    }

    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}

class B extends A {

    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }
}

class C extends B {
}

class D extends C {
}
class  mainTest{
    public static void main(String[] args) {

        A a = new A();
        B b = new B();
        C c = new C();
        D d = new D();

        // 在 A 中存在 show(A obj)，直接调用
        a.show(a); // A.show(A)
        // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
        a.show(b); // A.show(A)
        // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
        b.show(c); // A.show(C)
        // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
        b.show(d); // A.show(C)

        // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
        A ba = new B();
        ba.show(c); // A.show(C)
        ba.show(d); // A.show(C)
    }
}
```

##### 重载

**重载(overload)是在一个类里面，方法名字相同，但是参数类型、个数、顺序至少有一个不同。返回类型可以相同也可以不同。应该注意的是，返回值不同，其它都相同不算是重载。**

**每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。** 最常用的地方就是构造器的重载。

> **重载规则:**
>
> - 被重载的方法必须改变参数列表(参数个数或类型不一样)；
> - 被重载的方法可以改变返回类型；
> - 被重载的方法可以改变访问修饰符；
> - 被重载的方法可以声明新的或更广的检查异常；
> - 方法能够在同一个类中或者在一个子类中被重载。
> - 无法以返回值类型作为重载函数的区分标准。



```
public class Overloading {
    public int test(){
        System.out.println("test1");
        return 1;
    }
 
    public void test(int a){
        System.out.println("test2");
    }   
 
    //以下两个参数类型顺序不同
    public String test(int a,String s){
        System.out.println("test3");
        return "returntest3";
    }   
 
    public String test(String s,int a){
        System.out.println("test4");
        return "returntest4";
    }   
 
    public static void main(String[] args){
        Overloading o = new Overloading();
        System.out.println(o.test());
        o.test(1);
        System.out.println(o.test(1,"test3"));
        System.out.println(o.test("test4",1));
    }
}
```

**方法的重写(Override)和重载(Overload)是Java多态性的不同表现，重写是父类与子类之间多态性的一种表现，重载可以理解成多态的具体表现形式**

> - 方法重载是一个类中定义了多个方法名相同,而他们的参数的数量不同或数量相同而类型和次序不同,则称为方法的重载(Overloading)。
> - 方法重写是在子类存在方法与父类的方法的名字相同,而且参数的个数与类型一样,返回值也一样的方法,就称为重写(Overriding)。
> - 方法重载是一个类的多态性表现,而方法重写是子类与父类的一种多态性表现。


## 设计模式
> 设计模式是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结，使用设计模式是为了可重用代码、让代码更容易被他人理解并且保证代码可靠性。
>
> 虽然[GoF](https://baike.baidu.com/item/GoF/6406151?fr=aladdin)设计模式只有23个，但是它们各具特色，每个模式都为某一个可重复的设计问题提供了一套解决方案。**根据它们的用途，设计模式可分为创建型，结构型和行为型三种**，其中创建型模式主要用于描述如何创建对象，结构型模式主要用于描述如何实现类或对象的组合，行为型模式主要用于描述类或对象怎样交互以及怎样分配职责;在GoF23种设计模式中包含5种创建型设计模式、7种结构型设计模式和11种行为型设计模式。此外，根据某个模式主要是用于处理类之间的关系还是对象之间的关系，设计模式还可以分为类模式和对象模式。

设计模式是一套被反复使用的、多数人知晓的、经过分类编目��、代码设计经验的总结。使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性、高内聚低耦合。

<table>
    <tr>
        <th rowspan="1">设计模式类型</th>
        <th colspan="1">设计模式名称</th>
        <th colspan="1">介绍</th>
        <th colspan="1">学习难度</th>
        <th colspan="1">使用频率</th>
    </tr>
    <tr>
        <td rowspan="6">创建型模式(6种)</td>
        <td><a href="#单例模式">单例模式</a></td>
        <td>保证一个类仅有一个对象，并提供一个访问它的全局访问点。</td>
        <td>★☆☆☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td><a href="#简单工厂模式">简单工厂模式</a></td>
        <td>定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。</td>
        <td>★★☆☆☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#工厂方法模式">工厂方法模式</a></td>
        <td>定义一个用于创建对象的接口，让子类决定将哪一个类实例化。</td>
        <td>★★☆☆☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#抽象工厂模式">抽象工厂模式</a></td>
        <td>提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体类。</td>
        <td>★★★★☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#原型模式">原型模式</a></td>
        <td>使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。</td>
        <td>★★★☆☆</td>
        <td>★★★☆☆</td>
    </tr>
    <tr>
        <td><a href="#建造者模式">建造者模式</a></td>
        <td>将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。</td>
        <td>★★★★☆</td>
        <td>★★☆☆☆</td>
    </tr>
    <tr>
        <td rowspan="7">结构型模式(7种)</td>
        <td><a href="#适配器模式">适配器模式</a></td>
         <td>将一个类的接口转换成客户希望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。</td>
        <td>★★☆☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td><a href="#桥接模式">桥接模式</a></td>
        <td>将抽象部分与它的实现部分分离，使他们都可以独立地变化。</td>
        <td>★★★☆☆</td>
        <td>★★★☆☆</td>
    </tr>
    <tr>
        <td><a href="#组合模式">组合模式</a></td>
        <td>组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象和组合对象的使用具有一致性。</td>
        <td>★★★☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td><a href="#装饰模式">装饰模式</a></td>
        <td>动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。</td>
        <td>★★★☆☆</td>
        <td>★★★☆☆</td>
    </tr>
    <tr>
        <td><a href="#外观模式">外观模式</a></td>
        <td>为子系统中的一组接口提供一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。</td>
        <td>★☆☆☆☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#享元模式">享元模式</a></td>
        <td>运用共享技术有效地支持大量细粒度的对象。</td>
        <td>★★★★☆</td>
        <td>★☆☆☆☆</td>
    </tr>
    <tr>
        <td><a href="#代理模式">代理模式</a></td>
        <td>为其他对象提供一个代理以控制对这个对象的访问。</td>
        <td>★★★☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td rowspan="11">行为模式(11种)</th>
        <td><a href="#职责链模式">职责链模式</a></td>
        <td>为解除请求的发送者和接收者之间的耦合，而使多个对象都有机会处理这个请求。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它。</td>
        <td>★★★☆☆</td>
        <td>★★☆☆☆</td>
    </tr>
    <tr>
        <td><a href="#命令模式">命令模式</a></td>
        <td>将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可取消的操作。</td>
        <td>★★★☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td><a href="#解释器模式">解释器模式</a></td>
        <td>定义一个语言，定义它的文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子。</td>
        <td>★★★★★</td>
        <td>★☆☆☆☆</td>
    </tr>
    <tr>
        <td><a href="#迭代器模式">迭代器模式</a></td>
        <td>提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。</td>
        <td>★★★☆☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#中介者模式">中介者模式</a></td>
        <td>用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显示地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。</td>
        <td>★★★☆☆</td>
        <td>★★☆☆☆</td>
    </tr>
    <tr>
        <td><a href="#备忘录模式">备忘录模式</a></td>
        <td>在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保持该状态，这样以后就可以将该对象恢复到保存的状态。</td>
        <td>★★☆☆☆</td>
        <td>★★☆☆☆</td>
    </tr>
    <tr>
        <td><a href="#观察者模式">观察者模式</a></td>
        <td>定义对象间的一种一对多的依赖关系，以便当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动刷新。</td>
        <td>★★★☆☆</td>
        <td>★★★★★</td>
    </tr>
    <tr>
        <td><a href="#状态模式">状态模式</a></td>
        <td>允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它所属的类。</td>
        <td>★★★☆☆</td>
        <td>★★★☆☆</td>
    </tr>
    <tr>
        <td><a href="#策略模式">策略模式</a></td>
        <td>定义一系列的算法，把它们一个个封装起来，并且使他们可相互替换。本模式使得算法的变化可以独立于使用它的客户。</td>
        <td>★☆☆☆☆</td>
        <td>★★★★☆</td>
    </tr>
    <tr>
        <td><a href="#模板方法模式">模板方法模式</a></td>
        <td>定义一个操作中的算法的骨架，而将一些步骤延迟到子类。</td>
        <td>★★☆☆☆</td>
        <td>★★★☆☆</td>
    </tr>
    <tr>
        <td><a href="#访问者模式">访问者模式</a></td>
        <td>表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素类别的前提下定义作用于这些元素的新操作。</td>
        <td>★★★★☆</td>
        <td>★☆☆☆☆</td>
    </tr>
</table>


### 单例模式
> 单例模式：确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。

单例模式设计就是采用一定的方法保证在整个程序中,对某个类只能存在一个对象的实例,并且该类只提供一个取得其对象实例的方法.

>单例模式作用:<br>
> - 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如网站首页页面缓存）.
> - 避免对资源的多重占用（比如写文件操作）.

单例模式使用的场景：需要频繁的进行创建和销毁的对象、创建对象时耗时过多或耗费资源过多(即：重量级对象)，但又经常用到的对象、工具类对象、频繁访问数据库或文件的对象(比如数据源、`session` 工厂等)

单例模式的6种写法.

写法名称 | 优点 | 缺点
---|---|---
[饿汉式](#饿汉式) | 线程安全,写法简单 | 不懒加载,可能造成浪费 
[懒汉式(线程不安全)](#懒汉式(线程不安全)) | 懒加载 | 线程不安全
[懒汉式(线程安全)](#懒汉式(线程安全)) | 线程安全,懒加载 | 效率很低,反序列化破坏单例
[双重校验锁](#双重校验锁) | 线程安全,懒加载 | 反序列化破坏单例
[静态内部类式](#静态内部类式) | 线程安全,懒加载 | 反序列化破坏单例 
[枚举式](#枚举式) | 防止反射攻击,反序列化创建对象,写法简单 | 不能传参,继承其他类



#### 饿汉式
```
class Singleton {

    private Singleton() {}

    private static final Singleton instance = new Singleton();

    public static Singleton getInstance() {
        return instance;
    }
}
```
这种写法比较简单，就是在类加载的时候就完成实例化。避免了线程同步问题。但是在类装载的时候就完成实例化，没有达到懒加载的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费.

#### 线程不安全的懒汉式
```
class Singleton {
    private Singleton() {
    }

    private static Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
起到了懒加载的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了`if (singleton == null)`判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式.

#### 线程安全的懒汉式
```
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
虽然解决了线程安全问题但是效率太低了，每个线程在想获得类的实例时候，执行`getInstance()`方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接 `return`就行了。

#### 双重校验锁
```
class Singleton {

    /**
     * volatile在这作用: 禁止JVM指令重排
     */
    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
`Double-Check`概念是多线程开发中常使用到的，如代码中所示，我们进行了两次`if (singleton == null)`检查，这样就可以保证线程安全了。这样，实例化代码只用执行一次，后面再次访问时，判断`if (singleton == null)`，直接`return`实例化对象，也避免的反复进行方法同步.

#### 静态内部类式
```
class Singleton {

    private Singleton() {}

    private static class InnerClass {
        private static final Singleton instance = new Singleton();
    }
    public static Singleton getInstance() {
        return InnerClass.instance;
    }
}
```
>这种方式同样利用了`classloder`的机制来保证初始化`instance`时只有一个线程，它跟饿汉式不同的是（很细微的差别）：饿汉式是只要`Singleton`类被装载了，那么`instance`就会被实例化（没有达到`lazy loading`效果），而这种方式是`Singleton`类被装载了，`instance`不一定被初始化。因为`SingletonHolder`类没有被主动使用，只有显示通过调用`getInstance`方法时，才会显示装载`SingletonHolder`类，从而实例化`instance`。想象一下，如果实例化`instance`很消耗资源，我想让他延迟加载，另外一方面，我不希望在`Singleton`类加载时就实例化，因为我不能确保`Singleton`类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化`instance`显然是不合适的。这个时候，这种方式相比饿汉式更加合理。

#### 枚举式
```
enum Singleton {
    INSTAMCE;

    Singleton() {
    }
}
```
这种方式是《Effective Java》作者`Josh Bloch`提倡的方式.借助 JDK1.5 中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。

#### 单例与序列化
上述单例模式6种写法除了,枚举式单例外其他5种写法都存在序列化问题.**序列化可以破坏单例.原因是序列化会通过反射调用无参数的构造方法创建一个新的对象.**

要想防止序列化对单例的破坏，只要在单例类中定义`readResolve`方法就可以解决该问题.**原因是反序列化时,会通过反射的方式调用要被反序列化的类的`readResolve`方法**
主要在`Singleton`类中定义`readResolve`方法，并在该方法中指定要返回的对象的生成策略，就可以防止单例被破坏。

以`DCL`为例,在该单例类中插入`readResolve`方法。
```
public class Singleton implements Serializable{

    private volatile static Singleton singleton;
    
    private Singleton (){}
    
    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }

    private Object readResolve() {
        return singleton;
    }
}
```

### 工厂模式
工厂模式是将实例化的对象代码提取出来,放到一个类中统一管理和维护,达到和主项目的依赖关系的解耦,从而提高项目的扩展和维护性.创建对象实例时,不要直接`new`类而是把这个`new`类的动作放在一个工厂的方法中,并返回.不要让类继承具体的类,而是继承抽象类或者实现接口.

详情查看以下三种工厂设计模式.

#### 简单工厂模式
> 简单工厂模式：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态方法，因此简单工厂模式又被称为静态工厂方法模式，它属于类创建型模式。

```
public class SimpleFactoryDemo {
    public static void main(String[] args) {
        SimpleFactory.getTest(1);
    }
}

class SimpleFactoryImpl1 implements SimpleFactoryInterface {

    @Override
    public void test() {
        System.out.println("i am  simpleFactory 1 ...");
    }
}
class SimpleFactoryImpl2 implements SimpleFactoryInterface {

    @Override
    public void test() {
        System.out.println("i am  simpleFactory 2 ...");
    }
}

class SimpleFactory {
    public static void getTest(int n) {
        switch (n) {
            case 1:
                SimpleFactoryImpl1 simpleFactory = new SimpleFactoryImpl1();
                simpleFactory.test();
                break;
            case 2:
                SimpleFactoryImpl2 simpleFactoryImpl2 = new SimpleFactoryImpl2();
                simpleFactoryImpl2.test();
                break;
            default:
        }
    }
}
```
简单工厂模式总结:
- **优点** 实现了对象创建和使用的分离,代码解耦.符合面向接口(指对外暴露的接口或类)编程思想
- **缺点** 如果代码逻辑,职责过多,则简单工厂会变的十分臃肿.
系统代码扩展困难.一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时,有可能造成工厂逻辑过于复杂,不利于系统的扩展和维护.

简单工厂模式适用于创建的对象比较少,业务逻辑不太复杂的情景

#### 工厂方法模式
> 工厂方法模式：定义一个用于创建对象的接口，让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式，又可称作虚拟构造器模式或多态工厂模式。工厂方法模式是一种类创建型模式。

```
public class FactoryMethodDemo {
    public static void main(String[] args) {
        FoodFactory f = new ColdRiceNoodleFactory();
        f.getFood().eat();
        // 扩展需要增加产品及相应产品工厂并实现相关接口
    }
}

interface Food {
    void eat();
}
interface FoodFactory {
    Food getFood();
}

class RiceNoodle implements Food{

    @Override
    public void eat() {
        System.out.println("eat rice noodle ...");
    }
}
class RiceNoodleFactory implements FoodFactory{

    @Override
    public Food getFood() {
        return new RiceNoodle();
    }
}

class ColdRiceNoodle implements Food {

    @Override
    public void eat() {
        System.out.println("eat cold rice noodle ...");
    }
}
class ColdRiceNoodleFactory implements FoodFactory {

    @Override
    public Food getFood() {
        return new ColdRiceNoodle();
    }
}
```
工厂方法模式是简单工厂模式的延伸,它**继承了简单工厂模式的优点,同时还弥补了简单工厂模式的不足.**
使用工厂方法模式扩展时,无须修改抽象工厂和抽象产品提供的接口,无须修改客户端,也无须修改其他的具体工厂和具体产品,而只**要添加一个具体工厂和具体产品**就可以了，这样，系统的可扩展性也就变得非常好，完全符合“开闭原则”。

但是在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，**系统中类的个数将成对增加**，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。

#### 抽象工厂模式
> 抽象工厂模式：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式。

```
interface Food {
    void eat();
}

interface FoodFactory {
    ColdRiceNoodle getColdRiceNoodle();
    RiceNoodle getRiceNoodle();
}
class RiceNoodle implements Food{

    @Override
    public void eat() {
        System.out.println("eating rice noodle");
    }
}
class ColdRiceNoodle implements Food{

    @Override
    public void eat() {
        System.out.println("eating cold rice noodle");
    }
}
class RiceNoodleFactory implements FoodFactory {


    @Override
    public ColdRiceNoodle getColdRiceNoodle() {
        return new ColdRiceNoodle();
    }

    @Override
    public RiceNoodle getRiceNoodle() {
        return new RiceNoodle();
    }
}

class ColdRiceNoodleFactory implements FoodFactory {

    @Override
    public ColdRiceNoodle getColdRiceNoodle() {
        return new ColdRiceNoodle();
    }

    @Override
    public RiceNoodle getRiceNoodle() {
        return new RiceNoodle();
    }
}
```
抽象工厂模式是工厂方法模式的进一步延伸,仍然具有工厂方法和简单工厂的**优点**.抽象工厂模式隔离了具体类的生成,使得客户并不需要知道什么被创建。由于这种隔离，更换一个具体工厂就变得相对容易，所有的具体工厂都实现了抽象工厂中定义的那些公共接口，因此只需改变具体工厂的实例，就可以在某种程度上改变整个软件系统的行为.

但是抽象工厂也存在一些**缺点**,增加新的产品等级结构麻烦，需要对原有系统进行较大的修改，甚至需要修改抽象层代码，这显然会带来较大的不便，违背了“开闭原则”。

### 原型模式
在Java中通过`new`关键字创建的对象是非常繁琐的,在我们需要大量对象的情况下,原型模式就是我们可以考虑实现的方式.

> 原型模式：使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。原型模式是一种对象创建型模式

原型模式我们也称为克隆模式，即一个某个对象为原型克隆出来一个一模一样的对象，该对象的属性和原型对象一模一样。而且对于原型对象没有任何影响。**原型模式的克隆方式有两种：浅克隆和深度克隆;浅克隆和深克隆的主要区别在于是否支持引用类型的成员变量的复制.**

#### 浅克隆
在浅克隆中,当对象被复制时只复制它本身和其中包含的值类型的成员变量,而引用类型的成员对象并没有复制.

代码实现
```
public class ShallowClone {
    public static void main(String[] args) {
        CloneHuman cloneHuman = new CloneHuman("黑色","大眼睛","高鼻梁","大嘴巴",new Date(123231231231L));
        for (int i = 0; i < 20; i++) {
            try {
                CloneHuman clone = (CloneHuman)cloneHuman.clone();
                System.out.printf("头发：%s,眼睛：%s,鼻子：%s,嘴巴：%s,生日：%s",clone.getHair(),clone.getEye(),clone.getNodes(),clone.getMouse(),clone.getBirth());
                System.out.println();
                System.out.println("浅克隆，引用类型地址比较：" + (cloneHuman.getBirth() == clone.getBirth()));
            } catch (CloneNotSupportedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}

class CloneHuman  implements Cloneable {

    private String hair;

    private String eye;

    private String nodes;

    private String mouse;

    private Date birth;

    public String getHair() {
        return hair;
    }

    public void setHair(String hair) {
        this.hair = hair;
    }

    public String getEye() {
        return eye;
    }

    public void setEye(String eye) {
        this.eye = eye;
    }

    public String getNodes() {
        return nodes;
    }

    public void setNodes(String nodes) {
        this.nodes = nodes;
    }

    public String getMouse() {
        return mouse;
    }

    public void setMouse(String mouse) {
        this.mouse = mouse;
    }

    public CloneHuman(String hair, String eye, String nodes, String mouse,Date brith) {
        this.hair = hair;
        this.eye = eye;
        this.nodes = nodes;
        this.mouse = mouse;
        this.birth = brith;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }
}
```
> PS Java语言提供的`Cloneable`接口和`Serializable`接口的代码非常简单，它们都是空接口，这种空接口也称为标识接口，标识接口中没有任何方法的定义，其作用是告诉JRE这些接口的实现类是否具有某个功能，如是否支持克隆、是否支持序列化等。

应该注意的是，`clone()`方法并不是`Cloneable`接口的方法，而是`Object`的一个`protected`方法。`Cloneable`接口只是规定，如果一个类没有实现`Cloneable`接口又调用了`clone()`方法，就会抛出 `CloneNotSupportedException`。

#### 深克隆
在深克隆中,除了对象本身被复制外,对象所包含的所有成员变量也将复制.
深克隆有两种实现方式,第一种是在浅克隆的基础上实现,第二种是通过序列化和反序列化实现

**在浅克隆的基础上实现**
```
public class DeepClone {
    public static void main(String[] args) {
        CloneHuman cloneHuman = new CloneHuman("黑色","大眼睛","高鼻梁","大嘴巴",new Date(123231231231L));
        for (int i = 0; i < 20; i++) {
            try {
                CloneHuman clone = (CloneHuman)cloneHuman.clone();
                System.out.printf("头发：%s,眼睛：%s,鼻子：%s,嘴巴：%s,生日：%s",clone.getHair(),clone.getEye(),clone.getNodes(),clone.getMouse(),clone.getBirth());
                System.out.println();
                System.out.println("深克隆，引用类型地址比较：" + (cloneHuman.getBirth() == clone.getBirth()));
            } catch (CloneNotSupportedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
class CloneHuman  implements Cloneable {

    private String hair;

    private String eye;

    private String nodes;

    private String mouse;

    private Date birth;

    public String getHair() {
        return hair;
    }

    public void setHair(String hair) {
        this.hair = hair;
    }

    public String getEye() {
        return eye;
    }

    public void setEye(String eye) {
        this.eye = eye;
    }

    public String getNodes() {
        return nodes;
    }

    public void setNodes(String nodes) {
        this.nodes = nodes;
    }

    public String getMouse() {
        return mouse;
    }

    public void setMouse(String mouse) {
        this.mouse = mouse;
    }

    public CloneHuman(String hair, String eye, String nodes, String mouse,Date brith) {
        this.hair = hair;
        this.eye = eye;
        this.nodes = nodes;
        this.mouse = mouse;
        this.birth = brith;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        CloneHuman human = (CloneHuman)super.clone();
        human.birth = (Date)this.birth.clone();
        return human;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }
}
```
**序列化反序列化实现深克隆**
```
public class DeepClone2 {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        CloneHuman2 cloneHuman1 = new CloneHuman2("黑色","大眼睛","高鼻梁","大嘴巴",new Date(123231231231L));

        // 使用序列化和反序列化实现深克隆
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(cloneHuman1);
        byte[] bytes = bos.toByteArray();

        ByteArrayInputStream bis = new ByteArrayInputStream(bytes);
        ObjectInputStream ois = new ObjectInputStream(bis);

        // 克隆好的对象
        CloneHuman2 cloneHuman2 = (CloneHuman2) ois.readObject();
        System.out.println("深克隆，引用类型地址比较：" + (cloneHuman1.getBirth() == cloneHuman2.getBirth()));

    }
}
class CloneHuman2  implements Cloneable, Serializable {

    private String hair;

    private String eye;

    private String nodes;

    private String mouse;

    private Date birth;

    public String getHair() {
        return hair;
    }

    public void setHair(String hair) {
        this.hair = hair;
    }

    public String getEye() {
        return eye;
    }

    public void setEye(String eye) {
        this.eye = eye;
    }

    public String getNodes() {
        return nodes;
    }

    public void setNodes(String nodes) {
        this.nodes = nodes;
    }

    public String getMouse() {
        return mouse;
    }

    public void setMouse(String mouse) {
        this.mouse = mouse;
    }

    public CloneHuman2(String hair, String eye, String nodes, String mouse,Date brith) {
        this.hair = hair;
        this.eye = eye;
        this.nodes = nodes;
        this.mouse = mouse;
        this.birth = brith;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        CloneHuman2 human = (CloneHuman2)super.clone();
        human.birth = (Date)this.birth.clone();
        return human;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }
}
```

#### 总结
原型模式作为一种快速创建大量相同或相似对象的方式，在软件开发中应用较为广泛，很多软件提供的复制和粘贴操作就是原型模式的典型应用.

通过`clone`的方式在获取大量对象的时候性能开销基本没有什么影响，而`new`的方式随着实例的对象越来越多，性能会急剧下降，所以原型模式是一种比较重要的获取实例的方式.

**优点**
- 当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，提高创建对象的效率。
- 可以使用深克隆的方式保存对象的状态，使用原型模式将对象复制一份并将其状态保存起来，可辅助实现撤销操作。
- 扩展性较好，由于在原型模式中提供了抽象原型类，在客户端可以针对抽象原型类进行编程，而将具体原型类写在配置文件中，增加或减少产品类对原有系统都没有任何影响。

**缺点**
- 需要为每一个类配备一个克隆方法，而且该克隆方法位于一个类的内部，当对已有的类进行改造时，需要修改源代码，违背了“开闭原则”。
- 在实现深克隆时需要编写较为复杂的代码，而且当对象之间存在多重的嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来可能会比较麻烦。

**使用场景**

原型模式很少单独出现，一般是和工厂方法模式一起出现，通过`clone`的方法创建一个对象，然后由工厂方法提供给调用者。
spring中bean的创建实际就是两种：单例模式和原型模式。

- 创建新对象成本较大，新的对象可以通过原型模式对已有对象进行复制来获得。
- 系统要保存对象的状态，而对象的状态变化很小，或者对象本身占用内存较少时，可以使用原型模式配合备忘录模式来实现。

### 建造者模式
> 建造者模式：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。

建造者模式是较为复杂的创建型模式，它将客户端与包含多个组成部分（或部件）的复杂对象的创建过程分离，客户端无须知道复杂对象的内部组成部分与装配方式，只需要知道所需建造者的类型即可。它关注如何一步一步创建一个的复杂对象，不同的具体建造者定义了不同的创建过程，且具体建造者相互独立，增加新的建造者非常方便，无须修改已有代码，系统具有较好的扩展性。

代码实现
```
public class MainTest {
    public static void main(String[] args) {

        Director director = new Director();
        Builder commonBuilder = new CommonRole();

        director.construct(commonBuilder);
        Role commonRole = commonBuilder.getRole();
        System.out.println(commonRole);
    }
}

class Role {
    private String head;
    private String body;
    private String foot;
    private String sp;
    private String hp;
    private String name;

    public void setSp(String sp) {
        this.sp = sp;
    }

    public String getSp() {
        return sp;
    }

    public void setHp(String hp) {
        this.hp = hp;
    }

    public String getHp() {
        return hp;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Role{" +
                "head='" + head + '\'' +
                ", body='" + body + '\'' +
                ", foot='" + foot + '\'' +
                ", sp='" + sp + '\'' +
                ", hp='" + hp + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

abstract class Builder {

    public abstract void builderHead();

    public abstract void builderBody();

    public abstract void builderFoot();

    public abstract void builderSp();

    public abstract void builderHp();

    public abstract void builderName();

    public Role getRole() {
        return new Role();
    }
}

class CommonRole extends Builder {

    private Role role = new Role();

    @Override
    public void builderHead() {
        System.out.println("building head .....");
    }

    @Override
    public void builderBody() {
        System.out.println("building body .....");
    }

    @Override
    public void builderFoot() {
        System.out.println("building foot .....");
    }

    @Override
    public void builderSp() {
        role.setSp("100");
    }

    @Override
    public void builderHp() {
        role.setHp("100");
    }

    @Override
    public void builderName() {
        role.setName("lucy");
    }

    @Override
    public Role getRole() {
        return role;
    }
}
class Director {

    public void construct(Builder builder) {
        builder.builderHead();
        builder.builderBody();
        builder.builderFoot();
        builder.builderHp();
        builder.builderSp();
        builder.builderName();
    }
}
```

**优点**
- 客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。
- 建造者模式很容易进行扩展。如果有新的需求，通过实现一个新的建造者类就可以完成，基本上不用修改之前已经测试通过的代码，因此也就不会对原有功能引入风险。符合[开放封闭原则](#开放封闭原则)。

**缺点**
- 若产品内部发生变化，建造者都要修改，成本较大；若内部变化复杂，会有很多的建造类。
- 产品必须有共同点，使用范围有限。建造者模式创造出来的产品，其组成部分基本相同。如果产品之间的差异较大，则不适用这个模式。

**使用场景**
- 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性。
- 需要生成的产品对象的属性相互依赖，需要指定其生成顺序。
- 对象的创建过程独立于创建该对象的类。在建造者模式中引入了指挥者类，将创建过程封装在指挥者类中，而不在建造者类中。
- 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品。

### 适配器模式
> 适配器模式：将一个接口转换成客户希望的另一个**接口**(指广义的接口，它可以表示一个方法或者方法的集合)，使接口不兼容的那些类可以一起工作，其别名为包装器。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

在适配器模式中，我们通过增加一个新的适配器类来解决接口不兼容的问题，使得原本没有任何关系的类可以协同工作。**根据适配器类与适配者类的关系不同，适配器模式可分为对象适配器和类适配器两种，在对象适配器模式中，适配器与适配者之间是关联关系；在类适配器模式中，适配器与适配者之间是继承（或实现）关系。**

#### 对象适配器
由于在Java中不支持多重继承，而且有破坏封装之嫌。所以提倡[多用组合少用继承](#继承与组合)，在实际开发中推荐使用对象适配器模式。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        new RedHat(new Linux()).useInputMethod();
        System.out.println("==============");

        new Win10(new Windows()).useInputMethod();
        System.out.println("==============");

        Adapter adapter = new Adapter(new Windows());
        new RedHat(adapter).useInputMethod();
    }
}

interface LinuxSoftware {
    void inputMethod();
}

interface WindowsSoftware {
    void inputMethod();
}

class Linux implements LinuxSoftware {

    @Override
    public void inputMethod() {
        System.out.println("linux 系统输入法 ...");
    }
}

class Windows implements WindowsSoftware {

    @Override
    public void inputMethod() {
        System.out.println("windows 系统输入法 ...");
    }
}

class RedHat {
    private LinuxSoftware linuxSoftware;

    public RedHat(LinuxSoftware linuxSoftware) {
        this.linuxSoftware = linuxSoftware;
    }

    public void useInputMethod() {
        System.out.println("开始使用 redHat 系统输入法 ...");
        linuxSoftware.inputMethod();
        System.out.println("结束使用 redHat 系统输入法 ...");
    }
}

class Win10 {
    private WindowsSoftware windowsSoftware;

    public Win10(WindowsSoftware windowsSoftware) {
        this.windowsSoftware = windowsSoftware;
    }

    public void useInputMethod() {
        System.out.println("开始使用 win10 系统输入法 ...");
        windowsSoftware.inputMethod();
        System.out.println("结束使用 win10 系统输入法 ...");
    }
}

// 在 Linux 系统上使用 windows 输入法
class Adapter implements LinuxSoftware{

    private WindowsSoftware windowsSoftware;

    public Adapter(WindowsSoftware windowsSoftware) {
        this.windowsSoftware = windowsSoftware;
    }

    @Override
    public void inputMethod() {
        windowsSoftware.inputMethod();
    }
}
```

#### 类适配器
类适配器模式和对象适配器模式最大的区别在于适配器和适配者之间的关系不同，对象适配器模式中适配器和适配者之间是关联关系，而类适配器模式中适配器和适配者是继承关系。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        new RedHat(new Linux()).useInputMethod();
        System.out.println("==============");

        new Win10(new Windows()).useInputMethod();
        System.out.println("==============");

        Adapter adapter = new Adapter();
        new RedHat(adapter).useInputMethod();
    }
}

interface LinuxSoftware {
    void inputMethod();
}

interface WindowsSoftware {
    void inputMethod();
}

class Linux implements LinuxSoftware {

    @Override
    public void inputMethod() {
        System.out.println("linux 系统输入法 ...");
    }
}

class Windows implements WindowsSoftware {

    @Override
    public void inputMethod() {
        System.out.println("windows 系统输入法 ...");
    }
}

class RedHat {
    private LinuxSoftware linuxSoftware;

    public RedHat(LinuxSoftware linuxSoftware) {
        this.linuxSoftware = linuxSoftware;
    }

    public void useInputMethod() {
        System.out.println("开始使用 redHat 系统输入法 ...");
        linuxSoftware.inputMethod();
        System.out.println("结束使用 redHat 系统输入法 ...");
    }
}

class Win10 {
    private WindowsSoftware windowsSoftware;

    public Win10(WindowsSoftware windowsSoftware) {
        this.windowsSoftware = windowsSoftware;
    }

    public void useInputMethod() {
        System.out.println("开始使用 win10 系统输入法 ...");
        windowsSoftware.inputMethod();
        System.out.println("结束使用 win10 系统输入法 ...");
    }
}

// 在 Linux 系统上使用 windows 输入法
class Adapter extends Windows implements LinuxSoftware{
    @Override
    public void inputMethod() {
        super.inputMethod();
    }
}
```

#### 总结
适配器模式将现有接口转化为客户类所期望的接口，实现了对现有类的复用，它是一种使用频率非常高的设计模式，在软件开发中得以广泛应用，在Spring等开源框架、驱动程序设计（如JDBC中的数据库驱动程序）中也使用了适配器模式。

**优点**
- 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构。
- 增加了类的透明性和复用性，同时系统的灵活性和扩展性都非常好，更换适配器或者增加新的适配器都非常方便。可以在不修改原有代码的基础上增加新的适配器类，完全符合[开放封闭原则](#开放封闭原则)。


**缺点**
- 过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器。
- 对象适配器模式的缺点是很难置换适配者类的方法。
- 类适配器模式中的目标抽象类只能为接口，不能为类，其使用有一定的局限性。

**使用场景**

系统需要使用现有的类，而这些类的接口不符合系统的需要；想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类一起工作。

- 系统需要使用现有的类，而此类的接口不符合系统的需要。
- 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。
- 通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）

### 桥接模式
> 桥接模式：将实现与抽象放在两个不同的类层次中，使两个层次可以独立改变。它是一种对象结构型模式，又称为柄体模式或接口模式。

桥接模式是一种很实用的结构型设计模式，如果软件系统中某个类存在两个独立变化的维度，通过该模式可以将这两个维度分离出来，使两者可以独立扩展，让系统更加符合“单一职责原则”。

例如像手机制造：内存是一个公司生产，芯片是另一个公司生产，而品牌又是另一个公司。我们需要什么样子的手机就把相应的芯片、内存组装起来。桥接模式就是把两个不同维度的东西桥接起来。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        Phone_A phone_a = new Phone_A();
        phone_a.setAbstractChip(new Chip_A());
        phone_a.setAbstractMemory(new Memory_A());
        phone_a.finished();
        System.out.println("=================");
        Phone_B phone_b = new Phone_B();
        phone_b.setAbstractChip(new Chip_B());
        phone_b.setAbstractMemory(new Memory_A());
        phone_b.finished();
    }
}
abstract class AbstractMemory {
    protected abstract void size();
}

abstract class AbstractChip {
    protected abstract void type();
}
abstract class  AbstractPhone {
    protected AbstractMemory abstractMemory;
    protected AbstractChip abstractChip;

    public void setAbstractChip(AbstractChip abstractChip) {
        this.abstractChip = abstractChip;
    }

    public void setAbstractMemory(AbstractMemory abstractMemory) {
        this.abstractMemory = abstractMemory;
    }

    protected abstract void finished();
}

class Memory_A  extends AbstractMemory{

    @Override
    protected void size() {
        System.out.println("create 6G of memory ...");
    }
}
class Memory_B  extends AbstractMemory{

    @Override
    protected void size() {
        System.out.println("create 8G of memory ...");
    }
}

class Chip_A extends AbstractChip {

    @Override
    protected void type() {
        System.out.println("snapdragon 888 chip ...");
    }
}

class Chip_B extends AbstractChip {

    @Override
    protected void type() {
        System.out.println("A14 chip ...");
    }
}

class Phone_A  extends AbstractPhone{

    @Override
    protected void finished() {
        abstractMemory.size();
        abstractChip.type();
        System.out.println("phone_a assembly completed ...");
    }
}

class Phone_B  extends AbstractPhone{

    @Override
    protected void finished() {
        abstractMemory.size();
        abstractChip.type();
        System.out.println("phone_b assembly completed ...");
    }
}
```
桥接模式是设计Java虚拟机和实现JDBC等驱动程序的核心模式之一，应用较为广泛。在软件开发中如果一个类或一个系统有多个变化维度时，都可以尝试使用桥接模式对其进行设计。桥接模式为多维度变化的系统提供了一套完整的解决方案，并且降低了系统的复杂度。

**优点**
- 实现了抽象和实现部分的分离，从而极大的提供了系统的灵活性，让抽象部分和实现部分独立开来，这有助于系统进行分层设计，从而产生更好的结构化系统。
- 优秀的扩展能力。在两个变化维度中任意扩展一个维度，都不需要修改原有系统，符合[开放封闭原则](#开放封闭原则)。
- 桥接模式替代多层继承方案，可以减少子类的个数，降低系统的管理和维护成本。

**缺点**
- 桥接模式的引入会增加系统的理解与设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计与编程。
- 桥接模式要求正确识别出系统中两个独立变化的维度(抽象、和实现)，因此其使用范围有一定的局限性，即需要有这样的应用场景，如何正确识别两个独立维度也需要一定的经验积累。

**使用场景**
- 桥接模式主要解决在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活。对于两个独立变化的维度，使用桥接模式再适合不过了。
- 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。

### 组合模式
> 组合模式：组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象和组合对象的使用具有一致性，组合模式又可以称为“整体—部分”模式，它是一种对象结构型模式。

组合模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        Tree tree = new Tree();
        tree.add(new LeftNode());
        tree.add(new RightNode());
        tree.implmethod();
    }
}

abstract class AbstractTree {
    protected abstract void add(AbstractTree node);

    protected abstract void remove(AbstractTree node);

    protected abstract void implmethod();
}

class LeftNode extends AbstractTree {

    @Override
    protected void add(AbstractTree node) {
        System.out.println("Exception: the method is not supported. ");
    }

    @Override
    protected void remove(AbstractTree node) {
        System.out.println("Exception: the method is not supported. ");
    }

    @Override
    protected void implmethod() {
        System.out.println("left node method ...");
    }
}

class RightNode extends AbstractTree {

    @Override
    protected void add(AbstractTree node) {
        System.out.println("Exception: the method is not supported. ");
    }

    @Override
    protected void remove(AbstractTree node) {
        System.out.println("Exception: the method is not supported. ");
    }

    @Override
    protected void implmethod() {
        System.out.println("right node method ...");
    }
}

class Tree extends AbstractTree {

    private ArrayList<AbstractTree> treeList = new ArrayList<>();


    @Override
    protected void add(AbstractTree node) {
        treeList.add(node);
    }

    @Override
    protected void remove(AbstractTree node) {
        treeList.remove(node);
    }

    @Override
    protected void implmethod() {
        System.out.println("tree node");
        for (AbstractTree node : treeList) {
            node.implmethod();
        }
    }
}
```
组合模式使用面向对象的思想来实现树形结构的构建与处理，描述了如何将容器对象和叶子对象进行递归组合，实现简单，灵活性好。由于在软件开发中存在大量的树形结构，因此组合模式是一种使用频率较高的结构型设计模式。在XML解析、组织结构树处理、文件系统设计等领域，组合模式都得到了广泛应用。

**优点**
- 高层模块调用简单。客户端可以一致地使用一个组合结构或其中单个对象，不必关心处理的是单个对象还是整个组合结构。
- 在组合模式中节点增加自由，无须对现有类库进行任何修改，符合[开闭原则](#开放封闭原则)。

**缺点**
- 在使用组合模式时，其叶子和树的声明都是实现类，而不是接口，违反了[依赖倒置原则](#依赖倒置原则)。

**使用场景**
- 部分、整体场景，如树形菜单，文件、文件夹的管理。
- 希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象。

### 装饰模式
> 装饰模式：动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。装饰模式是一种对象结构型模式。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        Drink coffee = new ShortBlock();
        System.out.println("订单价格："+ coffee.cost());
        System.out.println("订单描述：" + coffee.getDesc());
        System.out.println("=====================");
        coffee = new Chocolate(coffee);
        System.out.println("订单价格："+ coffee.cost());
        System.out.println("订单描述：" + coffee.getDesc());
        System.out.println("=====================");
        coffee = new Milk(coffee);
        System.out.println("订单价格："+ coffee.cost());
        System.out.println("订单描述：" + coffee.getDesc());
    }
}

abstract class Drink {

    public String desc;

    private double price = 0.0;

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String getDesc() {
        return desc;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public double getPrice() {
        return price;
    }

    protected abstract double cost();
}

class ShortBlock extends Drink{

    public ShortBlock() {
        super.setPrice(3.0);
        super.setDesc("ShortBlock");
    }

    @Override
    protected double cost() {
        return super.getPrice();
    }
}

class LongBlock extends Drink {

    public LongBlock() {
        super.setPrice(5.0);
        super.setDesc("LongBlock");
    }

    @Override
    protected double cost() {
        return super.getPrice();
    }
}

class CoffeeShop extends Drink {

    private final Drink coffee;

    protected CoffeeShop(Drink coffee) {
        this.coffee = coffee;
    }

    @Override
    protected double cost() {
        return super.getPrice() + coffee.cost();
    }

    @Override
    public String getDesc() {
        return desc + " " + getPrice() +" && "+  coffee.getDesc();
    }
}

class Milk extends CoffeeShop {

    public Milk(Drink seasoning) {
        super(seasoning);
        super.setDesc("Milk");
        super.setPrice(6);
    }

}

class Chocolate extends CoffeeShop {

    public Chocolate(Drink seasoning) {
        super(seasoning);
        super.setDesc("Chocolate");
        super.setPrice(4);
    }

}
```

装饰模式降低了系统的耦合度，可以动态增加或删除对象的职责，并使得需要装饰的具体构件类和具体装饰类可以独立变化，以便增加新的具体构件类和具体装饰类。在软件开发中，装饰模式应用较为广泛，例如在JavaIO中的输入流和输出流的设计、javax.swing包中一些图形界面构件功能的增强等地方都运用了装饰模式。

**装饰者模式主要解决：**一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

**优点**

- 对于扩展一个对象的功能，装饰模式比继承更加灵活性，不会导致类的个数急剧增加。
- 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。

**缺点**

- 多层装饰比较复杂。
- 使用装饰模式进行系统设计时将产生很多小对象，这些对象的区别在于它们之间相互连接的方式有所不同，而不是它们的类或者属性值有所不同，大量小对象的产生势必会占用更多的系统资源，在一定程序上影响程序的性能。

**使用场景**

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。可代替继承。

- 扩展一个类的功能。 
- 动态增加功能，动态撤销。
- 当不能采用继承的方式对系统进行扩展或者采用继承不利于系统扩展和维护时可以使用装饰模式。不能采用继承的情况主要有两类：
  - 第一类是系统中存在大量独立的扩展，为支持每一种扩展或者扩展之间的组合将产生大量的子类，使得子类数目呈爆炸性增长
  - 第二类是因为类已定义为不能被继承



### 外观模式

> 外观模式：为子系统中的一组接口提供一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

外观模式又称为门面模式，它是一种对象结构型模式。外观模式是[迪米特法则](#迪米特法则)的一种具体实现，通过引入一个新的外观角色可以降低原有系统的复杂度，同时降低客户类与子系统的耦合度。

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.aTest();
    }
}
class Facade {
    private final A a;
    private final B b;
    private final C c;
    private final D d;

    public Facade() {
        this.a = A.getInstance();
        this.b = B.getInstance();
        this.c = C.getInstance();
        this.d = D.getInstance();
    }

    public void aTest() {
        a.aTest();
        b.aTest();
        c.aTest();
        d.aTest();
    }
}

/**
 * A调用B类，A调用C类，B调用C类，D调用B类，D调用C类
 */
class A {
   private final static A instance = new A();
   private A(){}
   public static A getInstance() {
       return instance;
   }
   public  void aTest() {
       B.getInstance().aTest();
       C.getInstance().aTest();
       System.out.println("A class test method ...");
   }
}

class B {
    private final static B instance = new B();
    private B(){}
    public static B getInstance() {
        return instance;
    }
    public  void aTest() {
        C.getInstance().aTest();
        System.out.println("B class test method ...");
    }
}

class C {
    private final static C instance = new C();
    private C(){}
    public  static C getInstance() {
        return instance;
    }
    public  void aTest() {
        System.out.println("C class test method ...");
    }
}

class D {
    private final static D instance = new D();
    private D(){}
    public static D getInstance() {
        return instance;
    }
    public  void aTest() {
        B.getInstance().aTest();
        C.getInstance().aTest();
        System.out.println("D class test method ...");
    }
}
```
外观模式的主要目的在于降低系统的复杂程度，在面向对象软件系统中，类与类之间的关系越多，不能表示系统设计得越好，反而表示系统中类之间的耦合度太大，这样的系统在维护和修改时都缺乏灵活性，因为一个类的改动会导致多个类发生变化，而外观模式的引入在很大程度上降低了类与类之间的耦合关系。引入外观模式之后，增加新的子系统或者移除子系统都非常方便，客户类无须进行修改（或者极少的修改），只需要在外观类中增加或移除对子系统的引用即可。从这一点来说，外观模式在一定程度上并不符合[开放封闭原则](#开放封闭原则)，增加新的子系统需要对原有系统进行一定的修改，虽然这个修改工作量不大。

**优点**
- 减少关联关系；对客户端屏蔽了具体的实现，使得子系统使用起来更加容易；客户端代码将变得很简单，与之关联的对象也很少。
- 解耦合；具体实现有改变不会影响到调用的客户端，只需要调整外观类即可。

**缺点**
- 如果设计不当，扩展时增加新的实现可能需要修改外观类的源代码，违背了[开放封闭原则](#开放封闭原则).
- 不能很好地限制客户端直接使用子系统类，如果对客户端访问子系统类做太多的限制则减少了可变性和灵活性。

**使用场景**
- 当要为访问一系列复杂的子系统提供一个简单入口时可以使用外观模式。
- 客户端程序与多个子系统之间存在很大的依赖性。引入外观类可以将子系统与客户端解耦，从而提高子系统的独立性和可移植性。
- 在层次化结构中，可以使用外观模式定义系统中每一层的入口，层与层之间不直接产生联系，而通过外观类建立联系，降低层之间的耦合度。

### 享元模式
> 享元模式：运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用。由于享元模式要求能够共享的对象必须是细粒度对象，因此它又称为轻量级模式，它是一种对象结构型模式。

代码实现

```
public class MainTest {
    public static void main(String[] args) {
        WebSiteFactory webSiteFactory = new WebSiteFactory();
        
        webSiteFactory.getWebSite("新闻").use(new User("lucy"));

        webSiteFactory.getWebSite("新闻").use(new User("jane"));

        webSiteFactory.getWebSite("博客").use(new User("jack"));

        webSiteFactory.getWebSite("博客").use(new User("maik"));

        webSiteFactory.getWebSite("博客").use(new User("seven"));

        System.out.println("网站类型共：" + webSiteFactory.countWebSiteType());
    }
}

abstract class WebSite{

    private String name;

    public abstract void use(User user);

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

class ConcreateWebSite extends WebSite {

    public ConcreateWebSite(String name) {
        super.setName(name);
    }

    @Override
    public void use(User user) {
        System.out.println("网站类型名称：" + super.getName() +"\t 网站用户：" + user.getUsername());
    }
}

class User {

    private String username;

    public User(String username) {
        this.username = username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}

class WebSiteFactory{

    private HashMap<String, WebSite> pool = new HashMap<>();

    public WebSite getWebSite(String webSiteName){
        if (!pool.containsKey(webSiteName)) {
            pool.put(webSiteName, new ConcreateWebSite(webSiteName));
        }
        return pool.get(webSiteName);
    }

    public Integer countWebSiteType() {
        return pool.size();
    }

}
```

当系统中存在大量相同或者相似的对象时，享元模式是一种较好的解决方案，它通过共享技术实现相同或相似的细粒度对象的复用，从而节约了内存空间，提高了系统性能。相比其他结构型设计模式，享元模式的使用频率并不算太高，但是作为一种以“节约内存，提高性能”为出发点的设计模式，它在软件开发中还是得到了一定程度的应用。例如：String、Java的池技术等。

享元模式主要解决在有大量对象时，有可能会造成内存溢出，我们把其中共同的部分抽象出来，如果有相同的业务请求，直接返回在内存中已有的对象，避免重新创建。

**优点**

- 大大减少对象的创建，降低系统的内存，使效率提高。

**缺点**

- 提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱。

**使用场景**

​	在使用享元模式时需要维护一个存储享元对象的享元池，而这需要耗费一定的系统资源，因此，应当在需要多次重复使用享元对象时才值得使用享元模式。

- 系统有大量相似对象。 
- 需要缓冲池的场景。





### 代理模式

> 代理模式：给某一个对象提供一个代理或占位符，并由代理对象来控制对原对象的访问。

代理模式是为一个对象提供一个替身，以控制对这个对象的访问。即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能。被代理的对象可以是远程对象、创建开销大的对象或需要安全控制的对象

代理模式有不同的形式, 主要有三种 静态代理、动态代理 (`JDK`代理、`Cglib`代理)。

#### 静态代理
```
public class MainTest {
    public static void main(String[] args) {
        ProxyImpl proxy = new ProxyImpl();
        StaticProxy staticProxy = new StaticProxy(proxy);
        staticProxy.test();
    }
}

interface ProxyInterface {
    void test();
}

class ProxyImpl implements ProxyInterface{

    @Override
    public void test() {
        System.out.println("helloWorld");
    }

}


class StaticProxy implements ProxyInterface{

    private ProxyImpl target;

    public StaticProxy (ProxyImpl target){
        this.target=target;
    }


    @Override
    public void test() {
        System.out.println("静态代理之前...");
        target.test();
        System.out.println("静态代理之后...");
    }
}
```
静态代理能在不修改目标对象的功能前提下，**能通过代理对象对目标进行扩展。**但是因为代理对象需要与目标对象实现相同的接口，所以会有很多代理类**一旦接口增加方法后，目标对象与代理对象都需要维护**。

#### 动态代理
动态代理，代理对象不需要实现接口，但是目标对象需要实现接口，否则不能实现动态代理。代理对象的生产是利用JDK的API，动态的在内存中构建代理对象。

##### JDK动态代理
```
public class MainTest {
    public static void main(String[] args) {
        ProxyImpl proxy = new ProxyImpl();
        ProxyInterface jdkProxyInterface = (ProxyInterface)new JDKProxy().bind(proxy);
        jdkProxyInterface.test();
    }
}
interface ProxyInterface {
    void test();
}

class ProxyImpl implements ProxyInterface{

    @Override
    public void test() {
        System.out.println("helloWorld");
    }

}

class JDKProxy {
    //通用类型，表示被代理的真实对象
    private Object target;

    public Object bind(Object target){
        this.target=target;
        //生成代理类（与被代理对象实现相同接口的兄弟类）
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), (proxy, method, args) -> {
            Object res;
            System.out.println("JDK动态代理前");
            res=method.invoke(target, args);
            System.out.println("JDK动态代理后");
            return res;
        });
    }

}
```

##### CGlib动态代理
`Cglib`代理也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能扩展。
`Cglib`是一个强大的高性能的代码生成包,它可以在运行期扩展 java 类与实现 java 接口.它广泛的被许多 AOP 的框架使用,例如 Spring AOP，实现方法拦截。

以下测试代码需要导入Cglib和asm相关jar包。
```
asm-3.3.1.jar
cglib-2.2.jar
```
```
PS：使用`CGLib`实现动态代理时出现了下面这个异常
Exception in thread "main" java.lang.IncompatibleClassChangeError: 
class net.sf.cglib.core.DebuggingClassWriter has interface org.objectweb.asm.ClassVisitor as super class
```
> - 原因：
> cglib.jar包含asm.jar包。报错内容是`ClassVisitor`的父类不相容。详细：[原因分析](https://my.oschina.net/itblog/blog/528613)<br>
>
> - 解决：
> 测试时用`cglib2.2.jar`和`asm3.3.1.jar`版本，解决jar包冲突问题。


```
public class MainTest {
    public static void main(String[] args) {
        Target target = new Target();
        Target bind = (Target)new CGLibProxy().bind(target);
        bind.test();
    }
}


class Target{

    public void test() {
        System.out.println("helloWorld");
    }

}

class CGLibProxy implements MethodInterceptor {

    private Object target;

    public  Object bind(Object target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object res;
        System.out.println("CGLib动态代理前");
        res=method.invoke(target, args);
        System.out.println("CGLib动态代理后");
        return res;
    }

}
```

#### 总结

代理模式是常用的结构型设计模式之一，它为对象的间接访问提供了一个解决方案，可以对对象的访问进行控制。代理模式主要解决：在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

**优点**

- 职责清晰。能够协调调用者和被调用者，在一定程度上降低了系统的耦合度。
- 高扩展性。增加和更换代理类无须修改源代码，符合[开闭原则](#开闭原则)，系统具有较好的灵活性和可扩展性。

**缺点**

- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。 
- 实现代理模式需要额外的工作，有些代理模式的实现非常复杂

**使用场景**

- 想在访问一个类时做一些控制。

### 职责链模式

> 职责链模式：避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。职责链模式是一种对象行为型模式。



代码实现

```
public class MainTest {
    public static void main(String[] args) {
        RequestEntity test = new RequestEntity("test", 2000);

        // 指定指责链
        BeforeHandler before = new BeforeHandler("before");
        AfterHandler after = new AfterHandler("after");
        PostHandler post = new PostHandler("post");

        // 形成链状闭环
        before.setHandler(after);
        after.setHandler(post);
        post.setHandler(before);

        after.handleRequest(test);
    }

}

class RequestEntity {

    public String name;

    public Integer grade;

    public RequestEntity(String name,Integer grade) {
        this.name = name;
        this.grade = grade;
    }

    public String getName() {
        return name;
    }

    public float getGrade() {
        return grade;
    }
}

abstract class Handler {

    // 下一个引用
    protected Handler handler;

    protected String name;

    public Handler(String name) {
        this.name = name;
    }

    public void setHandler(Handler handler) {
        this.handler = handler;
    }

    public abstract void handleRequest(RequestEntity requestEntity);

}

class BeforeHandler extends Handler {

    public BeforeHandler(String name){
        super(name);
    }


    @Override
    public void handleRequest(RequestEntity requestEntity) {
        if (requestEntity.getGrade() < 1000) {
            System.out.println("分数为：" + requestEntity.getGrade() + "，被" + this.name + "处理 ");
        }else {
            handler.handleRequest(requestEntity);
        }
    }
}

class AfterHandler extends Handler {

    public AfterHandler(String name){
        super(name);
    }

    @Override
    public void handleRequest(RequestEntity requestEntity) {
        if (requestEntity.getGrade() <= 2000) {
            System.out.println("分数为：" + requestEntity.getGrade() + "，被" + this.name + "处理 ");
        }else {
            handler.handleRequest(requestEntity);
        }
    }
}

class PostHandler extends Handler {

    public PostHandler(String name){
        super(name);
    }

    @Override
    public void handleRequest(RequestEntity requestEntity) {
        if (requestEntity.getGrade() > 3000) {
            System.out.println("分数为：" + requestEntity.getGrade() + "，被" + this.name + "处理 ");
        }else {
            handler.handleRequest(requestEntity);
        }
    }
}
```

职责链模式通过建立一条链来组织请求的处理者，请求将沿着链进行传递，请求发送者无须知道请求在何时、何处以及如何被处理，实现了请求发送者与处理者的解耦。在软件开发中，如果遇到有多个对象可以处理同一请求时可以应用职责链模式，例如在Web应用开发中创建一个过滤器(Filter)链来对请求数据进行过滤，在工作流系统中实现公文的分级审批等等，使用职责链模式可以较好地解决此类问题。

**优点**

- 增加新的请求处理类很方便，只需要在客户端重新建链即可，从这一点来看是符合[开闭原则](#开放封闭原则)的。
-  为请求创建了一个接收者对象的链式结构。对请求的发送者和接收者进行解耦。
- 简化了对象。使得对象不需要知道链的结构。
- 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。

**缺点**

- 系统性能将受到一定影响，可能会造成循环调用。特别是在链比较长的时候，因此需控制链中最大节点数量，一般通过在 Handler 中设置一个最大节点数量，在` setNext()`方法中判断是否已经超过阀值，超过则不允许该链建立，避免出现超长链无意识地破坏系统性能。
- 调试不方便。采用了类似递归的方式，调试时逻辑可能比较复杂。可能不容易观察运行时的特征，有碍于除错。

**使用场景**

- 在处理消息的时候以过滤很多道。
- 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。
- 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
- 可动态指定一组对象处理请求。

### 命令模式

> 命令模式：将一个请求封装为一个对象，从而让我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作模式或事务模式。

**命令模式可以将请求发送者和接收者完全解耦，发送者与接收者之间没有直接引用关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求**。

代码实现

```
public class MainTest {
    public static void main(String[] args) {
        LightReceiver lightReceiver = new LightReceiver();

        LightOnCommand lightOnCommand = new LightOnCommand(lightReceiver);
        LightOffCommand lightOffCommand = new LightOffCommand(lightReceiver);
        RemoteCommand remoteCommand = new RemoteCommand();
        // 测试
        remoteCommand.setCommand(0,lightOnCommand,lightOffCommand);
        // 按下开灯按钮
        System.out.println("按下开灯按钮 -------");
        remoteCommand.onButtonPushed(0);
        // 按下关灯按钮
        System.out.println("按下关灯按钮 -------");
        remoteCommand.offButtonPushed(0);
        // 按下撤销按钮
        System.out.println("按下撤销按钮 ---------");
        remoteCommand.undoButtonPushed(0);
    }
}

interface Command{

    /**
     * 执行命令
     */
    void exec();

    /**
     * 撤销命令
     */
    void undo();

}

class LightReceiver{

    public void on(){
        System.out.println("开灯 ...");
    }

    public void off() {
        System.out.println("关灯 ...");
    }


}

class LightOnCommand implements Command{

    private LightReceiver lightReceiver;

    public LightOnCommand(LightReceiver lightReceiver) {
        super();
        this.lightReceiver = lightReceiver;
    }

    @Override
    public void exec() {
        lightReceiver.on();
    }

    @Override
    public void undo() {
        lightReceiver.off();
    }
}

class LightOffCommand implements Command {

    private LightReceiver lightReceiver;

    public LightOffCommand(LightReceiver lightReceiver) {
        super();
        this.lightReceiver = lightReceiver;
    }

    @Override
    public void exec() {
        lightReceiver.off();
    }

    @Override
    public void undo() {
        lightReceiver.on();
    }
}

/**
 * 空执行 默认命令实现类
 */
class NoCommand implements Command {

    @Override
    public void exec() {
        System.out.println("默认命令执行");
    }

    @Override
    public void undo() {
        System.out.println("默认撤销方法");
    }
}

class RemoteCommand {

    // 存放开关命令
    private Command onCommands[];

    private Command offCommands[];

    // 存放撤销命令
    private Command undoCommands[];

    public RemoteCommand() {
        undoCommands = new Command[5];
        onCommands = new Command[5];
        offCommands = new Command[5];

        // 默认空命令
        for (int i = 0; i < 5; i++) {
            onCommands[i] = new NoCommand();
            offCommands[i] = new NoCommand();
        }
    }

    // 设置命令
    public  void  setCommand(int no, Command onCommand, Command offCommand) {
        onCommands[no] = onCommand;
        offCommands[no] = offCommand;
    }

    // 按下开的按钮
    public void onButtonPushed(int no) {
        onCommands[no].exec();
        // 记录撤销操作
        undoCommands[no] = onCommands[no];
    }

    // 按下关闭的按钮
    public void offButtonPushed(int no) {
        offCommands[no].exec();
        // 记录撤销操作
        undoCommands[no] = offCommands[no];
    }

    // 按下撤销按钮
    public void undoButtonPushed(int no) {
        undoCommands[no].undo();
    }


}
```

命令模式是一种使用频率非常高的设计模式，它可以将请求发送者与接收者解耦，请求发送者通过命令对象来间接引用请求接收者，使得系统具有更好的灵活性和可扩展性。在基于GUI的软件开发，无论是在电脑桌面应用还是在移动应用中，命令模式都得到了广泛的应用。

在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。

**优点**

- 降低了系统耦合度。
- 新的命令可以很容易地加入到系统中。由于增加新的具体命令类不会影响到其他类，因此增加新的具体命令类很容易，无须修改原有系统源代码，甚至客户类代码，满足[开闭原则](#开放封闭原则)的要求。

**缺点**

- 使用命令模式可能会导致某些系统有过多的具体命令类。因为针对每一个对请求接收者的调用操作都需要设计一个具体命令类，因此在某些系统中可能需要提供大量的具体命令类，这将影响命令模式的使用。

**使用场景**

- 请求调用者需要与请求接收者解耦时，命令模式可以使调用者和接收者不直接交互。
- 当系统需要支持命令的撤销操作和恢复操作时。
-  系统需要在不同的时间指定请求、将请求排队和执行请求。一个命令对象和请求的初始调用者可以有不同的生命期，换言之，最初的请求发出者可能已经不在了，而命令对象本身仍然是活动的，可以通过该命令对象去调用请求接收者，而无须关心请求调用者的存在性，可以通过请求日志文件等机制来具体实现。

### 解释器模式

> 解释器模式：定义一个语言的文法，并且建立一个解释器来解释该语言中的句子，这里的“语言”是指使用规定格式和语法的代码。解释器模式是一种类行为型模式。

代码实现

```
public class MainTest {
    public static void main(String[] args) throws IOException {
        String expStr = getExpStr();
        HashMap<String, Integer> var = getValue(expStr);
        Calculator calculator = new Calculator(expStr);
        System.out.println("运算结果：" + expStr + "=" + calculator.run(var));
    }

    // 获得表达式
    public static String getExpStr() throws IOException {
        System.out.print("请输入表达式：");
        return (new BufferedReader(new InputStreamReader(System.in))).readLine();
    }

    // 获得值映射
    public static HashMap<String, Integer> getValue(String expStr) throws IOException {
        HashMap<String, Integer> map = new HashMap<>();

        for (char ch : expStr.toCharArray()) {
            if (ch != '+' && ch != '-') {
                if (!map.containsKey(String.valueOf(ch))) {
                    System.out.print("请输入" + String.valueOf(ch) + "的值：");
                    String in = (new BufferedReader(new InputStreamReader(System.in))).readLine();
                    map.put(String.valueOf(ch), Integer.valueOf(in));
                }
            }
        }
        return map;
    }

}

abstract class AbstractExpression {

    /**
     * 表达式解释器
     */
    public abstract int interpreter(HashMap<String, Integer> var);

}

/**
 * 终结表达式
 */
class VarExpression extends AbstractExpression {

    private String key;

    public VarExpression(String key) {
        this.key = key;
    }

    @Override
    public int interpreter(HashMap<String, Integer> map) {
        return map.get(key);
    }
}

/**
 * 非终结表达式
 */
class SymbolExpression extends AbstractExpression {

    protected AbstractExpression left;

    protected AbstractExpression right;

    SymbolExpression(AbstractExpression left, AbstractExpression right) {
        this.left = left;
        this.right = right;
    }


    // 因为 SymbolExpression  是让其子类来实现，因此 interpreter 是一个默认实现
    @Override
    public int interpreter(HashMap<String, Integer> var) {
        return 0;
    }

}

/**
 * 减法
 */
class SubExpression extends SymbolExpression {

    public SubExpression(AbstractExpression left, AbstractExpression right) {
        super(left, right);
    }

    @Override
    public int interpreter(HashMap<String, Integer> var) {
        return super.left.interpreter(var) - super.right.interpreter(var);
    }

}

/**
 * 加法
 */
class AddExpression extends SymbolExpression {

    public AddExpression(AbstractExpression left, AbstractExpression right) {
        super(left, right);
    }

    @Override
    public int interpreter(HashMap<String, Integer> var) {
        return super.left.interpreter(var) + super.right.interpreter(var);
    }

}

/**
 * 计算器 调用加减法
 */
class Calculator {

    private AbstractExpression expression;

    public  Calculator(AbstractExpression expression) {
        this.expression = expression;
    }

    public Calculator(String expStr) {
        // 安排运算先后顺序
        Stack<AbstractExpression> stack = new Stack<>();
        // 表达式拆分成字符数组
        char[] charArray = expStr.toCharArray();


        AbstractExpression left = null;
        AbstractExpression right = null;
        //遍历我们的字符数组，  即遍历	[a, +, b]
        //针对不同的情况，做处理
        for (int i = 0; i < charArray.length; i++) {
            switch (charArray[i]) {
                case '+':
                    // 从 stack 取 出 left => "a"
                    left = stack.pop();
                    // 取出右表达式 "b"
                    right = new VarExpression(String.valueOf(charArray[++i]));
                    stack.push(new AddExpression(left, right));
                    // 然后根据得到 left 和 right 构建 AddExpresson 加入 stack
                    break;
                case '-':
                    left = stack.pop();
                    right = new VarExpression(String.valueOf(charArray[++i]));
                    stack.push(new SubExpression(left, right));
                    break;
                default:
                    //如果是一个 Var 就创建要给 VarExpression 对象，并 push 到 stack
                    stack.push(new VarExpression(String.valueOf(charArray[i])));

                    break;
            }
        }
        //当遍历完整个 charArray  数组后，stack 就得到最后
        this.expression = stack.pop();
    }


    public int run(HashMap<String, Integer> var) {
        //最后将表达式 a+b 和 var = {a=10,b=20}
        //然后传递给 expression 的 interpreter 进行解释执行
        return this.expression.interpreter(var);
    }

}

```

解释器模式为自定义语言的设计和实现提供了一种解决方案，它用于定义一组文法规则并通过这组文法规则来解释语言中的句子。虽然解释器模式的使用频率不是特别高，但是它在正则表达式、XML文档解释等领域还是得到了广泛使用。与解释器模式类似，目前还诞生了很多基于抽象语法树的源代码处理工具，例如Eclipse中的Eclipse AST，它可以用于表示Java语言的语法结构，用户可以通过扩展其功能，创建自己的文法规则。

**优点**

- 易于改变和扩展文法。增加新的解释表达式较为方便。如果用户需要增加新的解释表达式只需要对应增加一个新的终结符表达式或非终结符表达式类，原有表达式类代码无须修改，符合“[开闭原则](#开放封闭原则)”。
- 易于实现简单文法。

**缺点**

- 可利用场景比较少。
- 对于复杂的文法比较难维护。解释器模式会引起类膨胀。如果一个语言包含太多文法规则，类的个数将会急剧增加，导致系统难以管理和维护。
-  解释器模式采用递归调用方法。

**使用场景**

- 对于一些固定文法构建一个解释句子的解释器。
- 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树
- 一些重复出现的问题可以用一种简单的语言来进行表达。
-  一个简单语法需要解释的场景。



### 迭代器模式
> 迭代器模式：提供一种方法来访问聚合对象，而不用暴露这个对象的内部表示，其别名为游标。迭代器模式是一种对象行为型模式。

迭代器模式的重要用途就是帮助我们遍历容器。迭代器模式，提供一种遍历集合元素的统一接口，用一致的方法遍历集合元素，不需要知道集合对象的底层表示，即：不暴露其内部的结构。在迭代器模式结构中包含聚合和迭代器两个层次结构，考虑到系统的灵活性和可扩展性，在迭代器模式中应用了[工厂方法模式](#工厂方法模式).

代码实现
```
public class MainTest {
    public static void main(String[] args) {
        Bread bread = new Bread();
        bread.add("面粉");
        bread.add("黄油");
        bread.add("白糖");
        bread.add("鸡蛋");
        Iterator iterator = bread.getIterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}

class FoodIterator implements Iterator {

    String[] foods;
    int      position = 0;

    public FoodIterator(String[] foods) {
        this.foods = foods;
    }

    @Override
    public boolean hasNext() {
        return position != foods.length;
    }

    @Override
    public Object next() {
        String food = foods[position];
        position += 1;
        return food;
    }

}

interface Food {

    void add(String name);

    Iterator getIterator();
}

class Bread implements Food{
    private String[] foods    = new String[4];
    private int      position = 0;

    @Override
    public void add(String name) {
        foods[position] = name;
        position += 1;
    }

    @Override
    public Iterator getIterator() {
        return new FoodIterator(this.foods);
    }
}
```
> 迭代器模式是一种使用频率非常高的设计模式，通过引入迭代器可以将数据的遍历功能从聚合对象中分离出来，聚合对象只负责存储数据，而遍历数据由迭代器来完成。由于很多编程语言的类库都已经实现了迭代器模式，因此在实际开发中，我们只需要直接使用Java、C#等语言已定义好的迭代器即可，迭代器已经成为我们操作聚合对象的基本工具之一。

迭代器的使用现在非常广泛，因为Java中提供了`java.util.Iterator`。而且Java中的很多容器（`Collection、Set`）也都提供了对迭代器的支持。

**优点**
- 提供一个统一的方法遍历对象，客户不用再考虑聚合的类型，使用一种方法就可以遍历对象了。
- 隐藏了聚合的内部结构，客户端要遍历聚合的时候只能取到迭代器，而不会知道聚合的具体组成。
- 提供了一种设计思想，就是一个类应该只有一个引起变化的原因（[单一职责原则](#单一职责原则)）。在聚合类中，我们把迭代器分开，就是要把管理对象集合和遍历对象集合的责任分开，这样一来集合改变的话，只影响到聚合对象。而如果遍历方式改变的话，只影响到了迭代器。
- 当要展示一组相似对象，或者遍历一组相同对象时使用, 适合使用迭代器模式

**缺点**
- 由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性。

**使用场景**
- 需要为一个聚合对象提供多种遍历方式。
- 使用迭代器模式可以为遍历不同的聚合结构提供一个统一的接口，接口的实现类中为不同的聚合结构提供不同的遍历方式，而客户端可以一致性地操作该接口。

### 中介者模式（未完）

### 备忘录模式（未完）

### 观察者模式（未完）

### 状态模式（未完）

### 策略模式（未完）

### 模板方法模式（未完）

### 访问者模式（未完）

