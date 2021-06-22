---
title: "CPU过高定位分析"
date: 2021-06-21
draft: false
tags: ["linux"]
slug: "cpu-usage-too-high"
---

## CPU使用过高定位分析
一般在生产环境排查程序故障，都会查看日志什么的，但是有些故障日志是看不出来的，就比如：CPU使用过高。

那应该怎么办呢？我们需要结合linux命令和JDK相关命令来排查程序故障。

### 第一步
首先使用top命令，找出CPU占比最高的Java进程；然后进一步定位后台程序，可以使用`jps -l`命令或者`ps -ef java`命令。
![CPU使用过高-1](/myblog/posts/images/essays/CPU使用过高-1.png)

### 第二步
定位到具体的线程；使用`ps -mp 进程ID -o THREAD,tid,time`命令可以找到线程ID。
> ps -mp 进程ID -o THREAD,tid,time 说明：
>-m：显示所有线程
>-p：pid进程使用CPU的时间
>-o：该参数后是用户自定义参数

![CPU使用过高-2](/myblog/posts/images/essays/CPU使用过高-2.png)

### 第三步
获取到线程ID后，将线程ID转化为16进制格式，如果有英文要小写格式；可以用命令`printf "%x\n" 线程ID`,当然也可以使用工具从10进制转16进制。
```
printf "%x\n" 16 // 10
```

### 第四步
线程ID转成16进制后，执行最后一个命令：`jstack 进程ID | grep 16进制线程ID -A50`,就能看到有问题的代码。
![CPU使用过高-3](/myblog/posts/images/essays/CPU使用过高-3.png)


