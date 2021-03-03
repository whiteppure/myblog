---
title: "Java中的数组"
date: 2020-10-04
draft: false
tags: ["Java", "数组"]
slug: "java-array"
---

> 数组就是同类数据元素的集合

## 易错:
- 数组是Java的引用数据类型
- 数组也是对象,能够向下或者向上转型,能使用`instanceof`关键字
```
int[] a = new int[8];  
Object obj = a ; //数组的父类也是Object,可以将a向上转型到Object  

int[] b = (int[])obj;  //可以进行向下转型 

if(obj instanceof int[]){ 
    //可以用instanceof关键字进行类型判定 
}
```

## 优缺点

数组优点:
- 通过下标访问元素的效率很高，指定下标为n的元素的地址(和链表相比较)
- 数组可以保存若干个元素的值

数组缺点:
- 数组长度是固定的不能变的
- 数组进行元素的删除和插入操作的时候，效率比较低,需要移动大量的元素(这里可以举例: ArrayList 和LinkedList区别)
- 数组元素的类型只能是一种
- 数组元素的内存地址是连续分配的,对内存要求高一些;(相对于链表结构比较,链表的内存是连续不连续都可以)
- 数组没有提供任何的封装，所有对元素的操作，都是通过自定义的方法实现的，对数组元素的操作比较麻烦



## 常见处理数组的操作
### 遍历数组
````
 for (int i = 0; i < array1.length; i++) {
       System.out.println(array1[i]);
}
````

### 数组去重
````
//利用 hashSet 集合去重
Set<Integer> set2 = new HashSet<Integer>();
for (int i = 0; i < arr11.length; i++) {
    set2.add(arr11[i]);
}
````

### 数组与集合转换
```
Set<String> set=new HashSet<String>(Arrays.asList(array2));//将数组转成set集合
//将数组转化成 list  集合
String[] array={"1","2","3"};
List<String> list=new ArrayList<String>();
for (int i = 0; i < array.length; i++) {
    list.add(array2[i]);
}
//第二种 数组 转 list 
List<String > list2=java.util.Arrays.asList(array);
```

### 数组排序
```
 //原生方法
 Arrays.sort(arr);
 // 或8种排序算法
```

### 复制数组
```
int[] arr = {1, 2, 3, 4}; //待复制的数组
int[] arr2 = Arrays.copyOf(arr, 10);  //指定新数组的长度
int[] arr3 = Arrays.copyOfRange(arr, 1, 3); //只复制从索引[1]到索引[3]之间的元素（不包括索引[3]的元素）
for (int i = 0; i < arr3.length; i++) {
    System.out.println(arr3[i]);
}
```


## 相关延伸:
- `void  method_name(int ... value)`变参就是当数组处理的，优先使用定参的，编译后就是数组
- 一个方法只能有一个变参!即使是不同的类型也不行。
- 变参参数只能在形参列表的末尾，如果传入的是数组，则只能传一个