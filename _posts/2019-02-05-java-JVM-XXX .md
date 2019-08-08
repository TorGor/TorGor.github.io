---
layout: post
title:  Springboot 使用 ThreadPoolTaskExecutor 线程池
date:   2019-02-05 00:00:00 +0800
categories: document
tag: JVM
---

* content
{:toc}


### jstat （JVM statistics Monitoring）：虚拟机统计信息监视工具

选项 | 作用
--- | ---
-class | 监视类装载、卸载数量、总空间以及类装载所耗费的时间
-gc | 监视Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息 
-gccapacity | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间 
-gcutil | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
-gccause | 与 -gcutil 功能一样，但是会额外输出导致上一次GC产生的原因
-gcnew | 监视新生代GC状况
-gcnewcapacity | 监视内容与 -gcnew 基本相同，输出主要关注使用到的最大、最小空间
-gcold  | 监视老年代GC状况
-gcoldcapacity | 监视内容与 -gcold 基本相同，输出主要关注使用到的最大、最小空间
-gcpermcapacity | 输出永久代使用到的最大、最小空间
-compiler | 输出JIT编译器编译过的方法、耗时等信息 
-printcompilation | 输出已经被JIT编译的方法 

![jstat 命令]({{ '/styles/images/jmm/JVM-jstat-help.png' | prepend: site.baseurl  }})

### centos yum 安装 jdk 没有 jstat ，jps 命令解决办法

yum install java-1.8.0-openjdk-devel -y





