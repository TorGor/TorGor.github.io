---
layout: post
title:  Java 锁的是对象，如何实现的？加锁过程是什么？
date:   2020-4-4 00:00:00 +0800
categories: document
tag: lock
---

* content
{:toc}

>开篇思考  
1. 对象在堆中的数据结构？和锁有什么关系？
2. 对象的锁是如何升级的？

还是绕不开底层。曾经一遍遍来自灵魂的追问，别再深入了，又不是为爱"鼓掌"，有乐趣吗？  嘿，还真的越深入越有趣。  
其实对象锁是由 Synchronized 来进行操控的，因为由虚拟机运行加锁步骤，而且各种解释都是非常抽象，
所以很多人对底层加锁原理不是很理解。其实这个可以参考 JUC 里面提供的手动加锁机制来作为参考。  
如果想理解手动加锁过程，可以看看这篇介绍[《AQS 都不懂怎么能说懂并发？AQS 实现手动加锁原理分析》](https://torgor.github.io/2020/01/28/java-ReentrantLock/)

# 对象在堆的内存结构     
JVM 中的堆内存我们都知道是用来存储 Java 实例化对象的。到底存储了什么呢？用来存放动态产生的数据，比如 new 出来的对象。  
注意：创建出来的对象只包含属于各自的成员变量，并不包括成员方法。  
因为同一个类的对象拥有各自的成员变量，存储在各自的堆中，但是他们共享该类的方法，并不是创建新的对象就复制一份方法，方法都保存在方法区中。  
在堆中只会存储成员方法的地址，在调用的时候，根据地址去方法区中执行对应的成员方法。

堆中的数据结构：
![jvm-heap-oop-structure.png](https://torgor.github.io/styles/images/jmm/jvm-heap-oop-structure.png)  

由上图可以看出 Java 对象保存在内存中时，由以下三部分组成：
1. 对象头 : MarkWord，对象指针，锁标志位，GC分代年龄。
2. 实例数据 ：用来保存 new 出来的具体实例数据，比如属于实例对象的属性值：用户名=张三、性别=男、身份证号=XXX
3. 对齐填充字节：这个不是必要，因为内存的使用都会被填充为八字节的倍数，纯粹是为了补位。 
 
数组对象是有些特别的，会在头中多一个 int 类型 lenth 来表示数组长度。

# 对象头内存结构和锁   

常常说 synchronized 锁住对象，那么具体怎么锁的，通过什么来判断锁类型？  
其实在 jdk1.6 之前的 synchronized 锁都是重量级锁，从 jdk1.6 开始对锁进行了优化，
加入了从无锁-偏向锁-轻量级锁-自旋-重量级锁的升级流程。
如果需要了解更多关于锁的概念，看这篇关于锁的文章[《聊聊你知道的锁》](https://torgor.github.io/2020/01/28/java-lock/)

![Mark Word 网络图](https://torgor.github.io/styles/images/jmm/heap-object-head-structure.png)

64位的数据结构略有不同：  
无锁状态 25bits unused ，31bits hashcode；  
偏向锁状态 线程ID 54bits，其余和 32 位相同  

由上图我们看到了头部 MarkWord 中的内存结构，当无锁和偏向锁的时候，锁标志位都是 01 ，
只有偏向锁的时候前面内存中保存了线程的 ID ，用来判断下次进入锁的时候，是否是当前线程。
如是直接获取锁，性能很好，这也是偏向锁的原理。
偏向锁会偏向于前一个获得它的线程，在接下来的线程竞争执行过程中，假如该锁没有被其他线程所获取，
没有其他线程来竞争该锁，那么持有偏向锁的线程不需要进行同步加锁操作，可以认为 01 状态的都是一种无锁状态。

偏向锁是可以通过参数设置是否开启的，当线程不是偏向锁，而且又有锁竞争，就会 CAS 比较替换对象头中的 threadId，
如果替换成功，还是偏向锁。
如果不成功，说明有锁竞争，进行自旋等待，升级为轻量级锁。  
如果还是竞争失败，升级为重量级锁。

![加锁过程图（太长了画不下，精简了）](https://torgor.github.io/styles/images/jmm/synchronized-lock-process.png)

# 偏向锁的撤销  
如果升级轻量级锁，那么偏向锁就应该失效才行，锁失效撤销的过程大致如下：
1. 在一个安全点停止拥有锁的线程。
2. 遍历线程栈，如果存在锁记录的话，需要修复锁记录和Markword，使其变成无锁状态。
3. 唤醒当前线程，将当前锁升级成轻量级锁。
所以为了避免性能问题，我们需要分析代码中是否要经常两个不同线程同时争抢同步代码执行，如果是需要关闭偏向锁提高性能。  

偏向锁 JVM 参数： 
```
启用参数: 
-XX:+UseBiasedLocking
关闭延迟: 
-XX:BiasedLockingStartupDelay=0 
禁用参数: 
-XX:-UseBiasedLocking
```

# 轻量级锁升级 
偏向锁撤销升级为轻量级锁，对象的Markword也会进行相应的的变化。

1. 线程在自己的栈桢中创建锁记录 LockRecord。
2. 将锁对象的对象头中的 MarkWord 复制到线程的刚刚创建的锁记录中。
3. 将锁记录中的 Owner 指针指向锁对象。
4. 将锁对象的对象头的MarkWord替换为指向锁记录的指针。

# 重量级锁的具体实现
重量级锁依赖 JVM 中的 monitor 对象来实现。  
接下来我们以对象锁来分析，我们都知道 `synchronized(this)` 就是给对象上锁，this 就是指具体的对象。  
通过反编译可以看到有两个个关键字：  
1. monitorenter
2. monitorexit 

**monitorenter**  
每一个对象都会和一个监视器monitor关联。监视器被占用时会被锁住，其他线程无法来获取该monitor。
当 JVM 执行某个线程的某个方法内部的 monitorenter 时，它会尝试去获取当前对象对应的 monitor 的所有权。其过程如下：  

1. 若monior的进入数为0，线程可以进入 monitor，并将monitor的进入数置为1。当前线程成为 monitor 的持有者 
2. 若线程已拥有monitor的所有权，允许它重入monitor，并递增monitor的进入数
3. 若其他线程已经占有monitor的所有权，那么当前尝试获取monitor的所有权的线程会被阻塞，直到monitor的进入数变为0，
才能重新尝试获取monitor的所有权。

**monitorexit**  
1. 能执行 monitorexit 指令的线程一定是拥有当前对象的 monitor 的所有权的线程。
2. 执行 monitorexit 时会将 monitor 的进入数 -1。当monitor的进入数减为0时，当前线程退出monitor，
不再拥有 monitor 的所有权，此时其他被这个 monitor 阻塞的线程可以尝试去获取这个 monitor 的所有权。


如下是针对 synchronized 关键字的示例代码：
```
public class MySynchronizedTest {
    public MySynchronizedTest() {
    }

    public synchronized void testMonitor() {
        System.out.println("synchronized method");
    }

    public void testSynchronizedThis() {
        synchronized(this) {
            System.out.println("synchronized this");
        }
    }

    public static synchronized void testStatic() {
        System.out.println("synchronized static");
    }
}
```

`javap -v MySynchronizedTest.class` 命令查看编译后的内容：

```
public class com.holy.nacosconsumer.MySynchronizedTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #8.#25         // java/lang/Object."<init>":()V
   #2 = Fieldref           #26.#27        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #28            // synchronized method
   #4 = Methodref          #29.#30        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #31            // synchronized this
   #6 = String             #32            // synchronized static
   #7 = Class              #33            // com/holy/nacosconsumer/MySynchronizedTest
   #8 = Class              #34            // java/lang/Object
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               LocalVariableTable
  #14 = Utf8               this
  #15 = Utf8               Lcom/holy/nacosconsumer/MySynchronizedTest;
  #16 = Utf8               testMonitor
  #17 = Utf8               testSynchronizedThis
  #18 = Utf8               StackMapTable
  #19 = Class              #33            // com/holy/nacosconsumer/MySynchronizedTest
  #20 = Class              #34            // java/lang/Object
  #21 = Class              #35            // java/lang/Throwable
  #22 = Utf8               testStatic
  #23 = Utf8               SourceFile
  #24 = Utf8               MySynchronizedTest.java
  #25 = NameAndType        #9:#10         // "<init>":()V
  #26 = Class              #36            // java/lang/System
  #27 = NameAndType        #37:#38        // out:Ljava/io/PrintStream;
  #28 = Utf8               synchronized method
  #29 = Class              #39            // java/io/PrintStream
  #30 = NameAndType        #40:#41        // println:(Ljava/lang/String;)V
  #31 = Utf8               synchronized this
  #32 = Utf8               synchronized static
  #33 = Utf8               com/holy/nacosconsumer/MySynchronizedTest
  #34 = Utf8               java/lang/Object
  #35 = Utf8               java/lang/Throwable
  #36 = Utf8               java/lang/System
  #37 = Utf8               out
  #38 = Utf8               Ljava/io/PrintStream;
  #39 = Utf8               java/io/PrintStream
  #40 = Utf8               println
  #41 = Utf8               (Ljava/lang/String;)V
{
  public com.holy.nacosconsumer.MySynchronizedTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/holy/nacosconsumer/MySynchronizedTest;

  public synchronized void testMonitor();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String synchronized method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/holy/nacosconsumer/MySynchronizedTest;

  public void testSynchronizedThis();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #5                  // String synchronized this
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 10: 0
        line 11: 4
        line 12: 12
        line 13: 22
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      23     0  this   Lcom/holy/nacosconsumer/MySynchronizedTest;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class com/holy/nacosconsumer/MySynchronizedTest, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4

  public static synchronized void testStatic();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #6                  // String synchronized static
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 16: 0
        line 17: 8
}

```

# synchronized 相关知识树
![Mark Word 网络图](https://torgor.github.io/styles/images/jmm/synchronized-structure-xmind.png)

# 喜欢文章请关注我    
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)

![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








