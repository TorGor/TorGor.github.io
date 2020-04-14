---
layout: post
title:  Java 锁的是对象，如何实现的？加锁过程是什么？
date:   2020-4-4 00:00:00 +0800
categories: document
tag: lock
---

>开篇思考
1. 对象在堆中的数据结构？和锁有什么关系？
2. 对象的锁是如何升级的？

还是绕不开底层。曾经一遍遍来自灵魂的追问，别再深入了，又不是那啥，有乐趣吗？  嘿，还真的越深入越有趣。  
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

由上图我们看到了头部 MarkWord 中的内存结构，当无锁和偏向锁的时候，锁标志位都是 01 ，
只有偏向锁的时候前面内存中保存了线程的 ID ，用来判断下次进入锁的时候，是否是当前线程。
如果是直接获取锁，性能很好，这也是偏向锁的原理。



# synchronized 相关知识树
![Mark Word 网络图](https://torgor.github.io/styles/images/jmm/synchronized-structure-xmind.png)

# 喜欢文章请关注我    
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)

![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








