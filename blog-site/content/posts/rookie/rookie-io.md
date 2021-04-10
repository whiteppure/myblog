---
title: "Java IO"
date: 2021-04-09
draft: false
tags: ["Java", "面向菜鸟编程"]
slug: "rookie-io"
---

## 概念
> IO，即in和out，也就是输入和输出，指应用程序和外部设备之间的数据传递，常见的外部设备包括文件、管道、网络连接。

Java 中是通过*流技术*处理IO的。

> 流（Stream），是一个抽象的概念，是指一连串的数据（字符或字节），是以先进先出的方式发送信息的通道。
代表任何有能力产出数据的数据源对象或者是有能力接受数据的接收端对象。

流的作用就是为数据源和目的地建立一个输送通道

一般来说关于流的特性有下面几点：

- 先进先出：最先写入输出流的数据最先被输入流读取到。
- 顺序存取：可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（RandomAccessFile除外）
- 只读或只写：每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。
在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

## 流的分类
![JavaIO流分类](/myblog/posts/images/essays/JavaIO流分类.png)

根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。

主要的分类方式有以下3种：
- 按数据流的方向：输入流、输出流
- 按处理数据单位：字节流、字符流
- 按功能：节点流、处理流

### 输入流与输出流
此输入、输出是相对于我们写的代码程序而言。
- 输入流：从别的地方获取资源 输入到 我们的程序中
- 输出流：从我们的程序中输出到别的地方；例如：将一个字符串保存到本地文件中，就需要使用输出流。


### 字节流与字符流
字节流和字符流的用法几乎完成全一样，区别在于字节流和字符流所操作的数据单元不同，字节流操作的单元是数据单元是8位的字节，字符流操作的是数据单元为16位的字符。

>- 字符流的由来
>
>Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。
>为此，JAVA中引入了处理字符的流。因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。
>
>- 为什么要有字符流？
>
>Java中字符是采用Unicode标准，Unicode 编码中，一个英文为一个字节，一个中文为两个字节。
>如果使用字节流处理中文，如果一次读写一个字符对应的字节数就不会有问题，一旦将一个字符对应的字节分裂开来，就会出现乱码了。

- 字节流：每次读取(写出)一个字节，当传输的资源文件有中文时，就会出现乱码，
- 字符流：每次读取(写出)两个字节，有中文时，使用该流就可以正确传输显示中文。

>- 字节流一般用来处理图像、视频、音频、PPT、Word等类型的文件。字符流一般用于处理纯文本类型的文件，如TXT文件等，但不能处理图像视频等非文本文件。
用一句话说就是：字节流可以处理一切文件，而字符流只能处理纯文本文件。
>- 字节流本身没有缓冲区，缓冲字节流相对于字节流，效率提升非常高。而字符流本身就带有缓冲区，缓冲字符流相对于字符流效率提升就不是那么大了。

### 节点流与处理流
按功能不同分为 节点流、处理流：
- 节点流：以从或向一个特定的地方读写数据。如`FileInputStream　`.
- 处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如`BufferedReader`。
处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，

处理流是对节点流的封装，最终的数据处理还是由节点流完成的。

## 使用流
看上面的几个分类，可能会感觉到有些混乱，那什么时候用字节流，什么时候该用输出流呢？
>1、首先自己要知道是选择输入流还是输出流，这就要根据自己的情况而定，如果你想从程序写东西到别的地方，那么就选择输出流，反之用输入流
>2、然后考虑你传输数据时，是选择使用字节流传输还是字符流，也就是每次传1个字节还是2个字节，有中文肯定就选择字符流了。
>3、前面两步就可以选出一个合适的节点流了，比如字节输入流inputStream，如果要在此基础上增强功能，那么就在处理流中选择一个合适的即可。

### File类
如果我们想要操作流首先，那不得不首先说一下[File](https://docs.oracle.com/javase/8/docs/api/index.html?overview-summary.html)类。

Java中的File类以抽象的方式代表文件名和目录路径名。该类主要用于文件和目录的创建、文件的查找和文件的删除等。

File对象代表磁盘中实际存在的文件和目录。

使用 File 类,查找文件、删除文件、创建文件的代码演示
```
 public static void main(String args[]) throws IOException {
        String dirname = "/home/dir";
        File file = new File(dirname);

        // 判断目录或文件存在
        if (!file.exists()) {
            System.out.println(dirname + " 该路径或文件不存在");
            System.out.println("开始创建该文件或目录....");
            // 不存在则创建
            if (file.createNewFile()) {
                System.out.println(dirname + " 创建成功");
            }
        }

        // 判断是否为目录
        if (file.isDirectory()) {
            System.out.println("目录： " + dirname);
            String s[] = file.list();
            for (int i = 0; i < s.length; i++) {
                File f = new File(dirname + "/" + s[i]);
                if (f.isDirectory()) {
                    System.out.println(s[i] + " 是文件夹");
                } else {
                    System.out.println(s[i] + " 是文件");
                }
            }
        } else {
            System.out.println(dirname + " 不是一个目录");
            System.out.println("开始删除文件 ...");
            // 删除文件
            if (file.delete()) {
                System.out.println(dirname + "删除成功"); 
            }
        }

    }
```

### 操作字节流
![字节流](/myblog/posts/images/essays/字节流.png)

操作byte类型数据，主要操作类是`OutputStream、InputStream`的子类；不用缓冲区，直接对文件本身操作。

以下代码就是用`FileInputStream、FileOutputStream`操作字节流
```
public static void main(String[] args) throws IOException {
        File file = new File("./test.txt");
        write(file);
        System.out.println(read(file));
    }

    // 用字节流写入
    public static void write(File file) throws IOException {
        OutputStream os = new FileOutputStream(file, true);
        // 要写入的字符串
        String string = "awslawslawslawslawslawslawsl";
        // 写入文件
        os.write(string.getBytes());
        // 关闭流
        os.close();
    }

    // 用字节流读取
    public static String read(File file) throws IOException {
        InputStream in = new FileInputStream(file);
        // 一次性取多少个字节
        byte[] bytes = new byte[1024];
        // 用来接收读取的字节数组
        StringBuilder sb = new StringBuilder();
        // 读取到的字节数组长度，为-1时表示没有数据
        int length = 0;
        // 循环取数据
        while ((length = in.read(bytes)) != -1) {
            // 将读取的内容转换成字符串
            sb.append(new String(bytes, 0, length));
        }
        // 关闭流
        in.close();
        return sb.toString();
    }
```

缓冲字节流是为高效率而设计的，真正的读写操作还是靠`FileOutputStream`和`FileInputStream`
```
public static void main(String[] args) throws IOException {
        File file = new File("test.txt");
        write(file);
        System.out.println(read(file));
    }

    public static void write(File file) throws IOException {
        // 缓冲字节流，提高了效率
        BufferedOutputStream bis = new BufferedOutputStream(new FileOutputStream(file, true));
        // 要写入的字符串
        String string = "awslawslawslawslawslawslawsl";
        // 写入文件
        bis.write(string.getBytes());
        // 关闭流
        bis.close();
    }

    public static String read(File file) throws IOException {
        BufferedInputStream fis = new BufferedInputStream(new FileInputStream(file));
        // 一次性取多少个字节
        byte[] bytes = new byte[1024];
        // 用来接收读取的字节数组
        StringBuilder sb = new StringBuilder();
        // 读取到的字节数组长度，为-1时表示没有数据
        int length = 0;
        // 循环取数据
        while ((length = fis.read(bytes)) != -1) {
            // 将读取的内容转换成字符串
            sb.append(new String(bytes, 0, length));
        }
        // 关闭流
        fis.close();
        return sb.toString();
    }
```

### 操作字符流
![字符流](/myblog/posts/images/essays/字符流.png)

操作字符类型数据，主要操作类是`Reader、Writer`的子类；使用缓冲区缓冲字符，不关闭流就不会输出任何内容。

字符流适用于文本文件的读写，`OutputStreamWriter` 类其实也是借助 `FileOutputStream` 类实现的

以下代码就是用`InputStreamReader、OutputStreamWriter`操作字节流.
```
public static void main(String[] args) throws IOException {
        File file = new File("test.txt");
        write(file);
        System.out.println(read(file));
    }

    public static void write(File file) throws IOException {
        // OutputStreamWriter可以显示指定字符集，否则使用默认字符集
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(file, true), "UTF-8");
        // 要写入的字符串
        String string = "awslawslawslawslawslawslawslawsl";
        osw.write(string);
        osw.close();
    }

    public static String read(File file) throws IOException {
        InputStreamReader isr = new InputStreamReader(new FileInputStream(file), "UTF-8");
        // 字符数组：一次读取多少个字符
        char[] chars = new char[1024];
        // 每次读取的字符数组先append到StringBuilder中
        StringBuilder sb = new StringBuilder();
        // 读取到的字符数组长度，为-1时表示没有数据
        int length;
        // 循环取数据
        while ((length = isr.read(chars)) != -1) {
            // 将读取的内容转换成字符串
            sb.append(chars, 0, length);
        }
        // 关闭流
        isr.close();
        return sb.toString();
    }
```

字符缓冲流
```
public static void main(String[] args) throws IOException {
        File file = new File("test.txt");
        write(file);
        System.out.println(read(file));
    }

    public static void write(File file) throws IOException {
        // FileWriter可以大幅度简化代码
        BufferedWriter bw = new BufferedWriter(new FileWriter(file, true));
        // 要写入的字符串
        String string = "awslawslawslawslawslawslawslawsl";
        bw.write(string);
        bw.close();
    }

    public static String read(File file) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(file));
        // 用来接收读取的字节数组
        StringBuilder sb = new StringBuilder();
        // 按行读数据
        String line;
        // 循环取数据
        while ((line = br.readLine()) != null) {
            // 将读取的内容转换成字符串
            sb.append(line);
        }
        // 关闭流
        br.close();
        return sb.toString();
    }
```

字节流和字符流间的转换

- `OutputStreamWriter` 是字符流通向字节流的桥梁
```
    public static void main(String[] args) throws IOException {
        File f = new File("test.txt");

        // OutputStreamWriter 是字符流通向字节流的桥梁,创建了一个字符流通向字节流的对象
        OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(f),"UTF-8");

        osw.write("我是字符流转换成字节流输出的");
        osw.close();

    }
```

- `InputStreamReader` 是字节流通向字符流的桥梁
```
  public static void main(String[] args) throws IOException {
        
        File f = new File("test.txt");
        
        InputStreamReader inr = new InputStreamReader(new FileInputStream(f),"UTF-8");
        
        char[] buf = new char[1024];
        
        int len = inr.read(buf);
        System.out.println(new String(buf,0,len));
        
        inr.close();

    }

```



## 序列化

### 反序列化



## IO分类

### 阻塞式IO模型

### 非阻塞IO模型

### IO复用模型

### 信号驱动IO模型

### 异步IO模型



## IO种类

### AIO

### BIO

### NIO



## 网络编程

### Netty



