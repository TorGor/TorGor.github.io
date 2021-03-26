---
layout: post
title:  redis 集群搭建方式：docker-compose + 传统模式(未完待续)
date:   2020-4-6 00:00:00 +0800
categories: document
tag: juc
---

* content
{:toc}

# 闭锁

分代年龄：Survivor from to 次数 == 2^4-1=15 次后转到老年代；  

mark word 中的 Klass ：指向元空间中的 class 的二进制存储地址；  

元空间：直接内存中存储，JVM会自动根据类的元数据大小动态增加元空间的容量，解决内存溢出的一些情况。
JDK1.7及之后版本的JVM已经将字符串常量池从方法区中移了出来，在Java 堆（Heap）中开辟了一块区域存放字符串常量池。


 
# JVM

javap -c class > **.txt  将 class 文件反编译到 txt 文件，查看字节码

字节码解读
iadd 
imul
ireturn
iload

FILO ：先进后出

栈帧：局部变量表，操作数栈，动态链接，方法出口

操作数栈：临时存放局部变量的计算结果

# Thread
1.sleep()方法
在指定时间内让当前正在执行的线程暂停执行，但不会释放“锁标志”。  



```
    private static TrafficShapingController generateRater(/*@Valid*/ FlowRule rule) {
        if (rule.getGrade() == RuleConstant.FLOW_GRADE_QPS) {
            switch (rule.getControlBehavior()) {
                case RuleConstant.CONTROL_BEHAVIOR_WARM_UP:
                    return new WarmUpController(rule.getCount(), rule.getWarmUpPeriodSec(),
                        ColdFactorProperty.coldFactor);
                case RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER:
                    return new RateLimiterController(rule.getMaxQueueingTimeMs(), rule.getCount());
                case RuleConstant.CONTROL_BEHAVIOR_WARM_UP_RATE_LIMITER:
                    return new WarmUpRateLimiterController(rule.getCount(), rule.getWarmUpPeriodSec(),
                        rule.getMaxQueueingTimeMs(), ColdFactorProperty.coldFactor);
                case RuleConstant.CONTROL_BEHAVIOR_DEFAULT:
                default:
                    // Default mode or unknown mode: default traffic shaping controller (fast-reject).
            }
        }
        return new DefaultController(rule.getCount(), rule.getGrade());
    }
```   

AQS 中的可中断：

LockSupport.park();
LockSupport.unpark(thread);

Thread.yield();
    
# 喜欢文章请关注我  
  
> 程序领域  
**点击关注+转发，私信发送【面试】或者【资料】可以收获更多资源**

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)









