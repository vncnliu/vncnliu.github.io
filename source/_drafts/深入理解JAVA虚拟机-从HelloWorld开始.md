---
title: 深入理解JAVA虚拟机-读书笔记
categories:
- 读书笔记
tags:
- JVM
---

##### 代码是如何执行的

* .java源文件编译为.class文件
* jvm加载.class文件在虚拟机里执行，或者编译为机器码直接执行

执行时 程序计数器记录程序执行到哪
虚拟机栈（线程私有）里存放栈帧队列，第一个为当前执行信息，栈帧里有操作数栈用于存储方法信息

找个文本编辑器创建一个文件HelloWorld.java
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```
命令行执行
```
F:\workspace\read\shen_ru_li_jie_java_xu_ni_ji\src\main\java\top\vncnliu\read>javac HelloWorld.java

F:\workspace\read\shen_ru_li_jie_java_xu_ni_ji\src\main\java\top\vncnliu\read>java HelloWorld
Hello World
```
命令行输出 Hello World 说明程序正确执行
然后执行javap反汇编看一下类的详细信息
```
F:\workspace\read\shen_ru_li_jie_java_xu_ni_ji\src\main\java\top\vncnliu\read>javap -verbose HelloWorld
Classfile /F:/workspace/read/shen_ru_li_jie_java_xu_ni_ji/src/main/java/top/vncnliu/read/HelloWorld.class
  Last modified 2018-3-11; size 425 bytes
  MD5 checksum 1e678fa12f97b65fdde287539d5ce419
  Compiled from "HelloWorld.java"
public class HelloWorld
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // Hello World
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // HelloWorld
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               HelloWorld.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               Hello World
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               HelloWorld
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public HelloWorld();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 4: 0
        line 5: 8
}
SourceFile: "HelloWorld.java"
```
可以看到主要有常量池和两个方法，一构造方法一个main方法。
ldc 将一个常量加载到操作数栈
getstatic 将一个常量加载到虚拟机栈顶

