



# synchronized关键字解读

## 概述

synchronized 关键字我们经常在 Java 相关代码里见到，主要作用是在多线程并发时，保证线程访问共享数据时的线程安全。比如StringBuffer里为了实现线程安全，它为AbstractStringBuilder的方法加上了synchronized关键字。

它的作用：

1. 确保线程互斥的访问同步代码
2. 保证共享变量的修改及时可见
3. 有效解决指令重排（synchronized同步中的代码，JVM不会轻易优化重排序）

**后两点volatile关键字也可以实现。**

它可以修饰代码块或者是方法，如下：

```java
package com.java.test.syncTest;

public class SynchronizedDemo {

    public void syncBlock(){
        synchronized (this){
            System.out.println("hello block");
        }
    }

    public synchronized void syncMethod(){
        System.out.println("hello method");
    }
}
```
使用javac 编译生成class文件，然后使用javap -c 查看完整信息。如下：
```java
// javac 编译我们编写的程序
D:\JdkSourceLearn\src\com\java\test\syncTest>javac SynchronizedDemo.java

// javap -v 查看生成的字节码
D:\JdkSourceLearn\src\com\java\test\syncTest>javap -v SynchronizedDemo.class
    
Classfile /D:/JdkSourceLearn/src/com/java/test/syncTest/SynchronizedDemo.class
  Last modified 2020年6月28日; size 628 bytes
  MD5 checksum 085d5365515dc098557dd8ebd99eaa2c // MD5签名
  Compiled from "SynchronizedDemo.java"
public class com.java.test.syncTest.SynchronizedDemo
  minor version: 0						// 次版本 1~12 停用
  major version: 56						// 主要版本 jdk12 从45开始
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER	  // 访问标记
  this_class: #6                          // 全限定名 com/java/test/syncTest/SynchronizedDemo
  super_class: #7                         // 父类     java/lang/Object
  interfaces: 0, fields: 0, methods: 3, attributes: 1
Constant pool:						    // 常量池
   #1 = Methodref          #7.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #19.#20        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #21            // hello block
   #4 = Methodref          #22.#23        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #24            // hello method
   #6 = Class              #25            // com/java/test/syncTest/SynchronizedDemo
   #7 = Class              #26            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               syncBlock
  #13 = Utf8               StackMapTable
  #14 = Class              #27            // java/lang/Throwable
  #15 = Utf8               syncMethod
  #16 = Utf8               SourceFile
  #17 = Utf8               SynchronizedDemo.java
  #18 = NameAndType        #8:#9          // "<init>":()V
  #19 = Class              #28            // java/lang/System
  #20 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #21 = Utf8               hello block
  #22 = Class              #31            // java/io/PrintStream
  #23 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #24 = Utf8               hello method
  #25 = Utf8               com/java/test/syncTest/SynchronizedDemo
  #26 = Utf8               java/lang/Object
  #27 = Utf8               java/lang/Throwable
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
  public com.java.test.syncTest.SynchronizedDemo();	// 默认的无参构造方法
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public void syncBlock();					// 方法名
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC				// 访问标记
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter					// synchronized块 开始
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3                  // String hello block
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit						// synchronized块 结束
        14: goto          22				 // 方法正常结束 跳到22返回
        17: astore_2					     // 从这里开始是异常路径 
        18: aload_1
        19: monitorexit						// synchronized块 结束
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 6: 0
        line 7: 4
        line 8: 12
        line 9: 22
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class com/java/test/syncTest/SynchronizedDemo, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

  public synchronized void syncMethod();
    descriptor: ()V
    flags: (0x0021) ACC_PUBLIC, ACC_SYNCHRONIZED	// 这里添加了ACC_SYNCHRONIZED访问标记
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String hello method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 12: 0
        line 13: 8
}
SourceFile: "SynchronizedDemo.java"
```

从上面的中文注释处可以看到，对于**synchronized**关键字而言，`javac`在编译时，会生成对应的**monitorenter**和**monitorexit**指令分别对应**synchronized**同步块的进入和退出，有两个**monitorexit**指令的原因是：为了保证抛异常的情况下也能释放锁，所以`javac`为同步代码块添加了一个隐式的**try-finally**，在finally中会调用**monitorexit**命令释放锁。(**显式实现**)

而对于**synchronized**方法而言，`javac`为其生成了一个**ACC_SYNCHRONIZED**关键字，在JVM进行方法调用时，发现调用的方法被**ACC_SYNCHRONIZED**修饰，则会先尝试获得锁。(**隐式实现**)

> 本质上都是对一个对象的monitor进行获取，而这个获取的过程是排他的，也就是同一时刻只能有一个线程获得同步块对象的监视器monitor。

[Java 虚拟机说明](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.14)

## Synchronized 原理

在Hotspot虚拟机中，对象在内存中的存储布局，可以分为三块：对象头Header，实例数据Instance Data，对齐填充Padding。

### Java对象头

[对象头参考地址](https://blog.csdn.net/lkforce/article/details/81128115#commentBox)

Hotspot虚拟机的对象头一般包含了两部分信息：

1. Mark Word，用于存储对象自身的运行时数据，比如hash，gc分代年龄，锁状态的标志，线程持有锁，偏向ID，偏向时间戳等等。
2. Klass Pointer：对象指向它的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
3. 数组长度（只有数组对象才有）。

#### mark word

Mark Word记录了对象和锁有关的信息，当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。

Mark Word在32位JVM中的长度是32bit，在64位JVM中长度是64bit。

Mark Word在不同的锁状态下存储的内容不同，在32位JVM中是这么存的：

![对象头1](https://i.loli.net/2020/06/28/dUPvpEZjFQ3hGD7.png)

[图片来源](https://juejin.im/post/5bfe6ddee51d45491b0163eb#heading-0)

[hotspot的源码在线地址](http://hg.openjdk.java.net/jdk8u/jdk8u60/hotspot/file/37240c1019fd/src/share/vm/oops/markOop.hpp)

![对象头2](https://i.loli.net/2020/06/28/bkji9rde5zlAUus.png)

JVM一般是这样使用锁和Mark Word的(**锁的一个升级**)：

1. 当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0。
2. 当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。
3. 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码。
4. 当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用CAS操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步锁代码。如果抢锁失败，则继续执行步骤5。
5. 偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6。
6. 轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码，如果失败则继续执行步骤7。
7. 自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

> 锁的升级：无锁->偏向锁->轻量级锁->重量级锁。
>
> 从上面可以看出来锁的重量级不断上升，这也是为了效率考虑的。如果一开始就是重量级的锁，因为阻塞唤醒、用户态和内核态的切换、进程的上下文切换造成资源消耗较大，而有些时候线程占有锁之后很快就释放了，只需要让其等待一小会即可，当然自旋也会有消耗，因为自旋需要利用cpu时间，而这段时间什么也没有做。

#### Klass Pointer

该指针在32位JVM中的长度是32bit，在64位JVM中长度是64bit。

Java对象的类数据保存在方法区。

#### 数组长度

只有数组对象保存了这部分数据。

该数据在32位和64位JVM中长度都是32bit。

### 实例数据

对象的实例数据就是在java代码中能看到的属性和他们的值。

### 对齐填充

JVM要求java的对象占的内存大小应该是8bit的倍数，所以后面有几个字节用于把对象的大小补齐至8bit的倍数，没有特别的功能

## 参考

[死磕Synchronized底层实现--概论](https://juejin.im/post/5bfe6ddee51d45491b0163eb#heading-0)

[重学Java——Synchronized底层实现原理](https://juejin.im/post/5cfe3a556fb9a07ecf721a5a#heading-0)

[Java的对象头和对象组成详解](https://blog.csdn.net/lkforce/article/details/81128115#commentBox)

