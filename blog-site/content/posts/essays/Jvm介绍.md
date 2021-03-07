---
title: "Jvm介绍"
date: 2021-03-05
draft: false
tags: ["Java","Jvm", "使用介绍"]
slug: "jvm-start"
---

## 为什么要学习Jvm
大部分Java开发人员，除了会在项目中使用到与Java平台相关的各种高精尖技术，对于Java技术的核心Java虚拟机了解甚少。
一些有一定工作经验的开发人员，打心眼儿里觉得SSM、微服务等上层技术才是重点，基础技术并不重要，
这其实是一种本末倒置的“病态”。
如果我们把核心类库的API比做数学公式的话，那么Java虚拟机的知识就好比公式的推导过程。

> 不要以钱为你的最终目标，当你把技术做到位的时候，你会发现你的薪资待遇、社会地位，都会自然而然的提升上来。不要本末倒置、急功近利。

- 面试的需要(BATJ、TMD，PKQ等面试都爱问)
- 中高级程序员必备技能：项目管理、调优的需求
- 追求极客的精神，比如：垃圾回收算法、JIT（即时编译器）、底层原理

垃圾收集机制为我们打理了很多繁琐的工作，大大提高了开发的效率，
但是，垃圾收集也不是万能的，懂得JVM内部的内存结构、工作机制，是设计高扩展性应用和诊断运行时问题的基础，也是Java工程师进阶的必备能力。

## 虚拟机概念

所谓虚拟机（Virtual Machine），就是一台虚拟的计算机。它是一款软件，用来执行一系列虚拟计算机指令。
大体上，虚拟机可以分为系统虚拟机和程序虚拟机。

- 大名鼎鼎的Virtual Box，VMware就属于系统虚拟机，它们完全是对物理计算机的仿真，提供了一个可运行完整操作系统的软件平台。
- 程序虚拟机的典型代表就是Java虚拟机，它专门为执行单个计算机程序而设计，在Java虚拟机中执行的指令我们称为Java字节码指令。
- 无论是系统虚拟机还是程序虚拟机，在上面运行的软件都被限制于虚拟机提供的资源中.

### Java虚拟机
- Java虚拟机是一台执行Java字节码的虚拟计算机，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。
- JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回器，以及可靠的即时编译器。
- Java技术的核心就是Java虚拟机（JVM，Java Virtual Machine），因为所有的Java程序都运行在Java虚拟机内部。
- Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。

## Jvm整体结构
HotSpot VM是目前市面上高性能虚拟机的代表作之一。下图就是 HotSport 虚拟机结构图
![Jvm内存模型](/myblog/posts/images/essays/Jvm内存模型.png)

- 类加载子系统：将字节码文件加载到内存当中生成一个class文件。具体分为三部分：装载，链接，初始化。
- 运行时数据区：方法区、堆、Java栈（虚拟机栈）、本地方法栈、程序计数器。其中线程共享，堆和方法区；Java栈、本地方法栈和程序计数器为每个线程独有一份的。
- 执行引擎：包括：解释器、JIT及时编译器、GC垃圾回收器。

## Java代码执行流程
Java代码通过编译器生成字节码文件，字节码通过Java虚拟机跟操作系统交互。
![Java代码执行流程](/myblog/posts/images/essays/Java代码执行流程.png)

## Jvm架构模型
Java编译器输入的指令流基本上是一种 **基于栈的指令集架构** ，另外一种指令集架构则是 **基于寄存器的指令集架构** 。
Hotsport 是基于栈的指令集架构。

**基于栈式架构的特点**
- 设计和实现更简单，适用于资源受限的系统
- 避开了寄存器的分配难题：使用零地址指令方式分配
- 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译器容易实现
- 不需要硬件支持，可移植性更好，更好实现跨平台

**基于寄存器架构的特点**
- 典型的应用是x86的二进制指令集：比如传统的PC以及Android的Davlik虚拟机。
- 指令集架构则完全依赖硬件，与硬件的耦合度高，可移植性差
- 性能优秀和执行更高效
- 花费更少的指令去完成一项操作
- 在大部分情况下，基于寄存器架构的指令集往往都以一地址指令、二地址指令和三地址指令为主，而基于栈式架构的指令集却是以零地址指令为主

## Jvm生命周期

启动 --> 执行 --> 退出

**虚拟机的启动**
- Java虚拟机的启动是通过引导类加载器（bootstrap class loader）创建一个初始类（initial class）来完成的，这个类是由虚拟机的具体实现指定的。

**虚拟机的执行**
- 一个运行中的Java虚拟机有着一个清晰的任务：执行Java程序
- 程序开始执行时他才运行，程序结束时他就停止
- 执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程

**虚拟机的退出**
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统用现错误而导致Java虚拟机进程终止
- 某线程调用Runtime类或System类的exit()方法，或Runtime类的halt()方法，并且Java安全管理器也允许这次exit()或halt()操作。
- 除此之外，JNI（Java Native Interface）规范描述了用JNI Invocation API来加载或卸载 Java虚拟机时，Java虚拟机的退出情况。

## Jvm发展历程
Hotspot VM、JRockit、J9 是目前主要流行的Java虚拟机。所有虚拟机的原则：一次编译，到处运行。

具体JVM的内存结构，其实取决于其实现，不同厂商的JVM，或者同一厂商发布的不同版本，都有可能存在一定差异。
主要以oracle HotSpot VM为默认虚拟机。

### Sun Classic VM
早在1996 Java1.0 的时候，Sun 公司发布了了第一款名为 Sun Classic VM 的Java虚拟机，他同时也是**世界上第一款**商用的Java虚拟机，
JDK1.4 的时候完全被淘汰。这款虚拟机只提供解释器。

> Java虚拟机分为两类执行引擎
> - 解释型：一行一行执行代码，执行效率慢，类似于javascript、python这类解释型的编程语言
> - 及时编译型：将字节码中的热点代码编译成机器码，并且将机器码缓存到方法区的代码缓存区。

这款虚拟机只能使用纯解释器方式来执行Java代码，如果要使用即时编译器那 就必须进行外挂，
但是假如外挂了即时编译器的话，即时编译器就会完全接管虚拟机的执行系统，解释器便不能再工作了。
因此这个阶段的虚拟机 虽然用了即时编译器输出本地代码，
其执行效率也和传统的 C/C++ 程序有很大差距，“Java语言很慢”的 印象就是在这阶段开始在用户心中树立起来的。

Hotspot 虚拟机内置了 Sun Classic 虚拟机。

### Exact VM
Sun的虚拟机团队努力去解决Classic虚拟机所面临的各种问题，提升运行效率，
在JDK 1.2时，曾在 Solaris 平台上发布过一款名为 Exact VM 的虚拟机，
它的编译执行系统已经具备现代高性能虚拟机雏形，如热点探测、两级即时编译器、编译器与解释器混合工作模式等。

### Hotspot VM
Hotspot VM 是目前使用范围最广的Java虚拟机。Hotspot 并非由Sun公司开发，而是由一家名为“Longview Technologies”的小公司设计的；

HotSpot VM既继承了Sun之前两款商用虚拟机的优点（如前面提到的准确式内存管理），也有许多自己新的技术优势， 
如它名称中的HotSpot指的就是它的热点代码探测技术,HotSpot VM的热点代码探测能力可以通过执行计数器找出最具有编译价值的代码，然后通知JIT编译器以方法为单位进行编译。
如果一个方法被频繁调用，或方法中有效循环次数很多，将会分别触发标准编译和OSR（栈上替换）编译动作。
通过编译器与解释器恰当地协同工作，可以在最优化的程序响应时间与最佳执行性能中取得平衡，而且无须等待本地代码输出才能执行程序， 即时编译的时间压力也相对减小，
这样有助于引入更多的代码优化技术，输出质量更高的本地代码。

### JRockit
JRockit 专注服务端的应用，内部不包含解析器的实现，全部代码都靠即时编译器编译后执行。

> 大量的行业基准测试显示，**JRockit JVM是世界上最快的JVM。** JRockit面向延迟敏感型应用的解决方案JRockit Real Time提供以毫秒或微秒计的JVM响应时间，
适合财务前端办公、军事指挥与控制和电信网络的需要。使用JRockit产品，客户已经体验到了显著的性能提高（一些超过了70% ）和硬件成本的减少（达50%）。

全面的Java运行时解决方案
- JRockit 面向延迟敏感性应用的解决方案，JRockit Real Time 提供以毫秒或微秒级的JVM响应时间，适合财务、军事指挥、电信网络的需要。
- MissionControl 服务套件，它是一组以模板低的开销来监控、管理和分析生产环境中的应用程序的工具。

### J9 VM
IBM的J9全称：IBM Technology for Java Virtual Machine，简称IT4J，内部代号J9。

J9的市场定位与HotSpot接近，服务器端、桌面应用、嵌入式等多用途VM。

J9是目前由影响力的三大商业虚拟机之一，2017年IBM发布了开源J9 VM，命名为OpenJ9，交给Eclipse基金会管理，也称Eclipse OpenJ9。

> IBM J9直至今天仍旧非常活跃，IBM J9虚拟机的职责分离与模块化做得比HotSpot更优秀，
由J9 虚拟机中抽象封装出来的核心组件库（包括垃圾收集器、即时编译器、诊断监控子系统等）就单独构 成了IBM OMR项目，
可以在其他语言平台如Ruby、Python中快速组装成相应的功能。从2016年起， IBM逐步将OMR项目和J9虚拟机进行开源，
完全开源后便将它们捐献给了Eclipse基金会管理，并重新 命名为Eclipse OMR和OpenJ9。
如果为了学习虚拟机技术而去阅读源码，更加模块化的OpenJ9代码 其实是比HotSpot更好的选择。
如果为了使用Java虚拟机时多一种选择，那可以通过AdoptOpenJDK来 获得采用OpenJ9搭配上OpenJDK其他类库组成的完整JDK。

### KVM和CDC/CLDC Hotspot VM
Oracle 在Java Me 产品线上的两款虚拟机： CDC/CLDC Hotspot VM。目前移动领域地位的尴尬，智能手机被IOS和android二分天下。

KVM 简单、轻量，高度可移植性，在面向更低端的设备上还维持着自己的一片市场。
- 只能控制器、传感器。
- 老年手机、经济欠发达地区简单的功能手机。

### Azul VM
前面三大“高性能Java虚拟机”使用在通用硬件平台上这里Azu1VW和BEALiquid VM是与特定硬件平台绑定、软硬件配合的专有虚拟机I

高性能Java虚拟机中的战斗机。
Azul VM是Azu1Systems公司在HotSpot基础上进行大量改进，运行于Azul Systems公司的专有硬件Vega系统上的ava虚拟机。

每个Azu1VM实例都可以管理至少数十个CPU和数百GB内存的硬件资源，并提供在巨大内存范围内实现可控的GC时间的垃圾收集器、专有硬件优化的线程调度等优秀特性。

2010年，AzulSystems公司开始从硬件转向软件，发布了自己的zing JVM，可以在通用x86平台上提供接近于Vega系统的特性。

### Apache Marmony
Apache也曾经推出过与JDK1.5和JDK1.6兼容的Java运行平台Apache Harmony。

它是IElf和Inte1联合开发的开源JVM，受到同样开源的openJDK的压制，Sun坚决不让Harmony获得JCP认证，最终于2011年退役，IBM转而参与OpenJDK

虽然目前并没有Apache Harmony被大规模商用的案例，但是它的Java类库代码吸纳进了Android SDK。

### Micorsoft JVM
微软为了在IE3浏览器中支持Java Applets，开发了Microsoft JVM。

只能在window平台下运行。但确是当时Windows下性能最好的Java VM。

1997年，sun以侵犯商标、不正当竞争罪名指控微软成功，赔了sun很多钱。微软windowsXPSP3中抹掉了其VM。
现在windows上安装的jdk都是HotSpot.

### Taobao JVM
由AliJVM团队发布。阿里，国内使用Java最强大的公司，覆盖云计算、金融、物流、电商等众多领域，需要解决高并发、高可用、分布式的复合问题。有大量的开源产品。

基于openJDK开发了自己的定制版本AlibabaJDK，简称AJDK。是整个阿里Java体系的基石。

基于openJDK Hotspot VM发布的国内第一个优化、深度定制且开源的高性能服务器版Java虚拟机。

创新的GCIH（GCinvisible heap）技术实现了off-heap，即将生命周期较长的Java对象从heap中移到heap之外，并且Gc不能管理GCIH内部的Java对象，以此达到降低GC的回收频率和提升Gc的回收效率的目的。
GCIH中的对象还能够在多个Java虚拟机进程中实现共享
使用crc32指令实现JvM intrinsic降低JNI的调用开销
PMU hardware的Java profiling tool和诊断协助功能
针对大数据场景的ZenGc
taobao vm应用在阿里产品上性能高，硬件严重依赖inte1的cpu，损失了兼容性，但提高了性能

目前已经在淘宝、天猫上线，把oracle官方JvM版本全部替换了。

### Dalvik VM
谷歌开发的，应用于Android系统，并在Android2.2中提供了JIT，发展迅猛。

Dalvik y只能称作虚拟机，而不能称作“Java虚拟机”，它没有遵循 Java虚拟机规范

不能直接执行Java的Class文件

基于寄存器架构，不是jvm的栈架构。

执行的是编译以后的dex（Dalvik Executable）文件。执行效率比较高。

它执行的dex（Dalvik Executable）文件可以通过class文件转化而来，使用Java语法编写应用程序，可以直接使用大部分的Java API等。
Android 5.0使用支持提前编译（Ahead of Time Compilation，AoT）的ART VM替换Dalvik VM。

### Graal VM
2018年4月，oracle Labs公开了GraalvM，号称 "Run Programs Faster Anywhere"，勃勃野心。
与1995年Java的”write once，run anywhere"遥相呼应。

GraalVM在HotSpot VM基础上增强而成的跨语言全栈虚拟机，可以作为“任何语言” 的运行平台使用。语言包括：Java、Scala、Groovy、Kotlin；C、C++、Javascript、Ruby、Python、R等

支持不同语言中混用对方的接口和对象，支持这些语言使用已经编写好的本地库文件

工作原理是将这些语言的源代码或源代码编译后的中间格式，通过解释器转换为能被Graal VM接受的中间表示。Graal VM提供Truffle工具集快速构建面向一种新语言的解释器。在运行时还能进行即时编译优化，获得比原生编译器更优秀的执行效率。

**如果说HotSpot有一天真的被取代，Graalvm希望最大。** 但是Java的软件生态没有丝毫变化。