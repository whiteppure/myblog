---
title: "Java异常"
date: 2021-01-13
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-exception"
---

![Java异常分类](/myblog/posts/images/essays/Java异常分类.png)
## 异常类型

`Throwable` 可以用来表示任何可以作为异常抛出的类，分为两种：`Error` 和 `Exception`。

其中 `Error` 用来表示Java程序无法处理的错误；这类错误一般与硬件有关，与程序本身无关，通常由系统进行处理，程序本身无法捕获和处理。是不可控制的。

`Exception` 分为两种：运行时异常和检查型异常。

- 受检异常 ：需要用`` try...catch... ``语句捕获并进行处理，并且可以从异常中恢复；
  ```
  public void test() throws MyException{}
  ```
  Java编译器对检查性异常会要求我们进行`catch`，必须得进行捕获，否则编译不过去。Java认为检查性异常都可以被处理，所以必须显示的处理 `checked` 异常。
  常见的检查性异常有`IOException`，`SqlException`。
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
    也就不需要我们显⽰的进⾏处理。

`Exception` 表⽰程序需要捕捉、 需要处理的常，是由与程序设计的不完善⽽出现的问题，程序必须处理的问题。
>异常和错误的区别：异常(Exception)能被程序本身可以处理，错误(Error)是无法处理。

## 自定义异常
在 Java 中可以自定义异常。编写自己的异常类时需要注意：
- 所有异常都必须是 `Throwable` 的子类。
- 如果希望写一个**检查性异常**类，则需要继承 `Exception` 类。
- 如果你想写一个**运行时异常**类，那么需要继承 `RuntimeException` 类。

大多数情况下都会继承`RuntimeException`,自定义运行时异常。

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
>异常链是Java中⾮常流⾏的异常处理概念， 是指在进⾏⼀个异常处理时抛出了另外⼀个异常， 由此产⽣了⼀个异常链条。

如果因为因为异常你决定抛出⼀个新的异常， 你⼀定要包含原有的异常， 这样， 处理程序才可以通过`getCause()`和`initCause()`⽅法来访问异常最终的根源。

```
public class MainTest {
    public static void main(String[] args) {
        try {
            int a = 1;
            int b = a / 0;
        } catch (ArithmeticException e) {
            throw new RuntimeException("除以零异常", e);
        }
    }
}
```
在此示例中，当捕获到`ArithmeticException`时，将创建一个新的`RuntimeException`异常，并附加原始的异常原因，并将异常链抛出到下一个更高级别的异常处理程序。

## 处理异常

异常的处理⽅式有两种:
1. ⾃⼰处理。
2. 向上抛， 交给调⽤者处理。

不要丢弃异常，捕获异常后需要进行相关处理。如果用户觉得不能很好地处理该异常，就让它继续传播，传到别的地方去处理，或者把一个低级的异常转换成应用级的异常，重新抛出。
千万不能捕获了之后什么也不做。 或者只是使⽤`e.printStacktrace`。如果是练习这样写也就算了，但是在正式的环境上不能这样做。正式环境请使用日志记录。

>写完代码后请一定要检查下，代码中千万不要有printStackTrace()。因为printStackTrace()只会在控制台上输出错误的堆栈信息，他只适合于用来代码调试。

`catch`语句应该指定具体的异常类型。不能把不该捕获的异常也捕获了;如果`finally`里面也会抛出异常，也一样需要使用`try..catch`处理。

### try..catch..

在Java中如果需要处理异常，必须先对异常通过`try..catch..`进行捕获，然后再对异常情况进行处理。
```
try{
   // 程序代码
}catch(异常类型1 异常的变量名1){
  // 程序代码
}
```

一个 `try` 对应多个 `catch`,进行多重捕获。

可以在 `try` 语句后面添加任意数量的 `catch` 块。如果发生异常，异常被抛给第一个 `catch` 块。如果不匹配，它会被传递给第二个 `catch` 块。
如此，直到异常被捕获或者通过所有的 `catch` 块。

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
在JDK7之后，可以将 `catch` 语句块折叠,这意味着可以将多个不同类型的异常合并处理。
```
try{
  // 程序代码
} catch (异常类型1 | 异常类型2 | 异常类型3 e) {
  // 程序代码
}
```

### finally
`try..catch..`通常连用用来捕获异常；`try..catch..finally..`也可以连用。

```
try{
  // 程序代码
  return a;
}catch(异常类型2 异常的变量名2){
  // 程序代码
  return b;
}finally{
  // 程序代码
  return c;
}
```

对于`try..catch..finally..`语句块中执行顺序的解释

根据JVM规范: 
- 如果 `try` 语句块里边有返回值则返回 `try` 语句块里边的;
- 如果 `try` 语句块和 `finally` 语句块都有 `return`,则忽略 `try` 语句块里边的使用 `finally` 语句块里边的 `return`; 
- `finally` 语句块是在 `try` 语句块或者 `catch` 语句块中的 `return` 语句之前执行的;
- 无论是否发生异常，`finally` 代码块中的代码总会被执行;

如果方法有返回值，切忌不要再 `finally` 中使用 `return`，这样会使得程序结构变得混乱。

> finally语句块什么时候不执行 ？
>
>如果当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed）或退出虚拟机(`System.exit(0)`)，与其相对应的 finally 语句块可能不会执行。
还有更极端的情况，就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。


JVM先会把`try`或者 `catch`代码块中的返回值保留，再来执行 `finally` 代码块中的语句，等到 `finally` 代码块执行完毕之后，在把之前保留的返回值给返回出去。
这条规则（保留返回值），只适用于 `return` 和 `throw` 语句，不适用于 `break` 和 `continue` 语句，因为它们根本就没有返回值。

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

`try` 语句块不止可以与 `catch` 连用，也可以与 `finally` 连用，但是 `catch` 不能与 `finally` 连用。

```
try{
  // 程序代码
} finally{
  // 程序代码
}
```

### try-with-resources
由于`..finally..`语句块中的代码一般情况下一定会执行，所以经常用来关闭资源。

代码演示
```
public class MainTest {
    public static void main(String[] args) {
        InputStream in = null;
        try {
            in = new FileInputStream("awsl");
            in.read();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

自从JDK7之后，支持`try-with-resources`的写法,这种写法对比之前更清晰、明了：
```
public class MainTest {
    public static void main(String[] args) {
        try (InputStream in = new FileInputStream("awsl")) {
            in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
这种写法其实是Java语法糖。
> Java语法糖
>
> 1.什么是语法糖?
  语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·兰丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。
>
> 2.能够带来的好处
 语法糖让程序更加简洁，有更高的可读性
>
> 3.有哪些语法糖?
>    1.自动拆箱、装箱
     2.泛型擦除
     3.不定长参数
     4.迭代器
     5.枚举
     6.switch支持枚举和字符串
     7.内部类
     8.try-with-resources
     9.lambda


### throws、throw
处理异常的方法，除了可以捕获异常，也可以将其丢给调用者进行处理。

- `throws` 用在方法上声明异常,子类继承的时候要继承该异常或者该异常的子类,不处理异常,谁调用该方法谁处理异常；
`throws`抛出异常时，它的调用者也要申明抛出异常或者捕获，不然编译报错。
    ```
    public static void main(String[] args) throws Exception {}
    ```
- `throw`用于方法内部，抛出的是异常对象。调用者可以不申明或不捕获（这是非常不负责任的方式）但编译器不会报错。
    ```
    throw new RuntimeException("这是运行中的异常");
    ```

`throws`表示出现异常的一种可能性，告诉调用者这个方法是危险的，并不一定会发生这些异常；`throw`则是抛出了异常，执行`throw`则一定抛出了某种异常对象。
两者都是消极处理异常的方式（这里的消极并不是说这种方式不好），只是抛出或者可能抛出异常，但是不会由方法去处理异常，真正的处理异常由此方法的上层调用处理。

## 经验总结
参考文章：
- https://www.hollischuang.com/archives/5528
- https://stackify.com/best-practices-exceptions-java

### 不要滥用异常
要谨慎地使用异常，异常捕获的代价非常高昂，异常使用过多会严重影响程序的性能。
如果在程序中能够用if语句和`boolean`变量来进行逻辑判断，那么尽量减少异常的使用，从而避免不必要的异常捕获和处理。

### 空catch
千万不要使用空的`catch`块:
```
try{
 // ...
}catch(IOException e){
  // ...
}
```
在捕获了异常之后什么都不做，相当于忽略了这个异常。空的`catch`块意味着你在程序中隐藏了错误和异常，并且很可能导致程序出现不可控的执行结果。
如果你非常肯定捕获到的异常不会以任何方式对程序造成影响，最好用日志将该异常进行记录，以便日后方便更新和维护。

### 吞掉异常
请不要在`catch`块中吞掉异常:
```
catch (NoSuchMethodException e) {
   return null;
}
```
不要不处理异常，而返回`null`，这样异常就会被吞掉，无法获取到任何失败信息，会给日后的问题排查带来巨大困难。

### 精确处理异常
```
public void foo() throws Exception { //错误做法
}
```
一定要尽量避免上面的代码，因为他的调用者完全不知道错误的原因到底是什么。

在方法声明中，可以由方法抛出一些特定受检异常。如果有多个，那就分别抛出多个，这样这个方法的使用者才会分别针对每个异常做特定的处理，从而避免发生故障。
```
public void foo() throws SpecificException1, SpecificException2 { 
//正确做法
}
```

同样的在捕获异常时，也要注意，尽量捕获特定的子类，而不是直接捕获`Exception`类。
```
try {
    someMethod();
} 
catch (Exception e) {
    log.error("method has failed", e);
}
```
上面代码，最大的问题就是，如果`someMethod()`的开发者在里面新增了一个特定的异常，并且预期是调用方能够特殊的对他进行处理。

但是调用者直接`catch了Exception`类，就会导致永远无法知道`someMethod`的具体变化细节。这久可能导致在运行的过程中在某一个时间点程序崩溃。

更不要去捕获`Throwable`类。因为Java中的`Error`也可以是`Throwable`的子类。但是`Error`是Java虚拟机本身无法控制的。Java虚拟机甚至可能不会在出现任何错误时请求用户的`catch`子句。
```
try {
    someMethod();
} 
catch (Throwable e) {
    log.error("method has failed", e);
}
```
`OutOfMemoryError`和`StackOverflowError`便是典型的例子，它们都是由于一些超出应用处理范围的情况导致的。

### 抛出异常
通常情况下，在捕获异常的时候抛出异常，需要注意的是，要始终在自定义异常中，覆盖原有的异常，从而构成一条异常链，这样堆栈跟踪就不会丢失：
```
catch (NoSuchMethodException e) {
    throw new MyServiceException("Some information: " + e.getMessage());  //错误做法
}
```
上面的命令可能会丢失掉主异常的堆栈跟踪。正确的方法是:
```
catch (NoSuchMethodException e) {
     throw new MyServiceException("Some information: " , e);  //正确做法
}
```
需要注意的是，可以记录异常或抛出异常，但不要同时做：
```
catch (NoSuchMethodException e) {
   log.error("Some information", e);
   throw e;
}
```
抛出和日志记录可能会在日志文件中产生多个日志消息,这就会导致同一个问题，却在日志中有很多不同的错误信息，使得开发人员陷入混乱。


### 选择异常
一旦你决定抛出异常，你就要决定抛出抛出检查异常还是非检查异常。

检查异常导致了太多的`try…catch`代码，可能有很多检查异常对开发人员来说是无法合理地进行处理的，比如:`SQLException`，而开发人员却不得不去进行`try…catch`，这样就会导致经常出现这样一种情况：逻辑代码只有很少的几行，而进行异常捕获和处理的代码却有很多行。
这样不仅导致逻辑代码阅读起来晦涩难懂，而且降低了程序的性能。

建议尽量避免检查异常的使用，如果确实该异常情况出现很的普遍，需要提醒调用者注意处理的话，就使用检查异常；否则使用非检查异常。
因此，在一般情况下，尽量将检查异常转变为非检查异常交给上层处理。

### 不要在finally中抛异常
```
try {
  someMethod();  //抛出 exceptionOne
}finally{
  cleanUp();    //如果在这里再抛出一个异常，那么try中的exception将会丢失
}
```
在上面的例子中，如果`someMethod()`抛出一个异常，并且在`finally`块中，`cleanUp()`也抛出一个异常，那么初始的`exception`(正确的错误异常)将永远丢失。

但是，如果你不想处理`someMethod()`中的异常，但是仍然需要做一些清理工作，那么在`finally`块中进行清理。不要使用`catch`块。

