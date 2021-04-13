---
title: "Java对象"
date: 2021-04-12
draft: false
tags: ["Java", "JVM"]
slug: "java-object"
---

## 对象实例化

### 对象创建方式
- new：最常见的方式、单例类中调用`getInstance`的静态类方法，`XXXFactory`的静态方法
- `Class`的`newInstance`方法：在JDK9里面被标记为过时的方法，因为只能调用空参构造器
- `Constructor`的`newInstance(XXX)`：反射的方式，可以调用空参的，或者带参的构造器
- 使用`clone()`：不调用任何的构造器，要求当前的类需要实现`Cloneable`接口中的`clone`接口
- 使用序列化：序列化一般用于`Socket`的网络传输
- 第三方库 `Objenesis`