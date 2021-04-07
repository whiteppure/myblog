---
title: "Java异常"
date: 2021-01-13
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-exception"
---
![异常体系](/myblog/posts/images/essays/异常体系.png)
## 异常类型

`Throwable` 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。

其中 Error 用来表示 Java程序 无法处理的错误；这类错误一般与硬件有关，与程序本身无关，通常由系统进行处理，程序本身无法捕获和处理。是不可控制的。

Exception 分为两种：运行时异常和检查型异常。

- 受检异常 ：需要用`` try...catch... ``语句捕获并进行处理，并且可以从异常中恢复；
  ```
  public void test() throw new Exception{ }
  ```
  Java编译器对检查性异常会要求我们进行catch，必须得进行捕获，否则编译不过去。Java认为检查性异常都可以被处理，所以必须显示的处理 checked 异常。
  常见的检查性异常有IOException，SqlException。
  当我们希望我们的⽅法调⽤者， 明确的处理⼀些特殊情况的时候， 就应该使⽤受检异常。

  
- 非受检异常 ：是程序运行时错误。例如：除 0 会引发 `ArithmeticException` ，此时程序崩溃并且无法恢复。
    ```
    public void test() {
        int a = 1;
        int b = a/0;
    }
    ```

    这种异常⼀般可以理解为是代码原因导致的。 
    ⽐如发⽣空指针、 数组越界等。 所以， 只要代码写的没问题， 这些异常都是可以避免的。
    也就不需要我们显⽰的进⾏处理.

Exception 表⽰程序需要捕捉、 需要处理的常， 是由与程序设计的不完善⽽出现的问题， 程序必须处理的问题。

## 自定义异常

- 继承`Exception`
- 大多数情况下都会继承`RuntimeException`

```
public class MainTest {

    void context() throws TestException {
        throw new TestException();
    }
    
    void  context2() {
        throw new Test2Exception();
    }
}
// TestException 为受检异常 程序必须要处理否则编译不通过
class TestException extends Exception {
}

// Test2Exception为运行时异常 可以不进行处理
class Test2Exception extends RuntimeException{
}
```

## 异常链

**异常链 是Java中⾮常流⾏的异常处理概念， 是指在进⾏⼀个异常处理时抛出了另外⼀个异常， 由此产⽣了⼀个异常链条。**

如果因为因为异常你决定抛出⼀个新的异常， 你⼀定要包含原有的异常， 这样， 处理程序才可以通过`getCause()`和`initCause()`⽅法来访问异常最终的根源。

```
    public static void main(String[] args) {
        try {
            int a = 1;
            int b = a/0;
        } catch (Exception e) {
            throw new RuntimeException("Exception", e);
        }
    }
```

在此示例中，当捕获到`Exception`时，将创建一个新的`RuntimeException`异常，并附加原始的异常原因，并将异常链抛出到下一个更高级别的异常处理程序。

## 常见的异常

### 空指针异常
`java.lang.NullPointerException`

这个异常经常遇到，异常的原因是程序中有空指针，即程序中调用了未经初始化的对象或者是不存在的对象。
经常出现在创建对象，调用数组这些代码中，比如对象未经初始化，或者图片创建时的路径错误等等。对数组代码
中出现空指针，是把数组的初始化和数组元素的初始化搞混淆了。数组的初始化是对数组分配空间，而数组元素的
初始化，是给数组中的元素赋初始值


### 下标越界异常
`java.lang.IndexOutOfBoundsException`

查看程序中调用的数组或者字符串的下标值是不是超出了数组的范围，一般来说，显示调用数组不太容易出这
样的错，但隐式调用就有可能出错了，还有一种情况，是程序中定义的数组的长度是通过某些特定方法决定的，不是
事先声明的，这个时候可以先查看一下数组的length，以免出现这个异常


## 处理异常

异常的处理⽅式有两种。
1. ⾃⼰处理。
2. 向上抛， 交给调⽤者处理。

不要丢弃异常，捕获异常后需要进行相关处理。如果用户觉得不能很好地处理该异常，就让它继续传播，传到别的地方去处理，或者把一个低级的异常转换成应用级的异常，重新抛出。
异常， 千万不能捕获了之后什么也不做。 或者只是使⽤`e.printStacktrace`。

catch语句应该指定具体的异常类型。不能把不该捕获的异常也捕获了;
在finally里面释放资源。如果finally里面也会抛出异常，也一样需要使用`try..catch`处理


### try和catch、finally

一个try对应多个catch,进行多重捕获。

- 可以在 try 语句后面添加任意数量的 catch 块。
- 如果保护代码中发生异常，异常被抛给第一个 catch 块。
- 如果抛出异常的数据类型与 ExceptionType1 匹配，它在这里就会被捕获。
- 如果不匹配，它会被传递给第二个 catch 块。
- 如此，直到异常被捕获或者通过所有的 catch 块。

```
try{
   // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}catch(异常类型2 异常的变量名2){
  // 程序代码
}catch(异常类型3 异常的变量名3){
  // 程序代码
}
```

```
try{
  // 程序代码
} catch (异常类型1 | 异常类型2 | 异常类型3 e) {
  // 程序代码
}
```


根据JVM规范: 
- 如果 try 语句块里边有返回值则返回 try 语句块里边的;
- 如果 try 语句块和 finally 语句块都有return,则忽略 try 语句块里边的使用 finally 语句块里边的return; 
- finally 语句块是在 try 语句块或者 catch 语句块中的 return 语句之前执行的;
- 无论是否发生异常，finally 代码块中的代码总会被执行;

```
try{
  // 程序代码
  // 该情况下不执行该代码，执行finally块中的代码
  return a;
}catch(异常类型2 异常的变量名2){
  // 程序代码
  // 如果存在异常则执行，该情况下无论是否有异常也不执行
  return b;
}finally{
  // 程序代码
  return c;
}
```

**finally语句块什么时候不执行**

如果当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed）或退出虚拟机(`System.exit(0)`)，与其相对应的 finally 语句块可能不会执行。还有更极端的情况，就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。

**JVM先会把try或者catch代码块中的返回值保留，再来执行finally代码块中的语句，等到finally代码块执行完毕之后，在把之前保留的返回值给返回出去。  这条规则（保留返回值），只适用于 return和throw语句，不适用于break和continue语句，因为它们根本就没有返回值。**

``` 
public class MyTest {
 
	public static void main(String[] args) {
        // main 代码块中的执行结果为：1
		System.out.println("main 代码块中的执行结果为：" + myMethod());
	}
 
	public static int myMethod() {
		int i = 1;
		try {
			System.out.println("try 代码块被执行！");
			return i;
		} finally {
			++i;
			System.out.println("finally 代码块被执行！");
			System.out.println("finally 代码块中的i = " + i);
		}
 
	}
 
}
```



try后边不是必须跟catch{},可以跟finally{}

```
try{
  // 程序代码
} finally{
  // 程序代码
}
```

### throws、throw

- `throws` 用在方法上声明异常,子类继承的时候要继承该异常或者该异常的子类,不处理异常,谁调用该方法谁处理异常, 语法(可以抛出多个异常):

```
public static void main(String[] args) throws Exception {}
```

- `throw`是语句抛出一个异常 new 异常对象 例如:

```
throw new RuntimeException("这是运行中的异常");
```


// .. trycatch 语法糖 处理异常总结 
